### 存储过程：医嘱拆分（SplitOrder）
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
### 医嘱拆分涉及到的数据库表
**医嘱表（NURSE_ORDERS）**
```sql
create table NURSE_ORDERS
(
	PATIENT_ID VARCHAR2(20) not null,
	VISIT_ID NUMBER(20) not null,
	ORDER_NO NUMBER(20) not null,
	ORDER_SUB_NO NUMBER(20) not null,
	REPEAT_INDICATOR NUMBER(5),
	ORDER_CLASS VARCHAR2(50),
	ORDER_CODE VARCHAR2(20),
	ORDER_TEXT VARCHAR2(200),
	DOSAGE NUMBER(10,4),
	DOSAGE_UNITS VARCHAR2(20),
	ADMINISTRATION VARCHAR2(20),
	START_DATE_TIME DATE,
	STOP_DATE_TIME DATE,
	FREQUENCY VARCHAR2(20),
	FREQ_COUNTER NUMBER(5),
	FREQ_INTERVAL NUMBER(5),
	FREQ_INTERVAL_UNIT VARCHAR2(20),
	PERFORM_SCHEDULE VARCHAR2(30),
	DOCTOR VARCHAR2(20),
	NURSE VARCHAR2(20),
	STOP_DOCTOR VARCHAR2(20),
	STOP_NURSE VARCHAR2(20),
	ORDER_STATUS VARCHAR2(10),
	FREQ_DETAIL VARCHAR2(200),
	PERFORM_RESULT VARCHAR2(20),
	ORDERED_DEPT_CODE VARCHAR2(20),
	PERFORMED_DEPT_CODE VARCHAR2(20),
	ISOUT VARCHAR2(10),
	APP_NO VARCHAR2(20),
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	PQ_LAST_PRINT_DATETIME DATE,
	DRUG_BILLING_ATTR VARCHAR2(20),
	WARDCODE VARCHAR2(255 char),
	WARD_CODE VARCHAR2(30 char),
	UPDATE_DATE_TIME DATE,
	EXPAND4 VARCHAR2(20),
	EXPAND5 VARCHAR2(20),
	constraint BIN$JI6W3WRMQTCRFXMW1VCXW
		check ("PATIENT_ID" IS NOT NULL),
	constraint BIN$JULVA8TITWG8JSBHCCU5EW
		check ("ORDER_SUB_NO" IS NOT NULL),
	constraint BIN$YBWN875RSQQSXCP30RGMLG
		check ("VISIT_ID" IS NOT NULL),
	constraint BIN$Z418HPKJT8QLUCWUVZ20W
		check ("ORDER_NO" IS NOT NULL)
)
/

comment on table NURSE_ORDERS is '2.5通过PatientID和VisitId获取住院医嘱(getPatientOrders)'
/

comment on column NURSE_ORDERS.PATIENT_ID is '病人ID'
/

comment on column NURSE_ORDERS.VISIT_ID is '住院次数'
/

comment on column NURSE_ORDERS.ORDER_NO is '母医嘱序号 该字段标志患者每一组医嘱'
/

comment on column NURSE_ORDERS.ORDER_SUB_NO is '子医嘱序号'
/

comment on column NURSE_ORDERS.REPEAT_INDICATOR is '医嘱属性：长期、临时'
/

comment on column NURSE_ORDERS.ORDER_CLASS is '医嘱类型：药品、检查、治疗、化验、护理等，也可以理解为费用类型'
/

comment on column NURSE_ORDERS.ORDER_CODE is '医嘱项目代码'
/

comment on column NURSE_ORDERS.ORDER_TEXT is '医嘱内容'
/

comment on column NURSE_ORDERS.DOSAGE is '单次剂量，当orderClass为药品类时，剂量为必填'
/

comment on column NURSE_ORDERS.DOSAGE_UNITS is '剂量单位，当orderClass为药品类时，剂量单位为必填'
/

comment on column NURSE_ORDERS.ADMINISTRATION is '药品途径，当orderClass为药品类时，途径是必填'
/

comment on column NURSE_ORDERS.START_DATE_TIME is '医嘱开始日期'
/

comment on column NURSE_ORDERS.STOP_DATE_TIME is '医嘱结束日期，结束的医嘱为必填'
/

comment on column NURSE_ORDERS.FREQUENCY is '医嘱频次，当RepeatIndicator是长期时，医嘱频次一般不能为空'
/

comment on column NURSE_ORDERS.FREQ_COUNTER is '频次数量，例：1日两次的为2，1日三次的为3'
/

comment on column NURSE_ORDERS.FREQ_INTERVAL is '频次间隔，例：2日一次的为2，1日一次的为1'
/

comment on column NURSE_ORDERS.FREQ_INTERVAL_UNIT is '频次单位，例：qd为日， q8h为小时'
/

comment on column NURSE_ORDERS.PERFORM_SCHEDULE is '医嘱执行时间说明，一般会关联Frequency，如qd则为8：00，bid为8-16。注意，如果是长期药品医嘱，此执行时间为必填'
/

comment on column NURSE_ORDERS.DOCTOR is '开嘱医生'
/

comment on column NURSE_ORDERS.NURSE is '校对护士'
/

comment on column NURSE_ORDERS.STOP_DOCTOR is '停止医生'
/

comment on column NURSE_ORDERS.STOP_NURSE is '停止校对护士'
/

comment on column NURSE_ORDERS.ORDER_STATUS is '医嘱状态'
/

comment on column NURSE_ORDERS.FREQ_DETAIL is '医生说明'
/

comment on column NURSE_ORDERS.PERFORM_RESULT is '皮试结果，如果医嘱是皮试医嘱，且有皮试结果的，此项为必填'
/

comment on column NURSE_ORDERS.ORDERED_DEPT_CODE is '开单科室'
/

comment on column NURSE_ORDERS.PERFORMED_DEPT_CODE is '执行科室'
/

comment on column NURSE_ORDERS.ISOUT is '是否是出院带药医嘱，1=是，0为否'
/

comment on column NURSE_ORDERS.APP_NO is '如果orderClass为检查，则AppNo为对应的检查申请单号。如果orderClass为检验，则为化验申请单号。如果为手术，则为手术申请单号'
/

comment on column NURSE_ORDERS.EXPAND1 is '拓展字段1'
/

comment on column NURSE_ORDERS.EXPAND2 is '拓展字段2'
/

comment on column NURSE_ORDERS.EXPAND3 is '拓展字段3'
/

comment on column NURSE_ORDERS.WARD_CODE is '护理单元编码'
/

create unique index BIN$KPOR9J3RQHMQSHMH5BWKOA
	on NURSE_ORDERS (PATIENT_ID, VISIT_ID, ORDER_NO, ORDER_SUB_NO)
/

create index BIN$PBXL4L1FT9WMFMTEBPSNA
	on NURSE_ORDERS (REPEAT_INDICATOR)
/

create index IDX_NURSE_ORDERS_WARD
	on NURSE_ORDERS (WARD_CODE, ORDER_TEXT)
/
```
**执行单表（NURSE_EXECUTE）**
```sql
create table NURSE_EXECUTE
(
    EXECUTE_ID                VARCHAR2(20) not null
        constraint PRI_NURSE_EXECUTE
            primary key,
    BARCODE                   VARCHAR2(50),
    BARCODE_OTHER             VARCHAR2(50),
    EXECUTE_DATE_TIME         DATE,
    EXECUTE_TYPE              VARCHAR2(20),
    EXECUTE_FLAG              VARCHAR2(10),
    PATIENT_ID                VARCHAR2(20) not null,
    VISIT_ID                  VARCHAR2(10) not null,
    ORDER_NO                  VARCHAR2(10) not null,
    ORDER_SUB_NO              NUMBER(5)    not null,
    ITEM_CODE                 VARCHAR2(30),
    ITEM_NAME                 VARCHAR2(100),
    ITEM_SPEC                 VARCHAR2(100),
    DOSAGE                    NUMBER(12, 4),
    DOSAGE_UNITS              VARCHAR2(20),
    ADMINISTRATION            VARCHAR2(20),
    FREQUENCY                 VARCHAR2(20),
    PERFORM_SCHEDULE          VARCHAR2(100),
    FREQ_DETAIL               VARCHAR2(100),
    WARD_CODE                 VARCHAR2(20),
    BED_LABEL                 VARCHAR2(20),
    DEPT_CODE                 VARCHAR2(20),
    NAME                      VARCHAR2(50),
    SEX                       VARCHAR2(10),
    INP_NO                    VARCHAR2(20),
    AGE                       VARCHAR2(20),
    REPEAT_INDICATOR          NUMBER(2),
    GROUP_NO                  VARCHAR2(20),
    DRUG_BILLING_ATTR         VARCHAR2(20),
    MEMO                      VARCHAR2(100),
    CREATE_DATE_TIME          DATE,
    PRINT_FLAG                VARCHAR2(1) default 0,
    PRINT_DATE                DATE,
    PRINT_OPERATOR            VARCHAR2(20),
    PRINT_UUID                VARCHAR2(100),
    SAMP_NAME                 VARCHAR2(100),
    SAMP_CODE                 VARCHAR2(20),
    GLASS_NAME                VARCHAR2(100),
    GROUP_NAME                VARCHAR2(100),
    START_DATE_TIME           DATE,
    START_NURSE               VARCHAR2(10),
    START_VERIFY_NURSE        VARCHAR2(10),
    END_DATE_TIME             DATE,
    END_NURSE                 VARCHAR2(10),
    END_VERIFY_NURSE          VARCHAR2(10),
    DISPENSE_FLAG             VARCHAR2(1),
    DISPENSE_DATE_TIME        DATE,
    DISPENSE_NURSE            VARCHAR2(10),
    DISPENSE_VERIFY_DATE_TIME DATE,
    DISPENSE_VERIFY_NURSE     VARCHAR2(10),
    SPEED                     VARCHAR2(10),
    SPEED_UNIT                VARCHAR2(10),
    EXPECT_END_TIME           DATE,
    IS_VALID                  VARCHAR2(255),
    UPDATE_SPEED_REASON       VARCHAR2(255),
    BEI_HE_NURSE              VARCHAR2(255),
    BEI_HE_TIME               DATE,
    BEI_NURSE                 VARCHAR2(255),
    BEI_TIME                  DATE,
    CANCEL_DATE_TIME          DATE,
    CANCEL_NURSE              VARCHAR2(20),
    CANCEL_REASON             VARCHAR2(30),
    CONTINUE_NURSE            VARCHAR2(30),
    CONTINUE_TIME             DATE,
    EXPAND1                   VARCHAR2(255),
    EXPAND2                   VARCHAR2(255),
    EXPAND3                   VARCHAR2(255),
    MATCH_DOSAGE              VARCHAR2(255),
    PATROL_CONTENT            VARCHAR2(255),
    PATROL_NURSE              VARCHAR2(255),
    PATROL_TIME               DATE,
    PAUSE_DATE_TIME           DATE,
    PAUSE_NURSE               VARCHAR2(20),
    PAUSE_REASON              VARCHAR2(30),
    SKIN_RESULT               VARCHAR2(30),
    SKIN_RESULT_DATE_TIME     DATE,
    AMOUNT                    VARCHAR2(255),
    NURSE_MEMO                VARCHAR2(300),
    SUPPLEMENTARYRES          VARCHAR2(300),
    TYPE                      VARCHAR2(255),
    TYPE_REASON               VARCHAR2(255),
    END_VERIFY_TIME           DATE,
    START_VERIFY_TIME         DATE,
    SKIN_MEDI_CODE            VARCHAR2(50)
)
/

comment on column NURSE_EXECUTE.EXECUTE_ID is '主键'
/

comment on column NURSE_EXECUTE.BARCODE is '条码号'
/

comment on column NURSE_EXECUTE.BARCODE_OTHER is '条码号，第三方系统产生的'
/

comment on column NURSE_EXECUTE.EXECUTE_DATE_TIME is '预计执行时间，yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_EXECUTE.EXECUTE_TYPE is '执行单类型'
/

comment on column NURSE_EXECUTE.EXECUTE_FLAG is '执行状态，0=未执行，1=执行中，2=已结束，3=暂停，9=作废，可根据实际情况增加字义'
/

comment on column NURSE_EXECUTE.PATIENT_ID is '病人ID'
/

comment on column NURSE_EXECUTE.VISIT_ID is '住院标识'
/

comment on column NURSE_EXECUTE.ORDER_NO is '医嘱号'
/

comment on column NURSE_EXECUTE.ORDER_SUB_NO is '医嘱子序号'
/

comment on column NURSE_EXECUTE.ITEM_CODE is '医嘱项目代码'
/

comment on column NURSE_EXECUTE.ITEM_NAME is '医嘱内容名称'
/

comment on column NURSE_EXECUTE.ITEM_SPEC is '医嘱规格'
/

comment on column NURSE_EXECUTE.DOSAGE is '剂量/单次执行数量'
/

comment on column NURSE_EXECUTE.DOSAGE_UNITS is '剂量单位'
/

comment on column NURSE_EXECUTE.ADMINISTRATION is '途径'
/

comment on column NURSE_EXECUTE.FREQUENCY is '频次'
/

comment on column NURSE_EXECUTE.PERFORM_SCHEDULE is '执行时间说明'
/

comment on column NURSE_EXECUTE.FREQ_DETAIL is '医生说明'
/

comment on column NURSE_EXECUTE.WARD_CODE is '护理单元代码'
/

comment on column NURSE_EXECUTE.BED_LABEL is '床号'
/

comment on column NURSE_EXECUTE.DEPT_CODE is '科室代码'
/

comment on column NURSE_EXECUTE.NAME is '姓名'
/

comment on column NURSE_EXECUTE.SEX is '性别'
/

comment on column NURSE_EXECUTE.INP_NO is '住院号'
/

comment on column NURSE_EXECUTE.AGE is '年龄'
/

comment on column NURSE_EXECUTE.REPEAT_INDICATOR is '长期=1，临时=0'
/

comment on column NURSE_EXECUTE.GROUP_NO is '组号'
/

comment on column NURSE_EXECUTE.DRUG_BILLING_ATTR is '医嘱标识，一般为自带药、或出院带药'
/

comment on column NURSE_EXECUTE.MEMO is '备注'
/

comment on column NURSE_EXECUTE.CREATE_DATE_TIME is '数据生成日期'
/

comment on column NURSE_EXECUTE.PRINT_FLAG is '打印标志，1=已打印，0=未打印'
/

comment on column NURSE_EXECUTE.PRINT_DATE is '打印日期'
/

comment on column NURSE_EXECUTE.PRINT_OPERATOR is '打印人'
/

comment on column NURSE_EXECUTE.PRINT_UUID is '打印请求ID'
/

comment on column NURSE_EXECUTE.SAMP_NAME is 'LIS系统的标本，由LIS接口写入'
/

comment on column NURSE_EXECUTE.SAMP_CODE is 'LIS系统的标本代码，由LIS接口写入'
/

comment on column NURSE_EXECUTE.GLASS_NAME is 'LIS系统的容器（试管），由LIS接口写入'
/

comment on column NURSE_EXECUTE.GROUP_NAME is 'LIS系统的项目名称，由LIS接口写入'
/

comment on column NURSE_EXECUTE.START_DATE_TIME is '执行开始时间'
/

comment on column NURSE_EXECUTE.START_NURSE is '执行护士'
/

comment on column NURSE_EXECUTE.START_VERIFY_NURSE is '开始校对护士'
/

comment on column NURSE_EXECUTE.END_DATE_TIME is '执行结束时间'
/

comment on column NURSE_EXECUTE.END_NURSE is '结束护士'
/

comment on column NURSE_EXECUTE.END_VERIFY_NURSE is '结束校对护士'
/

comment on column NURSE_EXECUTE.DISPENSE_FLAG is '配液标志，1=已配液，2=已核对'
/

comment on column NURSE_EXECUTE.DISPENSE_DATE_TIME is '配液时间'
/

comment on column NURSE_EXECUTE.DISPENSE_NURSE is '配液护士'
/

comment on column NURSE_EXECUTE.DISPENSE_VERIFY_DATE_TIME is '配液核对时间'
/

comment on column NURSE_EXECUTE.DISPENSE_VERIFY_NURSE is '配液核对护士'
/

comment on column NURSE_EXECUTE.SPEED is '执行速度，一般用于输液'
/

comment on column NURSE_EXECUTE.SPEED_UNIT is '执行速度单位，小时/分钟'
/

comment on column NURSE_EXECUTE.EXPECT_END_TIME is '根据滴速算出来的预计停止时间'
/

create index IDX_NURSE_EXECUTE1
    on NURSE_EXECUTE (PATIENT_ID, VISIT_ID, ORDER_NO, ORDER_SUB_NO)
/

create index IDX_NURSE_EXECUTE2
    on NURSE_EXECUTE (ITEM_CODE)
/

create index IDX_NURSE_EXECUTE3
    on NURSE_EXECUTE (EXECUTE_DATE_TIME)
/

create index IDX_NURSE_EXECUTE4
    on NURSE_EXECUTE (BARCODE)
/

create index IDX_NURSE_EXECUTE5
    on NURSE_EXECUTE (BARCODE_OTHER)
/
```
**执行单拆分时间(NURSE_EXECUTE_SPLIT)**
```sql
create table NURSE_EXECUTE_SPLIT
(
    PATIENT_ID      VARCHAR2(50) not null,
    VISIT_ID        VARCHAR2(10) not null,
    ORDER_NO        VARCHAR2(20) not null,
    NEXT_SPLIT_DATE DATE,
    LAST_SPLIT_DATE DATE,
    PRINT_DATE      DATE,
    MEMO            VARCHAR2(100),
    constraint PRI_NURSE_EXECUTE_SPLIT
        primary key (PATIENT_ID, VISIT_ID, ORDER_NO)
)
/

comment on column NURSE_EXECUTE_SPLIT.PATIENT_ID is '病人ID'
/

comment on column NURSE_EXECUTE_SPLIT.VISIT_ID is '住院标识'
/

comment on column NURSE_EXECUTE_SPLIT.ORDER_NO is '医嘱号'
/

comment on column NURSE_EXECUTE_SPLIT.NEXT_SPLIT_DATE is '下次拆分日期'
/

comment on column NURSE_EXECUTE_SPLIT.LAST_SPLIT_DATE is '本次拆分日期'
/

comment on column NURSE_EXECUTE_SPLIT.PRINT_DATE is '瓶签打印日期'
/

comment on column NURSE_EXECUTE_SPLIT.MEMO is '备注'
/

```
**医嘱频次字典表（NURSE_FREQUENCY_DICT）**
```sql
create table NURSE_FREQUENCY_DICT
(
    FREQUENCY          VARCHAR2(50) not null
        constraint PRI_NURSE_FREQUENCY_DICT
            primary key,
    FREQ_COUNT         NUMBER(5),
    FREQ_INTERVAL      NUMBER(5),
    FREQ_INTERVAL_UNIT VARCHAR2(20),
    DEFAULT_TIME       VARCHAR2(50),
    FREQUENCY_DESC     VARCHAR2(100),
    MEMO               VARCHAR2(50)
)
/

comment on column NURSE_FREQUENCY_DICT.FREQUENCY is '频次'
/

comment on column NURSE_FREQUENCY_DICT.FREQ_COUNT is '频率次数，例：bid（一天2次），freq_counter = 2'
/

comment on column NURSE_FREQUENCY_DICT.FREQ_INTERVAL is '频率间隔，例：bid（一天2次），freq_interval = 1'
/

comment on column NURSE_FREQUENCY_DICT.FREQ_INTERVAL_UNIT is '频率间隔单位，限定：日、周、小时'
/

comment on column NURSE_FREQUENCY_DICT.DEFAULT_TIME is '默认执行时间'
/

comment on column NURSE_FREQUENCY_DICT.FREQUENCY_DESC is '频率的中文描述'
/

comment on column NURSE_FREQUENCY_DICT.MEMO is '备注'
/
```
**执行状态字典表（NURSE_EXECUTE_STATUS_DICT）**
```sql
create table NURSE_EXECUTE_STATUS_DICT
(
    EXECUTE_STATUS VARCHAR2(20),
    CANCEL_STATUS  VARCHAR2(20),
    STOP_STATUS    VARCHAR2(20)
)
/

comment on column NURSE_EXECUTE_STATUS_DICT.EXECUTE_STATUS is '对应的HIS系统的医嘱执行状态'
/

comment on column NURSE_EXECUTE_STATUS_DICT.CANCEL_STATUS is '对应的HIS系统的医嘱作废状态'
/

comment on column NURSE_EXECUTE_STATUS_DICT.STOP_STATUS is '对应的HIS系统的医嘱停止状态'
/

```

### 方法：生成条码号
```sql
--生成barcode
  function f_general_barcode(v_administration in varchar2) return varchar2 as
    v_prefix    varchar2(20);
    v_serial_no varchar2(20);
    n_seq_no    number(10);
    n_count     number(5);
  begin

    n_count := 0;
    select count(1)
      into n_count
      from nurse_administration_dict
     where ADMINISTRATION_NAME = v_administration;
    if n_count > 0 then

      SELECT BARCODE_CLASS--execute_prefix
        into v_prefix
        from nurse_administration_dict
       where ADMINISTRATION_NAME = v_administration;
    end if;

    if v_prefix is null then
      v_prefix := 'QT';
    end if;

    select healthcare.nurse_execute_sequence.nextval
      into n_seq_no
      from dual;

    v_serial_no := lpad(to_char(n_seq_no), 10, '0');

    return 'ZXD' || v_prefix || v_serial_no;

  end f_general_barcode;
```
### 方法：生成执行时间数组
```sql
function f_split_time(v_perform_schedule in varchar2, --执行时间，以分号隔开的时间格式，如9:30;10:30;
                        d_execute_date     date) return dateArray --执行日期yyyy-mm-dd
   as
    v_time varchar2(500);
    v_temp varchar2(500);

    d_return_date date;

    d_date_array dateArray;

    i     number(5);
    n_pos number(5);
  begin

    d_date_array := dateArray();

    v_temp := v_perform_schedule;

    if instr(v_temp, '补') > 0 or instr(v_temp, '术中') > 0 or
       instr(v_temp, '取药') > 0 or instr(v_temp, '领药') > 0 then
      v_temp := null;
    end if;

    if v_temp is null then
      d_date_array.EXTEND;
      d_date_array(1) := d_execute_date;
      return d_date_array;
    end if;

    --时间格式是mm-dd hh24:mi
    if length(v_temp) = 11 and substr(v_temp, 3, 1) = '-' and
       substr(v_temp, 9, 1) = ':' then
      d_date_array.EXTEND;
      d_date_array(1) := to_date(to_char(sysdate, 'yyyy') || '-' || v_temp || ':' || '00',
                                 'yyyy-mm-dd hh24:mi:ss');

      return d_date_array;
    end if;

    if instr(v_temp, '-') > 0 and instr(v_temp, ':') > 0 then
      d_date_array.EXTEND;
      d_date_array(1) := to_date(to_char(sysdate, 'yyyy') || '-' || v_temp || ':' || '00',
                                 'yyyy-mm-dd hh24:mi:ss');

      return d_date_array;
    end if;

    select replace(v_temp, '-', ';') into v_temp from dual;
    select replace(v_temp, '；', ';') into v_temp from dual;
    select replace(v_temp, '：', ':') into v_temp from dual;

    v_temp := v_temp || ';';
    i      := 0;

    --拆分时间点
    while instr(v_temp, ';') > 0 loop
      n_pos  := instr(v_temp, ';');
      v_time := substr(v_temp, 0, n_pos - 1);

      if instr(v_time, '24') = 1 then
        v_time := '23:59';
      end if;

      if instr(v_time, ':') <= 0 then
        v_time := v_time || ':00:00';
      else
        v_time := v_time || ':00';
      end if;

      select to_date(to_char(d_execute_date, 'yyyy-mm-dd') || ' ' || v_time,
                     'yyyy-mm-dd hh24:mi:ss')
        into d_return_date
        from dual;

      i := i + 1;

      d_date_array.EXTEND;
      d_date_array(i) := d_return_date;

      v_temp := substr(v_temp, n_pos + 1);
    end loop;

    return d_date_array;

  end f_split_time;
```
