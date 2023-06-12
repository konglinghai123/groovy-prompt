### 医嘱拆分
```sql
procedure SplitOrder(v_patient_id in varchar2, --病人ID
                       v_visit_id   in varchar2, --住院标识
                       v_order_no   in varchar2, --医嘱号
                       errorcode    out int, --执行结果标识(1成功,-1失败)
                       errorMsg     out varchar2, --执行结果描述
                       n_execute    out int) --执行单数量

   as
    v_frequency          varchar2(20); --频次
    v_administration     varchar2(20); --途径
    v_perform_schedule   varchar2(500); --执行时间
    v_order_status       varchar2(20); --医嘱状态
    v_freq_interval_unit varchar2(20); --间隔单位
    v_first_day_num      varchar2(20); --首日执行次数
    v_barcode            varchar2(50); --条码号
    v_week               varchar2(20); --周几

    v_status_cancel  varchar2(20); --取消状态
    v_status_execute varchar2(20); --执行状态
    v_status_stop    varchar2(20); --停止状态

    d_start_date_time   date; --开医嘱时间
    d_stop_date_time    date; --停止时间
    d_next_split_date   date; --下次拆分时间
    d_execute_date_time date; --执行时间
    d_temp_start        date; --要生成执行单的开始拆分时间
    d_temp_end          date; --要生成执行单的结束拆分时间
    d_temp              date; --

    d_execute_array dateArray; --执行时间列表

    n_freq_counter     number(5); --执行次数
    n_freq_interval    number(5); --执行间隔
    n_repeat_indicator number(5); --长期临时标志
    n_count            number(5); --计数器
    n_day              number(5); --医嘱执行的相隔天数

  begin
    n_execute := 0;
    SuccessPackage('执行成功', errorCode, errorMsg);

    n_count := 0;
    select count(1)
      into n_count
      from nurse_orders a
     where a.patient_id = v_patient_id
       and a.visit_id = v_visit_id
       and a.order_no = v_order_no
       and a.order_sub_no = 1;
    if n_count <= 0 then
      ErrorPackage('查询医嘱失败：没有找到医嘱号为' || v_order_no || '的医嘱信息',
                   errorCode,
                   errorMsg);
      return;
    end if;

    --1、提取医嘱信息
    errorMsg := '提取医嘱信息：';
    select to_Number(a.repeat_indicator),
           a.start_date_time,
           a.frequency,
           a.administration,
           rtrim(ltrim(replace(a.perform_schedule, '：', ':'))),
           a.stop_date_time,
           a.order_status,
           to_Number(a.freq_counter),
           to_Number(a.freq_interval),
           a.freq_interval_unit,
           '' first_day_num
      into n_repeat_indicator, --长期、临时标志
           d_start_date_time, --医嘱开始时间
           v_frequency, --频率
           v_administration, --途径
           v_perform_schedule, --执行时间，如果此时间为空，则会从nurse_frequency_dict中取默认值
           d_stop_date_time, --医嘱停止时间
           v_order_status, --医嘱状态，会根据医嘱状态不同生成相应处理
           n_freq_counter, --频率次数
           n_freq_interval, --频率间隔
           v_freq_interval_unit, --间隔单位
           v_first_day_num --首日执行次数
      from nurse_orders a
     where a.patient_id = v_patient_id
       and a.visit_id = v_visit_id
       and a.order_no = v_order_no
       and a.order_sub_no = 1;

    --2、如果频次不为空，但默认执行时间为空的，取频次字典表的信息
    errorMsg := '如果频次不为空：';
    if v_frequency is not null and v_perform_schedule is null then

      n_count := 0;
      select count(1)
        into n_count
        from nurse_frequency_dict a
       where frequency = v_frequency;

      if n_count <= 0 then
        ErrorPackage('医嘱拆分失败：没有找到频次号为' || v_frequency || '的字典信息',
                     errorCode,
                     errorMsg);
        return;
      end if;

      select rtrim(ltrim(replace(a.default_time, '：', ':'))),
             a.freq_count,
             a.freq_interval,
             a.freq_interval_unit
        into v_perform_schedule,
             n_freq_counter,
             n_freq_interval,
             v_freq_interval_unit
        from nurse_frequency_dict a
       where frequency = v_frequency;

    end if;

    --3、医嘱的执行间隔，主要是要生成相隔天数
    errorMsg := '医嘱的执行间隔：';
    if n_freq_interval is null or n_freq_interval = 0 then
      n_day := 1.0; --默认是每天
    else
      n_day := n_freq_interval * 1;
    end if;

    if instr(v_freq_interval_unit, '周') > 0 then
      n_day := n_day * 7; --如果是周的，则*7
    elsif instr(v_freq_interval_unit, '时') > 0 then
      n_day          := 1;
      n_freq_counter := 24 / n_freq_interval;
    end if;

    if n_day = 0 or n_day is null then
      n_day := 1; --默认是每天执行
    end if;

    if n_freq_counter is null or n_freq_counter = 0 then
      n_freq_counter := 1; --默认是1次
    end if;

    --4、执行时间perform_schedule为空的处理
    errorMsg := '执行时间perform_schedule为空的处理：';
    if v_perform_schedule is null then
      v_perform_schedule := '';

      if instr(v_freq_interval_unit, '周') > 0 then

        --开嘱日期是周几
        select to_number(decode(to_char(d_start_date_time, 'D'),
                                '1',
                                '8',
                                to_char(d_start_date_time, 'D'))) - 1
          into v_week
          from dual;


        --周的执行时间为 1;7，其中1和7指的是周几
        for i in 1 .. n_freq_counter loop

          if i = 1 then
            v_perform_schedule := v_week;
          else
            v_perform_schedule := v_perform_schedule || ';' || v_week;
          end if;

          select to_number(decode(to_char(d_start_date_time + (7 - i) / 2,
                                          'D'),
                                  '1',
                                  '8',
                                  to_char(d_start_date_time + (7 - i) / 2,
                                          'D'))) - 1
            into v_week
            from dual;

        end loop;
      else
        for i in 1 .. n_freq_counter loop

          --从0点开始，平均分配一天的时间点
          if d_temp is null then
            d_temp := trunc(sysdate);
          else
            select d_temp + 1.0 / n_freq_counter into d_temp from dual;
          end if;

          if v_perform_schedule is null then
            v_perform_schedule := to_char(d_temp, 'hh24:mi');
          else
            v_perform_schedule := v_perform_schedule || ';' ||
                                  to_char(d_temp, 'hh24:mi');
          end if;
        end loop;
      end if;
    end if;


    --5、执行状态定义
    errorMsg := '执行状态定义配置表：';
    select a.execute_status, a.stop_status, a.cancel_status
      into v_status_execute, v_status_stop, v_status_cancel
      from nurse_execute_status_dict a
     where rownum = 1;

    if v_status_execute is null then
      ErrorPackage('请联系管理员设置执行状态', errorCode, errorMsg);
      return;
    end if;

    if v_status_stop is null then
      ErrorPackage('请联系管理员设置停止状态', errorCode, errorMsg);
      return;
    end if;

    if v_status_cancel is null then
      ErrorPackage('请联系管理员设置作废状态', errorCode, errorMsg);
      return;
    end if;

    --6、医嘱状态判断，只有执行、停止、作废状态的医嘱才处理，其他状态的医嘱直接返回
    errorMsg := '状态判断：';
    if instr(v_status_execute, v_order_status) <= 0 and
       instr(v_status_stop, v_order_status) <= 0 and
       instr(v_status_cancel, v_order_status) <= 0 then
      ErrorPackage('拆分失败：医嘱不是执行、停止或者作废状态',
                   errorCode,
                   errorMsg);
      return;
    end if;

    --7、下次拆分时间
    errorMsg := '下次拆分时间：';
    n_count  := 0;
    select count(1)
      into n_count
      from healthcare.nurse_execute_split a
     where a.patient_id = v_patient_id
       and a.visit_id = v_visit_id
       and a.order_no = v_order_no;
    if n_count > 0 then
      select next_split_date
        into d_next_split_date
        from healthcare.nurse_execute_split a
       where a.patient_id = v_patient_id
         and a.visit_id = v_visit_id
         and a.order_no = v_order_no;
    end if;

    --8、作废状态的处理,把执行单作废
    errorMsg := '作废状态处理：';
    if instr(v_status_cancel, v_order_status) > 0 then
      update nurse_execute
         set execute_flag = '9',
             memo         = '作废执行单:' ||
                            to_char(sysdate, 'yyyy-mm-dd hh24:mi:ss')
       where patient_id = v_patient_id
         and visit_id = v_visit_id
         and order_no = v_order_no;

      --commit;

      SuccessPackage('执行成功', errorCode, errorMsg);
      return;
    end if;

    --9、根据执行时间描述，生成执行时间数组
    errorMsg        := '时间拆分：' || v_perform_schedule;
    d_execute_array := f_split_time(v_perform_schedule, sysdate);

    --执行时间数组的处理，如果数组个数小于n_freq_counter
    if d_execute_array.count < n_freq_counter then
      select rtrim(ltrim(replace(a.default_time, '：', ':'))),
             a.freq_count,
             a.freq_interval,
             a.freq_interval_unit
        into v_perform_schedule,
             n_freq_counter,
             n_freq_interval,
             v_freq_interval_unit
        from nurse_frequency_dict a
       where frequency = v_frequency;

      if v_perform_schedule is null then
        v_perform_schedule := '';

        for i in 1 .. n_freq_counter loop

          --从0点开始，平均分配一天的时间点
          if d_temp is null then
            d_temp := trunc(sysdate);
          else
            select d_temp + 1.0 / n_freq_counter into d_temp from dual;
          end if;

          if v_perform_schedule is null then
            v_perform_schedule := to_char(d_temp, 'hh24:mi');
          else
            v_perform_schedule := v_perform_schedule || ';' ||
                                  to_char(d_temp, 'hh24:mi');
          end if;
        end loop;
      end if;

      d_execute_array := f_split_time(v_perform_schedule, sysdate);

    end if;

    --10、临时医嘱的处理
    errorMsg := '临时医嘱处理：';
    if n_repeat_indicator = 0 then

      --如果已经有拆分时间的，说明已经拆分过，此处不再做拆分处理
      if d_next_split_date is not null then
        n_count := 0;
        select count(1)
          into n_count
          from nurse_execute
         where patient_id = v_patient_id
           and visit_id = v_visit_id
           and order_no = v_order_no;

        if n_count > 0 then
          NoDataPackage('已有拆分记录', errorCode, errorMsg);
          return;
        end if;
      end if;

      --临时医嘱 拆分
      --1.生成执行时间，根据频次和间隔
      --2.生成条码(含类型)
      --3.插入表
      -- dbms_output.put_line('n_freq_counter' || '='|| n_freq_counter);
      n_execute := 0;
      for i in 1 .. n_freq_counter loop
        d_execute_date_time := to_date(to_char(d_start_date_time,
                                               'yyyy-mm-dd') || ' ' ||
                                       to_char(d_execute_array(i),
                                               'hh24:mi:ss'),
                                       'yyyy-mm-dd hh24:mi:ss');

        -- dbms_output.put_line(to_char(d_execute_date_time,
        --                            'yyyy-mm-dd hh24:mi:ss'));

        v_barcode := f_general_barcode(v_administration);

        --写入执行单
        splitInsertExecute(v_patient_id,
                           v_visit_id,
                           v_order_no,
                           v_barcode,
                           d_execute_date_time,
                           i || '/' || n_freq_counter,
                           errorcode,
                           errorMsg);

        if errorCode <> 1 then
          ErrorPackage(errorMsg, errorcode, errorMsg);
          return;
        end if;

        n_execute := n_execute + 1;

      end loop;

      --写入拆分记录日志
      splitInsertLog(v_patient_id,
                     v_visit_id,
                     v_order_no,
                     sysdate,
                     d_execute_date_time,
                     '生成了' || n_execute || '条执行单',
                     errorcode,
                     errorMsg);
      commit;

      SuccessPackage('执行成功,生成了' || n_execute || '条执行单',
                     errorCode,
                     errorMsg);

      return;
    end if; --临时医嘱的处理完毕

    --以下是长期医嘱的处理
    --11、长期医嘱--停止的处理
    if v_order_status = v_status_stop then
      if d_next_split_date is not null and
         d_next_split_date = d_stop_date_time then
        SuccessPackage('执行完成，医嘱已停止', errorCode, errorMsg);
        return;
      end if;

      /* --停止多余的执行单
      stopExecute(v_patient_id,
                  v_visit_id,
                  v_order_no,
                  d_stop_date_time,
                  errorcode,
                  errormsg);
      return; */
    end if;

    /*
    执行or停止的处理：生成执行单
    1.生成执行时间，根据频次和间隔
    2.开始日期、拆分日期到明天为止
    3.去掉大于执行日期的执行单
    4.生成条码(含类型)
    5.插入表
    */

    --12、长期医嘱的处理1：确定要生成的执行单的时间区间
    --开始拆分的日期
    if d_next_split_date is not null then
      --拆分过的设置开始日期为拆分日期的下一个执行日期
      d_temp_start := d_next_split_date;
    else
      --未拆分过的，从开始日期开始拆分
      select trunc(d_start_date_time) into d_temp_start from dual;
    end if;

    --生成结束日期
    if d_stop_date_time is not null then

      --删掉多余的执行单信息
      delete from nurse_execute a
       where patient_id = v_patient_id
         and visit_id = v_visit_id
         and order_no = v_order_no
         and execute_date_time > d_stop_date_time
         and a.execute_flag = '0';

      update nurse_execute_split a
         set a.next_split_date = d_stop_date_time
       where patient_id = v_patient_id
         and visit_id = v_visit_id
         and order_no = v_order_no;

      -- commit;

      --已停止的，设置为停止日期
      select trunc(d_stop_date_time) into d_temp_end from dual;
    else
      --未停止的，设置结束日期为下一个执行日期
      select trunc(sysdate) + n_day + 1 into d_temp_end from dual;
    end if;

    --如果结束日期小于开始日期
    if d_temp_end < d_temp_start then
      SuccessPackage('执行完成，结束日期小于开始日期', errorCode, errorMsg);
      return;
    end if;

    n_execute := 0;

    --单位为周的处理

    if instr(v_freq_interval_unit, '周') > 0 then

      while trunc(d_temp_start) <= trunc(d_temp_end) loop
        --当前日期是周几
        select to_number(decode(to_char(d_temp_start, 'D'),
                                '1',
                                '8',
                                to_char(d_temp_start, 'D'))) - 1
          into v_week
          from dual;

        if instr(v_perform_schedule, v_week) > 0 then
          --执行日期
          d_execute_date_time := to_date(to_char(d_temp_start, 'yyyy-mm-dd') ||
                                         ' 00:00:00',
                                         'yyyy-mm-dd hh24:mi:ss');

          --去掉比停止日期大的
          if d_execute_date_time > d_stop_date_time and
             d_stop_date_time is not null then
            continue;
          end if;

          --条码号
          v_barcode := f_general_barcode(v_administration);

          --写入执行单
          splitInsertExecute(v_patient_id,
                             v_visit_id,
                             v_order_no,
                             v_barcode,
                             d_execute_date_time,
                             '1/1',
                             errorcode,
                             errorMsg);

          if errorCode <> 1 then
            ErrorPackage(errorMsg, errorcode, errorMsg);
            return;
          end if;

          n_execute := n_execute + 1;

        end if;

        select d_temp_start + 1 into d_temp_start from dual;

      end loop;

    else
      --单位非周的处理
      while trunc(d_temp_start) <= trunc(d_temp_end) loop

        for i in 1 .. n_freq_counter loop
          d_execute_date_time := to_date(to_char(d_temp_start, 'yyyy-mm-dd') || ' ' ||
                                         to_char(d_execute_array(i),
                                                 'hh24:mi:ss'),
                                         'yyyy-mm-dd hh24:mi:ss');

          --去掉比开始日期小的
          if d_execute_date_time < d_start_date_time then
            continue;
          end if;

          --去掉比停止日期大的
          if d_execute_date_time > d_stop_date_time and
             d_stop_date_time is not null then
            continue;
          end if;

          v_barcode := f_general_barcode(v_administration);

          --写入执行单
          splitInsertExecute(v_patient_id,
                             v_visit_id,
                             v_order_no,
                             v_barcode,
                             d_execute_date_time,
                             i || '/' || n_freq_counter,
                             errorcode,
                             errorMsg);

          if errorCode <> 1 then
            ErrorPackage(errorMsg, errorcode, errorMsg);
            return;
          end if;

          n_execute := n_execute + 1;

        end loop;

        select d_temp_start + n_day into d_temp_start from dual;

      end loop;
      --单位非周的处理结束
    end if;

    if d_stop_date_time is not null then
      d_temp_start := d_stop_date_time;
    end if;

    --写入拆分记录日志
    splitInsertLog(v_patient_id,
                   v_visit_id,
                   v_order_no,
                   sysdate,
                   d_temp_start,
                   '生成了' || n_execute || '条执行单',
                   errorcode,
                   errorMsg);
    --commit;

    SuccessPackage('执行成功,生成了' || n_execute || '条执行单',
                   errorCode,
                   errorMsg);

  exception
    when others then
      --rollback;
      ErrorPackage(errorMsg || ' ' || sqlerrm, errorcode, errormsg);
      return;
  end SplitOrder;
```
