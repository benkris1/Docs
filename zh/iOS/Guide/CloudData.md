# 数据存储

## 简介

### 什么是数据存储服务
 Cloud Data 是 MaxLeap 提供的数据存储服务，它建立在对象`MLObject`的基础上，每个`MLObject`包含若干键值对。所有`MLObject`均存储在 MaxLeap 上，您可以通过 iOS/Android Core SDK 对其进行操作，也可在 Console 中管理所有的对象。此外 MaxLeap 还提供一些特殊的对象，如`MLUser`(用户)，`MLFile`(文件)，`MLGeoPoint` (地理位置)，他们都是基于 `MLObject` 的对象。

### 为何需要数据存储服务
 Cloud Data 将帮助您解决数据库基础设施的构建和维护，从而专注于实现真正带来价值的应用业务逻辑。其优势在于：

* 解决硬件资源的部署和运维
* 提供标准而又完整的数据访问API
* 不同于传统关系型数据库，向云端存储数据无需提前建表，数据对象以 JSON 格式随存随取，高并发访问轻松无压力
* 可可结合代码托管服务，实现云端数据的 Hook （详见 [MaxLeap 云代码](ML_DOCS_LINK_PLACEHOLDER_USERMANUAL#CLOUD_CODE_ZH)）

## 准备

如果您尚未安装 SDK，请先查阅[快速入门指南](ML_DOCS_LINK_PLACEHOLDER_SDK_QUICKSTART_IOS)，安装 SDK 并使之在 Xcode 中运行。
您还可以查看我们的 [API 参考](ML_DOCS_LINK_PLACEHOLDER_API_REF_IOS)，了解有关我们 SDK 的更多详细信息。

**注意**：我们支持 iOS 7.0 及以上版本。

## Cloud Object

存储在  Cloud Data 的对象称为 `MLObject`，而每个 `MLObject` 被规划至不同的 `class` 中（类似“表”的概念)。`MLObject` 包含若干键值对，且值为兼容 JSON 格式的数据。考虑到数据安全，MaxLeap 禁止客户端修改数据仓库的结构。您需要预先在 MaxLeap 开发者平台上创建需要用到的表，然后仔细定义每个表中的字段和其值类型。

### 新建

假设我们要保存一条数据到 `Comment` 类(数据表)，它包含以下属性：

属性名|值|值类型
-------|-------|-----
content|"我很喜欢这条分享"|字符串
pubUserId|1314520|数字
isRead|false|布尔

我们建议使用驼峰式命名法来命名类名和字段名（如：NameYourclassesLikeThis(类名首字母大写), nameYourKeysLikeThis(列名首字母小写)），让代码看起来整齐美观。

首先，需要在云端数据仓库中添加 `Comment` 类，才能够往里面插入数据。
有关添加类等操作的说明，请查阅：[控制台用户手册 － 云数据](ML_DOCS_LINK_PLACEHOLDER_USERMANUAL#CLOUD_DATA_ZH)


`MLObject` 接口与 `NSMutableDictionary` 类似，但多了 `saveInBackground` 方法。现在我们保存一条 `Comment`:

```objective_c
MLObject *myComment = [MLObject objectWithClassName:@"Comment"];
myComment[@"content"] = @"我很喜欢这条分享";
myComment[@"pubUserId"] = @1314520;
myComment[@"isRead"] = @NO;
[myComment saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        // myComment save succeed
    } else {
        // there was an error
    }
}];
```

该代码运行后，您可能想知道是否真的执行了相关操作。为确保数据正确保存，您可以在 MaxLeap 开发中心查看应用中的数据浏览器。您应该会看到类似于以下的内容：

```
objectId: "xWMyZ4YEGZ", content: "我很喜欢这条分享", pubUserId: 1314520, isRead: false,
createdAt:"2011-06-10T18:33:42Z", updatedAt:"2011-06-10T18:33:42Z"
```

注意：

* **Comment表何时创建:** 出于数据安全考虑，MaxLeap 禁止客户端建表，所以在保存这条数据之前，必须先在开发者中心创建 Comment 这个表。
* **表中同一属性值类型一致:** 新建 comment 对象时，对应属性的值的数据类型要和创建该属性时一致，否则保存数据将失败。
* **客户端无法修改后端数据结构：** 例如，如果 Comment 表中没有 `isRead` 这个字段，那么保存将会失败
* **内建的属性:** 每个 MLObject 对象有以下几个字段是不需要开发者指定的。这些字段的创建和更新是由系统自动完成的，请不要在代码里使用这些字段来保存数据。

属性名|值|
-------|-------|
`objectId`|对象的唯一标识符
`createdAt`|对象的创建时间
`updatedAt`|对象的最后修改时间

* **大小限制：** `MLObject` 的大小被限制在128K以内。
* 键的名称可以由英文字母、数字和下划线组成，但必须以字母开头，值的类型可为字符, 数字, 布尔, 数组或是 `MLObject`，为支持 JSON 编码的类型即可.

### 检索

##### 获取 `MLObject`

您可以通过某条数据的 `objectId`, 获取这条数据的完整内容:

```objective_c
MLQuery *query = [MLQuery queryWithClassName:@"Comment"];
[query getObjectInBackgroundWithId:@"objectId" block:^(MLObject *object, NSError *error) {
    // Do something with the returned MLObject in the myComment variable.
    NSLog(@"%@", myComment);
}];
// The InBackground methods are asynchronous, so any code after this will run
// immediately.  Any code that depends on the query result should be moved
// inside the completion block above.
```

##### 获取 `MLObject` 属性值

要从检索到的 `MLObject` 实例中获取值，您可以使用 `objectForKey:` 方法或 `[]` 操作符：

```objective_c
int pubUserId = [[myComment objectForKey:@"pubUserId"] intValue];
NSString *content = myComment[@"content"];
BOOL pubUserId = [myComment[@"cheatMode"] boolValue];
```

有三个特殊的值以属性的方式提供：

```objective_c
NSString *objectId = myComment.objectId;
NSDate *updatedAt = myComment.updatedAt;
NSDate *createdAt = myComment.createdAt;
```

若需要刷新已有对象，可以调用 `-fetchInBackgroundWithBlock:` 方法：

```
[myObject fetchInBackgroundWithBlock:^(MLObject *object, NSError *error) {
    // object 就是使用服务器数据填充后的 myObject
}];
```

### 更新

更新 `MLObject` 需要两步：首先获取需要更新的 `MLObject`，然后修改并保存。

```objective_c
// 根据 objectId 获取 MLObject
MLObject *object = [MLObject objectWithoutDataWithClassName:@"Comment" objectId:@"objectId"];
[object fetchInBackgroundWithBlock:^(MLObject *myComment, NSError *error) {
    // Now let's update it with some new data. In this case only isRead will get sent to the cloud
    myComment[@"isRead"] = @YES;
    [myComment saveInBackgroundWithBlock:nil];
}];
// The InBackground methods are asynchronous, so any code after this will run
// immediately.  Any code that depends on the query result should be moved
// inside the completion block above.
```

客户端会自动找出被修改的数据，所以只有 “dirty” 字段会被发送到服务器。您不需要担心其中会包含您不想更新的数据。

### 删除对象

##### 删除 `MLObject`

```objective_c
[myComment deleteInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        //
    } else {
 	     // there was an error
	}
}];
```
##### 批量删除 `MLObject`

```
[MLObject deleteAllInBackground:objectsToDelete block:^(BOOL succeeded, NSError *error) {
	 if (succeeded) {
    	//
    } else {
   	   // there was an error
    }
}];
```

##### 删除 `MLObject` 实例的某一属性

除了完整删除一个对象实例外，您还可以只删除实例中的某些指定的值。请注意只有调用 `-saveInBackgroundWithBlock:` 之后，修改才会同步到云端。

```objective_c
// After this, the content field will be empty
[myComment removeObjectForKey:@"content"];
// Saves the field deletion to the MaxLeap
[myComment saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        //
    } else {
        // there was an error
    }
}];
```

### 计数器
计数器是应用常见的功能需求之一。当某一数值类型的字段会被频繁更新，且每次更新操作都是将原有的值增加某一数值，此时，我们可以借助计数器功能，更高效的完成数据操作。并且避免短时间内大量数据修改请求引发冲突和覆盖。

比如纪录某用户游戏分数的字段"score"，我们便会频繁地修改，并且当有几个客户端同时请求数据修改时，如果我们每次都在客户端请求获取该数据，并且修改后保存至云端，便很容易造成冲突和覆盖。

##### 递增计数器
此时，我们可以利用`-incrementKey:`(增量为1)，高效并且更安全地更新计数器类型的字段。如，为了更新记录某帖子的阅读次数字段 `readCount`，我们可以使用如下方式：

```objective_c
[myPost incrementKey:@"readCount"];
[myPost saveInBackgroundWithBlock:nil];
```

##### 指定增量
您还可以使用 `-incrementKey:byAmount:` 实现任何数量的递增。注意，增量无需为整数，您还可以指定增量为浮点类型的数值。

##### 递减计数器

要实现递减计数器，只需要向 `-incrementKey:byAmount:` 接口传入一个负数即可：

```objective_c
[myPost incrementKey:@"readCount" byAmount:@(-1)];
[myPost saveInBackgroundWithBlock:nil];
```

### 数组

您可以通过以下方式，将数组类型的值保存至 `MLObject` 的某字段(如下例中的 `tags` 字段)下：

##### 增加至数组尾部
您可以使用 `addObject:forKey:` 和 `addObjectsFromArray:forKey:`向`tags`属性的值的尾部，增加一个或多个值。

```objective_c
[myPost addUniqueObjectsFromArray:@[@"flying", @"kungfu"] forKey:@"tags"];
[myPost saveInBackgroundWithBlock:nil]
```

同时，您还可以通过`-addUniqueObject:forKey:` 和 `addUniqueObjectsFromArray:forKey:`，仅增加与已有数组中所有 item 都不同的值。插入位置是不确定的。

##### 使用新数组覆盖

可以通过 `setObject:forKey:` 方法使用一个新数组覆盖 `tags` 中原有数组：

```
[myPost setObject:@[] forKey:@"tags"]
```

##### 删除某数组字段的值

`-removeObject:forKey:` 和 `-removeObjectsInArray:forKey:` 会从数组字段中删除每个给定对象的所有实例。

请注意 `removeObject:forKey` 与 `removeObjectForKey:` 的区别。 

**注意：Remove 和 Add/AddUnique 必需分开调用保存函数，否则数据不能正常上传和保存。**

### 关系数据

对象可以与其他对象相联系。如前面所述，我们可以把一个 `MLObject` 的实例 a，当成另一个 `MLObject` 实例 b 的属性值保存起来。这可以解决数据之间一对一或者一对多的关系映射，就像数据库中的主外键关系一样。

注：MaxLeap Services 是通过 `Pointer` 类型来解决这种数据引用的，并不会将数据 a 在数据 b 的表中再额外存储一份，这也可以保证数据的一致性。

#### 使用 `Pointer` 实现

例如：一条微博信息会有多条评论。创建一条微博，并添加一条评论，您可以这样写：

```objective_c
// Create the post
MLObject *myPost = [MLObject objectWithClassName:@"Post"];
myPost[@"title"] = @"I'm Hungry";
myPost[@"content"] = @"Where should we go for lunch?";
// Create the comment
MLObject *myComment = [MLObject objectWithClassName:@"Comment"];
myComment[@"content"] = @"Let's do Sushirrito.";
// Add a relation between the Post and Comment
myComment[@"parent"] = myPost;
// This will save both myPost and myComment
[myComment saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        //
    } else {
        // there was an error
    }
}];
```

我们可以使用 `query` 来获取这条微博所有的评论：

```
MLObject *myPost = ...
MLQuery *query = [MLQuery queryWithClassName:@"Comment"];
[query whereKey:@"parent" equalTo:myPost];
[query findObjectsInBackgroundWithBlock:^(NSArray *allComments, NSError *error) {
    // do something with all the comments of myPost
}];
```

您也可以通过 `objectId` 来关联已有的对象：

```objective_c
// Add a relation between the Post with objectId "1zEcyElZ80" and the comment
myComment[@"parent"] = [MLObject objectWithoutDataWithclassName:@"Post" objectId:@"1zEcyElZ80"];
```

默认情况下，当您获取一个对象的时候，关联的 `MLObject` 不会被获取。这些对象除了 `objectId` 之外，其他属性值都是空的，要得到关联对象的全部属性数据，需要再次调用 `fetch` 系方法（下面的例子假设已经通过 `MLQuery` 得到了 `Comment` 的实例）:

```objective_c
MLObject *post = fetchedComment[@"parent"];
[post fetchInBackgroundWithBlock:^(MLObject *post, NSError *error) {
    NSString *title = post[@"title"];
    // do something with your title variable
}];
```

#### 使用 `MLRelation` 实现关联

您可以使用 `MLRelation` 来建模多对多关系。这有点像 List 链表，但是区别之处在于，在获取附加属性的时候，`MLRelation` 不需要同步获取关联的所有 `MLRelation` 实例。这使得 `MLRelation` 比链表的方式可以支持更多实例，读取方式也更加灵活。例如，一个 `User` 可以赞很多 `Post`。这种情况下，就可以用`getRelation()`方法保存一个用户喜欢的所有 Post 集合。为了新增一个喜欢的 `Post`，您可以这样做：

```objective_c
MLUser *user = [MLUser currentUser];
MLRelation *relation = [user relationForKey:@"likes"];
[relation addObject:post];
[post saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
    if (succeeded) {
        //
    } else {
        // there was an error
    }
}];
```

您可以从 `MLRelation` 删除一个帖子，代码如下：

```objective_c
[relation removeObject:post];
```

默认情况下，这种关系中的对象列表不会被下载。您可以将 `[relation query]` 返回的 `MLQuery` 传入 `-[query findObjectsInBackgroundWithBlock:]` 获取 `Post` 列表。代码应如下所示：

```objective_c
MLQuery *query = [relation query];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    if (error) {
        // There was an error
    } else {
        // objects has all the Posts the current user liked.
    }
}];
```

若您只想要 `Post` 的一个子集，可以对 `-[MLRelation query]` 返回的 `MLQuery` 添加额外限制条件：

```objective_c
MLQuery *query = [relation query];
[query whereKey:@"title" hasSuffix:@"We"];
// Add other query constraints.
```

若要了解有关 `MLQuery` 的更多详细信息，请查看本指南的查询部分。`MLRelation` 的工作方式类似于 `MLObject` 的 `NSArray`，因此您能对对象数组进行的任何查询（不含 `includeKey:`）均可对 `MLRelation` 执行。

### 数据类型

目前，我们使用的值的数据类型有 `NSString`、`NSNumber` 和 `MLObject`。MaxLeap 还支持 `NSDate`、`NSData` 和 `NSNull`。

您可以嵌套 `NSDictionary` 和 `NSArray` 对象，以在单一 `MLObject` 中存储具有复杂结构的数据。

一些示例：

```objective_c
NSNumber *number = @42;
NSString *string = [NSString stringWithFormat:@"the number is %@", number];
NSDate *date = [NSDate date];
NSData *data = [@"foo" dataUsingEncoding:NSUTF8StringEncoding];
NSArray *array = @[string, number];
NSDictionary *dictionary = @{@"number": number,
                             @"string": string};

NSNull *null = [NSNull null];

MLObject *bigObject = [MLObject objectWithclassName:@"BigObject"];
bigObject[@"myNumber"] = number;
bigObject[@"myString"] = string;
bigObject[@"myDate"] = date;
bigObject[@"myData"] = data;
bigObject[@"myArray"] = array;
bigObject[@"myDictionary"] = dictionary;
bigObject[@"myNull"] = null;
[bigObject saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
    if (error) {
        // There was an error
    } else {
        // objects has all the Posts the current user liked.
    }
}];
```

我们不建议通过在 `MLObject` 中使用 `NSData` 字段来存储图像或文档等大型二进制数据。`MLObject` 的大小不应超过 128 KB。要存储更多数据，我们建议您使用 `MLFile` 或者 `MLPrivateFile`。更多详细信息请参考本指南的[文件](#文件)部分。

## 文件

### MLFile 的创建和上传

`MLFile` 可以让您的应用程序将文件存储到服务器中，以应对文件太大或太多，不适宜放入普通 `MLObject` 的情况。比如常见的文件类型图像文件、影像文件、音乐文件和任何其他二进制数据（大小不超过 100 MB）都可以使用。

`MLFile` 上手很容易。首先，你要由 `NSData` 类型的数据，然后创建一个 `MLFile` 实例。下面的例子中，我们只是使用一个字符串：

```objective_c
NSData *data = [@"Working at MaxLeap is great!" dataUsingEncoding:NSUTF8StringEncoding];
MLFile *file = [MLFile fileWithName:@"resume.txt" data:data];
```

**注意**，在这个例子中，我们把文件命名为 `resume.txt`。这里要注意两点：

- 您不需要担心文件名冲突。每次上传都会获得一个唯一标识符，所以上传多个文件名为 `resume.txt` 的文件不同出现任何问题。
- 重要的是，您要提供一个带扩展名的文件名。这样 MaxLeap 就能够判断文件类型，并对文件进行相应的处理。所以，若您要储存 PNG 图片，务必使文件名以 .png 结尾。

然后，您可以把文件保存到云中。与 `MLObject` 相同，使用 `-save` 方法。

```objective_c
[file saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
    // Handle success or failure here ...
}];
```

最后，保存完成后，您可以像其他数据一样把 `MLFile` 与 `MLObject` 关联起来：

```objective_c
MLObject *jobApplication = [MLObject objectWithclassName:@"JobApplication"]
jobApplication[@"applicantName"] = @"Joe Smith";
jobApplication[@"applicantResumeFile"] = file;
[jobApplication saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
    // Handle success or failure here ...
}];
```

您可以调用 `-getDataInBackgroundWithBlock:` 重新获取此数据。这里我们从另一 `JobApplication` 对象获取恢复文件：

```objective_c
MLFile *applicantResume = anotherApplication[@"applicantResumeFile"];
[applicationResume getDataInBackgroundWithBlock:^(NSData *data, NSError *err) {
    if (!error) {
        NSData *resumeData = data;
    }
}];
```

##### 图像

通过将图片转换成 `NSData` 然后使用 `MLFile` 就可以轻松地储存图片。假设您有一个名为 `image` 的 `UIImage`，并想把它另存为 `MLFile`：

```objective_c
UIImage *image = ...;
NSData *imageData = UIImagePNGRepresentation(image);
MLFile *imageFile = [MLFile fileWithName:@"image.png" data:imageData];

MLObject *userPhoto = [MLObject objectWithClassName:@"UserPhoto"];
userPhoto[@"imageName"] = @"My trip to Hawaii!";
userPhoto[@"imageFile"] = imageFile;
[userPhoto saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
    // ...
}];
```

您的 `MLFile` 将作为保存操作的一部分被上传到 `userPhoto` 对象。还可以跟踪 `MLFile` 的*上传和下载进度*。

您可以调用 `-getDataInBackgroundWithBlock:` 重新获取此图像。这里我们从另一个名为 `anotherPhoto` 的 `UserPhoto` 获取图像文件：

```objective_c
MLFile *userImageFile = anotherPhoto[@"imageFile"];
[userImageFile getDataInBackgroundWithBlock:^(NSData *imageData, NSError *error) {
    if (!error) {
        UIImage *image = [UIImage imageWithData:imageData];
    }
}];
```

### 进度

使用 `saveInBackgroundWithBlock:progressBlock:` 和 `getDataInBackgroundWithBlock:progressBlock::` 可以分别轻松了解 `MLFile` 的上传和下载进度。例如：

```
NSData *data = [@"MaxLeap is great!" dataUsingEncoding:NSUTF8StringEncoding];
MLFile *file = [MLFile fileWithName:@"resume.txt" data:data];
[file saveInBackgroundWithBlock:^(BOOL succeeded, NSError *error) {
  // 成功或失败处理...
} progressBlock:^(int percentDone) {
  // 更新进度数据，percentDone 介于 0 和 100。
}];
```

您可以用 [REST API](ML_DOCS_LINK_PLACEHOLDER_API_REF_IOS) 删除对象引用的文件。您需要提供主密钥才能删除文件。

如果您的文件未被应用中的任何对象引用，则不能通过 [REST API](ML_DOCS_LINK_PLACEHOLDER_API_REF_IOS) 删除它们。您可以在应用的“设置”页面请求清理未使用的文件。请记住，该操作可能会破坏依赖于访问未被引用文件（通过其地址属性）的功能。当前与对象关联的文件将不会受到影响。

## 查询

我们已经知道如何使用 `getObjectInBackgroundWithId:block:]` 从 MaxLeap 中检索单个 `MLObject`。使用 `MLQuery`，还有其他多种检索数据的方法 —— 您可以一次检索多个对象，设置检索对象的条件等。

### 基本查询

使用 `MLQuery` 查询 `MLObject` 分三步：

1. 创建一个 `MLQuery` 对象，并指定对应的 `leapClassName`；
2. 为 `MLQuery` 添加过滤条件；
3. 执行 `MLQuery`：调用 `findObjectsInBackgroundWithBlock:` 来查询与过滤条件匹配的 `MLObject` 数据。

例如，查询指定人员所发的微博，可以使用 `whereKey:equalTo:` 方法限定键值：

```objective_c
MLQuery *query = [MLQuery queryWithclassName:@"Post"];
[query whereKey:@"publisher" equalTo:@"MaxLeap"];
[query findObjectsInBackgroundWithBlock:^(NSArray *posts, NSError *error) {
    if (!error) {
        // The find succeeded.
        NSLog(@"Successfully retrieved %d posts.", psots.count);
        // Do something with the found objects
        for (MLObject *object in posts) {
            NSLog(@"%@", object.objectId);
        }
    } else {
        // Log details of the failure
        NSLog(@"Error: %@ %@", error, [error userInfo]);
    }
}];
```

`findObjectsInBackgroundWithBlock:` 方法确保不阻塞当前线程并完成网络请求，然后在主线程执行 `block`。

### 查询约束

为了充分利用 `MLQuery`，我们建议使用下列方法添加限制条件。但是，若您更喜欢用 `NSPredicate`，创建 `MLQuery` 时提供 `NSPredicate` 即可指定一系列的限制条件。

```objective_c
NSPredicate *predicate = [NSPredicate predicateWithFormat:
@"publisher = 'MaxLeap'"];
MLQuery *query = [MLQuery queryWithclassName:@"Post" predicate:predicate];
```

支持以下特性：

- 指定一个字段和一个常数的简单比较操作，比如： `=`、`!=`、`<`、`>`、`<=`、`>=` 和 `BETWEEN`
- 正则表达式，如 `LIKE`、`MATCHES`、`CONTAINS` 或 `ENDSWITH`。
- 限定谓语，如 `x IN {1, 2, 3}`。
- 键存在谓语，如 `x IN SELF`。
- `BEGINSWITH` 表达式。
- 带 `AND`、`OR` 和 `NOT` 的复合谓语。
- 带 `"key IN %@", subquery` 的子查询。

不支持以下类型的谓语：

- 聚合操作，如 ANY、SOME、ALL 或 NONE。
- 将一个键与另一个键比较的谓语。
- 带多个 `OR` 子句的复杂谓语。

##### 设置过滤条件

有几种方法可以对 `MLQuery` 可以查到的对象设置限制条件。您可以用 `whereKey:notEqualTo:` 将具有特定键值对的对象过滤出来：

```objective_c
[query whereKey:@"publisher" notEqualTo:@"xiaoming"];
```

您可以给定多个限制条件，只有满足所有限制条件的对象才会出现在结果中。换句话说，这类似于 AND 类型的限制条件。

```objective_c
[query whereKey:@"publisher" notEqualTo:@"xiaoming"];
[query whereKey:@"createdAt" greaterThan:[NSDate dateWithTimeIntervalSinceNow:-3600]];
```

您可以通过设置 `limit` 来限制结果数量。默认结果数量限制为 100，但是 1 到 1000 之间的任意值都有效：

```objective_c
query.limit = 10; // limit to at most 10 results
```

`skip` 用来跳过返回结果中开头的一些条目，配合 `limit` 可以对结果分页：

```
query.skip = 10; // 跳过前 10 条结果
```

如果您想要确切的一个结果，更加方便的方法是使用 `getFirstObjectInBackgroundWithBlock:` 而不是 `findObjectsInBackgroundWithBlock:`。

```objective_c
MLQuery *query = [MLQuery queryWithclassName:@"Post"];
[query whereKey:@"playerEmail" equalTo:@"xiaoming@example.com"];
[query getFirstObjectInBackgroundWithBlock:^(MLObject *object, NSError *error) {
    if (!object) {
        NSLog(@"The getFirstObject request failed.");
    } else {
        // The find succeeded.
        NSLog(@"Successfully retrieved the object.");
    }
}];
```

##### 对结果排序

对于可排序的数据，如数字和字符串，您可以控制结果返回的顺序：

```objective_c
// Sorts the results in ascending order by the createdAt field
[query orderByAscending:@"createdAt"];
// Sorts the results in descending order by the createdAt field
[query orderByDescending:@"createdAt"];
```

一个查询可以使用多个排序键，如下：

```objective_c
// Sorts the results in ascending order by the score field if the previous sort keys are equal.
[query addAscendingOrder:@"score"];
// Sorts the results in descending order by the score field if the previous sort keys are equal.
[query addDescendingOrder:@"username"];
```

##### 设置数值大小约束

对于可排序的数据，你还可以在查询中使用对比：

```objective_c
// Restricts to wins < 50
[query whereKey:@"wins" lessThan:@50];
// Restricts to wins <= 50
[query whereKey:@"wins" lessThanOrEqualTo:@50];
// Restricts to wins > 50
[query whereKey:@"wins" greaterThan:@50];
// Restricts to wins >= 50
[query whereKey:@"wins" greaterThanOrEqualTo:@50];
```

##### 设置返回数据包含的属性

您可以限制返回的字段，通过调用 `selectKeys:` 并传入一个字段数组来实现。若要检索只包含 `score` 和 `playerName` 字段（以及特殊内建字段，如 `objectId`、`createdAt` 和 `updatedAt`）的对象：

```objective_c
MLQuery *query = [MLQuery queryWithclassName:@"Post"];
[query selectKeys:@[@"contents", @"publisher"]];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    // objects in results will only contain the contents and publisher fields
}];
```

稍后，可以通过对返回的对象调用  `fetchIfNeededInBackgroundWithBlock:` 提取其余的字段：

```objective_c
MLObject *object = (MLObject*)results[0];
[object fetchIfNeededInBackgroundWithBlock::^(MLObject *object, NSError *error) {
    // all fields of the object will now be available here.
}];
```

##### 设置更多约束

若您想要检索与几个不同值匹配的对象，您可以使用 `whereKey:containedIn:`，并提供一组可接受的值。这通常在用单一查询替代多个查询时比较有用。例如，如果您检索某几个用户发的微博：

```objective_c
// Finds posts from any of Jonathan, Dario, or Shawn
NSArray *names = @[@"Jonathan Walsh", @"Dario Wunsch", @"Shawn Simon"];
[query whereKey:@"publisher" containedIn:names];
```

若您想要检索与几个值都不匹配的对象，您可以使用 `whereKey:notContainedIn:`，并提供一组可接受的值。例如，如果您想检索不在某个列表里的用户发的微博：

```objective_c
// Finds posts from anyone who is neither Jonathan, Dario, nor Shawn
NSArray *names = @[@"Jonathan Walsh", @"Dario Wunsch", @"Shawn Simon"];
[query whereKey:@"playerName" notContainedIn:names];
```

若您想要检索有某一特定键集的对象，可以使用 `whereKeyExists:`。相反，若您想要检索没有某一特定键集的对象，可以使用 `whereKeyDoesNotExist:`。

您可以使用 `whereKey:matchesKey:inQuery:` 方法获取符合以下要求的对象：对象中的一个键值与另一查询所得结果的对象集中的某一键值匹配。例如，获取用户所有粉丝发的微博：

```objective_c
MLQuery *commentQuery = [MLQuery queryWithClassName:@"Comment"];
[commentQuery whereKey:@"parent" equalTo:post];
MLQuery *postsQuery = [MLQuery queryWithClassName:@"Post"];
[postsQuery whereKey:@"author" matchesKey:@"author" inQuery:postsQuery];
[postsQuery findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    // ...
}];
```

类似地，您可以使用 `whereKey:doesNotMatchKey:inQuery:` 获取不符合以下要求的对象，对象中的一个键值与另一查询所得结果的对象集中的某一键值匹配。

### 不同属性值类型的查询

#### 值类型为数组的查询

对于数组类型的键，您可以查找键的数组值包含 2 的对象，如下所示：

```objective_c
// Find objects where the array in arrayKey contains 2.
[query whereKey:@"arrayKey" equalTo:@2];
```

您还可以查找键数组值包含值 2、3 或 4 的对象，如下所示：

```objective_c
// Find objects where the array in arrayKey contains each of the
// elements 2, 3, and 4.
[query whereKey:@"arrayKey" containsAllObjectsInArray:@[@2, @3, @4]];
```

####值类型为字符串的查询

使用 `whereKey:hasPrefix:` 将结果限制为以某一特定字符串开头的字符串值。与 MySQL `LIKE` 运算符类似，它包含索引，所以对大型数据集很有效：

```objective_c
// Finds barbecue sauces that start with "Big Daddy's".
MLQuery *query = [MLQuery queryWithclassName:@"Post"];
[query whereKey:@"title" hasPrefix:@"Big Daddy's"];
```

#### 值类型为 `MLObject` 查询

##### `MLObject` 类型字段匹配 `MLObject`

有几种方法可以用于关系型数据查询。如果您想检索有字段与某一特定 `MLObject` 匹配的对象，可以像检索其他类型的数据一样使用 `whereKey:equalTo:`。例如，如果每个 `Comment` 在 `parent` 字段中有一个 `Post` 对象，您可以提取某一特定 `Post` 的评论：

```objective_c
// Assume MLObject *myPost was previously created.
MLQuery *query = [MLQuery queryWithClassName:@"Comment"];
[query whereKey:@"post" equalTo:myPost];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    // comments now contains the comments for myPost
}];
```

您还可以用 `objectId` 进行关系型查询：

```objective_c
MLObject *object = [MLObject objectWithoutDataWithClassName:@"Post" objectId:@"1zEcyElZ80"];
[query whereKey:@"parent" equalTo:object];
```

##### `MLObject` 类型字段匹配 `Query`

如果想要检索的对象中，有字段包含与其他查询匹配的 `MLObject`，您可以使用 `whereKey:matchesQuery:`。**注意**，默认限值 100 和最大限值 1000 也适用于内部查询，因此在大型数据集中进行查询时，您可能需要谨慎构建查询条件才能按需要进行查询。为了查找包含图像的帖子的评论，您可以这样：

```objective_c
MLQuery *innerQuery = [MLQuery queryWithClassName:@"Post"];
[innerQuery whereKeyExists:@"image"];
MLQuery *query = [MLQuery queryWithClassName:@"Comment"];
[query whereKey:@"post" matchesQuery:innerQuery];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    // comments now contains the comments for posts with images
}];
```

如果想要检索的对象中，有字段包含与其他查询不匹配的 `MLObject`，您可以使用 `whereKey:doesNotMatchQuery:`。为了查找不包含图像的帖子的评论，您可以这样：

```objective_c
MLQuery *innerQuery = [MLQuery queryWithClassName:@"Post"];
[innerQuery whereKeyExists:@"image"];
MLQuery *query = [MLQuery queryWithClassName:@"Comment"];
[query whereKey:@"post" doesNotMatchQuery:innerQuery];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    // comments now contains the comments for posts without images
}];
```

##### 返回指定 `MLObject` 类型的字段
在一些情况下，您可能想要在一个查询中返回多种类型的相关对象。您可以用 `includeKey:` 方法达到这个目的。例如，假设您要检索最新的十条评论，并且想要同时检索这些评论的相关帖子：

```objective_c
MLQuery *query = [MLQuery queryWithClassName:@"Comment"];
// Retrieve the most recent ones
[query orderByDescending:@"createdAt"];
// Only retrieve the MLt ten
query.limit = 10;
// Include the post data with each comment
[query includeKey:@"post"];
[query findObjectsInBackgroundWithBlock:^(NSArray *comments, NSError *error) {
    // Comments now contains the MLt ten comments, and the "post" field
    // has been populated. For example:
    for (MLObject *comment in comments) {
        // This does not require a network access.
        MLObject *post = comment[@"post"];
        NSLog(@"retrieved related post: %@", post);
    }
}];
```

您也可以使用点标记进行多层级检索。如果您想要包含帖子的评论以及帖子的作者，您可以操作如下：

```objective_c
[query includeKey:@"post.author"];
```

您可以通过多次调用 `includeKey:`，进行包含多个字段的查询。此功能也适用于 `getFirstObjectInBackgroundWithBlock:` 和 `getObjectInBackgroundWithId:block:` 等 `MLQuery` 辅助方法。

### 个数查询

计数查询可以对拥有 1000 条以上数据的类返回大概结果。如果您只需要计算符合查询的对象数量，不需要检索匹配的对象，可以使用 `countObjects`，而不是 `findObjects`。例如，要计算某一特定玩家玩过多少种游戏：

```objective_c
MLQuery *query = [MLQuery queryWithclassName:@"Post"];
[query whereKey:@"publisher" equalTo:@"Sean"];
[query countObjectsInBackgroundWithBlock:^(int count, NSError *error) {
    if (!error) {
        // The count request succeeded. Log the count
        NSLog(@"Sean has played %d games", count);
    } else {
        // The request failed
    }
}];
```

对于含超过 1,000 个对象的类，计数操作受超时设定的限制。这种情况下，可能经常遇到超时错误，或只能返回近似正确的结果。因此，在应用程序的设计中，最好能做到避免此类计数操作。

###复合查询

如果想要查找与几个查询中的其中一个匹配的对象，您可以使用 `orQueryWithSubqueries:` 方法。例如，如果您想要查找赢得多场胜利或几场胜利的玩家，您可以：

```objective_c
MLQuery *fewReader = [MLQuery queryWithClassName:@"Post"];
[fewReader whereKey:@"readCount" lessThan:@10];
MLQuery *lotsOfReader = [MLQuery queryWithClassName:@"Post"];
[lotsOfReader whereKey:@"readCount" greaterThan:@100];
MLQuery *query = [MLQuery orQueryWithSubqueries:@[fewReader, lotsOfReader]];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    // results contains players with lots of wins or only a few wins.
}];
```

您可以给新创建的 `MLQuery` 添加额外限制条件，这相当于 “and” 运算符。

但是，请注意：在混合查询结果中查询时，我们不支持非过滤型限制条件（如 `limit`、`skip`、`orderBy...:`、`includeKey:`）。

###缓存查询

##  `MLObject` 子类

MaxLeap 的设计能让您尽快上手使用。您可以使用 `MLObject` 类访问所有数据，以及通过 `objectForKey:` 或 `[]` 操作符访问任何字段。在成熟的代码库中，子类具有许多优势，包括简洁性、可扩展性和支持自动完成。子类化纯属可选操作，但它会将以下代码：

```objective_c
MLObject *game = [MLObject objectWithclassName:@"Game"];
game[@"displayName"] = @"Bird";
game[@"multiplayer"] = @YES;
game[@"price"] = @0.99;
```

转换为：

```objective_c
Game *game = [Game object];
game.displayName = @"Bird";
game.multiplayer = @YES;
game.price = @0.99;
```

###创建 `MLObject` 子类

创建 `MLObject` 子类的步骤：

1. 声明符合 `MLSubclassing` 协议的子类。
2. 实现子类方法 `MLclassName`。这是您传给 `-initWithclassName:` 方法的字符串，这样以后就不必再传类名了。
3. 将 `MLObject+Subclass.h` 导入您的 .m 文件。该操作导入了 `MLSubclassing` 协议中的所有方法的实现。其中 `MLclassName` 的默认实现是返回类名(指 Objective C 中的类)。
4. 在 `+[MaxLeap setApplicationId:clientKey:]` 之前调用 `+[Yourclass registerSubclass]`。一个简单的方法是在类的 [+load][+load api reference] (Obj-C only) 或者 [+initialize][+initialize api reference] (both Obj-C and Swift) 方法中做这个事情。

下面的代码成功地声明、实现和注册了 `MLObject` 的 `Game` 子类：

```objective_c
// Game.h
@interface Game : MLObject <MLSubclassing>
+ (NSString *)leapClassName;
@end

// Game.m
// Import this header to let Armor know that MLObject privately provides most
// of the methods for MLSubclassing.
#import <MaxLeap/MLObject+Subclass.h>
@implementation Game
+ (void)load {
    [self registerSubclass];
}
+ (NSString *)leapClassName {
    return @"Game";
}
@end
```

### 属性的访问/修改

向 `MLObject` 子类添加自定义属性和方法有助于封装关于这个类的逻辑。借助 `MLSubclassing`，您可以将与同一个主题的所有相关逻辑放在一起，而不必分别针对事务逻辑和存储/传输逻辑使用单独的类。

`MLObject` 支持动态合成器(dynamic synthesizers)，这一点与 `NSManagedObject` 类似。像平常一样声明一个属性，但是在您的 .m 文件中使用 `@dynamic` 而不用 `@synthesize`。下面的示例在 `Game` 类中创建了 `displayName` 属性：

```objective_c
// Game.h
@interface Game : MLObject <MLSubclassing>
+ (NSString *)leapClassName;
@property (retain) NSString *displayName;
@end

// Game.m
@dynamic displayName;
```

现在，您可以使用 `game.displayName` 或 `[game displayName]` 访问 `displayName` 属性，并使用 `game.displayName = @"Bird"` 或 `[game setDisplayName:@"Bird"]` 对其进行赋值。动态属性可以让 Xcode 提供自动完成功能和简单的纠错。

`NSNumber` 属性可使用 `NSNumber` 或其相应的基本类型来实现。请看下例：

```objective_c
@property BOOL multiplayer;
@property float price;
```

这种情况下，`game[@"multiplayer"]` 将返回一个 `NSNumber`，可以使用 `boolValue` 访问；`game[@"price"]` 将返回一个 `NSNumber`，可以使用 `floatValue` 访问。但是，`fireProof` 属性实际上是 `BOOL`，`rupees` 属性实际上是 `float`。动态 `getter` 会自动提取 `BOOL` 或 `int` 值，动态 `setter` 会自动将值装入 `NSNumber` 中。您可以使用任一格式。原始属性类型更易于使用，但是 `NSNumber` 属性类型明显支持 `nil` 值。

### 定义函数

如果您需要比简单属性访问更加复杂的逻辑，您也可以声明自己的方法：

```objective_c

@dynamic iconFile;

- (UIImageView *)iconView {
    MLImageView *view = [[MLImageView alloc] initWithImage:kPlaceholderImage];
    view.file = self.iconFile;
    [view loadInBackground];
    return view;
}
```

### 创建子类的实例

您应该使用类方法 `object` 创建新的对象。这样可以构建一个您定义的类型的实例，并正确处理子类化。要创建现有对象的引用，使用 `objectWithoutDataWithObjectId:`。

### 子类的查询

您可以使用类方法 `query` 获取对特定子类对象的查询。下面的示例查询了用户可购买的装备：

```objective_c
MLQuery *query = [Game query];
[query whereKey:@"rupees" lessThanOrEqualTo:@0.99];
[query findObjectsInBackgroundWithBlock:^(NSArray *objects, NSError *error) {
    if (!error) {
        Game *firstArmor = objects[0];
        // ...
    }
}];
```

## 地理位置

MaxLeap 让您可以把真实的纬度和经度坐标与对象关联起来。通过在 `MLObject` 中添加 MLGeoPoint，可以在查询时实现将对象与参考点的距离临近性纳入考虑。这可以让您轻松某些事情，如找出距离与某个用户最近的其他用户或者距离某个用户最近的地标。

#### MLGeoPoint 字段说明

#### 创建 MLGeoPoint

要将某个地点与对象联系起来，您首先要创建一个 `MLGeoPoint`。例如，要创建一个纬度为 40.0 度，经度为 -30.0 的点：

```objective_c
MLGeoPoint *point = [MLGeoPoint geoPointWithLatitude:40.0 longitude:-30.0];
```

然后，该点被作为常规字段储存在对象中。

```objective_c
placeObject[@"location"] = point;
```

#### 地理位置查询

##### 查询距离某地理位置最近的对象

有了一些具有空间坐标的对象后，找到哪些对象距离某个点最近将会产生很好的效应。这可以通过使用 `whereKey:nearGeoPoint:` 对 `MLQuery` 添加另一限制条件完成。举例而言，找出距离某个用户最近的十个地点的方法如下：

```objective_c
// User's location
MLUser *userObject;
MLGeoPoint *userGeoPoint = userObject[@"location"];

// Create a query for places
MLQuery *query = [MLQuery queryWithClassName:@"PlaceObject"];

// Interested in locations near user.
[query whereKey:@"location" nearGeoPoint:userGeoPoint];

// Limit what could be a lot of points.
query.limit = 10;

// Final list of objects
[query findObjectsInBackgroundWithBlock:^(NSArray *placesObjects, NSError *error) {
    if (error) {
        // there was an error
    } else {
        // do something with placesObjects
    }
}];
```

此时，`placesObjects` 是按照与 `userGeoPoint` 的距离（由近及远）排列的一组对象。注意，若应用另一个 `orderByAscending:`/`orderByDescending:` 限制条件，该限制条件将优先于距离顺序。

##### 查询某地理位置一定距离内的对象

若要用距离来限定获得哪些结果，请使用 `whereKey:nearGeoPoint:withinMiles:`、`whereKey:nearGeoPoint:withinKilometers:` 和 `whereKey:nearGeoPoint:withinRadians:`。

##### 查询一定地理位置范围内对象

您还可以查询包含在特定区域内的对象集合。若要查找位于某个矩形区域内的对象，请将 `whereKey:withinGeoBoxFromSouthwest:toNortheast:` 限制条件添加至您的 `MLQuery`。

```objective_c
MLGeoPoint *swOfSF = [MLGeoPoint geoPointWithLatitude:37.708813 longitude:-122.526398];
MLGeoPoint *neOfSF = [MLGeoPoint geoPointWithLatitude:37.822802 longitude:-122.373962];
MLQuery *query = [MLQuery queryWithclassName:@"PizzaPlaceObject"];
[query whereKey:@"location" withinGeoBoxFromSouthwest:swOfSF toNortheast:neOfSF];
[query findObjectsInBackgroundWithBlock:^(NSArray *pizzaPlacesInSF, NSError *error) {
    if (error) {
        // there was an error
    } else {
        // do something with pizzaPlacesInSF
    }
}];
```

请注意：

1. 每个 `MLObject` 类仅可能有一个带 `MLGeoPoint` 对象的键。
2. 点不应等于或大于最大范围值。纬度不能为 -90.0 或 90.0。经度不能为 -180.0 或 180.0。若纬度或经度设置超出边界，会引起错误。

## 数据安全
每个到达 MaxLeap 云服务的请求是由移动端 SDK，管理后台，云代码或其他客户端发出，每个请求都附带一个 security token。MaxLeap 后台可以根据请求的 security token 确定请求发送者的身份和授权，并在处理数据请求的时候，根据发送者的授权过滤掉没有权限的数据。

具体的介绍及操作方法，请参考 [数据存储-使用指南](ML_DOCS_LINK_PLACEHOLDER_USERMANUAL#CLOUD_DATA_ZH)


[+load api reference]: https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/classes/NSObject_class/#//apple_ref/occ/clm/NSObject/load

[+initialize api reference]: https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/classes/NSObject_class/#//apple_ref/occ/clm/NSObject/initialize

[access control list]: http://en.wikipedia.org/wiki/Access_control_list
[role-based access control]: http://en.wikipedia.org/wiki/Role-based_access_control

[set up a facebook app]: https://developers.facebook.com/apps

[getting started with the facebook sdk]: https://developers.facebook.com/docs/getting-started/facebook-sdk-for-ios/

[facebook permissions]: https://developers.facebook.com/docs/reference/api/permissions/

[facebook sdk reference]: https://developers.facebook.com/docs/reference/ios/current/

[set up twitter app]: https://dev.twitter.com/apps

[twitter documentation]: https://dev.twitter.com/docs

[twitter rest api]: https://dev.twitter.com/docs/api


[weibo_develop_site]: http://open.weibo.com/
[set up weibo app]: http://open.weibo.com/apps/new?sort=mobile
[weibo documentation]: http://open.weibo.com/wiki/%E9%A6%96%E9%A1%B5

[wechat_develop_site]: https://open.weixin.qq.com
[wechat documentation]: https://open.weixin.qq.com/cgi-bin/showdocument?action=dir_list&t=resource/res_list&verify=1&lang=zh_CN

[open_qq_site]: http://open.qq.com/
[set_up_qq_app]: http://op.open.qq.com/appregv2/
[qq_documentation]: http://wiki.open.qq.com/wiki/IOS_API%E8%B0%83%E7%94%A8%E8%AF%B4%E6%98%8E

[maxleap_console]: https://maxleap.cn
