# 即时通讯

## 简介

即时通讯服务是 MaxLeap 研发组件中的重要基础组件，可以为开发者提供实时聊天、群聊、好友关系管理等功能，支持语音、图片等附件信息。

`MaxIMLib` 是不含界面的基础 IM 通讯能力库，封装了通信能力和会话、消息等对象。引用到 App 工程中后，需要开发者自己实现 UI 界面，相对较轻量，适用于对 UI 有较高订制需求的开发者。

## 安装

1. 安装 SDK

    <a class="download-sdk" href="https://github.com/MaxLeap/SDK-Android/releases" target="_blank">下载 MaxLeap SDK</a>

    解压后将 `maxleap-imlib-xxx.jar` 包导入工程的 `libs` 目录下。

2. 配置权限

    ```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    ```

3. 添加依赖

    ```gradle
    compile('io.socket:socket.io-client:0.6.3') {
        exclude group: 'org.json', module: 'json'
    }
    compile "com.squareup.okhttp3:okhttp:3.1.2"
    ```


## MLParrot

### 获得 MLParrot 实例

`MLParrot` 是 IMLib 的管理类，所有的相关方法都可以通过 `MLParrot` 的实例进行调用。`MLParrot` 有以下两种方式可以用于获得实例：

第一种方式，使用这种方式所有回调都会发生在 UI 线程上。通常情况下应该总是使用这种方式。

```java
MLParrot parrot = MLParrot.getInstance();
```

第二种方式，使用这种方式所有回调都会发生在自定义的 Handler 所对应的线程上。

```java
MLParrot parrot = MLParrot.getInstance(handler);
```

### MLParrot 的方法

`MLParrot` 提供的方法主要有两种：

1. `onXXX()` 或 `offXXX()` 开头的基于 Socket 的方法以及其它基于 Http 请求的方法。在使用基于 Socket 的方法时请注意在调用 `onXXX()` 开启功能后及时调用 `offXXX()` 进行关闭。（例: 在 Activity 的 `onResume()` 时调用 `onMessage()` 以便在前台时获得实时消息，在 `onPause()` 时调用 `offMessage()` 在后台时关闭实时消息的获取。）

2. MLParrot 同时支持 Fluent API，所以你可以使用这样的方法进行链式调用：`parront.methodA().methodB().methodC()`。

## 登录

IMLib 支持两种方式进行登录，用户自定的账户系统或者基于 MaxLeap 系统的 MLUser。

在调用具体的登录方法前需要先指定使用哪种方式进行登录。

### 使用用户已有账号系统

**1、作为普通用户登录**

```java
parrot.initWithCustomAccount("your application id", "your api key", clientId, installId);
```

其中 `installId` 主要用于推送，如果不需要推送的话可以直接传 `null`。

**2、作为游客登录**

```java
parrot.initWithGuest("your application id", "your api key", clientId, jsonExtras);
```

其中 `clientId` 可以为空，此时系统会自动分配 ID，如果指定的话则强制使用指定的 ID 作为游客的身份识别。最后一个参数为 JSON 形式的对象，代表登录游客的自定义属性，可以为空。

### 使用 MaxLeap 账号系统

**1、使用用户名和密码验证登录**

```java
parrot.initWithMLUser("your application id", "your api key",  username, password);
```

以上传入的参数为已经注册过的 `MLUser` 的用户名和密码。

**2、使用手机号和短信验证码登录**

先使用 MaxLeap 核心库获得手机验证码

```java
MLUserManager.requestLoginSmsCodeInBackground("your phone number", new RequestSmsCodeCallback() {
    @Override
    public void done(MLException e) {
    }
});
```

然后再进行初始化操作

```java
parrot.initWithPhoneNumber("your application id", "your api key",  phoneNumber, verifyCode);
```

**3、使用第三方平台认证信息登录**

```java
parrotAndroid.initWithAuth(App.APPLICATION_ID, App.API_KEY, authDataJSON);
```

以上 `authDataJSON` 是 MaxLeap 系统的 `MLUser` 对应的 `authData`。

### 创建连接


指定好登录的方式后就可以通过以下方式建立连接，如果使用自定义的账户系统则登录成功返回的就是传入的 `clientId`，如果使用 `MLUser` 则返回的是该 `MLUser` 的 `objectId`。

```java
parrot.login(new DataHandler<String>() {
    @Override
    public void onSuccess(String id) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

登录成功后就可以进行后续的操作了。


## 消息

在 `IMLib` 中 `Message` 代表消息。一个 `Message` 通常有两部分组成: 代表消息来源或目标的 `MessageSource`，代表消息内容的 `MessageContent`。

`Message` 可以通过构造方法创建，但更好的方式是使用 `MessageBuilder` 来创建。
`MessageSource` 可以用于判断消息来自好友还是群组还是聊天室。
`MessageContent` 用于构建消息的内容和类型，目前共有四种类型的消息：`text`, `image`, `audio`, `video`。

创建一个消息对象

```java
Message msg = MessageBuilder.newBuilder()
                .to(targetFriend)
                .text(message);
```

## 单聊

单聊即单个用户之间的点对点聊天系统。单聊的前提是需要先将对方添加为好友，可以查找好友关系管理一节。

### 发送消息

```java
Message msg = MessageBuilder.newBuilder()
                .to(targetFriend)	// 目标对象的 ClientId
                .text(message);
parrot.sendMessage(msg, new SimpleDataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }
});
```

### 接收消息

```java
parrot.onMessage(new SimpleDataHandler<Message>() {
	@Override
    public void onSuccess(Message message) {
        if (message.getFrom().fromFriend()) {
        	//	do something
        }
    }
});
```

接收到消息后可以通过 `message.getType()` 获得消息的类型以判断消息来自哪里。消息一共有以下三种类型：`MessageSource.TYPE_FRIEND`, `MessageSource.TYPE_GROUP`, `MessageSource.TYPE_ROOM`。也可以通过 `fromFriend()`, `fromGroup()`, `fromRoom()` 进行判断。

## 群组

### 发送消息

与单聊基本一样，只是创建 `Message` 时使用的是 `toGroup()`。

```java
Message msg = MessageBuilder.newBuilder()
                .toGroup(targetGroup)	// 目标的 Group 的 Id
                .text(message);
parrot.sendMessage(msg, new SimpleDataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }
});
```

### 接收消息

与单聊基本一样，只是接收时的判断条件变成了 `fromGroup()`。

```java
parrot.onMessage(new SimpleDataHandler<Message>() {
	@Override
    public void onSuccess(Message message) {
        if (message.getFrom().fromGroup()) {
        	//	do something
        }
    }
});
```

## 聊天室

聊天室很像群组，但是聊天室没有离线消息也没有历史记录。

### 发送消息

与单聊基本一样，只是创建 `Message` 时使用的是 `toRoom()`。


```java
Message msg = MessageBuilder.newBuilder()
                .toRoom(targetRoom)	// 目标的 Room 的 Id
                .text(message);
parrot.sendMessage(msg, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
        toastP("sendMessage() success");
    }

    @Override
    public void onError(ParrotException e) {
        toastE("sendMessage()", e);
    }
});
```

### 接收消息

与单聊基本一样，只是接收时的判断条件变成了 `fromRoom()`。

```java
parrot.onMessage(new SimpleDataHandler<Message>() {
	@Override
    public void onSuccess(Message message) {
        if (message.getFrom().fromRoom()) {
        	//	do something
        }
    }
});
```

## 系统消息

### 发送系统消息

```java
parrot.sendSystemMessage(targetFriend, message, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```


### 接收系统消息

```java
parrot.onSystemMessage(new SimpleDataHandler<Message>() {
	@Override
    public void onSuccess(Message message) {
    }
});
```

## 在线状态管理

### 监控上线事件

```java
parrot.onFriendOnline(new SimpleDataHandler<String>() {
    @Override
    public void onSuccess(String friend) {
    }
})
```

### 监控下线事件

```java
parrot.onFriendOffline(new SimpleDataHandler<String>() {
    @Override
    public void onSuccess(String friend) {
    }
})
```

## 好友关系管理

### 添加好友

在发送消息前你需要先添加对方为朋友，此时可以调用以下方法：

```java
parrot.addFriend("friend's client id", new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 删除好友

```java
parrot.removeFriend("friend's client id", new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 获得指定好友的信息

可以用于查询指定好友现在是否在线

```java
parrot.getFriendInfo("friend's client id", new DataHandler<FriendInfo>() {
    @Override
    public void onSuccess(FriendInfo friendInfo) {
        tvStatus.setText(friendInfo.isOnline() ? "Online" : "Offline");
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 获取所有好友

可以用于查询所有好友的在线状态

```java
parrot.listFriends(new DataListHandler<Friend>() {
    @Override
    public void onSuccess(List<Friend> friends) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

以上方法返回的是 `Friend` 对象的列表。`Friend` 有两个属性：`id` 代表着该好友的 `client id`，`online` 代表着该好友当前是否在线。

## 群组关系管理

### 获得所有群组

```java
parrot.listGroups(new DataListHandler<Group>() {
    @Override
    public void onSuccess(List<Group> t) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 获得指定群组的信息

```java
parrot.getGroupInfo("group id", new DataHandler<Group>() {
    @Override
    public void onSuccess(Group group) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 创建群组

```java
parrot.createGroup("group name",
    Arrays.asList("member client id 1", "member client id 2"),
    new DataHandler<Group>() {
        @Override
        public void onSuccess(Group group) {
        }

        @Override
        public void onError(ParrotException e) {
        }

    });
```

### 删除群组

```java
parrot.deleteGroup("group id", new DataHandler<Void>() {
        @Override
        public void onSuccess(Void aVoid) {
        }

        @Override
        public void onError(ParrotException e) {
        }
    });
```

### 更新群组

```java
Group group = new Group();
group.setId("group id");
group.setOwner("owner");
group.setName("group name");
group.setMembers(Arrays.asList("member client id 1", "member client id 2"));
parrot.updateGroup(group, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 添加成员

```java
parrot.addGroupMembers("group id", newMembers, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 移除成员

```java
parrot.removeGroupMembers("group id", removedMembers, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

## 聊天室关系管理

### 获得所有聊天室

```java
parrot.listRooms(new DataListHandler<Room>() {
    @Override
    public void onSuccess(List<Room> room) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 获得指定聊天室的信息

```java
parrot.getRoomInfo("room id", new DataHandler<Room>() {
    @Override
    public void onSuccess(Room room) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 创建聊天室

```java
parrot.createRoom("room name",
    Arrays.asList("member client id 1", "member client id 2"),
    new DataHandler<Room>() {
        @Override
        public void onSuccess(Room room) {
        }

        @Override
        public void onError(ParrotException e) {
        }

    });
```

### 删除聊天室

```java
parrot.deleteRoom("room id", new DataHandler<Void>() {
        @Override
        public void onSuccess(Void aVoid) {
        }

        @Override
        public void onError(ParrotException e) {
        }
    });
```


### 更新聊天室

```java
parrot.updateRoom(room, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 添加成员

```java
parrot.addRoomMembers("room id", newMembers, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 移除成员

```java
parrot.removeRoomMembers("group id", removedMembers, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```


## 自定义属性

用户，群组和聊天室在使用时可以任意设置自定义属性。自定义的属性本身是一个普通的 JSONObject 对象。

### 用户

创建或更新自定义属性

```java
parrot.saveOrUpdateUserExtras(jsonExtras, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

创建自定义属性（会覆盖原来设置的所有自定义属性）

```java
parrot.saveUserExtras(jsonExtras, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

获取自定义属性

```java
parrot.getUserExtras(new DataHandler<JSONObject>() {
    @Override
    public void onSuccess(JSONObject jsonObject) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

删除自定义属性

```java
parrot.deleteUserExtras(new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```


### 群组

创建或更新自定义属性

```java
parrot.saveOrUpdateGroupExtras(groupId, jsonExtras, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

创建自定义属性（会覆盖原来设置的所有自定义属性）

```java
parrot.saveGroupExtras(groupId, jsonExtras, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

获取自定义属性

```java
parrot.getGroupExtras(groupId, new DataHandler<JSONObject>() {
    @Override
    public void onSuccess(JSONObject jsonObject) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

删除自定义属性

```java
parrot.deleteGroupExtras(groupId, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```


### 聊天室

创建或更新自定义属性

```java
parrot.saveOrUpdateRoomExtras(roomId, jsonExtras, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

创建自定义属性（会覆盖原来设置的所有自定义属性）

```java
parrot.saveRoomExtras(roomId, jsonExtras, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

获取自定义属性

```java
parrot.getRoomExtras(roomId, new DataHandler<JSONObject>() {
    @Override
    public void onSuccess(JSONObject jsonObject) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

删除自定义属性

```java
parrot.deleteRoomExtras(roomId, new DataHandler<Void>() {
    @Override
    public void onSuccess(Void aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

## 游客管理

### 创建游客

游客在创建时可用直接指定 ID 和自定义属性，两者都是可选项。如果指定了游客 ID 则系统会以 ID 为准创建或更新游客，如果没有指定 ID 则系统会自行分配 ID 并返回。

```java
parrot.createOrUpdateGuest("guest id", jsonExtras, new DataHandler<String>() {
    @Override
    public void onSuccess(String id) {

    }

    @Override
    public void onError(ParrotException e) {

    }
});
```

### 获得游客信息

```java
parrot.getGuestInfo("guest id", new DataHandler<JSONObject>() {
    @Override
    public void onSuccess(JSONObject jsonObject) {

    }

    @Override
    public void onError(ParrotException e) {

    }
});
```

### 获取游客历史消息

```java
parrot.recentGuestMessages("guest id", ts, limit, new DataListHandler<MessageHistory>() {
    @Override
    public void onSuccess(List<MessageHistory> t) {

    }

    @Override
    public void onError(ParrotException e) {

    }
});
```

## 搜索支持

搜索功能主要有 4 个参数，query, sort, skip 和 limit。
其中 skip 和 limit 为 int 类型数值，主要用于分页。
sort 用于进行排序，值为需要使用的属性的字符串序列。单独指定属性名表示顺序，在属性名前加上 `-` 表示逆序，多个条件之间使用 `,` 分隔（如：`-size,-ts` 表示先按 size 逆序再按 ts 逆序）。
query 表示查询条件，主要用于查询自定义属性，值为普通的 Map 类型（如 foo:bar 表示查询拥有自定义属性为 foo 且 foo 值为 bar 的对象）。

### 搜索用户

```java
Map<String, String> query = new HashMap<String, String>();
query.put("foo", "bar");
String sort = "-size,-ts";
int skip = 0;
int limit = 10;
parrot.searchUsers(query, sort, skip, limit, new DataListHandler<User>() {
    @Override
    public void onSuccess(List<User> users) {

    }

    @Override
    public void onError(ParrotException e) {

    }
});
```

### 搜索群组

```java
Map<String, String> query = new HashMap<String, String>();
query.put("foo", "bar");
String sort = "-size,-ts";
int skip = 0;
int limit = 10;
parrot.searchGroups(query, sort, skip, limit, new DataListHandler<Group>() {
    @Override
    public void onSuccess(List<Group> groups) {

    }

    @Override
    public void onError(ParrotException e) {

    }
});
```

### 搜索聊天室

```java
Map<String, String> query = new HashMap<String, String>();
query.put("foo", "bar");
String sort = "-size,-ts";
int skip = 0;
int limit = 10;
parrot.searchRooms(query, sort, skip, limit, new DataListHandler<Room>() {
    @Override
    public void onSuccess(List<Room> rooms) {

    }

    @Override
    public void onError(ParrotException e) {

    }
});
```

## 多媒体支持

除了基本的文字聊天，`IMLib` 也支持多媒体聊天。在使用多媒体聊天时你需要先调用上传接口将多媒体文件上传到服务器上，为了聊天的实时性，请先在上传前严格控制文件的大小。

上传文件

```java
parrot.uploadFile(sd.getFile(), new DataHandler<String>() {
    @Override
    public void onSuccess(String url) {
        toastP("success");
    }

    @Override
    public void onError(ParrotException e) {
        toastE("Upload File", e);
    }
});
```

以上方法返回的是上传的文件的路径。

上传完成后可以调用以下方法构建多媒体消息，之后可以像之前一样调用 `sendMessage()` 方法发送消息。

```java
Message msg = MessageBuilder.newBuilder()
            .to(targetFriend)
            .image(url);	// 上传后返回的路径
```

除了图片，消息也支持音频，视频文件，只需要将 `image()` 替换成 `audio()` 或 `video()` 即可。


## 多设备同步

`IMLib` 支持多设备同步，即你在 A 设备发送的消息在 B 设备也能同样受到。为了实现此功能，你需要调用以下方法：

```java
parrot.onSelfMessage(new DataHandler<Message>() {
	@Override
    public void onSuccess(Message message) {
        toastP("onSelfMessage() success");
    }

    @Override
    public void onError(ParrotException e) {
        toastE("onSelfMessage()", e);
    }
});
```

## 聊天记录

### 获得与指定好友的聊天记录

获得的聊天记录按时间顺序从小到大排列

```java
parrot.recentMessages("friend's client id",
    timestamp,
    limit,
    new DataListHandler<MessageHistory>() {
        @Override
        public void onSuccess(List<MessageHistory> messageHistory) {
        }

        @Override
        public void onError(ParrotException e) {
        }
    });
```

### 获得指定群组的聊天记录

获得的聊天记录按时间顺序从小到大排列

```java
parrot.recent GroupMessages("group id",
    timestamp,
    limit,
    new DataListHandler<MessageHistory>() {
        @Override
        public void onSuccess(List<MessageHistory> messageHistory) {
        }

        @Override
        public void onError(ParrotException e) {
        }
    });
```


## 离线推送

离线推送需要依赖于 `maxleap-core-xxx.jar` 包，具体配置步骤可以参见 Marketing 的 LPNS 章节。

## 登出与释放资源

### 登出

如果继承了离线推送功能后，应用退出后就能够接受到推送的离线消息。如果不希望接受到推送的消息则需要使用登出功能。

```xml
parrot.logOut(installId, new DataHandler<Void>() {
   @Override
    public void onSuccess(List<Void> aVoid) {
    }

    @Override
    public void onError(ParrotException e) {
    }
});
```

### 释放资源

当用户退出应用后，应确保调用以下方法释放资源。

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    MLParrot.getInstance().destroy();
}
```

以上方法会关闭 Socket 连接并停止内部线程池，在调用后如果试图再次发送请求会导致线程池的错误。
所以以上方法应该只在退出应用时调用，而不是在每个 Activity 的 `onDestroy()` 中调用。


