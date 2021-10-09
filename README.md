# 用户行为埋点规范

> 用户行为埋点的规范主要用于用户端产品，包括小程序、App、H5活动链接等。
>
> 规范的内容包括：
>
> - 数据规范
> - 需求规范



## 1.  记录用户行为的规范

> 主要说明在采集用户行为时，默认收集的信息。



在收集信息的过程中遵循以下原则：

- 原则一：不在采集数据的环节添加业务逻辑或统计逻辑。
- 原则二：尽量收集完整的行为数据，采集环节不过滤数据。
- 原则三：未上报成功的数据能够续传至服务端存储，已上传的数据不重复存储。
- 原则四：对有埋点的旧功能模块进行技术改造时，需要考虑埋点对象是否需要延续。

按照原则一，例如“当天第一次访问诺友圈首页”这样的埋点事件不合理，应该记录为“访问诺友圈首页”，“第一次访问”的逻辑放在分析阶段；

按照原则二，例如“注册用户访问PGC文章详情页”这样的埋点事件不合理，应该记录为“访问PGC文章详情页”，应该记录包括匿名用户在内所有用户的行为，过滤数据放在分析阶段。



### 1.1  用户行为命名



**系统名称\_模块名称-页面名称-对象名称\_操作类型**

完整命名包含【系统名称】、【模块名称-页面名称-对象名称】、【操作类型】

- 三部分通过下划线"_"连接。
- 每一部分内部用中划线"-"连接。

|                            | 中文可选                                    | 英文可选                                           |
| -------------------------- | ------------------------------------------- | -------------------------------------------------- |
| 系统名称                   | 三诺健康小程序                              | healthmini                                         |
| 模块名称-页面名称-对象名称 | 按功能显示的名称显示                        | 按元件功能title/name显示                           |
| 操作类型                   | 启动/退出/点击/浏览/输入/滑动/关闭/后台响应 | start/quit/click/visit/input/scroll/close/response |



### 1.2 用户行为数据结构



**用户行为= 用户Who+动作Action+对象Object**

> 以下说明为埋点基本公共字段，如有额外字段，可以添加到单个事件，或提炼为公共基本字段。



**行为数据公共结构：**


	data:
	{
		user:{//用户
      who:string,//匿名ID
			userID:string,//系统ID
			objectID:string//对象ID
		},
		action:{//动作
			type:string,//动作类型
      ext:{//动作包含的特定属性
		  	...//在下文分别描述
		  }, 
			when:timestamp,//动作发生时间
			IP:string,//动作IP
			env:{//动作设备
				device:string//设备品牌
				system:string,//设备系统
				browser:string,//设备浏览器
				browserVer:string,//设备浏览器版本
				screenWidth:int,//屏幕宽
				screenHeight:int//屏幕高
		  }
		},
		object:{
			product:string,//用户使用的系统
			productVer:string,//用户使用的系统版本
			url:string,//当前对象所在页面的Url，含参数
			divName:string//当前对象的xpath中div属性
      elementType:string,//当前对象的元件类型
      elementName:string//当前对象的xpath中元件属性
		}
	}




#### 1.2.1 用户Who



**用户数据公共结构：**

	user:{//用户
  	who:string,//匿名ID
		userID:string,//系统ID
		objectID:string//对象ID
	}

字段详细说明：

- 匿名ID：用户在未登录状态，为用户分配的标识

> 微信生态：UnionID
>
> H5/Web：cookie
>
> App：待定

- 系统ID：用户当前访问哪个系统，他在该系统分配的ID

> 微信生态：accountID
>
> H5/Web：待定
>
> App生态：待定

- 对象ID：用户在用户平台建立对象生成的ID。

> 目前特指t_archives_user.id

当用户匿名登录时，应该根据规则，捕获到用户的匿名ID，系统ID为空，对象ID根据用户当时的认证信息赋值。

当用户登录后，匿名ID不变，系统ID和对象ID为登录后用户信息赋值。对于登录状态的用户，如果前端取不到系统ID和对象ID，需要强制拉取用户信息赋值到系统ID和对象ID。



#### 1.2.2 动作Action

目前定义动作为以下几种：

> 在定义以下动作时，需要根据需要确认动作记录的条件，如埋点记录的触发实在JS验证之前还是之后



**动作数据公共结构：**

	action:{//动作
		type:string,//动作类型
  	ext:{//动作包含的特定属性
	  	...//在下文分别描述
	  }, 
		when:timestamp,//动作发生时间
		IP:string,//动作IP
		env:{//动作设备
			device:string//设备品牌
			system:string,//设备系统
			browser:string,//设备浏览器
			browserVer:string,//设备浏览器版本
			screenWidth:int,//屏幕宽
			screenHeight:int//屏幕高
	  }
	}




##### 1.2.2.1 启动（start）：系统启动

> 启动操作包括冷启动和热启动。热启动是指小程序或App等在后台进程未完全关闭的情况下被再次激活，冷启动是指小程序或App等经过自身初始化加载到手机进程中。



action:{//动作
	type:"start"//动作类型
  ext:{
  	sceneId:string,//标识进入小程序/App/H5的路径，小程序为各生态的内置场景值ID
  	scene:string,//场景值名称
	},
	when:"2021-08-08 18:18:18",//动作发生时间
	IP:"220.220.220.220",//动作IP
	env:{//动作设备
		device:"iPhone 11"//设备品牌
		system:"iOS",//设备系统
		browser:"Safari",//设备浏览器
		browserVer:"14.1.2 (16611.3.10.1.3)",//设备浏览器版本
		screenWidth:1920,//屏幕宽
		screenHeight:1080//屏幕高
	}
}




**附：各生态小程序默认场景值**

微信小程序：https://developers.weixin.qq.com/miniprogram/dev/reference/scene-list.html 

支付宝小程序：https://opendocs.alipay.com/mini/framework/scene

字节小程序：https://microapp.bytedance.com/docs/zh-CN/mini-app/develop/framework/scene-value

百度小程序：https://smartprogram.baidu.com/docs/data/scene/

QQ小程序：https://q.qq.com/wiki/develop/game/frame/scene/



##### 1.2.2.2 退出（quit）：系统退出


action:{//动作
	type:"quit"//动作类型
  ext:{
  	referUrl:string,//退出前所在的页面
	},
	...//when/IP/env属性
}




##### 1.2.2.3 点击（click）：对页面元素单击/双击的操作

> 点击动作暂无扩展属性


action:{//动作
	type:"click"//动作类型
  ext:{
	},
	...//when/IP/env属性
}




##### 1.2.2.4 浏览（visit）：成功进入页面即为浏览


action:{//动作
	type:"visit",//动作类型
  ext:{
		referUrl:string,//浏览来源页面
		utm_source:string,//来源页参数，标识渠道
    utm_medium:string,//来源页参数，标识媒介
    utm_term:string,//来源页参数，标识关键词
    utm_campaign:string,//来源页参数，标识运营活动
    utm_content:string// 来源页参数，标识内容
	},
	...//when/IP/env属性
}




##### 1.2.2.5 输入（input）：通过键盘向输入框输入字符、数字、文字等，在输入框失去焦点时结束

> 输入动作暂无扩展属性


action:{//动作
	type:"input",//动作类型
  ext:{
  },
  ...//when/IP/env属性
}




##### 1.2.2.6 滑动（scroll）：按住对象移动


action:{//动作
	type:"scroll",//动作类型
  ext:{
  	x0:int,//对象在屏幕横向坐标起始位置
  	x1:int,//对象在屏幕横向坐标结束位置
  	y0:int,//对象在屏幕纵向坐标结束位置
  	y1:int//对象在屏幕纵向坐标结束位置
	},
	...//when/IP/env属性
}




##### 1.2.2.7 关闭（close）：关闭/返回操作


action:{//动作
	type:"close",//动作类型
  ext:{
  	duration:int//从元件打开到离开元件持续了多少秒
	},
  ...//when/IP/env属性
}




##### 1.2.2.8 后台响应（response）：接收后台返回结果


action:{//动作
	type:"response",//动作类型
  ext:{
  	result:string,//后台返回的结果值
    request:string//后台响应对应的请求，带完整参数
	},
  ...//when/IP/env属性
}




#### 1.2.3 对象Object


object:{
	product:string,//用户使用的系统
	productVer:string,//用户使用的系统版本
	url:string,//当前对象所在Url，含参数
	divName:string,//当前对象的xpath中div属性
  elementType:string,//当前对象的元件类型
 	elementName:string//当前对象的元件名称或标题
}


- 当动作类型action.type为start、quit、response时，只有product、productVer属性，无url、divName、elementType、elementName属性
- 当动作类型object.elementType为page时，只有product、productVer、url、elementName（title）属性，无divName属性



**用户使用的系统（product）**

用户操作对象所在的公司软件产品，具备区分性，如：三诺健康小程序、三诺健康App、三诺教育平台公众号。



**用户使用的系统版本（productVer）**

软件产品系统在生产环境的版本。



**当前对象所在Url（url）**

用户操作对象所在页面url的绝对路径。



**当前对象所在模块（divName）**

用户操作对象处于页面的模块，用xpath中的div名称标识。



**当前对象元件类型（elementType）**

- 主要类型：按钮（button）、输入框（input）、弹窗（popout）、页面（page）



### 1.3 用户行为数据存储

行为数据存储不限定格式，但需要存储的基本信息为：


{
 	"用户行为中文命名",
 	"用户行为英文命名",
 	"用户行为数据结构":{
 		"用户",
 		"动作",
 		"对象"
 	}
}




## 2.  埋点需求的规范

在整理埋点需求是遵循以下原则：

- 原则一：根据业务指标的口径，拆解用户的事实动作。
- 原则二：收集必要的或潜在的用户行为，不过度记录、重复记录。



### 2.1  埋点需求提出

| 事件中文                                                    | 事件英文名                               | 事件触发条件                             | 用户字段                                                     | 事件字段                                                     | 对象字段                                                     |
| <!--按照1.1.1中的事件结构命名-->                            | <!--按照1.1.1中的事件结构命名-->         | <!--描述对象的具体位置以及触发条件-->    | <!--如果没有额外的字段，统一记录为公共字段，额外的字段，标明字段来源、字段类型。如“用户手机号”，来源为用户保存在数据库的电话号码，类型为字符/string型。--> | <!--如果没有额外的字段，统一记录为公共字段，额外的字段，标明字段来源、字段类型。--> | <!--描述对象的字段信息的填入规则-->                          |
| 例：三诺健康小程序\_我的-我的积分-每日任务-去完成按钮\_点击 | mini_mine-mylegral-dailytask-todo\_click | 点击任意每日任务下任意“去完成”按钮时记录 | 公共字段                                                     | 公共字段                                                     | 在对象中divName均为每日任务的div模块名称；<br />elementType均为button；<br />elementName为点击按钮的标题，如“上传血糖”、“阅读文章” |



### 2.2  埋点版本管理

| 事件中文                                                    | 事件英文名                                       | 上线日期                | 当前状态                                                     | 备注                                                         |

| <!--按照1.1.1中的事件结构命名，与埋点需求对应-->            | <!--按照1.1.1中的事件结构命名，与埋点需求对应--> | <!--埋点需求上线日期--> | <!--该埋点处于废弃状态还是使用状态。默认为使用状态，当功能删除时，埋点处于废弃状态--> | <!--该埋点提出的原因（统计口径）、废弃的原因。-->            |
| 例：三诺健康小程序\_我的-我的积分-每日任务-去完成按钮\_点击 | mini_mine-mylegral-dailytask-todo\_click         | 20210804                | 使用中                                                       | 提出原因：用于统计用户主要获取哪一类型的积分，获取的频次和间隔 |


















