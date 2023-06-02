#简介
    数据交换平台是百辰源自主研发的异构数据集成平台，主要用于对接医院异构的数据接口，并由平台对外开放服务接口，
    各应用系统调用平台基本服务层的服务接口实现对各服务的调用，实现各业务系统的互联互通和数据交换。
    
#关于数据交换平台的groovy脚本编写要求

# 通用例子：CommonDemo类

CommonDemo类是一个Groovy插件，主要包括以下功能。

## 类名：自定义

**固定import包名:**

`com.cr.hesb.core.bean.InvokeBean`
`com.cr.hesb.integration.plugin.GroovyPlugin`

**实现接口：** `com.cr.hesb.integration.plugin.GroovyPlugin`


## 方法：action

**函数签名：** `action(InvokeBean bean, Map<String, Object> inboundData) -> Map<String, Object>`

该方法主要负责处理入参和出参的设置，它接收两个参数：`bean`（InvokeBean类型）和`inboundData`（键值对类型的Map）。函数返回一个键值对类型的Map，包含了处理后的出参。

### 入参设置

入参可以在函数形参 `inboundData` 中根据入参的键获得。例如有以下结构的Json：

```json
{
    "patient_id": "00001E11",
    "visit_id": 1,
    "exam_list": [],
    "address": {
    
    }  
}
```

那么获取参数的脚本如下：
```groovy

def patientId = inboundData?.get("patient_id")
List<HashMap> dataList = inboundData?.get("exam_list")
def address = inboundData?.get("address")

```
### 中间处理过程

在这个阶段，需要处理例如数据库调用，http请求，webservice请求，数据结构处理等

#### 数据库调用

**数据源获取方式：**
```groovy    
// 2.1从spring获取数据源
DataSourceAdapter sourceAdapter = ApplicationContextHolder.getBean(DataSourceAdapter.class);
HikariDataSource ds = sourceAdapter.get(DATA_SOURCE_ID);
```
`DATA_SOURCE_ID` 是一个固定的值，需要按照实际数据源配置的ID配置

**数据库实际调用：**
   1.原生jdbc调用
   2.基于spring的jdbcTemplate调用
   3.基于hutool工具类的调用
   
#### http请求
使用基于hutool工具类的调用即可

#### webservice请求
使用基于hutool工具类调用即可

#### 数据结构处理
灵活运用groovy的各种办法处理即可

### 出参设置

出参要求为一个 Map 类型，例如要求返回如下结构：
```json
{
    "errorCode": "1",
    "errorMsg": "查询成功",
    "hospitalData": []
}
```
groovy代码如下：
```
def result = [:]
result.put("errorCode",1);
result.put("errorMsg","查询成功");
result.put("hospitalData",[]);
return result;
```
