# 融云WEB端口开发之聊天室

标签（空格分隔）： 公司

---

# 接上一篇文章【融云WEB端口开发】

#分析
- [x] 1.服务端先注册用户，然后加入一个token，这样就现成了一个token对应一个用户的概念；
- [x] 2.客户端初始化融云，这样我们的客户端就能实时的接收到其他用户所发过来的消息了；
- [x] 3.这个是另外一个（聊天室接口）开发；主要有 **加入聊天室、聊天室发消息、聊天室发题目、退出聊天室、获取聊天室信息**。(1.先获取聊天室信息【所有用户、消息记录】2.发送消息【所有人收到消息】)


---


# 参考文档

 1. 客户端
  [客户端文档](https://www.rongcloud.cn/docs/web_api_demo.html#introduction)
  [客户端源码下载地址](https://github.com/rongcloud/websdk-demo/tree/master/im)
  [客户端WebSDK demo](https://github.com/rongcloud/websdk-demo)
  [客户端WebIM 集成引导](https://rongcloud.github.io/websdk-demo/integrate/guide.html)
  [消息接口-融云 SDK 消息结构详解](https://www.rongcloud.cn/docs/message_architecture.html#text_message)

 2. 服务端
  [服务端文档](https://www.rongcloud.cn/docs/server.html#open_source_sdk)
  [服务端-PHP源码下载地址](https://github.com/rongcloud/server-sdk-php)

---

# 功能接口

## 客户端

### 聊天室接口
- [ ] 加入聊天室
- [ ] 聊天室发消息
- [ ] 聊天室发题目
- [ ] 退出聊天室
- [ ] 获取聊天室用戶信息


## 服务端
- [x] 用户注册加入融云token值



---

# 日程

 - 20081007 今天又要研究的是聊天室接口，主要是有**加入聊天室、聊天室发消息、聊天室发题目、退出聊天室、获取聊天室用戶信息**

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



### 聊天室接口

* **加入聊天室**
> 加入聊天室之后，同时会获之前取聊天记录

    ```js
    /* 加入聊天室 */
    function chatroom_join(){
    	var chatRoomId = "10086"; // 聊天室 Id。
    	var count = 20;// 拉取最近聊天最多 50 条。
    	RongIMClient.getInstance().joinChatRoom(chatRoomId, count, {
    	  onSuccess: function(message) {
    	       // 加入聊天室成功。
    	       console.log('加入聊天室成功');
    	  },
    	  onError: function(error) {
    	  	console.log('加入聊天室失败');
    	    // 加入聊天室失败
    	  }
    	});
    }
    ```


* **聊天室发消息**
    ```js
    /* 聊天室发消息 */
    function chatroom_sendMsg(){
    	var content = {
    		content:"hello，time：" + new Date().getTime(),
    		extra:[_currentUid,_currentHeadimg,_currentName]
    	};
    
    	var chatRoomId = "10086";// 聊天室 Id。
    	var conversationType = RongIMLib.ConversationType.CHATROOM; // 私聊
    	var msg = new RongIMLib.TextMessage(content);
    	
    	RongIMClient.getInstance().sendMessage(conversationType, chatRoomId, msg, {
            onSuccess: function (message) {
            	console.log("发送聊天室消息成功");
            	console.log(message);
            },
            onError: function (errorCode,message) {
            	console.log("发送聊天室消息失败");
            	console.log(message);
            }
        });
    }
    ```


* **聊天室发题目**



* **退出聊天室**
    ```js
    /* 退出聊天室 */
    function chatroom_quit(){
    	var chatRoomId = "10086"; // 聊天室 Id。
    	RongIMClient.getInstance().quitChatRoom(chatRoomId, {
    	  onSuccess: function() {
    	       // 退出聊天室成功。
    	       console.log('退出聊天室成功');
    	  },
    	  onError: function(error) {
    	    // 退出聊天室失败。
    	    console.log('退出聊天室失败');
    	  }
    	});
    }
    ```


* **获取聊天室用戶信息**
    ```js
    /* 获取聊天室用戶信息 */
    function chatroom_list(){
    	var chatRoomId = "10086";// 聊天室 Id。
    	var count = 20; // 获取聊天室人数 （范围 0-20 ）
    	var order = RongIMLib.GetChatRoomType.REVERSE;// 排序方式。
    	RongIMClient.getInstance().getChatRoomInfo(chatRoomId, count, order, {
    	  onSuccess: function(chatRoom) {
    	  	console.log("获取聊天室信息成功");
    	  	console.log(chatRoom);
    	       // chatRoom => 聊天室信息。
    	    // chatRoom.userInfos => 返回聊天室成员。
    	     // chatRoom.userTotalNums => 当前聊天室总人数。
    	  },
    	  onError: function(error) {
    	  	console.log("获取聊天室信息失败");
    	    // 获取聊天室信息失败。
    	  }
    	});	
    }
    ```


## 服务端

* **注册加入融云token**
参考这篇文章，[https://github.com/rongcloud/server-sdk-php](https://github.com/rongcloud/server-sdk-php),到时候集成到注册当中就行了；