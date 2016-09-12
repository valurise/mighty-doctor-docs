# 绝世好医API接入文档

绝世好医服务接口是提供给第三方合作伙伴完成与绝世好医服务进行对接所需的URL接口。具体接口定义如下：

------

### 系统URI入参定义

- _cid：第三方合作伙伴代码
- _ts: 客户端时间戳
- _sign：URL请求参数校验的MD5编码，由URI参数(按字母顺序)加第三方合作访问密钥联合组成，例如md5hex(_cid=xxx&_ts=xxx&sid=xxx密钥)

---

### 获取病症列表

- 请求地址: http://mydoctor.valurise.com/apis/getAllSymptom.json
- 请求方式: GET
- 入参: 无
- 返回:

```
[
  {
    "body":"全身", // 身体部位名（中文）
    "gender": "all",// 性别(all, female, male)
    "symptom": "麻刺感", // 症状名（中文）
    "symptomId": 419 // 症状ID
  },
  ...
]
```



### 获取服务号

- 请求地址: http://mydoctor.valurise.com/apis/startSession.json
- 请求方式: POST
- Content Type: application/x-www-form-urlencoded
- Body入参：

```
symptomId=419&uid=139xxxxxxxx&sex=1&height=175&weight=70&birth=1980-05-10
```

| 参数名       | 含义   | 是否必填 | 备注            |
| --------- | ---- | ---- | ------------- |
| symptomId | 症状ID | 是    | 从获取病症列表接口中获取  |
| uid       | 用户ID | 是    | 用户唯一标识ID      |
| sex       | 性别   | 是    | 1：男性，2：女性     |
| height    | 身高   | 是    | 单位:厘米         |
| weight    | 体重   | 是    | 单位:公斤         |
| birth     | 生日   | 是    | 格式:yyyy-MM-dd |

- 返回：

```
{
  "sid": "xxxxxx", // 服务号,对应一次问诊服务, 使用一次后作废, 有效时间30分钟
  "ts":xxxxx // 时间戳(毫秒), 服务器当前时间
}
```



### 获取问题/结论节点

- 请求地址: http://mydoctor.valurise.com/apis/fetchQuestion.json
- 请求方式: POST
- Content Type: application/json
- URI入参:

```
sid=xxxxxxx //问诊服务号
```

- Body入参

> 用户本次输入的问题答案集合, 其中问题共有3种:单选题、多选题、输入题。
>
> 获取首个问题时无需提交此参数；
>
> 单选题必须要求用户做出一个选择；
>
> 多选题可以允许用户不做任何选择；
>
> 输入题可以允许用户不输入值，由服务端判断后续的处理逻辑。

1\. 单选题

```
{
  "questionId":3810,// 所回答的问题编号
  "answer":"1" // 所选择的答案编号
}
```

2\. 多选题

```
{
  "questionId":3812, // 所回答的问题编号
  "answers":"1,2,5,7"  // 所选择的多个答案编号。如未选择任何答案，则置空字符串即可。
}
```

3\. 输入题

```
{
  "questionId":3815,
  "values":[{"key":"RSBP","value":"170"},{"key":"RDBP","value":"94"}] //用户所输入的信息对应输入项的key.
}
```

*其中key值包括: RSBP(最近收缩压)，USBP（通常收缩压），RDBP（最近舒张压），UDBP（通常舒张压），DOS（症状持续时间），GA（孕周）*

- 返回

> 返回值按nodeType分为Question, Conclusion两种情况: 
>
> 1. nodeType = Question, 需要用户答题, 然后继续下一步问诊, 以下列出 3种题型的样例，以questionType（SingleSelection, MultiSelection, ValueEntry）进行区分。
> 2. nodeType=Conclusion, 问诊结束, 给出结论

情况一：nodeType=Question

1\. 单选题

```
{
  "nodeType":"Question",
  "questionId": 3885,
  "questionText": “下颌或口腔有过受伤吗？例如跌落、猛击或其他击打导致疼痛。",
  "questionType": "SingleSelection"
  "answers": [
		{
           "answerId": 1,
           "answerText": "是"
         },
         {
           "answerId": 2,
           "answerText": "否"
         }
	]
}
```

2\. 多选题

```
{
  "nodeType": "Question"
  "questionId": 292,
  "questionText": “存在任何下列可能的紧急症状吗？请选择所有符合的情况。",
  "questionType": "MultiSelection"
  "answers": [
		{
           "answerId": 1,
           "answerText": "声音低沉或无法完全张开嘴"
         },
         {
           "answerId": 2,
           "answerText": "呼吸困难"
         },
		{
           "answerId": 3,
           "answerText": "嘴唇、舌头或口腔突然肿胀"
         }
	]
}
```

3\. 问答题

```
{
  "nodeType": "Question"
  "questionId": 292,
  "questionText": "近期大多数时候的收缩压值为多少（血压读数中的上值）？",
  "questionType": "ValueEntry",
  "entries":[ // 输入项列表
		{
		  "key":"hbp",   // 输入项KEY
		  "label":"收缩血压",   // 输入项标题
		  "valueType":"number", // 输入项类型
		  "value":"120" // 输入项默认值
		},   
		...
      ] 
}
```



情况二：nodeType = Conclusion

```
{
  "nodeType":"Conclusion",
  "verifyCode":"E20D23",// 用于医疗分诊后台的访问授权码
  "conclusions":[
    {
      "actionType":"hospital",// 建议的行动类型，home: 在家静养、nurse: 咨询专业护士、hospital: 去医院就诊、emergency：急救
      "actionTime":"24h", // 建议的采取行动的时间, 例如24小时内   
	  "actionLabel":"请在24小时内前往社区医院就医", // 建议采取行动
	  "emergencyLevel": "8", // 紧急程度（最高紧急程度为1）
	  "departInfo": "内科", // 建议挂号科室
	  "title": "可能是百日咳", // 结论标题
	  "content": "百日咳的典型症状是一阵长时间的咳嗽，接着大口喘气，听起来就像是在呐喊。一段长时间的咳嗽后也可能会出现呕吐。百日咳是一种在起初时很像普通的感冒的病毒性感染，但一至两周后咳嗽变得湿润，并有浓厚的粘液。长时间的咳嗽让人十分痛苦。如果不进行治疗，感染会持续至少6周，且可能需要3个月才能完全消除。 如果能及早使用抗生素治疗可能会有所帮助，并且可能缩短病程。处方药也可用于治疗咳嗽。",// 结论描述
	  "suggets":["让别人开车送你去医院。","如果没人开车送你去医院，建议拨打120或当地急救电话寻求帮助","在向医生进行咨询之前，不要吃或喝任何东西。","如果可能，请带上所有服用的药物清单（包括非处方药、维生素或补充剂）；如果没有药物清单，也可以带上药瓶。"]// 补充建议
    },
    ...
  ]
}
```



### 返回上一个问题/结论节点

- 请求地址：http://mydoctor.valurise.com/apis/rollbackQuestion.json
- 请求方式：POST
- URI入参：

```
sid=xxxxxxx //问诊服务号
```

- 返回：数据结构同获取问题接口
