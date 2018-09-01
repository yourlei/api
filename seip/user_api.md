# 省战略性新兴产业监测分析预警平台

#　2用户端

## 2.9生成报告

报告页面由服务端渲染, 前端负责提交表单数据, 后台根据报告模板查询季报/半年报/年报数据生成实时图表数据,　图表使用echart生成可进行交互, 用户可选择查看报告指定章节的内容;　当用户发送导出报告请求时, 由node生成实时可视化图片, 并将图片加入到docx文档(两个主要依赖是 **officegen**, **node-echart**)

### 2.9.1led行业报告

- method: get
- url: domain/api/v1/led?query={body}&token=value

- body 参数

``` json
{
  "title": "报告标题",
　"catalog": "1000001",  
  "classify": "0",
  "year": "2010",
  "period": "0"
}
```

- 参数说明

| name         |  desc        |
|:------------:|:------------:|
| catalog      | 选择查看的报告章节内容, 通过0,1组成的字符串表示, 每个字符代表一个章/节, 1为输出该章节内容至报告, 0则不输出 |
| classify     | 报告类型（年报 , 半年报,季度报）   |
| year         | 查询年份 |
| period       | 每个报告类型的时间段|


- 报告中可选的章节

- led 第一章(只能选到章), 第二章下五个小节, 第三章(只能选到章), 共有７个可选的章节, 如1000001表示查看第一章和第三章内容, 1100000 表示查看第一章和第二章第一小节内容

- ngit 第一章(只能选到章), 第二章下17个小节, 第三章(只能选到章), 共有19个可选的章节


### 2.9.2新一代信息技术产业报告

- method: get
- url: domain/api/v1/ngit?query={body}&token=value



- 参数说明同上


## 2.10导出报告

- method: get
- url: domain/report/docx/:name?query={body}

name: *led|ngit*

- body 参数

``` json
{
  "title": "报告标题",
　"catalog": "1000001", # 长度为７, 
  "classify": "0", # 0年报，1半年报, 2季度报
  "year": "2010",　　# 报告年份
  "period": "0"　 # 0:年报/上半年/第一季度,1:下半年/第二季度, 2:第三季度报, 3:第四季度（0, 1时根据classify确定其值）
}
```


- 返回结果

``` json
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "filename": "led.docx"
}
```

- 说明:

| code         |  msg         |   说明   |
|:------------:|:------------:|:--------:|
| 1            | 参数错误       |          |



**20180202更新**
---

## 2.11专利申请趋势数据

- method: post
- url: domain/api/v1/warning/trend

- 参数

``` json
{
	"industry": "led",
	"province": "广东" #　可选
}
```

- 返回结果

``` json
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "data": {
    "d1": [], # 发明
    "d2": [], # 外观
    "d3": []  # 实用
  }
}
```

*三组数据对应的时间顺序为1985-2017*


## 2.12生命周期分析

- method: post
- url: domain/api/v1/warning/index

- 参数

``` json
{
	"industry": "led",
	"province": "广东" #　可选
}
```

- 返回结果

``` json
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "data": {
    "growthRate": [], # 成长率
    "growthx": [],    # 成熟系数
    "agingx": []      # 衰老系数
  }
}
```

*三组数据对应的时间顺序为2000-2017*