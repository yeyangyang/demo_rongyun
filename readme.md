# 融云WEB端口开发

标签（空格分隔）： 公司

---


#分析
- [x] 1.服务端先注册用户，然后加入一个token，这样就现成了一个token对应一个用户的概念；
- [x] 2.客户端初始化融云，这样我们的客户端就能实时的接收到其他用户所发过来的消息了；
- [x] 3.客户端进行消息发送（也可以用服务端，占时先用客户端的进行发送消息）；
- [x] 4.客户端使用 `消息监听` 来实时接受消息
- [x] 5.客户端接下来是做会话列表（这样展示出来我所聊过的所有用户，到时候好进行相对应的用户操作）；使用`获取会话列表`，这样子就能获取我所聊过的所有用户（他这个所有列表还可以获得用户聊天的最后一条记录，**就是不知道能不能判断他是否读取过当前这条数据了**）；
- [x] 6.客户端的会话功能，既然能获取他的列表，那么就代表肯定能删除他的列表数据了；
- [x] 7.最后只要每次进入和这个用户的聊天框时，既可以查看上次的聊天记录（获取单群聊历史消息）；


---


# 参考文档

 1. 客户端
  [客户端文档](https://www.rongcloud.cn/docs/web_api_demo.html#introduction)
  [客户端源码下载地址](https://github.com/rongcloud/websdk-demo/tree/master/im)
  [客户端WebSDK demo](https://github.com/rongcloud/websdk-demo)
  [客户端WebIM 集成引导](https://rongcloud.github.io/websdk-demo/integrate/guide.html)
 2. 服务端
  [服务端文档](https://www.rongcloud.cn/docs/server.html#open_source_sdk)
  [服务端-PHP源码下载地址](https://github.com/rongcloud/server-sdk-php)

---

# 功能接口

## 客户端

### 消息接口
- [x] 发送接口
- [x] 获取单群聊历史消息

### 会话接口
- [x] 获取会话列表
- [x] 删除指定会话
- [x] 删除所有会话


## 服务端
- [x] 用户注册加入融云token值

---

# 日程

 - 20180913 今天把基本的融云知识了解了一下，其中就是初步的会使用了他的聊天（发消息、获取消息列表、获取聊天记录、删除会话等操作），这些基础的操作，现在做的也只是按照文档上面来一步一步操作下来的，里面的含义还没有理解进入

---


# 操作
## 客户端
### 初始化
* **初始化融云 SDK**
必须要在html头部中引用融云的js`<script src="http://cdn.ronghub.com/RongIMLib-2.2.5.min.js"></script>`
```js
    //AppKey：应用的唯一标识。
    RongIMLib.RongIMClient.init("AppKey");
```

* **设置监听**
必须设置监听器后，再连接融云服务器，示例代码如下：

```js
// 设置连接监听状态 （ status 标识当前连接状态 ）
 // 连接状态监听器
 RongIMClient.setConnectionStatusListener({
    onChanged: function (status) {
        switch (status) {
            case RongIMLib.ConnectionStatus.CONNECTED:
                console.log('链接成功');
                break;
            case RongIMLib.ConnectionStatus.CONNECTING:
                console.log('正在链接');
                break;
            case RongIMLib.ConnectionStatus.DISCONNECTED:
                console.log('断开连接');
                break;
            case RongIMLib.ConnectionStatus.KICKED_OFFLINE_BY_OTHER_CLIENT:
                console.log('其他设备登录');
                break;
              case RongIMLib.ConnectionStatus.DOMAIN_INCORRECT:
                console.log('域名不正确');
                break;
            case RongIMLib.ConnectionStatus.NETWORK_UNAVAILABLE:
              console.log('网络不可用');
              break;
            }
    }});

 // 消息监听器
 RongIMClient.setOnReceiveMessageListener({
    // 接收到的消息
    onReceived: function (message) {
        // 判断消息类型
        switch(message.messageType){
            case RongIMClient.MessageType.TextMessage:
                // message.content.content => 消息内容
                break;
            case RongIMClient.MessageType.VoiceMessage:
                // 对声音进行预加载                
                // message.content.content 格式为 AMR 格式的 base64 码
                break;
            case RongIMClient.MessageType.ImageMessage:
               // message.content.content => 图片缩略图 base64。
               // message.content.imageUri => 原图 URL。
               break;
            case RongIMClient.MessageType.DiscussionNotificationMessage:
               // message.content.extension => 讨论组中的人员。
               break;
            case RongIMClient.MessageType.LocationMessage:
               // message.content.latiude => 纬度。
               // message.content.longitude => 经度。
               // message.content.content => 位置图片 base64。
               break;
            case RongIMClient.MessageType.RichContentMessage:
               // message.content.content => 文本消息内容。
               // message.content.imageUri => 图片 base64。
               // message.content.url => 原图 URL。
               break;
            case RongIMClient.MessageType.InformationNotificationMessage:
                // do something...
               break;
            case RongIMClient.MessageType.ContactNotificationMessage:
                // do something...
               break;
            case RongIMClient.MessageType.ProfileNotificationMessage:
                // do something...
               break;
            case RongIMClient.MessageType.CommandNotificationMessage:
                // do something...
               break;
            case RongIMClient.MessageType.CommandMessage:
                // do something...
               break;
            case RongIMClient.MessageType.UnknownMessage:
                // do something...
               break;
            default:
                // do something...
        }
    }
});

```

* **连接服务器**
连接融云必须在执行 `RongIMLib.RongIMClient.init(appkey);` 之后，并且所有除监听以外的方法都必须在 `connect成功之后` 再调用。

```js
  var token = "mKmyKqTSf7aNDinwAFMnz7NXKILeV3X0+CCRBOxmtOApmvQjMathViWrePIfq0GuTu9jELQqsckv4AhfjCAKgQ==";

  RongIMClient.connect(token, {
        onSuccess: function(userId) {
          console.log("Connect successfully." + userId);
        },
        onTokenIncorrect: function() {
          console.log('token无效');
        },
        onError:function(errorCode){
              var info = '';
              switch (errorCode) {
                case RongIMLib.ErrorCode.TIMEOUT:
                  info = '超时';
                  break;
                case RongIMLib.ConnectionState.UNACCEPTABLE_PAROTOCOL_VERSION:
                  info = '不可接受的协议版本';
                  break;
                case RongIMLib.ConnectionState.IDENTIFIER_REJECTED:
                  info = 'appkey不正确';
                  break;
                case RongIMLib.ConnectionState.SERVER_UNAVAILABLE:
                  info = '服务器不可用';
                  break;
              }
              console.log(errorCode);
            }
      });
      
```      

* **重新连接**

```js
    var callback = {
        onSuccess: function(userId) {
            console.log("Reconnect successfully." + userId);
        },
        onTokenIncorrect: function() {
            console.log('token无效');
        },
        onError:function(errorCode){
            console.log(errorcode);
        }
    };
    var config = {
        // 默认 false, true 启用自动重连，启用则为必选参数
        auto: true,
        // 重试频率 [100, 1000, 3000, 6000, 10000, 18000] 单位为毫秒，可选
        url: 'cdn.ronghub.com/RongIMLib-2.2.6.min.js',
        // 网络嗅探地址 [http(s)://]cdn.ronghub.com/RongIMLib-2.2.6.min.js 可选
        rate: [100, 1000, 3000, 6000, 10000]
    };
    RongIMClient.reconnect(callback, config);
```

>到这一步就可以,只要连接上了就可以了
>连接上了，接下来就可以去实现发消息的功能了
>连接成功会在console上面显示出来

### 消息接口

* **发送消息**
使用`客户端`的代码进行发送消息

```js
/* 发送接口  */
function send_message(content="hello",extra="附加信息",targetId="userId2"){
		 var msg = new RongIMLib.TextMessage({content:"hello RongCloud!",extra:['这个是用户id','这个是头像路径','这个是昵称']});
		 var conversationtype = RongIMLib.ConversationType.PRIVATE; // 单聊,其他会话选择相应的消息类型即可。
		 var targetId = "userId2"; // 目标 Id
		 RongIMClient.getInstance().sendMessage(conversationtype, targetId, msg, {
                onSuccess: function (message) {
                    //message 为发送的消息对象并且包含服务器返回的消息唯一Id和发送消息时间戳
                    console.log("Send successfully");
                },
                onError: function (errorCode,message) {
                    var info = '';
                    switch (errorCode) {
                        case RongIMLib.ErrorCode.TIMEOUT:
                            info = '超时';
                            break;
                        case RongIMLib.ErrorCode.UNKNOWN_ERROR:
                            info = '未知错误';
                            break;
                        case RongIMLib.ErrorCode.REJECTED_BY_BLACKLIST:
                            info = '在黑名单中，无法向对方发送消息';
                            break;
                        case RongIMLib.ErrorCode.NOT_IN_DISCUSSION:
                            info = '不在讨论组中';
                            break;
                        case RongIMLib.ErrorCode.NOT_IN_GROUP:
                            info = '不在群组中';
                            break;
                        case RongIMLib.ErrorCode.NOT_IN_CHATROOM:
                            info = '不在聊天室中';
                            break;
                        default :
                            info = x;
                            break;
                    }
                    console.log('发送失败:' + info);
                }
            }
        );	
}

```

* **消息接受**

一般在上面几步就已经写好了这个方法,用`消息监听器`来监控是否有新的消息
```js
// 消息监听器
 RongIMClient.setOnReceiveMessageListener({
    // 接收到的消息
    onReceived: function (message) {
        // 判断消息类型
        switch(message.messageType){
            case RongIMClient.MessageType.TextMessage:
                // message.content.content => 消息内容
                break;
            case RongIMClient.MessageType.VoiceMessage:
                // 对声音进行预加载                
                // message.content.content 格式为 AMR 格式的 base64 码
                break;
            case RongIMClient.MessageType.ImageMessage:
               // message.content.content => 图片缩略图 base64。
               // message.content.imageUri => 原图 URL。
               break;
            case RongIMClient.MessageType.DiscussionNotificationMessage:
               // message.content.extension => 讨论组中的人员。
               break;
            case RongIMClient.MessageType.LocationMessage:
               // message.content.latiude => 纬度。
               // message.content.longitude => 经度。
               // message.content.content => 位置图片 base64。
               break;
            case RongIMClient.MessageType.RichContentMessage:
               // message.content.content => 文本消息内容。
               // message.content.imageUri => 图片 base64。
               // message.content.url => 原图 URL。
               break;
            case RongIMClient.MessageType.InformationNotificationMessage:
                // do something...
               break;
            case RongIMClient.MessageType.ContactNotificationMessage:
                // do something...
               break;
            case RongIMClient.MessageType.ProfileNotificationMessage:
                // do something...
               break;
            case RongIMClient.MessageType.CommandNotificationMessage:
                // do something...
               break;
            case RongIMClient.MessageType.CommandMessage:
                // do something...
               break;
            case RongIMClient.MessageType.UnknownMessage:
                // do something...
               break;
            default:
                // do something...
        }
    }
});
```

### 会话接口

* **获取会话列表**

```js
/* 获取会话列表 */
function dialogue_list(){
	RongIMClient.getInstance().getConversationList({
	  onSuccess: function(list) {
	  	console.log(list);
	    // list => 会话列表集合。
	  },
	  onError: function(error) {
	  	console.log("errorcode: " + error);
	     // do something...
	  }
	},null);
}

```

* **删除会话**

```js
/* 删除指定会话 */
function dialogue_del(){
	RongIMClient.getInstance().removeConversation(RongIMLib.ConversationType.PRIVATE, 'userId2', { 
		onSuccess:function(result){ 
            //showResult("删除会话成功",result,start);
		}, 
		onError:function(error){ 
		// error => 清除会话错误码。 
		    //showResult("删除会话失败",error,start);
		} 
	});
}

```

* **删除所有会话**

```js
/* 删除所有会话 */
function dialogue_del_q(){
	RongIMClient.getInstance().clearConversations({
	    onSuccess:function(){
	      // 清除会话成功
	    },
	    onError:function(error){
	      // error => 清除会话错误码。
	    }
	});
}
```

## 服务端

* **注册加入融云token**
参考这篇文章，[https://github.com/rongcloud/server-sdk-php](https://github.com/rongcloud/server-sdk-php),到时候集成到注册当中就行了；