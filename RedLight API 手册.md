

# RedLight API 手册（ver 1.01） #

## 一、简要约定：

项目名称暂定“Redlight”。以下说明中，API主域名以“http://www.redlight.com/api/"为示例，测试服务器及最终产品服务器，以实际部署域名为准。

API请求需加入当前版本号，1.00~1.99版本为V1，2.00~2.99版本为V2……以此类推。举例http://www.redlight.com/api/v1/

API返回数据均为JSON字串

**API使用前提：**RedLight具备教师账号数据库信息，即拥有edx平台的教师id及对应登录密码

## 二、API 介绍：

### 1. 授权登录 signin（GET）：

获得redlight授权。

成功登录后，redlight会返回一个session字符串，后续的API请求会利用这个字串做登录校验。

USAGE： 主域名+api路径+参数

| 参数名 | 类型   | 说明     |
| ------ | ------ | -------- |
| techID | string | 教师id   |
| passwd | string | 登录密码 |

***说明：***

*1.session字串代表用户的登录状态，有效期为30分钟。30分钟内用户没有调用API的动作，session失效，视为logout；*

*2.在session有效期内，任何api的调用，均会刷新session有效期，使其有效期恢复为30分钟；*

*3.如果调用api，收到session过期的错误，可重新调用signin方法获取session字符串，再进行后续操作。*



举例：

``` http://www.redlight.com/api/v1/signin?techID=12345&passwd=abcdef ```

成功返回：

``` { 
{
	ErrCode:0,    //错误码，0为成功，非0为失败
	ErrMsg:"",    //出错原因
	Session:"alskdjf54alsdkfj42asslakdjfl123asjkd"  //SessionCode
}
```

失败返回：

``` 
{
	ErrCode:1,    //错误码，0为成功，非0为失败
	ErrMsg:"can not find techid",    //出错原因
	Session:""  //SessionCode
}

```

### 2. 创建房间模板room（GET / POST）：

创建虚拟直播间。如果成功创建虚拟直播间，则会返回2个随机密码，一个为普通用户密码attendeePW，一个为主持人密码moderatorPW。启动、加入会议时，需要用到密码作为参数。

GET方法为普通模式，建立一个标准房间模板。

POST方法为增强模式，参数以JSON方式传递，多了预传递课件URL的参数，也可以加入自定义参数作为扩展数据。

**USEAGE（GET）：**主域名+api路径+参数+session字串

| 参数名      | 类型   | 说明                                                        |
| ----------- | ------ | ----------------------------------------------------------- |
| meetingID   | string | 直播间id（required）                                        |
| name        | string | 直播间名称（required）                                      |
| startDay    | string | 开始日，取值范围 ”周一“ ~ ”周日“                            |
| startTime   | string | 开始时间，取值范围”00:00“-”23:59“                           |
| endTime     | string | 结束时间，取值范围 ”00:00“-”23:59“                          |
| muteOnStart | bool   | 用户加入时静音：true为静音，默认为false                     |
| guestPolicy | bool   | 用户加入时需要主持人批准：true为需要主持人批准，默认为false |

***attendeePW与moderatorPW的说明：***

*1. 教师默认为主持人身份，默认使用moderatorPW密码进入直播间;*

*2. 学生如果携attendeePW密码进入直播间为普通用户，如果携moderatorPW密码进入直播间，则为主持人身份*



举例:

1. 直播间id：jisfenasdfk；直播间名称：history ：

``` http://www.redlight.com/api/v1/room?meetingID=jisfenasdfk&name=history&session=alskdjf54alsdkfj42asslakdjfl123asjkd ```

2. 直播间id：jisfenasdfk；直播间名称：history ；用户加入需要主持人批准：

   ``` 
   http://www.redlight.com/api/v1/room?meetingID=jisfenasdfk&name=history&guestPolicy=true&session=alskdjf54alsdkfj42asslakdjfl123asjkd

成功返回:

```
{
	ErrCode:0,
	ErrMsg:"",
	attendeePW:"iwuerowj"     //普通用户密码
	moderatorPW:"ldjfsidfj"   //主持人密码
	meetingURL:"http://www.redlight.com/b/ksjaofejl"  //直播间入口URL
}
```

失败返回：

``` 
{
	ErrCode:1,
	ErrMsg:"the meeting room was created earlier with this meetingID",
	attendeePW:""     //普通用户密码
	moderatorPW:""   //主持人密码
	meetingURL:""  //直播间入口URL
}
```

**USEAGE（POST）：**主域名+api路径+JSON数据

| 参数名      | 类型         | 说明                                                        |
| ----------- | ------------ | ----------------------------------------------------------- |
| meetingID   | string       | 直播间id（required）                                        |
| name        | string       | 直播间名称（required）                                      |
| startDay    | string       | 开始日，取值范围 ”周一“ ~ ”周日“                            |
| startTime   | string       | 开始时间，取值范围”00:00“-”23:59“                           |
| endTime     | string       | 结束时间，取值范围 ”00:00“-”23:59“                          |
| muteOnStart | bool         | 用户加入时静音：true为静音，默认为false                     |
| guestPolicy | bool         | 用户加入时需要主持人批准：true为需要主持人批准，默认为false |
| slidesList  | object Array | 课件URL列表                                                 |
| customParam | object       | 自定义参数（以备将来扩展使用）                              |

举例：

1. 直播间id：jisfenasdfk；直播间名称：history ；每周三13:00-14:30上课，有2个课件需要预传：

``` 
http://www.redlight.com/api/v1/room
JSON 数据：
{
	meetingID:"jisfenasdfk",
	name:"history",
	startDay:"周三",
	startTime:"13:00",
	endTime:"14:30",
	slidesList:[{
		name:"history lesson1.pdf",
		url:"http://www.xxxxx.com/isijfs/history1.pdf"
		},{
		name:"history lesson1.pdf",
		url:"http://www.xxxxx.com/isijfs/history1.pdf"
		}],
	customParam:{
		one:xxxx,
		two:sdfsf
	},
	session:"alskdjf54alsdkfj42asslakdjfl123asjkd"
}
```

### 3. 修改虚拟直播间 room(PUT)

该PUT方法，会更新虚拟直播间的设置参数。参数以JSON字串方式传递，类似POST方法

**USEAGE（PUT）：**主域名+api路径+JSON数据

| 参数名      | 类型         | 说明                                                        |
| ----------- | ------------ | ----------------------------------------------------------- |
| meetingID   | string       | 直播间id（required）                                        |
| name        | string       | 直播间名称（required）                                      |
| startDay    | string       | 开始日，取值范围 ”周一“ ~ ”周日“                            |
| startTime   | string       | 开始时间，取值范围”00:00“-”23:59“                           |
| endTime     | string       | 结束时间，取值范围 ”00:00“-”23:59“                          |
| muteOnStart | bool         | 用户加入时静音：true为静音，默认为false                     |
| guestPolicy | bool         | 用户加入时需要主持人批准：true为需要主持人批准，默认为false |
| slidesList  | object Array | 课件URL列表                                                 |
| customParam | object       | 自定义参数（以备将来扩展使用）                              |

举例：

1. 修改直播间id：jisfenasdfk；直播间名称：history ；用户加入需要批准；每周五13:00-14:30上课，有1个课件需要预传：

``` 
http://www.redlight.com/api/v1/room
JSON 数据：
{
	meetingID:"jisfenasdfk",
	name:"history",
	startDay:"周五",
	startTime:"13:00",
	endTime:"14:30",
	guestPolicy:true,
	slidesList:[{
		name:"history lesson1.pdf",
		url:"http://www.xxxxx.com/isijfs/history1.pdf"
		}],
	customParam:{
		one:xxxx,
		two:sdfsf
	},
	session:"alskdjf54alsdkfj42asslakdjfl123asjkd"
}
```

成功返回：

``` 
{
	ErrCode:0,    //错误码，0为成功，非0为失败
	ErrMsg:""    //出错原因
}
```

出错返回：

``` 
{
	ErrCode:1,    //错误码，0为成功，非0为失败
	ErrMsg:"room update failed"   //出错原因
}
```



### 4. 删除虚拟直播间 room （DELETE）

该方法会删除指定的虚拟直播间。

**USEAGE（DELETE）：**主域名+api路径+参数+session字串

| 参数名    | 类型   | 说明                 |
| --------- | ------ | -------------------- |
| meetingID | string | 直播间id（required） |

举例删除meetingID为12313的虚拟直播间:

``` 
http://www.redlight.com/api/v1/room?meetingID=12313&session=alskdjf54alsdkfj42asslakdjfl123asjkd
```

成功返回：

``` 
{
	ErrCode:0,    //错误码，0为成功，非0为失败
	ErrMsg:""    //出错原因
}
```

出错返回：

``` 
{
	ErrCode:1,    //错误码，0为成功，非0为失败
	ErrMsg:"meetingID is wrong"    //出错原因
}
```



### 5. 开始会议start（GET）：

启动并进入会议。该指令的使用分为2种情况：

* 老师使用（URL尾部有session字串），视为启动会议，默认为主持人身份。
* 学生使用（无session字串），视为加入会议，如果加入会议时使用moderatorPW（主持人密码）进入也被视为主持人。

**USEAGE**：

老师：主域名+api路径+参数+session字串

| 参数名     | 类型   | 说明                       |
| ---------- | ------ | -------------------------- |
| meetingURL | string | 直播间入口URL（requeired） |

举例：

``` 
http://www.redlight.com/api/v1/start?meetingURL=http://www.redlight.com/b/ksjaofejl&session=alskdjf54alsdkfj42asslakdjfl123asjkd
```

**USEAGE**： 

学生：主域名+api路径+参数

| 参数名     | 类型   | 说明                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| meetingURL | string | 直播间入口URL（requeired）                                   |
| userName   | string | 学生姓名（requeired）                                        |
| passwd     | string | 会议密码（requeired），如果用attendeePW则为普通用户，用moderatorPW则为主持人 |
| userID     | string | 学生id（requeired），学生的数据库id，用来识别用户身份        |

举例：

``` 
http://www.redlight.com/api/v1/start?meetingURL=http://www.redlight.com/b/ksjaofejl&userName=lell&passwd=lskjfis&userID=12313
```

成功返回：

``` 
{
	ErrCode:0,
	ErrMsg:"",
	JoinURL:"http://www.redlight.com/b/jisflj"  //直播间URL
}
```

失败返回：

``` 
{
	ErrCode:1,
	ErrMsg:"start meeting failed",
	JoinURL:""  //直播间URL
}
```



如果调用成功，则跳转到joinURL提供的URL地址，进入直播间。

### 6. 获取虚拟直播间列表 roomList (GET) ###

获取创建的虚拟直播间列表。

**USEAGE：**主域名+api路径+session字串

举例：

``` 
http://www.redlight.com/api/v1/roomList?session=alskdjf54alsdkfj42asslakdjfl123asjkd
```

成功返回：

```
{
	ErrCode:0,
	ErrMsg:"",
	RoomList:[{
		name:"sfaf",
		meetingID:"sdfjslfsf",
		attendeePW:”sdfasfd“,
		moderatorPW:"sfasdfasdf",
        muteOnStart:true,
        guestPolicy:false		
	},{
		name:"safasdf",
		meetingID:"sdfjsdfasdf",
		attendeePW:”sdfsdfasfd“,
		moderatorPW:"sf234asdfasdf",
        muteOnStart:true,
        guestPolicy:false
	}]
}

```

失败返回：

``` 
{
	ErrCode:1,
	ErrMsg:"get room list failed",
	RoomList:[]
}
```
## 三 、更新记录：

### V1.02:

1.删除创建房间模板API中attendeePW参数，密码管理交给redlinght。在创建房间模板时redlight会自动生成attendeePW和moderatorPW两个密码，并在创建成功后将2个密码返回。

2.老师调用start方法进入直播间不需要带密码参数，redlinght会自动匹配上moderatorPW，以主持人身份登录。

3.学生调用start方法时，需要密码参数，传入attendeePW是以普通身份加入直播间，传入moderatorPW则以主持人身份进入直播间。具体使用哪个密码，视具体的业务场景。

4.修改部分手误

### V1.01:

1. API中的参数字面量，统一为小写驼峰命名，例如：attendeePW,name等；
2. 将SessionCode参数的命名改为session；
3. 删除room系列API中的AllModerator参数，该功能属于业务功能，需要业务逻辑去维护。
4. 修改一些手误。

### V1.00:

redlight init
