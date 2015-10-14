云代码 SDK支持 JDK6, 7, 8，推荐使用JDK8。

##安装Maven

Eclipse:

1.	点击"Help" >> "Install New Software.."
2.	在"Work with"中输入：`http://download.eclipse.org/technology/m2e/releases`，在列表中选择"Maven Integration for Eclipse"，即可安装Maven插件。

##安装 MaxLeap命令行工具（MaxLeap-CLI）
####下载MaxLeap-CLI
下载地址：[https://github.com/LeapCloud/MaxLeap-CLI/releases](https://github.com/LeapCloud/MaxLeap-CLI/releases)

根据平台选择对应的客户端：

1.  Windows：[maxleap.exe](https://github.com/LeapCloud/MaxLeap-CLI/releases/download/v0.1/maxleap.exe)
2.  Linux/Mac OSX：[maxleapformac.tar](https://github.com/LeapCloud/MaxLeap-CLI/releases/download/v0.1/maxleapformac.tar)

下载完成后，您可直接在终端中使用 MaxLeap-CLI。进入下载目录(macos版本需要解压后使用)，查看MaxLeap-CLI版本

```shell
./maxleap -v
```
    
显示`zcc version 0.1`表示MaxLeap客户端安装成功

*	maxleap命令添加到环境变量

每次执行maxleap命令都需要进入下载安装目录才能执行命令，你可以将maxleap添加到环境变量，这样你可以随时随地使用maxleap了

1.  LINUX和MAC：

    ```
    vim ~/.bash_profile
    ```
    
    编辑profile文件，将MaxLeap安装目录追加到PATH中，比如你的MaxLeap安装目录为`/usr/local/maxleap-cli`
        
    `export PATH=/usr/local/maxleap-cli:$PATH`
    
    最后让profile生效：`source ~/.bash_profile`

2.  WINDOWS：

    //TODO:
	
##	安装SDK

### 使用模板创建 MaxLeap 云代码项目

获取 MaxLeap 云代码 Java项目模板

```shell
git clone https://github.com/LeapCloud/Demo-CloudCode-Java.git
```

### 修改配置
在/src/main/resources/config（请确保此路径存在）中，添加global.json文件，并在其中添加如下配置：

```java
{
	"applicationName" : "HelloWorld",
	"applicationId": "YOUR_APPLICATION_ID",
	"applicationKey": "YOUR_MASTER_KEY",
	"lang" : "java",
	"javaMain": "Main",
	"packageHook" : "YOUR_HOOK_PACKAGE_NAME",
	"packageClasses" : "YOUR_ENTITY_PACKAGE_NAME",
	"version": "0.0.1"
}
```

根据创建应用时获取的key，修改下列键的值：
	
键|值|
------------|-------|
applicationName|MaxLeap应用名称
applicationId|Application ID
applicationKey|Master Key
javaMain|入口main函数类名
packageHook|Hook包名
packageClasses|Class实体包名
version|当前云代码项目版本号

### 定义一个简单的function

```Java
import com.maxleap.code.MLLoader;
import com.maxleap.code.Response;
import com.maxleap.code.impl.GlobalConfig;
import com.maxleap.code.impl.LoaderBase;
import com.maxleap.code.impl.Response;

public class Main extends LoaderBase implements Loader {
    @Override
    public void main(GlobalConfig globalConfig) {

    	//定义Cloud Function
        defineFunction("hello", request -> {
            Response<String> response = new Response<String>(String.class);
            response.setResult("Hello, " + request.parameter(Map.class).get("name") + "!");
            return response;
        });
    }
}
```

注意：

* Main class的main method是云代码项目启动的入口（在global.json中指定），需要继承LoaderBase并实现Loader接口，在main方法中需要注册所有的cloud function和job。

### 打包

在当前项目根目录下运行Maven命令：

`mvn package`

我们将在项目根目录下的target文件夹中发现 *xxx-1.0-SNAPSHOT-mod.zip* 文件，这便是我们想要的package.

## 云代码的上传及部署
1. 登录：`maxleap login <UserName> -region <CN or US ...>`
2. 选择所要部署的目标应用，作为后续操作的上下文：`maxleap use <AppName>` ,如果你不记得你的AppName，可以通过`maxleap apps`来枚举你的所有应用列表
3. 上传Package： `maxleap upload <PackageLocation>`
4. 部署云代码：`maxleap deploy <VersionNumber>`

**注意：**

*	这里的VersionNumber定义在您云代码项目中的global.json文件中（version字段的值）
* 	若您在部署之前，已经部署过某个版本的云代码，需要先卸载该版本的云代码，才能部署新版本。
*	使用`maxleap help`来获取所有相关命令帮助，你也可以查看[lcc使用向导](ML_DOCS_GUIDE_LINK_PLACEHOLDER_JAVA)，以获取lcc的更多信息。

### 测试

通过 curl，我们向部署好的Cloud Function发送如下POST请求，以测试我们的Function是否部署成功：

```shell
curl -X POST \
-H "X-ML-AppId: YOUR_APPID" \
-H "X-ML-APIKey: YOUR_APIKEY" \
-H "Content-Type: application/json" \
-d '{"name":"David Wang"}' \
https://api.leap.as/functions/hello
```
此时，我们将得到如下结果：

```shell
Hello, David Wang!
```
表明测试通过，部署成功。

注意:

* X-ML-APIKey的值为应用的API KEY，而非云代码项目中使用的Master Key.

## 下一步
 至此，您已经完成 MaxLeap SDK 的安装与必要的配置。请移步至[云代码 SDK使用教程](ML_DOCS_GUIDE_LINK_PLACEHOLDER_JAVA)以获取 MaxLeap 云代码 SDK 的详细功能介绍以及使用方法。