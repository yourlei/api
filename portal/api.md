# portal_v1

## 文档大纲

根据系统的模块划分，该文档主要描述的内容有以下部分：

**0.[相关规范](#0相关规范)**

- [响应数据](#01响应数据)
- [http状态码说明](#02http状态码)
- [接口权限说明](#03http请求头)

**1.[用户管理](#1用户管理)**

- [用户登录](#10登录)
- [用户注册](#11注册)
- [注销用户](#12注销用户)
- [禁启用用户](#13禁用|启用用户)
- [审核用户](#14审核用户)
- [编辑用户](#15编辑用户)
- [修改用户密码](#16修改用户密码)
- [登出系统](#17登出)
- [用户列表](#18用户列表)

**2.[验证码](#2验证码)**

- [图片验证码](#20图片验证码)

**3.[角色管理](#3角色管理)**

- [角色列表](#31角色列表)
- [创建角色](#32创建角色)
- [删除角色](#33删除角色)
- [编辑角色](#34编辑角色)
- [查看角色组下用户](#34获取角色组下用户)
- [用户迁移](#35转移角色组下用户)

**4.[权限管理](#4权限管理)**

- [资源列表](#41资源列表)
- [获取角色组下资源](#42获取角色组下资源)
- [修改权限](#43修改权限)
- [获取角色组下资源(用户端)](#44获取角色组下资源用户端)

**5.[资源管理](#5资源管理)**

**菜单资源**

- [创建菜单](#511创建菜单)
- [菜单列表](#512菜单列表)
- [编辑菜单](#513编辑菜单)
- [删除菜单](#514删除菜单)
- [查看一级菜单](#515获取一级菜单)

**接口资源**

- [创建接口](#521创建接口)
- [接口列表](#522接口列表)
- [编辑接口](#523编辑接口)
- [删除接口](#524删除接口)

**6.[日志管理](#6日志管理)**

- [日志列表](#61日志列表)
- [日志导出](#62日志导出)

~~**7.[数据接口](#7数据接口)**~~

- [ES数据查询](#71es数据查询)
- [NLP接口](#72情感分析)
- [http请求转发接口](#73http请求转发接口)

---

# 0相关规范

## 0.1响应数据

- 返回的数据格式，每个接口固定返回*code*, *error*信息，当http请求资源成功时 *code*的返回的值为 *0*， 对应的非0则表示请求失败，具体的错误信息可查看*error*中的*msg*。 固定返回的字段如下：


``` js
{
  "code": 0,
  "error": {
    "msg": "这里是错误信息，成功时为空"
  }
}
```

- 错误代码说明

| code | 说明 |
|:----:|:----:|
|0 |执行成功|
|1 |请求参数错误|
|2 |刷新token|
|404 |资源不存在|
|500|服务出错|
|1000xx | 用户管理相关的错误码  |
|2000xx | 角色管理相关的错误码  |
|3000xx | 资源管理相关的错误码  |
|4000xx | 权限相关的错误码 |
|5000xx | 数据集管理相关的错误码 |


## 0.2http状态码

- 200: GET请求成功, 及DELETE或PATCH同步请求完成，或者PUT同步更新一个已存在的资源
- 201: POST 同步请求完成，或者PUT同步创建一个新的资源

- 400 参数错误
- 401 Unauthorized: 用户未认证，请求失败
- 403 Forbidden: 用户无权限访问该资源，请求失败
- 404 Not found: 资源不存在

- 500 Internal Server Error: 服务器错误，确认状态并报告问题

## 0.3http请求头

对于需要验证token的接口, 请求头中需包含 **x-user-token**, **x-user-id**(登录后返回的用户id)

---

# 1用户管理

管理用户端及管理员端用户, 包括用户的增删改查操作以及用户状态变更(禁用, 启用,注销)

## 1.0登录

- method: post
- url: domain/api/v1/admin/signin | domain/api/v1/users/signin

- 参数 

``` js
{
	"email": "admin@ibbd.net",
	"password": "scut2017",
	"uuid": "", # 验证码标识
	"code": "TtfHg" # 前端输入的验证码
}
```

- 请求参数

| 参数 |类型  |说明  |范围及格式|
|:----:|:----:|:----:|:--------:|
|email |string|邮箱  |          |
|password | string   | 密码|    |
|uuid|string|验证码标识||
|code|string|验证码||

- 返回结果:

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "data": {
    "id": "",
    "role_id": "",
    "token": "",
    "name": "",
    "email": "",
    "status": ""
  }
}
```

- data返回结果

| name | desc  |
|:----:|:----:|
|id |用户id|
|role_id | 角色id   |
|token | 服务端返回的token|
|name|用户名|
|email|邮箱|

- *code*值有:

| code | msg  |
|:----:|:----:|
|10001 |用户名或密码错误|
|10002 |无效的用户,请联系管理员|
|10003|没有登录权限,请使用管理员用户登录|          |
|10004|验证码不正确|
|10005|验证码失效|
|10006|帐号或密码连续输错多次，请5分钟之后再试|
|10007|用户已在其他主机登录,请先下线后再登录|

**说明**

- 非管理员用户不能登录管理员平台, 管理员用户可登录用户及管理员端
- 注册或新增的用户需要通过管理员审核通过后账户才有效
- 登录时用户账户或密码连续输错超过3次, 账户将被锁定5分钟
- 单个账户不允许在多台主机同时登录用户端或管理员端
- 用户登录后token有效期为60分钟, 有效期内若用户在系统上有发生操作, 则系统自动刷新token; 若无则token到期用户本次登录失效

## 1.1注册

- method: post
- url: domain/api/v1/signup(doamin/api/v1/admin/users/new)

- 参数

``` js
{
  "name": "Janni",
  "mobile": "18888888885",
  "email": "fanni@gmail.com",
  "password": "",
  "roleId": 1
}
```

**参数说明**

| name         |  desc         |   说明   |
|:------------:|:------------:|:--------:|
| name         | 用户名        |          |  
| mobile       | 手机号        | 11位数字, 1(3|4|5|7|8)开头| 
| email        | 邮箱          |          |
| password     | 密码          | 数字,字母及特殊字符(!@#$)任意两两组合|

- 返回结果:

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

- **code说明**

| code         |  msg         |   说明   |
|:------------:|:------------:|:--------:|
| 10009        | 用户名已注册 |          |  
| 100010       | 邮箱已注册   |          | 
| 100011       | 手机号已注册 |          |


## 1.2注销用户

- method: delete
- URL: domain/api/v1/users/:id

- 参数

``` js
{
  "remark": "注销leimi"
}
```

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

## 1.3禁用|启用用户

- method: patch
- URL: domain/api/v1/users/status/:id


- 参数

``` js
{
	"status": 1,  //1 启用, -1 禁用
	"remark": "启用leimi"
}
```

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

## 1.4审核用户

- method: patch
- URL: domain/api/v1/users/check/:id


- 参数

``` js
{
  "check_status": 1,
  "check_remark": "审核leimi通过"
}
```

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

## 1.5编辑用户

- method: patch
- URL: domain/api/v1/users/:id

- 参数

``` js
{
  "name": "",
  "password": "",
  "mobile": ""
}
```

- 参数说明

| name         |  说明         |
|:------------:|:------------:|
| name         | 用户名        |
| password     | 密码          |
| mobile       | 手机号        |

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

## 1.6修改用户密码

- 请求方式: post
- URL: domain/api/v1/admin/passwd/change

- 参数:

``` js
{
  "passwd": "",      // 原密码
  "new_passwd": ""  // 新密码
}
```

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

## 1.7登出

- 请求方式: get
- URL: domain/api/v1/users/logout/:name

- name: 取值user|admin, 分别标识登出用户端|管理员端

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

## 1.8用户列表

- 请求方式: get
- URL: domain/api/v1/users?query={}

- 参数

``` js
{
  "offset": 0,
  "limit": 10,
  "where": {
    "email": "邮箱模糊查询",
    "mobile": "手机号模糊查询",
    "group": "角色组查询",
    "status": 1, // 用户状态
    "check_status": 1, // 用户审核状态
    "created_at": {
      "$gt": "2017-02-02",
      "$lt": "2017-02-10"
    }
	}
}
```

- 返回结果

``` js
{
 {
  "code": 0,
  "error": {
    "msg": ""
  },
  "total": 1,
  "data": [
    {
      "created_at": "",
      "updated_at": "",
      "id": 1,
      "name": "admin",
      "email": "admin@ibbd.net",
      "mobile": "18888888888",
      "status": 1,
      "check_status": 1,
      "role": "管理员组"
    }
  ]
}
```

---

# 2验证码

## 2.0图片验证码

- method: get
- url: domain/api/v1/image/base64

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "data": {
    "img": "data:image/bmp;base64,...", // 图片的base64编码
    "uuid": "" # 验证码id标识
  }
}
```

**接口更新**

- 考虑到安全性, 原来的客户端验证修改为服务端验证方式,该接口返回的**uuid**作为验证码标识,提交时带上用户输入的验证码及uuid,具体看**0.用户登录**

- 验证码有效时间为60s,过期后需重新获取
- 验证码不区分大小写

---

# 3角色管理

- 管理平台角色, 默认分配管理员及游客两种角色, 管理员可自行添加角色组; 

- 平台权限基于角色设计, 不同角色组用户可拥有查看不同资源的权限;

- 用户迁移功能可将用户迁移至其他角色组;

## 3.1角色列表

- 请求方式: get
- URL: domain/api/v1/roles?query={body}

- body参数

``` js
{
  "offset": 0,
  "limit": 8,
  "where": {
    "name": "",
    "created_at": {
      "$gt": "2017-02-02",
      "$lt": "2017-02-10"
    },
    "updated_at": {
      "$gt": "",
      "$lt": ""
    }
	}
}
```

- 返回结果

``` js
{
  "code": 0,
  "errors": {
    "msg": ""
  },
  "total": 3,
  "data": [
    {
      "id": 1,
      "name": "管理员组",
      "desc": "管理员",
      "status": 1,
      "created_at": "2017-10-11 17:11:54",
      "updated_at": "2017-10-11 17:11:54",
    }
  ]
}
```

## 3.2创建角色

- 请求方式: post
- URL: domain/api/v1/roles

- 参数

``` js
{
	"name": "开发组",
	"desc": "开发小组"
}
```

- 错误代码

| code         |  msg         |   说明   |
|:------------:|:------------:|:--------:|
| 20001        | 角色组已存在 |     |


- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

## 3.3删除角色

- 请求方式: delete
- URL: domain/api/v1/roles/:id

- 返回结果

``` js

{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

## 3.4编辑角色

- 请求方式: patch
- URL: domain/api/v1/roles/:id

- 参数

``` js
{
	"name": "测试组",
	"desc": "测试小组"
}
```

- 返回结果

``` js

{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

## 3.4获取角色组下用户

- 请求方式: get
- URL: domain/api/v1/roles/users/:id

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "total": 1,
  "data": [
    {
      "id": 4,
      "name": "雷米"
    }
  ]
}
```

## 3.5转移角色组下用户

- 请求方式: patch
- URL: domain/api/v1/roles/users

- 参数

``` js
{
  "roleId": 3,
  "userId": [5,7]
}
```

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

---

# 4权限管理

根据角色对资源(菜单, 接口)分配权限, 用户登录用户端平台后通过接口返回其所在角色组下的资源列表, 前端根据返回列表渲染页面菜单, 每个菜单对应不同的可视化界面; 接口权限用于验证用户是否有调用某个接口的权限, 若该用户所在角色组没有获得此接口的使用权限, 则返回401状态码, 该部分的功能通过中间件实现;

实现该模块的功能的数据表设计如下:

**portal_role_resources(角色资源表)**

``` mysql
+-------------+---------+------+-----+---------+-------+
| Field       | Type    | Null | Key | Default | Extra |
+-------------+---------+------+-----+---------+-------+
| role_id     | int(11) | NO   | PRI | NULL    |       |
| resource_id | int(11) | NO   | PRI | NULL    |       |
```

**portal_resources(资源总表)**

``` mysql
+------------+-------------+------+-----+---------------------+----------------+
| Field      | Type        | Null | Key | Default             | Extra          |
+------------+-------------+------+-----+---------------------+----------------+
| id         | int(11)     | NO   | PRI | NULL                | auto_increment |
| detail_id  | int(11)     | NO   |     | NULL                |                |
| name       | varchar(32) | NO   |     | NULL                |                |
| type_id    | tinyint(2)  | NO   |     | 0                   |                |
| deleted_at | datetime    | NO   |     | 0000-01-01 00:00:00 |                |
```

**portal_menus(菜单表)**

``` mysql
+------------+--------------+------+-----+---------------------+----------------+
| Field      | Type         | Null | Key | Default             | Extra          |
+------------+--------------+------+-----+---------------------+----------------+
| id         | int(11)      | NO   | PRI | NULL                | auto_increment |
| item       | varchar(20)  | NO   | PRI | NULL                |                |
| name       | varchar(32)  | NO   |     | NULL                |                |
| parent     | int(11)      | YES  |     | NULL                |                |
| schema     | longtext     | YES  |     | NULL                |                |
| action     | tinyint(1)   | YES  |     | NULL                |                |
| priority   | int(2)       | NO   |     | NULL                |                |
| remark     | varchar(128) | YES  |     | NULL                |                |
| created_at | datetime     | YES  |     | NULL                |                |
| updated_at | datetime     | YES  |     | NULL                |                |
| deleted_at | datetime     | NO   |     | 0000-01-01 00:00:00 |                |
```

**protal_interfaces(接口表)**

``` mysql
+------------+--------------+------+-----+---------------------+----------------+
| Field      | Type         | Null | Key | Default             | Extra          |
+------------+--------------+------+-----+---------------------+----------------+
| id         | int(11)      | NO   | PRI | NULL                | auto_increment |
| name       | varchar(32)  | NO   |     | NULL                |                |
| url        | varchar(32)  | NO   |     | NULL                |                |
| method     | varchar(32)  | YES  |     | NULL                |                |
| module     | varchar(32)  | YES  |     | NULL                |                |
| remark     | varchar(128) | YES  |     | NULL                |                |
| created_at | datetime     | YES  |     | NULL                |                |
| updated_at | datetime     | YES  |     | NULL                |                |
| deleted_at | datetime     | NO   |     | 0000-01-01 00:00:00 |                |
```

- portal_role_resources:　角色资源关联表, 分配或撤销权限直接操作该表

- portal_resources: 资源总表, 建立与不同类型资源的关联关系

## 4.1资源列表

- method: get
- url: domain/api/v1/resources
- header: x-user-token

- 返回结果

``` js
{
    "code": 0,
    "error": {
      "msg": ""
    },
    "data": {
    "menus": [
      [
        {
          "id": 4, // 父菜单
          "name": "国际"
        },
        {
          "id": 6,
          "name": "新闻搜索"
        }
      ],
      [
        {
          "id": 7, // 父菜单
          "name": "人工智能"
        },
        {
          "id": 8,
          "name": "机器学习"
        }
      ]
    ],,
    "interfaces": {
        "users": [
          "登录",
          "注册"
        ]
    }
  }
}
```

- 结果说明:

**menus以父菜单进行分组, 第一个元素为父菜单,其余元素则为该父菜单下的子菜单**

## 4.2获取角色组下资源

- method: get
- url: domain/api/v1/roles/resource/:id

**id: 角色id**

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "data": {
    "menus": [
        {
            "id": 4,
            "name": "国际"
        },
        {
            "id": 6,
            "name": "新闻搜索"
        }
    ],
    "interfaces": {}
  }
}
```

## 4.3修改权限

- method: post
- url: domain/api/v1/roles/role2resource

``` js
{
	"id": 1,               # 角色id
	"menus": {
		"associate": [4, 5], # 分配权限的菜单id
		"dissociate": [1, 2] # 移除权限的菜单id
	},
	"interfaces": {
		"associate": ["登录"], # 分配权限的接口名称
		"dissociate": ["删除用户"] # 移除权限的接口名称
	}
}
```

- 返回结果

``` js
{
    "code": 0,
    "error": {
      "msg": ""
    }
}
```

## 4.4获取角色组下资源(用户端)

- method: get
- url: domain/api/v1/users/resource/:roleId

- 返回结果

``` js
{
    "code": 0,
    "error": {
      "msg": ""
    },
    "data": {
      "menus": [
        {
          "id": 1,
          "name": "管理数据",
          "action": 0,
          "priority": 8,
          "parent": 0,
          "schema": "{\"params\":\"\",\"form\":[],\"iframe\":[]}",
          "item": "data",
          "child": []
        },
        {
          "id": 4,
          "name": "搜索",
          "action": 1,
          "priority": 3,
          "parent": 0,
          "schema": "{\"params\":\"http://baidu.com\",\"form\":[],\"iframe\":[]}",
          "item": "find"
        }
      ],
      "interfaces": [
        {
          "created_at": "2018-01-26 14:25:08",
          "updated_at": "2018-01-26 14:25:08",
          "id": 2,
          "name": "登录",
          "url": "api/v1/users/signin"
        }
      ]
    }
}
```

---

# 5资源管理

- 提供对资源的存取操作, 保证数据持久化
- 菜单资源单独对应一个router页面, **schema**参数由前端自定义配置

## 5.1菜单资源

### 5.1.1创建菜单

- method: post
- url: domain/api/v1/resource/menus

- 参数

``` js
{
  "name": "数仓",
	"item": "data",
	"parent": 1,
	"action": 1,
  "priority": 3,
  "schema": {
    "params": "http://baidu.com",
    "formComponet": [
      {
        "name": "time", # 表单域名称
        "label": "时段", # 表单域标签
        "input": "select", # date | select
        "values": ["深圳", "广州", "香港"], # input 为date时, values 为空[]
        "size": "large",
        "rules": [] # 对应前端的表单域验证规则
      }
    ],
    "iframePage":[
      {
        "url":{},
        "width": "",
        "height": ""
      }
    ]
  }
  "remark": ""
}
```

- 参数说明

| 字段 | 类型 | 说明 |是否必填 |
|:----:|:---:|:---:|:--- |
| name | string | 菜单名称 | Y |
| item | string | 菜单项 (一级菜单必填)| N |
| parent | nubmer | 父级菜单ID, 值为0表示该菜单为一级菜单 | Y |
| action | number | 动作类型(0: 显示子菜单, 1: 打开iframe页面, 2: route, 3: 打开search页面) | Y |
| schema | string | 菜单对应页面的配置 | Y |
| priority | number | 权重 | Y |

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

- 错误码说明

| code |说明 |
|:----:|:---:|
| 30001 | 资源冲突, 菜单名已占用, **注意：一级菜单及相同父菜单下name, item 须唯一** |

### 5.1.2菜单列表

- method: get
- url: domain/api/v1/resource/menus?query={body}

- 参数

``` js
{
  "offset":: 0,
  "limit": 10,
  "where": {
    "name": "",
    "action": 0,
    "created_at": {
      "$gt": "2017-02-02",
      "$lt": "2017-02-10"
    },
    "updated_at": {
      "$gt": "",
      "$lt": ""
    }
  }
}
```

- 参数说明

| 字段 | 类型 | 说明 |是否必填 |
|:----:|:---:|:---:|:--- |
| offset | number | 分页 | N |
| limit | number | 页宽 | N |
| name | string | 菜单名 | N |
| action | number | 动作类型 | N |
| created_at | object | 创建时间 | N |
| updated_at | object | 修改时间 | N |

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "data": [
    {
      "created_at": "2018-02-26 13:54:42",
      "updated_at": "2018-02-26 13:54:42",
      "id": 4,
      "name": "国际",
      "item": "FinanceNew",
      "params": "",
      "parent": "",
      "action": 0,
      "priority": 7,
      "schema": {},
      "group": "游客组,研发"
    }
  ]
}
```

### 5.1.3编辑菜单

- method: put
- url: domain/api/v1/resource/menus/:id

- 参数

``` js
{
  "name": "",
  "schema": {},
  "params": "",
  "remark": "",
  "priority": 9
}
```

- 参数说明

| 字段 | 类型 | 说明 |是否必填 |
|:----:|:---:|:---:|:--- |
| name | string |  | N |
| schema | object |  | N |
| params | string |  | N |
| remark | string |  | N |

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

### 5.1.4删除菜单

- method: delete
- url: domain/api/v1/resource/menus/:id

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

### 5.1.5获取一级菜单

- method: get
- url: domain/resources/menu/parent

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "data": [
    {
        "id": 1,
        "name": "管理数据"
    },
    {
        "id": 5,
        "name": "搜索-A"
    }
  ]
}
```

## 5.2接口资源

### 5.2.1创建接口

- method: post
- url: domain/api/v1/resource/interfaces

- 参数

``` js
{
  "name": "注册f",
	"url": "/api/v1/users/signup",
	"method": "post",
	"module": "user",
  "remark": ""
}
```

| 字段 | 类型 | 说明 |是否必填 |
|:----:|:---:|:---:|:--- |
| name | string | 接口名称 | Y |
| url | string | 接口地址 | Y |
| method | string | 方法 | Y |
| module | string | 模块 | Y |
| remark | string | 描述 | N |

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

### 5.2.2接口列表

- method: get
- url: domain/api/v1/resource/interfaces?query={body}

- 参数

``` js
{
  "offset":: 0,
  "limit": 10,
  "where": {
    "name": "",
    "module": "",
    "created_at": {
      "$gt": "2017-02-02",
      "$lt": "2017-02-10"
    },
    "updated_at": {
      "$gt": "",
      "$lt": ""
    }
  },
  "fields": []
}
```

- 参数说明

| 字段 | 类型 | 说明 |是否必填 |
|:----:|:---:|:---:|:--- |
| offset | number | 页码 | N |
| limit | number | 页宽 | N |
| name | string | 接口名 | N |
| module | string | 模块 | N |
| created_at | object | 创建时间 | N |

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "data": []
}
```

### 5.2.3编辑接口

- method: put
- url: domain/api/v1/resource/interfaces/:id

- 参数

``` js
{
  "name": "",
  "url": "",
  "remark": ""
}
```

- 参数说明

| 字段 | 类型 | 说明 |是否必填 |
|:----:|:---:|:---:|:--- |
| name | string |  | N |
| url | string |  | N |
| remark | string |  | N |

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

### 5.2.4删除接口

- method: delete
- url: domain/api/v1/resource/interfaces/:id

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  }
}
```

---

# 6日志管理

记录用户在平台用户端及管理员端发生的操作, 主要记录信息有操作用户, 主机地址, 操作内容, 接口地址

## 6.1日志列表

- 请求方式: get
- URL: domain/api/v1/logs?query=body

- body参数

``` js
{
  "offset": 0,
  "limit": 8,
  "where": {
    "account": "",    # 用户名
    "account_id": 1,　# 用户id
    "action": "",     # 操作内容 
    "ip": "",         # 主机地址
    "created_at": {
      "$gt": "2017-02-02",
      "$lt": "2017-02-10"
    }
   }
}
```

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "total": 821,
  "data": [
    {
      "id": 1,
      "account_id": 1,
      "account": "",
      "action": "更新密码",
      "method": "POST",
      "url": "/api/v1/user/passwd",
      "ip": "127.0.0.1",
      "created_at": "2017-10-11T22:56:27.000Z",
      "updated_at": "2017-10-11T22:56:27.000Z"
    }]
}
```

## 6.2日志导出

- method: get
- url: domain/api/v1/logs/download?query={body}

- body 

``` js
{
  "offset": 0,
  "limit": 8,
  "where":{
    "account": "",   # 用户名
    "action": "",
    "ip": ,
    "created_at": {
      "$gt": "2017-02-02",
      "$lt": "2017-02-10"
    }
  }
}
```

- 返回结果

**xlsx**

---

# 7数据接口

## 7.1ES数据查询

- method: get
- url: domain/api/v1/es/search?query={body}

- body查询体

``` js
{
  "offset":0,
  "limit": 10,
  "where": {
    "index":"qualitrip", 　 # 数据集
    "key":"天津",  　　　　　 # 搜索参数
    "name": "菜单名"
  },
  "order": ["sum", "desc"] # ["排序字段", "排序规则(asc(升序), desc(降序)"]
}
```

- 返回结果

``` js
{
  "code": 0,
  "error": {
    "msg": ""
  },
  "total": 2351,
  "columns": [
    {
      "name": "sail_time", # 列名
      "sorting": true  # 是否可排序
    },
    {
      "name": "city",
      "sorting": false
    },
    ],
  "data": []
}
```

**结果说明:前端根据columns展示表头**

## 7.2情感分析

- method: post
- url: domain/api/v1/sentiments/predict

- body查询体

``` js
{
	"contents": ["市民于2015年12月29日购买一台魅族NOTE2电信版手机，颜色是白色，价值700元，市民表示购买时要求商家提供移动联通4G版，但商家提供的是电信版。市民要求商家退款，商家拒绝处理。", "5月14日，国家主席习近平在北京出席“一带一路”国际合作高峰论坛开幕式，并发表题为《携手推进“一带一路”建设》的主旨演讲。"]
}
```

- 返回结果 

```
{
  "code": 0,
  "message": "",
  "data": [
    0.19140599999999997,
    0.970703
  ]
}
```

## 7.3http请求转发接口

- method: post
- url: domain/api/v1/fetch

- body 

``` js
{
	"method": "post",
	"url": "http://api.example.com",
	"data": {},
  "dataType": "js",
  "contentType": "js"
}
```

- 参数说明

| 字段 | 类型 | 说明 |是否必填 |
|:----:|:---:|:---:|:--- |
| method | string | get, post, put | Y |
| url | string |api地址  | Y |
| data | object | body数据 | Y |
| dataType | string | 默认为js | N |
| contentType | string | 默认为js | N |

- 返回结果 

```
{
  "code": 0,
  "message": "",
  "data": {}
}
```








