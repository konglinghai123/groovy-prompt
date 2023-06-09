

#入科出科记录
```sql
create table NURSE_ADT_LOG
(
	PATIENT_ID VARCHAR2(20) not null,
	VISIT_ID NUMBER(20) not null,
	WARD_CODE VARCHAR2(20),
	BED_NO VARCHAR2(20),
	DEPT_CODE VARCHAR2(20),
	ADMISSION_DATE_TIME DATE not null,
	DISCHARGE_DATE_TIME DATE,
	NURSE VARCHAR2(20),
	DOCTOR VARCHAR2(20),
	TRANSFER_FROM VARCHAR2(20),
	TRANSFER_TO VARCHAR2(20),
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	LEND_DEPT_CODE VARCHAR2(20),
	BED_LABEL VARCHAR2(20 char),
	TRANSFER_FROM_DEPT_CODE VARCHAR2(20 char),
	TRANSFER_TO_DEPT_CODE VARCHAR2(20 char),
	UPDATE_DATE_TIME DATE,
	constraint BIN$AXDQQJ4PT9K6SHYMC14FQ
		check ("PATIENT_ID" IS NOT NULL),
	constraint BIN$VEM4Y5ISRVKJ9EFD1QRHMG
		check ("VISIT_ID" IS NOT NULL),
	constraint BIN$XQQR4BQTTYJIPOZ6Q9QUA
		check ("ADMISSION_DATE_TIME" IS NOT NULL)
)
/

comment on table NURSE_ADT_LOG is '2.7通过PatientID和VisitId获取入科出科记录(getPatientTransfer)'
/

comment on column NURSE_ADT_LOG.PATIENT_ID is '病人ID'
/

comment on column NURSE_ADT_LOG.VISIT_ID is '住院次数'
/

comment on column NURSE_ADT_LOG.WARD_CODE is '护理单元代码'
/

comment on column NURSE_ADT_LOG.BED_NO is '床序号（注意不是床标号）'
/

comment on column NURSE_ADT_LOG.DEPT_CODE is '住院科室代码'
/

comment on column NURSE_ADT_LOG.ADMISSION_DATE_TIME is '入科时间yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_ADT_LOG.DISCHARGE_DATE_TIME is '出科时间yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_ADT_LOG.NURSE is '责任护士代码'
/

comment on column NURSE_ADT_LOG.DOCTOR is '住院医生代码'
/

comment on column NURSE_ADT_LOG.TRANSFER_FROM is '源科室：转入时，不能为空'
/

comment on column NURSE_ADT_LOG.TRANSFER_TO is '去向科室：转出时，不能为空'
/

comment on column NURSE_ADT_LOG.EXPAND1 is '拓展字段1'
/

comment on column NURSE_ADT_LOG.EXPAND2 is '拓展字段2'
/

comment on column NURSE_ADT_LOG.EXPAND3 is '拓展字段3'
/

comment on column NURSE_ADT_LOG.LEND_DEPT_CODE is '借床科室代码'
/

comment on column NURSE_ADT_LOG.BED_LABEL is '床号'
/

comment on column NURSE_ADT_LOG.TRANSFER_FROM_DEPT_CODE is '来源科室：转入时，不能为空'
/

comment on column NURSE_ADT_LOG.TRANSFER_TO_DEPT_CODE is '去向科室：转出时，不能为空'
/

create unique index BIN$UHZ3IRA2QW2CXSAWBTO9PQ
	on NURSE_ADT_LOG (PATIENT_ID, VISIT_ID, ADMISSION_DATE_TIME)
/

create index BIN$HDRPCP3HRJARWTJYJBT8QA
	on NURSE_ADT_LOG (DISCHARGE_DATE_TIME)
/

create index BIN$MQIPFAIRYUQONECQXUGBQ
	on NURSE_ADT_LOG (ADMISSION_DATE_TIME)
/
```

#诊断表 

```sql
create table NURSE_DIAGNOSIS
(
	PATIENT_ID VARCHAR2(20) not null,
	VISIT_ID NUMBER(20) not null,
	DIAGNOSIS_DATE DATE,
	DIAGNOSIS_TYPE VARCHAR2(10) not null,
	DIAGNOSIS_NO NUMBER(5) not null,
	DIAGNOSIS_CODE VARCHAR2(50),
	DIAGNOSIS_DESC VARCHAR2(200),
	CURE_RESULT NUMBER(3),
	EXPAND1 VARCHAR2(10),
	EXPAND2 VARCHAR2(10),
	EXPAND3 VARCHAR2(10),
	CREATE_DATE_TIME DATE,
	ID NUMBER,
	IS_DELETED NUMBER(1),
	CREATE_TIME DATE,
	UPDATE_DATE_TIME DATE,
	constraint BIN$EFDGQ4FJRREGA4S7XMHMBA
		check ("DIAGNOSIS_NO" IS NOT NULL),
	constraint BIN$KRA4UEWKSDOENYIJ6CDE2G
		check ("PATIENT_ID" IS NOT NULL),
	constraint BIN$NHMYJPLGRSQQKAFNWFPX7A0
		check ("VISIT_ID" IS NOT NULL),
	constraint BIN$ZSLBWZO4SMUNTESO0B9DFW
		check ("DIAGNOSIS_TYPE" IS NOT NULL)
)
/

comment on table NURSE_DIAGNOSIS is '2.9通过PatientID和VisitId获取诊断表(getPatientDiagnosis)'
/

comment on column NURSE_DIAGNOSIS.PATIENT_ID is '病人ID'
/

comment on column NURSE_DIAGNOSIS.VISIT_ID is '住院次数'
/

comment on column NURSE_DIAGNOSIS.DIAGNOSIS_DATE is '诊断日期 yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_DIAGNOSIS.DIAGNOSIS_TYPE is '诊断类别，1=门诊诊断，2=入院诊断，3=出院诊断'
/

comment on column NURSE_DIAGNOSIS.DIAGNOSIS_NO is '每个诊断类别下的诊断序号'
/

comment on column NURSE_DIAGNOSIS.DIAGNOSIS_CODE is 'ICD10码'
/

comment on column NURSE_DIAGNOSIS.DIAGNOSIS_DESC is '诊断说明'
/

comment on column NURSE_DIAGNOSIS.CURE_RESULT is '治疗效果。1、治愈；2、好转；3、未愈；4、死亡'
/

comment on column NURSE_DIAGNOSIS.EXPAND1 is '拓展字段1'
/

comment on column NURSE_DIAGNOSIS.EXPAND2 is '拓展字段2'
/

comment on column NURSE_DIAGNOSIS.EXPAND3 is '拓展字段3'
/

create unique index BIN$L44BBOKDREQIVUUBPQ6PIG
	on NURSE_DIAGNOSIS (PATIENT_ID, VISIT_ID, DIAGNOSIS_TYPE, DIAGNOSIS_NO)
/

create index BIN$KIMEUN4KS1IHW0U5IWJVG
	on NURSE_DIAGNOSIS (DIAGNOSIS_DATE)
/
```

#病人检查申请记录

```sql
create table NURSE_EXAM_MASTER
(
	EXAM_NO VARCHAR2(25) not null,
	PATIENT_ID VARCHAR2(20) not null,
	VISIT_ID NUMBER(20),
	VISIT_NO VARCHAR2(20),
	EXAM_ITEM VARCHAR2(500),
	EXAM_CLASS VARCHAR2(20),
	REQ_DATE_TIME DATE,
	REQ_DOCTOR VARCHAR2(20),
	REQ_DEPT_CODE VARCHAR2(20),
	REPORT_DATE_TIME DATE,
	REPORT_DOCTOR VARCHAR2(20),
	EXAM_STATUS VARCHAR2(10),
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	ORDER_CODE VARCHAR2(20 char),
	UPDATE_DATE_TIME DATE,
	constraint BIN$HXATO4WHRTWZ9LITRG6MBQ
		check ("EXAM_NO" IS NOT NULL)
)
/

comment on table NURSE_EXAM_MASTER is '2.18通过PatientID获取病人检查申请记录（getExamList)'
/

comment on column NURSE_EXAM_MASTER.EXAM_NO is '检查申请号，主键'
/

comment on column NURSE_EXAM_MASTER.PATIENT_ID is '病人ID'
/

comment on column NURSE_EXAM_MASTER.VISIT_ID is '住院次数，如果是住院申请记录，则visit不能为空'
/

comment on column NURSE_EXAM_MASTER.VISIT_NO is '门诊就诊号，如果是门诊申请记录，则VisitNo不能为空'
/

comment on column NURSE_EXAM_MASTER.EXAM_ITEM is '检查项目'
/

comment on column NURSE_EXAM_MASTER.EXAM_CLASS is '检查类别：CT、MR、心电图等'
/

comment on column NURSE_EXAM_MASTER.REQ_DATE_TIME is '申请日期 yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_EXAM_MASTER.REQ_DOCTOR is '申请医生'
/

comment on column NURSE_EXAM_MASTER.REQ_DEPT_CODE is '申请科室代码'
/

comment on column NURSE_EXAM_MASTER.REPORT_DATE_TIME is '报告日期 yyyy-mm-dd hh24:mi:ss，当检查有结果时，不能为空'
/

comment on column NURSE_EXAM_MASTER.REPORT_DOCTOR is '报告医生，当检查有结果时，不能为空'
/

comment on column NURSE_EXAM_MASTER.EXAM_STATUS is '检查申请状态：提交申请、确认报告、审核报告等'
/

comment on column NURSE_EXAM_MASTER.EXPAND1 is '拓展字段1'
/

comment on column NURSE_EXAM_MASTER.EXPAND2 is '拓展字段2'
/

comment on column NURSE_EXAM_MASTER.EXPAND3 is '拓展字段3'
/

comment on column NURSE_EXAM_MASTER.ORDER_CODE is '贵医大新增字段,别的医院不需要'
/

create index BIN$9Z35T7JSTHMYXBX8VAHQVA
	on NURSE_EXAM_MASTER (PATIENT_ID)
/

create index BIN$B4JXC1XMSGKHWPXXMVP77Q
	on NURSE_EXAM_MASTER (REPORT_DATE_TIME)
/

create index BIN$G52QAO8WQCO7IKXAVJEONG
	on NURSE_EXAM_MASTER (REQ_DATE_TIME)
/

create unique index BIN$PJRTKYYDQJYHNWYOLQ0VVQ
	on NURSE_EXAM_MASTER (EXAM_NO)
/

create unique index BIN$RQCLQ63SS0IEACS4ZZNYQ
	on NURSE_EXAM_MASTER (EXAM_NO, PATIENT_ID)
/
```

#检查结果表

```sql
create table NURSE_EXAM_RESULT
(
	EXAM_NO VARCHAR2(25) not null,
	IMPRESSION VARCHAR2(2000),
	DESCRIPTION VARCHAR2(2000),
	IS_ABNORMAL VARCHAR2(10),
	REPORT_DATE_TIME DATE,
	REPORT_DOCTOR VARCHAR2(20),
	IMAGE_URL VARCHAR2(2000),
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	UPDATE_DATE_TIME DATE,
	constraint BIN$T0RDTT9CRKYQ5CF8S9O44W
		check ("EXAM_NO" IS NOT NULL)
)
/

comment on table NURSE_EXAM_RESULT is '@Column(name="",length=2000)'
/

comment on column NURSE_EXAM_RESULT.EXAM_NO is '检查申请号'
/

comment on column NURSE_EXAM_RESULT.IMPRESSION is '印象'
/

comment on column NURSE_EXAM_RESULT.DESCRIPTION is '检查所见'
/

comment on column NURSE_EXAM_RESULT.IS_ABNORMAL is '是否异常：是/否'
/

comment on column NURSE_EXAM_RESULT.REPORT_DATE_TIME is '报告日期 yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_EXAM_RESULT.REPORT_DOCTOR is '报告医生'
/

comment on column NURSE_EXAM_RESULT.IMAGE_URL is '检查报告图片地址，支持JPG/PDF/BMP/Dicom，多个地址以;隔开'
/

comment on column NURSE_EXAM_RESULT.EXPAND1 is '拓展字段1'
/

comment on column NURSE_EXAM_RESULT.EXPAND2 is '拓展字段2'
/

comment on column NURSE_EXAM_RESULT.EXPAND3 is '拓展字段3'
/

create unique index BIN$NXVN5O13RQKXMNK2WMQWDQ
	on NURSE_EXAM_RESULT (EXAM_NO)
/

create index BIN$YLWVCFUREYDSYZ6XVCXKQ
	on NURSE_EXAM_RESULT (REPORT_DATE_TIME)
/
```

#护士执行单表

```sql
create table NURSE_ORDERS_EXECUTE_LC
(
	BARCODE VARCHAR2(50 char) not null,
	ITEM_NO NUMBER(10) not null,
	CREATE_DATE_TIME DATE,
	UPDATE_REASON VARCHAR2(255 char),
	ADMINISTRATION VARCHAR2(20 char),
	BEI_NURSE VARCHAR2(255 char),
	BEI_TIME DATE,
	DEPT_CODE VARCHAR2(20 char),
	DOSAGE VARCHAR2(255 char),
	DOSAGE_UNITS VARCHAR2(20 char),
	DRIP_SPEED VARCHAR2(255 char),
	END_DATE_TIME DATE,
	END_NURSE VARCHAR2(20 char),
	END_NURSE_CHECK VARCHAR2(20 char),
	EXECUTE_DATE_TIME DATE,
	EXECUTE_STATUS VARCHAR2(10 char),
	EXECUTE_TYPE VARCHAR2(20 char),
	EXPAND1 VARCHAR2(20 char),
	EXPAND2 VARCHAR2(20 char)
		constraint SYS_NURSE_ORDERS_EXECUTE_LC
			unique,
	EXPAND3 VARCHAR2(20 char),
	FREQUENCY VARCHAR2(20 char),
	GROUP_NO VARCHAR2(20 char),
	HE_NURSE VARCHAR2(255 char),
	HE_TIME DATE,
	ITEM_NAME VARCHAR2(200 char),
	ITEM_SPEC VARCHAR2(50 char),
	OPERATION_STATUS VARCHAR2(255 char),
	ORDER_NO VARCHAR2(255 char),
	ORDER_SUB_NO NUMBER(10),
	PATIENT_ID VARCHAR2(20 char),
	PAUSE_DATE_TIME DATE,
	PAUSE_NURSE VARCHAR2(20 char),
	PAUSE_REASON VARCHAR2(30 char),
	PEI_NURSE VARCHAR2(255 char),
	PEI_TIME DATE,
	REPEAT_INDICATOR VARCHAR2(10 char),
	SKIN_BATCH VARCHAR2(30 char),
	SKIN_RESULT VARCHAR2(30 char),
	SKIN_RESULT_DATE_TIME DATE,
	START_DATE_TIME DATE,
	START_NURSE VARCHAR2(20 char),
	START_NURSE_CHECK VARCHAR2(20 char),
	VISIT_ID NUMBER(10),
	WARD_CODE VARCHAR2(20 char),
	TYPE VARCHAR2(255 char),
	VALID_FLAG VARCHAR2(255 char),
	FREQEUNCY VARCHAR2(20 char),
	TYPE_REASON VARCHAR2(255 char),
	ADMINISTRATION_CODE VARCHAR2(255 char),
	ORDER_CLASS VARCHAR2(20 char),
	ORDERS_EXECUTE_NO VARCHAR2(20 char),
	CANCEL_DATE_TIME DATE,
	CANCEL_NURSE VARCHAR2(20 char),
	CANCEL_REASON VARCHAR2(30 char),
	CONTINUE_NURSE VARCHAR2(30 char),
	CONTINUE_TIME DATE,
	EXECUTE_DATE_TIME_NEW VARCHAR2(30 char),
	BEI_HE_NURSE VARCHAR2(255 char),
	BEI_HE_TIME DATE,
	DRIP_SPEED_UNIT VARCHAR2(255 char),
	IS_BEI_FLAG VARCHAR2(255 char),
	AMOUNT VARCHAR2(255 char),
	PRINT_DATE DATE,
	PRINT_FLAG VARCHAR2(10 char),
	PRINT_OPERATOR VARCHAR2(100 char),
	PRINT_UUID VARCHAR2(100 char),
	FREQ_DETAIL VARCHAR2(20 char),
	BARCODE_OTHER VARCHAR2(20 char),
	UPDATE_DATE_TIME DATE,
	EXPECT_END_TIME DATE,
	EXPAND4 VARCHAR2(50 char),
	EXPAND5 VARCHAR2(50 char),
	EXPAND6 VARCHAR2(50 char),
	primary key (BARCODE, ITEM_NO),
	constraint BIN$HGER440ISQKXYF11NWETRW
		check ("BARCODE" IS NOT NULL),
	constraint BIN$XSLL4FCUSBEVYEEVBBPIKA
		check ("ITEM_NO" IS NOT NULL)
)
/

comment on table NURSE_ORDERS_EXECUTE_LC is '3.1通过PatientId获取执行单信息(getOrdersExecute)(聊城二院执行单是以抽取然后回写在数据库)'
/

comment on column NURSE_ORDERS_EXECUTE_LC.BARCODE is '条码号'
/

comment on column NURSE_ORDERS_EXECUTE_LC.ITEM_NO is '同组条码的项目序号'
/

comment on column NURSE_ORDERS_EXECUTE_LC.UPDATE_REASON is '修改输液滴速的原因'
/

comment on column NURSE_ORDERS_EXECUTE_LC.ADMINISTRATION is '途径'
/

comment on column NURSE_ORDERS_EXECUTE_LC.BEI_NURSE is '摆药人'
/

comment on column NURSE_ORDERS_EXECUTE_LC.BEI_TIME is '摆药时间'
/

comment on column NURSE_ORDERS_EXECUTE_LC.DEPT_CODE is '所在科室代码'
/

comment on column NURSE_ORDERS_EXECUTE_LC.DOSAGE is '单次剂量（单次数量）'
/

comment on column NURSE_ORDERS_EXECUTE_LC.DOSAGE_UNITS is '剂量单位'
/

comment on column NURSE_ORDERS_EXECUTE_LC.DRIP_SPEED is '输液的滴速'
/

comment on column NURSE_ORDERS_EXECUTE_LC.END_DATE_TIME is '执行结束日期yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_ORDERS_EXECUTE_LC.END_NURSE is '结束护士'
/

comment on column NURSE_ORDERS_EXECUTE_LC.END_NURSE_CHECK is '双签结束护士'
/

comment on column NURSE_ORDERS_EXECUTE_LC.EXECUTE_DATE_TIME is '预计执行时间yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_ORDERS_EXECUTE_LC.EXECUTE_STATUS is '执行单状态:0-未执行、1-执行中（输液中）、2-暂停输液、3-已完成（结束输液）'
/

comment on column NURSE_ORDERS_EXECUTE_LC.EXECUTE_TYPE is '类别：输液、注射、口服、雾化、皮试、治疗（理疗）、标本、输血等'
/

comment on column NURSE_ORDERS_EXECUTE_LC.EXPAND1 is '拓展字段1'
/

comment on column NURSE_ORDERS_EXECUTE_LC.EXPAND2 is '拓展字段2'
/

comment on column NURSE_ORDERS_EXECUTE_LC.EXPAND3 is '拓展字段3'
/

comment on column NURSE_ORDERS_EXECUTE_LC.GROUP_NO is '组号（组号是指长期医嘱中，一天需要执行多次的，组号 = 第几次 / 共几次）
     如bid的医嘱，groupNo应该为1/2 ， 1/2，分别代表第一组和第二组'
/

comment on column NURSE_ORDERS_EXECUTE_LC.HE_NURSE is '核对人'
/

comment on column NURSE_ORDERS_EXECUTE_LC.HE_TIME is '核对时间'
/

comment on column NURSE_ORDERS_EXECUTE_LC.ITEM_NAME is '执行单内容'
/

comment on column NURSE_ORDERS_EXECUTE_LC.ITEM_SPEC is '规格'
/

comment on column NURSE_ORDERS_EXECUTE_LC.OPERATION_STATUS is '操作状态(摆药配药):null-未摆药、1-已摆药、2-已配药、3、已核对'
/

comment on column NURSE_ORDERS_EXECUTE_LC.ORDER_NO is '对应的医嘱号'
/

comment on column NURSE_ORDERS_EXECUTE_LC.ORDER_SUB_NO is '对应的医嘱子序号'
/

comment on column NURSE_ORDERS_EXECUTE_LC.PATIENT_ID is '病人ID'
/

comment on column NURSE_ORDERS_EXECUTE_LC.PAUSE_DATE_TIME is '暂停时间'
/

comment on column NURSE_ORDERS_EXECUTE_LC.PAUSE_NURSE is '输液暂停人'
/

comment on column NURSE_ORDERS_EXECUTE_LC.PAUSE_REASON is '暂停原因'
/

comment on column NURSE_ORDERS_EXECUTE_LC.PEI_NURSE is '配药人'
/

comment on column NURSE_ORDERS_EXECUTE_LC.PEI_TIME is '配药时间'
/

comment on column NURSE_ORDERS_EXECUTE_LC.REPEAT_INDICATOR is '医嘱类型:1-长期 ,0-临时'
/

comment on column NURSE_ORDERS_EXECUTE_LC.SKIN_BATCH is '皮试批号'
/

comment on column NURSE_ORDERS_EXECUTE_LC.SKIN_RESULT is '皮试结果'
/

comment on column NURSE_ORDERS_EXECUTE_LC.SKIN_RESULT_DATE_TIME is '填皮试结果的时间'
/

comment on column NURSE_ORDERS_EXECUTE_LC.START_DATE_TIME is '执行开始日期yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_ORDERS_EXECUTE_LC.START_NURSE is '开始护士工号'
/

comment on column NURSE_ORDERS_EXECUTE_LC.START_NURSE_CHECK is '双签护士工号'
/

comment on column NURSE_ORDERS_EXECUTE_LC.VISIT_ID is '住院次数'
/

comment on column NURSE_ORDERS_EXECUTE_LC.WARD_CODE is '所在护理单元代码'
/

comment on column NURSE_ORDERS_EXECUTE_LC.TYPE is '补执行标识：0-正常执行 1-补执行'
/

comment on column NURSE_ORDERS_EXECUTE_LC.VALID_FLAG is '执行单有效标识：1-有效、0-无效'
/

comment on column NURSE_ORDERS_EXECUTE_LC.FREQEUNCY is '频次'
/

comment on column NURSE_ORDERS_EXECUTE_LC.TYPE_REASON is '补执行的原因'
/

comment on column NURSE_ORDERS_EXECUTE_LC.ADMINISTRATION_CODE is '途径编码'
/

comment on column NURSE_ORDERS_EXECUTE_LC.ORDER_CLASS is '医嘱类型'
/

comment on column NURSE_ORDERS_EXECUTE_LC.ORDERS_EXECUTE_NO is '医嘱回传ID'
/

comment on column NURSE_ORDERS_EXECUTE_LC.CANCEL_DATE_TIME is '异常取消时间'
/

comment on column NURSE_ORDERS_EXECUTE_LC.CANCEL_NURSE is '异常取消人'
/

comment on column NURSE_ORDERS_EXECUTE_LC.CANCEL_REASON is '异常取消原因'
/

comment on column NURSE_ORDERS_EXECUTE_LC.CONTINUE_NURSE is '继续执行人'
/

comment on column NURSE_ORDERS_EXECUTE_LC.CONTINUE_TIME is '继续执行时间'
/

comment on column NURSE_ORDERS_EXECUTE_LC.EXECUTE_DATE_TIME_NEW is '预计执行时间yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_ORDERS_EXECUTE_LC.BEI_HE_NURSE is '摆药核对人'
/

comment on column NURSE_ORDERS_EXECUTE_LC.BEI_HE_TIME is '摆药核对时间'
/

comment on column NURSE_ORDERS_EXECUTE_LC.DRIP_SPEED_UNIT is '输液滴速单位'
/

comment on column NURSE_ORDERS_EXECUTE_LC.IS_BEI_FLAG is '是否摆药配药:0-不需要摆药  ,1-需要摆药*/   //是否静配 0表示静配 ，1不是静配'
/

comment on column NURSE_ORDERS_EXECUTE_LC.BARCODE_OTHER is '第三方条码'
/

create index IDX_NOEL_ORDERNO
	on NURSE_ORDERS_EXECUTE_LC (ORDER_NO, PATIENT_ID, VISIT_ID, EXECUTE_DATE_TIME)
/
```

#费用明细

```sql
create table NURSE_INP_BILL_DETAIL
(
	PATIENT_ID VARCHAR2(20) not null,
	VISIT_ID NUMBER(20) not null,
	ITEM_NO NUMBER(10) not null,
	BILLING_DATE_TIME DATE not null,
	ITEM_CODE VARCHAR2(20) not null,
	ITEM_NAME VARCHAR2(200),
	AMOUNT NUMBER(10,4),
	UNITS VARCHAR2(40),
	PRICE NUMBER(10,4),
	COSTS NUMBER(10,4),
	FEE_CLASS VARCHAR2(20),
	INSUR_FLAG VARCHAR2(10),
	ORDER_NO VARCHAR2(20),
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	UPDATE_DATE_TIME DATE,
	constraint BIN$7T6N0CNRRQQJLI2ORM6XA
		check ("VISIT_ID" IS NOT NULL),
	constraint BIN$EKRCPYZ4SMHXMV48EUZZQ
		check ("BILLING_DATE_TIME" IS NOT NULL),
	constraint BIN$LQJNZGYPRXGH4MTI1KITQA
		check ("PATIENT_ID" IS NOT NULL),
	constraint BIN$LSPUCGCQ089HJ05WTCOW
		check ("ITEM_CODE" IS NOT NULL)
)
/

comment on table NURSE_INP_BILL_DETAIL is '2.13通过PatientID和VisitId获取费用明细(getPatientBill)'
/

comment on column NURSE_INP_BILL_DETAIL.PATIENT_ID is '病人ID'
/

comment on column NURSE_INP_BILL_DETAIL.VISIT_ID is '住院次数'
/

comment on column NURSE_INP_BILL_DETAIL.ITEM_NO is '序号'
/

comment on column NURSE_INP_BILL_DETAIL.BILLING_DATE_TIME is '费用时间yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_INP_BILL_DETAIL.ITEM_CODE is '项目代码'
/

comment on column NURSE_INP_BILL_DETAIL.ITEM_NAME is '费用名称'
/

comment on column NURSE_INP_BILL_DETAIL.AMOUNT is '数量'
/

comment on column NURSE_INP_BILL_DETAIL.UNITS is '计价单位'
/

comment on column NURSE_INP_BILL_DETAIL.PRICE is '单价'
/

comment on column NURSE_INP_BILL_DETAIL.COSTS is '金额'
/

comment on column NURSE_INP_BILL_DETAIL.FEE_CLASS is '费用分类：西药、中成药、检查费、化验费等'
/

comment on column NURSE_INP_BILL_DETAIL.INSUR_FLAG is '医保类型：
  1 = 甲类
  2 = 乙类
  3 = 自费'
/

comment on column NURSE_INP_BILL_DETAIL.ORDER_NO is '费用来源医嘱的医嘱号'
/

comment on column NURSE_INP_BILL_DETAIL.EXPAND1 is '拓展字段1'
/

comment on column NURSE_INP_BILL_DETAIL.EXPAND2 is '拓展字段2'
/

comment on column NURSE_INP_BILL_DETAIL.EXPAND3 is '拓展字段3'
/

create unique index BIN$YZXMXVJYQRKYXRTXDXOSQ
	on NURSE_INP_BILL_DETAIL (PATIENT_ID, VISIT_ID, ITEM_NO, BILLING_DATE_TIME, ITEM_CODE)
/

create unique index BIN$5WCNITPRKQJBNJX9OAG
	on NURSE_INP_BILL_DETAIL (PATIENT_ID, VISIT_ID, ITEM_NO)
/

create index BIN$CDZJ0T9SRPUAFGQIF6EU9A
	on NURSE_INP_BILL_DETAIL (BILLING_DATE_TIME)
/

create index BIN$HJMRLIZS96BNILMS3I8Q
	on NURSE_INP_BILL_DETAIL (ITEM_CODE)
/
```

#病人化验申请记录

```sql
create table NURSE_LAB_MASTER
(
	TEST_NO VARCHAR2(20) not null,
	PATIENT_ID VARCHAR2(20),
	VISIT_ID NUMBER(20),
	VISIT_NO VARCHAR2(20),
	TEST_ITEM VARCHAR2(2000),
	SPECIMEN VARCHAR2(20),
	REQ_DATE_TIME DATE,
	REQ_DOCTOR VARCHAR2(50),
	REQ_DEPT_CODE VARCHAR2(20),
	REPORT_DATE_TIME DATE,
	REPORT_DOCTOR VARCHAR2(20),
	REPORT_DEPT_CODE VARCHAR2(20),
	TEST_STATUS VARCHAR2(10),
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	UPDATE_DATE_TIME DATE,
	constraint BIN$JUVHKX6WRDWOU0JE1LB4LQ
		check ("TEST_NO" IS NOT NULL)
)
/

comment on table NURSE_LAB_MASTER is '2.20通过PatientID获取病人化验申请记录（GetLabList)'
/

comment on column NURSE_LAB_MASTER.TEST_NO is '化验申请号，主键'
/

comment on column NURSE_LAB_MASTER.PATIENT_ID is '病人ID'
/

comment on column NURSE_LAB_MASTER.VISIT_ID is '住院次数，如果是住院申请记录，则visit不能为空'
/

comment on column NURSE_LAB_MASTER.VISIT_NO is '门诊就诊号，如果是门诊申请记录，则VisitNo不能为空'
/

comment on column NURSE_LAB_MASTER.TEST_ITEM is '化验项目'
/

comment on column NURSE_LAB_MASTER.SPECIMEN is '标本'
/

comment on column NURSE_LAB_MASTER.REQ_DATE_TIME is '申请日期 yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_LAB_MASTER.REQ_DOCTOR is '申请医生'
/

comment on column NURSE_LAB_MASTER.REQ_DEPT_CODE is '申请科室'
/

comment on column NURSE_LAB_MASTER.REPORT_DATE_TIME is '报告日期 yyyy-mm-dd hh24:mi:ss，有报告结果时不能为空'
/

comment on column NURSE_LAB_MASTER.REPORT_DOCTOR is '报告医生，有报告结果时不能为空'
/

comment on column NURSE_LAB_MASTER.REPORT_DEPT_CODE is '执行科室'
/

comment on column NURSE_LAB_MASTER.TEST_STATUS is '化验申请状态：提交申请、确认报告、审核报告等'
/

comment on column NURSE_LAB_MASTER.EXPAND1 is '拓展字段1'
/

comment on column NURSE_LAB_MASTER.EXPAND2 is '拓展字段2'
/

comment on column NURSE_LAB_MASTER.EXPAND3 is '拓展字段3'
/

create unique index BIN$ZFBCBRNISPKMG9WV0G4TWG
	on NURSE_LAB_MASTER (TEST_NO)
/

create index BIN$1Z4Y7TIAQPGMRL6PUM1MPG
	on NURSE_LAB_MASTER (REPORT_DOCTOR)
/

create index BIN$MXSJLT5WQVAJPUI643JIXA
	on NURSE_LAB_MASTER (PATIENT_ID)
/

create index BIN$WJXFMJATJKEHI01U9WSW
	on NURSE_LAB_MASTER (REQ_DATE_TIME)
/
```

#病人检验结果信息

```sql
create table NURSE_LAB_RESULT
(
	TEST_NO VARCHAR2(20) not null,
	ITEM_NO NUMBER(5) not null,
	RESULT_DATE_TIME DATE,
	ITEM_NAME VARCHAR2(100),
	RESULT VARCHAR2(200),
	UNITS VARCHAR2(30),
	ABNORMAL VARCHAR2(20),
	REFERENCE VARCHAR2(500),
	CH_NAME VARCHAR2(20),
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	PATIENT_ID VARCHAR2(20),
	VISIT_ID NUMBER(20),
	UPDATE_DATE_TIME DATE,
	constraint BIN$BCNYVNL5SIK1OKKTTL0UW
		check ("TEST_NO" IS NOT NULL),
	constraint BIN$FBPXB01ETNCBNWJI47XOFQ
		check ("ITEM_NO" IS NOT NULL)
)
/

comment on table NURSE_LAB_RESULT is '2.21通过PatientId和TestNo获取病人检验结果信息（GetLabResult）'
/

comment on column NURSE_LAB_RESULT.TEST_NO is '化验申请号，主键'
/

comment on column NURSE_LAB_RESULT.ITEM_NO is '序号'
/

comment on column NURSE_LAB_RESULT.RESULT_DATE_TIME is '报告日期yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_LAB_RESULT.ITEM_NAME is '化验项目名称'
/

comment on column NURSE_LAB_RESULT.RESULT is '结果值'
/

comment on column NURSE_LAB_RESULT.UNITS is '单位'
/

comment on column NURSE_LAB_RESULT.ABNORMAL is '异常标志：高、低、正常'
/

comment on column NURSE_LAB_RESULT.REFERENCE is '参考值范围，一般不为空'
/

comment on column NURSE_LAB_RESULT.CH_NAME is '细菌名称'
/

comment on column NURSE_LAB_RESULT.EXPAND1 is '拓展字段1'
/

comment on column NURSE_LAB_RESULT.EXPAND2 is '拓展字段2'
/

comment on column NURSE_LAB_RESULT.EXPAND3 is '拓展字段3'
/

create unique index BIN$I0MDM251TC611VWNDXSPOW
	on NURSE_LAB_RESULT (TEST_NO, ITEM_NO)
/

create index BIN$3XT6HBTVS2Y9RE8XCKSROA
	on NURSE_LAB_RESULT (RESULT_DATE_TIME)
/

create index BIN$IUXIMGRUMKPBXCBGZ0G
	on NURSE_LAB_RESULT (PATIENT_ID, VISIT_ID)
/
```

#手术信息

```sql

create table NURSE_OPERATION
(
	PATIENT_ID VARCHAR2(20) not null,
	VISIT_ID NUMBER(20) not null,
	OPER_ID NUMBER(25) not null,
	REQ_DATE_TIME DATE,
	SCHEDULE_DATE_TIME DATE,
	OPER_STATUS VARCHAR2(10),
	OPER_DEPT_CODE VARCHAR2(20),
	OPER_ROOM VARCHAR2(20),
	DEPT_STAYED VARCHAR2(20),
	WARD_CODE VARCHAR2(20),
	BED_NO NUMBER(5),
	OPER_DOCTOR VARCHAR2(20),
	OPER_NAME VARCHAR2(200),
	OPER_CLASS VARCHAR2(20),
	IS_SEPERATE VARCHAR2(10),
	IS_EMERGENCY VARCHAR2(10),
	ASSISANT VARCHAR2(200),
	NURSE VARCHAR2(200),
	AROUND_NURSE VARCHAR2(200),
	ANESTHESIA_DOCTOR VARCHAR2(200),
	ANESTHESIA_METHOD VARCHAR2(200),
	ANESTHESIA_ASSISANT VARCHAR2(200),
	IN_DATE_TIME DATE,
	ANES_DATE_TIME DATE,
	START_DATE_TIME DATE,
	END_DATE_TIME DATE,
	ANESEND_TIME DATE,
	OUT_DATE_TIME DATE,
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	OUT_WARD_TIME DATE,
	BACK_WARD_TIME DATE,
	OUT_WARD_NURSE VARCHAR2(20),
	BACK_WARD_NURSE VARCHAR2(20),
	ID NUMBER,
	IS_DELETED NUMBER(1),
	CREATE_TIME DATE,
	UPDATE_DATE_TIME DATE,
	constraint BIN$FUULWXVTQE2HSJ0YBYBBGG
		check ("OPER_ID" IS NOT NULL),
	constraint BIN$KU4TQTWYTWIUND8KMWJJ7W
		check ("PATIENT_ID" IS NOT NULL),
	constraint BIN$XJ8LNELARBAZE9NG15U9A
		check ("VISIT_ID" IS NOT NULL)
)
/

comment on table NURSE_OPERATION is '2.11通过PatientID和VisitId获取手术信息(getPatientOperation)'
/

comment on column NURSE_OPERATION.PATIENT_ID is '病人ID'
/

comment on column NURSE_OPERATION.VISIT_ID is '住院次数'
/

comment on column NURSE_OPERATION.OPER_ID is '手术ID'
/

comment on column NURSE_OPERATION.REQ_DATE_TIME is '申请时间yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_OPERATION.SCHEDULE_DATE_TIME is '预计手术日期yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_OPERATION.OPER_STATUS is '手术状态：
  1=医生提交手术申请
  2=手术室已排期
  3=手术进行中
  4=手术已结束'
/

comment on column NURSE_OPERATION.OPER_DEPT_CODE is '手术室代码'
/

comment on column NURSE_OPERATION.OPER_ROOM is '手术间'
/

comment on column NURSE_OPERATION.DEPT_STAYED is '病人所在科室'
/

comment on column NURSE_OPERATION.WARD_CODE is '病人所在护理单元'
/

comment on column NURSE_OPERATION.BED_NO is '床序号（注意不是床标号）'
/

comment on column NURSE_OPERATION.OPER_DOCTOR is '手术主刀医生'
/

comment on column NURSE_OPERATION.OPER_NAME is '手术名称，如果一次有多个手术，手术名称用;隔开'
/

comment on column NURSE_OPERATION.OPER_CLASS is '手术等级'
/

comment on column NURSE_OPERATION.IS_SEPERATE is '是否隔离：是、否'
/

comment on column NURSE_OPERATION.IS_EMERGENCY is '是否急诊：是、否'
/

comment on column NURSE_OPERATION.ASSISANT is '手术助手，如果有多个，用空格隔开'
/

comment on column NURSE_OPERATION.NURSE is '洗手护士（器械护士），如果有多个，用空格隔开'
/

comment on column NURSE_OPERATION.AROUND_NURSE is '巡回护士，如果有多个，用空格隔开'
/

comment on column NURSE_OPERATION.ANESTHESIA_DOCTOR is '麻醉医生'
/

comment on column NURSE_OPERATION.ANESTHESIA_METHOD is '麻醉方法'
/

comment on column NURSE_OPERATION.ANESTHESIA_ASSISANT is '麻醉助手'
/

comment on column NURSE_OPERATION.IN_DATE_TIME is '进手术室时间yyyy-mm-dd hh24:mi:ss，根据实际情况填写'
/

comment on column NURSE_OPERATION.ANES_DATE_TIME is '麻醉开始时间yyyy-mm-dd hh24:mi:ss, 根据实际情况填写写'
/

comment on column NURSE_OPERATION.START_DATE_TIME is '手术开始时间yyyy-mm-dd hh24:mi:ss, 根据实际情况填写'
/

comment on column NURSE_OPERATION.END_DATE_TIME is '手术结束时间yyyy-mm-dd hh24:mi:ss, 根据实际情况填写'
/

comment on column NURSE_OPERATION.ANESEND_TIME is '复苏时间yyyy-mm-dd hh24:mi:ss, 根据实际情况填写'
/

comment on column NURSE_OPERATION.OUT_DATE_TIME is '出手术室时间yyyy-mm-dd hh24:mi:ss, 根据实际情况填写'
/

comment on column NURSE_OPERATION.EXPAND1 is '拓展字段1'
/

comment on column NURSE_OPERATION.EXPAND2 is '拓展字段2'
/

comment on column NURSE_OPERATION.EXPAND3 is '拓展字段3'
/

comment on column NURSE_OPERATION.OUT_WARD_TIME is '出病房时间'
/

comment on column NURSE_OPERATION.BACK_WARD_TIME is '返回病房时间'
/

comment on column NURSE_OPERATION.OUT_WARD_NURSE is '出病房操作护士 '
/

comment on column NURSE_OPERATION.BACK_WARD_NURSE is '返回病房操作护士 '
/

create unique index BIN$TF8AL5OS5Y29LCRRM39G
	on NURSE_OPERATION (PATIENT_ID, VISIT_ID, OPER_ID)
/

create index BIN$EGZWST73Q0SUNFIGZFVTA
	on NURSE_OPERATION (SCHEDULE_DATE_TIME)
/
```

#医嘱信息

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

#出院病人列表

```sql

create table NURSE_PAT_VISIT
(
	PATIENT_ID VARCHAR2(20) not null,
	VISIT_ID NUMBER(20) not null,
	ADMISSION_DATE_TIME DATE,
	ADMISSION_DEPT_CODE VARCHAR2(20),
	ADMISSION_WARD_CODE VARCHAR2(20),
	ADMISSION_BED_NO NUMBER(5),
	ADMISSION_BED_LABEL VARCHAR2(20),
	DISCHARGE_DEPT_CODE VARCHAR2(20),
	DISCHARGE_WARD_CODE VARCHAR2(32),
	DISCHARGE_DATE_TIME DATE,
	DISCHARGE_BED_NO NUMBER(5),
	DISCHARGE_BED_LABEL VARCHAR2(20),
	OUTP_DOCTOR VARCHAR2(20),
	DOCTOR_IN_CHARGE VARCHAR2(20),
	MAIN_DOCTOR VARCHAR2(20),
	CHIEF_DOCTOR VARCHAR2(20),
	DUTY_NURSE VARCHAR2(20),
	IS_EMERGENCY VARCHAR2(10),
	PATIENT_CONDITION VARCHAR2(10),
	CHARGE_TYPE VARCHAR2(20),
	IS_DEAD VARCHAR2(10),
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	DIAGNOSIS VARCHAR2(100),
	BODY_WEIGHT NUMBER(4,1),
	BODY_HEIGHT NUMBER(7,2),
	NEWBORN_OUT VARCHAR2(10),
	CR_WARD_CODE VARCHAR2(20),
	ADMISSION_WARD_DATE_TIME DATE,
	OUT_PAT_ID VARCHAR2(20),
	ZS VARCHAR2(20),
	EXPAND4 VARCHAR2(20 char),
	EXPAND5 VARCHAR2(20 char),
	ID NUMBER,
	IS_DELETED NUMBER(1),
	CREATE_TIME DATE,
	NURSING_CLASS VARCHAR2(10 char),
	UPDATE_DATE_TIME DATE,
	constraint BIN$ASUVJJNTWGMLW2M6RRGIA
		check ("VISIT_ID" IS NOT NULL),
	constraint BIN$OUQM2JEFQMOITK57MBJTIQ
		check ("PATIENT_ID" IS NOT NULL)
)
/

comment on table NURSE_PAT_VISIT is '2.2通过始日期和结束日期获取出院病人列表(getDischargePatients)(这表没有主键)'
/

comment on column NURSE_PAT_VISIT.PATIENT_ID is '病人ID'
/

comment on column NURSE_PAT_VISIT.VISIT_ID is '住院次数'
/

comment on column NURSE_PAT_VISIT.ADMISSION_DATE_TIME is '入院日期'
/

comment on column NURSE_PAT_VISIT.ADMISSION_DEPT_CODE is '入院科室代码，与dept_dict关联'
/

comment on column NURSE_PAT_VISIT.ADMISSION_WARD_CODE is '入院时的护理单元名称，与dept_dict关联'
/

comment on column NURSE_PAT_VISIT.ADMISSION_BED_NO is '入院时床序号（注意不是床号）'
/

comment on column NURSE_PAT_VISIT.ADMISSION_BED_LABEL is '入院时床号'
/

comment on column NURSE_PAT_VISIT.DISCHARGE_DEPT_CODE is '出院科室名称，如果是出院患者，则不能为空，与dept_dict关联'
/

comment on column NURSE_PAT_VISIT.DISCHARGE_WARD_CODE is '出院时的护理单元名称，如果是出院患者，则不能为空，与dept_dict关联'
/

comment on column NURSE_PAT_VISIT.DISCHARGE_DATE_TIME is '出院时间yyyy-mm-dd hh24:mi:ss，如果是出院患者，则不能为空'
/

comment on column NURSE_PAT_VISIT.DISCHARGE_BED_NO is '出院时床序号（注意不是床号）'
/

comment on column NURSE_PAT_VISIT.DISCHARGE_BED_LABEL is '出院床号'
/

comment on column NURSE_PAT_VISIT.OUTP_DOCTOR is '门诊医生'
/

comment on column NURSE_PAT_VISIT.DOCTOR_IN_CHARGE is '住院医生'
/

comment on column NURSE_PAT_VISIT.MAIN_DOCTOR is '主治医生'
/

comment on column NURSE_PAT_VISIT.CHIEF_DOCTOR is '(副)主任医生'
/

comment on column NURSE_PAT_VISIT.DUTY_NURSE is '责任护士'
/

comment on column NURSE_PAT_VISIT.IS_EMERGENCY is '是否急诊入院，1为是，0为否'
/

comment on column NURSE_PAT_VISIT.PATIENT_CONDITION is '入院病情：病危、病重、普通'
/

comment on column NURSE_PAT_VISIT.CHARGE_TYPE is '费别：自费、医保、农合'
/

comment on column NURSE_PAT_VISIT.IS_DEAD is '是否死亡'
/

comment on column NURSE_PAT_VISIT.EXPAND1 is '拓展字段1'
/

comment on column NURSE_PAT_VISIT.EXPAND2 is '拓展字段2'
/

comment on column NURSE_PAT_VISIT.EXPAND3 is '拓展字段3'
/

comment on column NURSE_PAT_VISIT.ADMISSION_WARD_DATE_TIME is '入科日期'
/

create unique index BIN$UAKFORQST2PCQBBW7OMWA
	on NURSE_PAT_VISIT (PATIENT_ID, VISIT_ID)
/

create index BIN$SBSGNNKNQVQ7MYLXFWZILG
	on NURSE_PAT_VISIT (ADMISSION_DATE_TIME, PATIENT_ID, VISIT_ID, DISCHARGE_DATE_TIME)
/
```

#预交金信息

```sql
create table NURSE_PREPAY
(
	RCPT_NO VARCHAR2(20),
	PATIENT_ID VARCHAR2(20),
	VISIT_ID NUMBER(20),
	TRANSACT_DATE DATE,
	PAY_WAY VARCHAR2(20),
	OPERATOR VARCHAR2(20),
	MONEY NUMBER(10,2),
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	UPDATE_DATE_TIME DATE,
	constraint BIN$A9AGJ852SFKYOLPRCQEYGQ
		check ("TRANSACT_DATE" IS NOT NULL),
	constraint BIN$AFTZNLG9SPC2PXQWKYLNXW
		check ("PAY_WAY" IS NOT NULL),
	constraint BIN$NY8FI0GOS5WCQ3VIREFEIG
		check ("RCPT_NO" IS NOT NULL),
	constraint BIN$OO8R4PA8S3OPAIWARXPVAQ
		check ("PATIENT_ID" IS NOT NULL),
	constraint BIN$W1FF68XATYWPLX5DHOMUW
		check ("VISIT_ID" IS NOT NULL)
)
/

comment on table NURSE_PREPAY is '2.12通过PatientID和VisitId获取预交金信息(getPatientPrepay)'
/

comment on column NURSE_PREPAY.RCPT_NO is '收据号'
/

comment on column NURSE_PREPAY.PATIENT_ID is '病人ID'
/

comment on column NURSE_PREPAY.VISIT_ID is '住院次数'
/

comment on column NURSE_PREPAY.TRANSACT_DATE is '交款时间yyyy-mm-dd hh24:mi:ss'
/

comment on column NURSE_PREPAY.PAY_WAY is '交款类型：现金、刷卡、支票'
/

comment on column NURSE_PREPAY.OPERATOR is '收费人'
/

comment on column NURSE_PREPAY.MONEY is '金额'
/

comment on column NURSE_PREPAY.EXPAND1 is '拓展字段1'
/

comment on column NURSE_PREPAY.EXPAND2 is '拓展字段2'
/

comment on column NURSE_PREPAY.EXPAND3 is '拓展字段3'
/

create index BIN$IRUXGMMTYECWGMZGD52G
	on NURSE_PREPAY (TRANSACT_DATE)
/

create unique index BIN$1TUCLCMORFCHWQFOICI2ZG
	on NURSE_PREPAY (RCPT_NO)
/

create index BIN$FWJRJCDURVIJ702DDM4OKW
	on NURSE_PREPAY (PATIENT_ID, VISIT_ID)
/
```

#在院病人列表
```sql
create table NURSE_PATS_IN_HOSPITAL
(
	PATIENT_ID VARCHAR2(20) not null,
	VISIT_ID NUMBER(20) not null,
	WARD_CODE VARCHAR2(20) not null,
	BED_NO NUMBER(5),
	DEPT_CODE VARCHAR2(20),
	NURSING_CLASS VARCHAR2(20),
	ADMISSION_DATE_TIME DATE,
	ADMISSION_WARD_DATE_TIME DATE,
	DOCTOR VARCHAR2(20),
	PATIENT_CONDITION VARCHAR2(20),
	LEND_DEPT_CODE VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	PREPAYMENTS NUMBER(20,4),
	TOTAL_COSTS NUMBER(20,4),
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	BED_LABEL VARCHAR2(30),
	CR_WARD_CODE VARCHAR2(30),
	DIAGNOSIS VARCHAR2(100),
	NEWBORN_OUT VARCHAR2(2),
	OLD_ADMISSION_DATE_TIME DATE,
	EXPAND4 VARCHAR2(20 char),
	EXPAND5 VARCHAR2(20 char),
	ID NUMBER,
	IS_DELETED NUMBER(1),
	CREATE_TIME DATE,
	UPDATE_DATE_TIME DATE,
	constraint BIN$FRF7O5EDRVMCSMYT3C39SG
		check ("VISIT_ID" IS NOT NULL),
	constraint BIN$R5KYTDQTGQGZMSEIIZ1A
		check ("PATIENT_ID" IS NOT NULL)
)
/

comment on table NURSE_PATS_IN_HOSPITAL is '2.1通过护理单元代码获取当前在院病人列表(getCurrentPatients)'
/

comment on column NURSE_PATS_IN_HOSPITAL.PATIENT_ID is '病人ID'
/

comment on column NURSE_PATS_IN_HOSPITAL.VISIT_ID is '住院次数'
/

comment on column NURSE_PATS_IN_HOSPITAL.WARD_CODE is '护理单元代码'
/

comment on column NURSE_PATS_IN_HOSPITAL.BED_NO is '床序号'
/

comment on column NURSE_PATS_IN_HOSPITAL.DEPT_CODE is '所属科室代码'
/

comment on column NURSE_PATS_IN_HOSPITAL.NURSING_CLASS is '护理等级:一级护理,二级护理,三级护理,特级护理'
/

comment on column NURSE_PATS_IN_HOSPITAL.ADMISSION_DATE_TIME is '入院日期'
/

comment on column NURSE_PATS_IN_HOSPITAL.ADMISSION_WARD_DATE_TIME is '入科日期'
/

comment on column NURSE_PATS_IN_HOSPITAL.DOCTOR is '医生'
/

comment on column NURSE_PATS_IN_HOSPITAL.PATIENT_CONDITION is '病情:危,重,一般'
/

comment on column NURSE_PATS_IN_HOSPITAL.LEND_DEPT_CODE is '入科日期'
/

comment on column NURSE_PATS_IN_HOSPITAL.PREPAYMENTS is '预付款'
/

comment on column NURSE_PATS_IN_HOSPITAL.TOTAL_COSTS is '总费用'
/

comment on column NURSE_PATS_IN_HOSPITAL.EXPAND1 is '拓展字段1'
/

comment on column NURSE_PATS_IN_HOSPITAL.EXPAND2 is '拓展字段2'
/

comment on column NURSE_PATS_IN_HOSPITAL.EXPAND3 is '拓展字段3'
/

comment on column NURSE_PATS_IN_HOSPITAL.BED_LABEL is '床号'
/

comment on column NURSE_PATS_IN_HOSPITAL.OLD_ADMISSION_DATE_TIME is '存旧入院日期'
/

comment on column NURSE_PATS_IN_HOSPITAL.EXPAND4 is '拓展字段4'
/

comment on column NURSE_PATS_IN_HOSPITAL.EXPAND5 is '拓展字段5'
/

create unique index BIN$RDW8IJIPS8ERHQXUXYRW
	on NURSE_PATS_IN_HOSPITAL (PATIENT_ID, VISIT_ID, WARD_CODE)
/

create index BIN$5P63W9WRR9E4UPWYWV7FG
	on NURSE_PATS_IN_HOSPITAL (DEPT_CODE)
/

create index BIN$AVQEXXESTYOIPDYVX1OHJA
	on NURSE_PATS_IN_HOSPITAL (ADMISSION_DATE_TIME)
/

create index BIN$N26AEFEIRTQMXGWODQRAMQ
	on NURSE_PATS_IN_HOSPITAL (WARD_CODE)
/

create index BIN$XRYGYC4QRJKMFCFEDRQQJQ
	on NURSE_PATS_IN_HOSPITAL (LEND_DEPT_CODE)
/

create index BIN$Z2ZNDXMUROIVCEIL30K8VW
	on NURSE_PATS_IN_HOSPITAL (CR_WARD_CODE)
/
```

#病人基本信息表

```sql
create table NURSE_PAT_MASTER_INDEX
(
	PATIENT_ID VARCHAR2(20) not null,
	INP_NO VARCHAR2(20),
	NAME VARCHAR2(100),
	SEX VARCHAR2(10),
	DATE_OF_BIRTH DATE,
	MARRIAGE VARCHAR2(20),
	OCCUPATION VARCHAR2(200),
	NATION VARCHAR2(20),
	IDNO VARCHAR2(40),
	COUNTRY VARCHAR2(20),
	ADDRESS VARCHAR2(100),
	PHONE VARCHAR2(50),
	COMPANY_ADDRESS VARCHAR2(100),
	COMPANY_PHONE VARCHAR2(50),
	CHARGE_TYPE VARCHAR2(20),
	CONTACT_NAME VARCHAR2(50),
	RELATIONSHIP VARCHAR2(20),
	CONTACT_ADDR VARCHAR2(100),
	CONTACT_PHONE VARCHAR2(50),
	EXPAND1 VARCHAR2(20),
	EXPAND2 VARCHAR2(20),
	EXPAND3 VARCHAR2(20),
	CREATE_DATE_TIME DATE,
	NATIVE_PLACE VARCHAR2(200),
	BIRTH_PLACE VARCHAR2(255 char),
	ID NUMBER,
	IS_DELETED NUMBER(1),
	CREATE_TIME DATE,
	UPDATE_DATE_TIME DATE,
	constraint BIN$KMZGB3AASNU9IRWKFHBBA
		check ("PATIENT_ID" IS NOT NULL)
)
/

comment on table NURSE_PAT_MASTER_INDEX is '2.3通过PatientID获取病人基本信息(getPatientInfo)'
/

comment on column NURSE_PAT_MASTER_INDEX.PATIENT_ID is '病人ID'
/

comment on column NURSE_PAT_MASTER_INDEX.INP_NO is '住院次数'
/

comment on column NURSE_PAT_MASTER_INDEX.NAME is '患者姓名'
/

comment on column NURSE_PAT_MASTER_INDEX.SEX is '性别'
/

comment on column NURSE_PAT_MASTER_INDEX.DATE_OF_BIRTH is '出生年月yyyy-mm-dd'
/

comment on column NURSE_PAT_MASTER_INDEX.MARRIAGE is '婚姻状态：已婚、未婚、离异'
/

comment on column NURSE_PAT_MASTER_INDEX.OCCUPATION is '职业：教师、医生、律师'
/

comment on column NURSE_PAT_MASTER_INDEX.NATION is '民族'
/

comment on column NURSE_PAT_MASTER_INDEX.IDNO is '身份证号'
/

comment on column NURSE_PAT_MASTER_INDEX.COUNTRY is '国籍：中国、美国'
/

comment on column NURSE_PAT_MASTER_INDEX.ADDRESS is '住址'
/

comment on column NURSE_PAT_MASTER_INDEX.PHONE is '电话'
/

comment on column NURSE_PAT_MASTER_INDEX.COMPANY_ADDRESS is '工作单位地址'
/

comment on column NURSE_PAT_MASTER_INDEX.COMPANY_PHONE is '工作单位电话'
/

comment on column NURSE_PAT_MASTER_INDEX.CHARGE_TYPE is '费别：自费、医保、农合'
/

comment on column NURSE_PAT_MASTER_INDEX.CONTACT_NAME is '联系人'
/

comment on column NURSE_PAT_MASTER_INDEX.RELATIONSHIP is '联系人关系'
/

comment on column NURSE_PAT_MASTER_INDEX.CONTACT_ADDR is '联系人住址'
/

comment on column NURSE_PAT_MASTER_INDEX.CONTACT_PHONE is '联系人电话'
/

comment on column NURSE_PAT_MASTER_INDEX.EXPAND1 is '拓展字段1'
/

comment on column NURSE_PAT_MASTER_INDEX.EXPAND2 is '拓展字段2'
/

comment on column NURSE_PAT_MASTER_INDEX.EXPAND3 is '拓展字段3'
/

comment on column NURSE_PAT_MASTER_INDEX.NATIVE_PLACE is '籍贯'
/

comment on column NURSE_PAT_MASTER_INDEX.BIRTH_PLACE is '出生地'
/

create unique index BIN$BTHRE5ORC1DDXNAZFIW
	on NURSE_PAT_MASTER_INDEX (PATIENT_ID)
/
```

#床位信息表

```sql
create table NURSE_BED_REC
(
	BED_NO NUMBER(6),
	WARD_CODE VARCHAR2(20),
	BED_LABEL VARCHAR2(20),
	ROOM_NO VARCHAR2(20),
	SEX VARCHAR2(20),
	BED_TYPE VARCHAR2(20),
	CR_WARD_CODE VARCHAR2(20),
	STATUS VARCHAR2(20 char),
	PATIENT_ID VARCHAR2(20),
	FLAG_SHARE VARCHAR2(20 char),
	EXPAND1 VARCHAR2(20 char),
	constraint BIN$OYLPBGVRISDHJ2NWZHDYQ
		check ("BED_NO" IS NOT NULL),
	constraint BIN$THYMNN8NSZA2MJRL18APA
		check ("WARD_CODE" IS NOT NULL)
)
/

comment on table NURSE_BED_REC is '科室字典表'
/

comment on column NURSE_BED_REC.BED_NO is '序号'
/

comment on column NURSE_BED_REC.WARD_CODE is '护理单元代码'
/

comment on column NURSE_BED_REC.BED_LABEL is '床标号'
/

comment on column NURSE_BED_REC.ROOM_NO is '房间'
/

comment on column NURSE_BED_REC.SEX is '性别'
/

comment on column NURSE_BED_REC.BED_TYPE is '类别'
/

comment on column NURSE_BED_REC.CR_WARD_CODE is '所属小科室'
/

comment on column NURSE_BED_REC.STATUS is '状态'
/

create index BIN$HUGVVXNLTIUGJWIUGUAEQG
	on NURSE_BED_REC (CR_WARD_CODE)
/

create unique index BIN$JVKEJYH3SDKWNCHMOBXUZA
	on NURSE_BED_REC (WARD_CODE, BED_NO)
/

create index IDX_NURSE_BED_REC_WARD
	on NURSE_BED_REC (WARD_CODE, BED_LABEL)
/
```
