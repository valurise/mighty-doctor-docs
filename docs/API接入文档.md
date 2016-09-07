绝世好医API文档
===
绝世好医服务接口是提供给第三方合作伙伴完成与绝世好医服务进行对接所需的URL接口。具体接口定义如下：

---

### 系统URI入参定义
- _cid：第三方合作伙伴代码
- _ts: 客户端时间戳
- _sign：URL请求参数校验的MD5编码，由URI参数列表(按字母顺序)加第三方合作访问密钥联合组成

----

### 获取病症列表
- 请求地址: http://mydoctor.valurise.com/apis/getAllSymptom.json
- 请求方式: GET
- 入参: 无
- 返回:
```
[
	{
       		"body": "全身",// 身体部位名（中文）
        	"gender": "all",// 性别(all, female, male)
      		"symptom": "麻刺感", // 症状名（中文）
       		"symptomId": 419 // 症状ID
	},
	…
]
```

### 获取服务号
- 请求地址: http://mydoctor.valurise.com/apis/startSession.json
- 请求方式: POST
- Content Type: application/x-www-form-urlencoded
- Body参数: symptomId=419&uid=139xxxxxxxx&sex=1&height=175&weight=70&birth=1980-05-10

参数名|含义|是否必填|备注
---|---|---|---
symptomId|症状ID|是|从获取病症列表接口中获取
uid|用户ID|是|用户唯一标识ID
sex|性别|是|1：男性，2：女性
height|身高|是|单位:厘米
weight|体重|是|单位:公斤
birth|生日|是|格式:yyyy-MM-dd

- 返回

```
{
	"sid": "xxxxxx", // 服务号,对应一次问诊服务, 使用一次后作废, 有效时间30分钟
	"ts":xxxxx // 时间戳(毫秒), 服务器当前时间
}
```






