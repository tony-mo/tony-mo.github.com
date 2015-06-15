---
layout: post
title: "iOS平台编写Cordova插件"
date: 2015-06-08 15:20:20 +0800
comments: true
categories:  
---


> Cordova提供的功能能够满足一般应用，但是对于复杂的应用或者对性能要求比较严格的应用来说，并不是很理想的。所以就需要在某些场景下自己写代码来弥补这些不足，Cordova也提供了Plugin功能。Cordova本身访问Native接口都是通过Plugin的方式提供的，可以参考官方Plugin代码，而且GitHub上也存在不少开源的Cordova Plugin，所以这些都是最好的教程。

> Plugin的分类大概有两种：
 
>JavaScript-only Plugin：不需要写Native代码，不依赖平台的共通的JS代码

>Native Plugin：弥补Cordova提供的功能以外的Native调用，依赖各个平台写不同的Native代码

引言出自[Fish Where The Fish Are](http://rensanning.iteye.com/blog/2029362)

文章这里只介绍Native Plugin在iOS平台下的编写。日后或许再添加JavaScript-only Plugin以及安卓平台下的插件编写:)

***

###如何在iOS平台下编写Cordova插件
(文章末尾有提供安卓平台下编写Cordova插件连接)

*由于文章篇幅有点长，所以建议大家可以参考插件的文件目录结构以及各文件内容，边看边拷贝文件内容，快速编写一个插件demo出来，首先有个大概的了解。毕竟纸上得来终觉浅，看完了不写，相当于没看。在2.各文件内容以及作用里面，会详细介绍配置文件以及各元素的作用。*



####1.<a name="file_structure">插件的文件目录结</a>

在桌面或者你认为适合放插件的文件夹下，新建文件夹。新建的文件夹的名称是该插件的id，格式为:com.xxxxxxxx.xxx。

比如，我在桌面新建文件夹，命名为:com.technologystudios.logout

![file_name](http://f.cl.ly/items/1Z3a1n1V2n1n3F1v0y0n/plugin_name.png)

在新建名称为com.technologystudios.logout的文件夹后，依次新建以下的文件(com.technologystudios.logout不必再新建，只需新建README.md，fetch.json，package.json，plugin.xml，src和www文件夹以及下级的文件，其中package.json,plugin.xml,src,www为必须的文件)

新建后的文件目录结构(这只是个人新建插件文件目录结构的习惯，并非标准。)

	com.technologystudios.logout
	├── README.md
	├── fetch.json
	├── package.json
	├── plugin.xml
	├── src
	│   └── ios
	│       ├── CDVLogoutPlugin.h
	│       └── CDVLogoutPlugin.m
	└── www
	    └── logoutplugin.js
	
####2.各文件内容以及作用

#####README.md文件
作为帮助文件的存在，内容描述的是插件如何使用以及定义返回的参数格式等。

	Logout Plugin
	==============
	
	Non-Cross-platform LogoutPlugin 
	
	Note:Just for LogoutPlugin
	
	
	## Using the plugin ##
	
	LogoutPlugin
	    使用方法:
	    var logoutPlugin = cordova.require("com.technologystudios.logout.logoutplugin")
	    logoutPlugin.executeLogout    
	    {
	    "type":"Logout"
	    }
	
	    返回的参数格式:
	    {
	    	"code":0,
	    	"data":{},
	    	"message":"登出成功!",
	    	"success":"1"
	    }
	## Licence ##
	
	The PHOENIX License
	
	Copyright (c) 2015 PHOENIX Tony Mo

#####fetch.json文件
fetch.json文件，在Cordova项目中，会出现在不同的文件夹。对于插件里面的fetch.json文件，是需要手动配置插件的信息。而对于位置在:项目名称/plugins/fetch.json的fetch.json文件则不需要手动配置，在通过Cordova CLI安装插件后，会自动生成该插件的信息。


例子:

{% codeblock lang:json %}	    
{
 "com.technologystudios.logout": 
 {
	"source": 
	{
   		"type": "non-registry",
   		"id": "com.technologystudios.logout"
	}
 }
}
{% endcodeblock %}   

该文件描述的是插件资源的类型以及插件id。

"type"的值有registry，git。

"registry":表示插件在ng-cordova已经注册的插件
"git":表示通过git url来进行安装(register类型也可以通过git url来安装，但是fetch.json文件的type值为"registry")

上面的"type":"non-registry"是
"com.technologystudios.logout"是插件的id，"type"描述的为该插件是否上传到在Cordova插件库并成功注册.
 
 
#####package.json文件

> 每个项目的根目录下面，一般都有一个package.json文件，定义了这个项目所需要的各种模块，以及项目的配置信息（比如名称、版本、许可证等元数据）。npm install 命令根据这个配置文件，自动下载所需的模块，也就是配置项目所需的运行和开发环境。
 
 引言出自[阮一峰博客](http://javascript.ruanyifeng.com/nodejs/packagejson.html)
 
 > npm 是 Node.js 的模块依赖管理工具。作为开发者使用的工具，主要解决开发 Node.js 时会遇到的问题。如同 RubyGems 对于 Ruby 开发者和 Maven 对于 Java 开发者的重要性，npm 对与 Node.js 的开发者和社区的重要性不言而喻。本文包括五点：package.json 、npm 的配置、npm install 命令、npm link 命令和其它 npm 命令。
 
 InfoQ的文章:[如何使用NPM来管理你的Node.js依赖](http://www.infoq.com/cn/articles/msh-using-npm-manage-node.js-dependence)
 
 
 插件package.json文件例子:

  {% codeblock lang:json %}
  {
   "version": "0.0.1",
   "name": "com.technologystudios.logout",
   "cordova_name": "LogoutPlugin",
   "description": "LogoutPlugin.",
   "license": "PHOENIX",
   "platforms": [
       "ios"
   ],
   "engines": [
       {
           "name": "cordova",
           "version": ">=3.0.0"
       }
   ]
  }
  {% endcodeblock %}

"version":必须字段,为该插件的版本信息。

"name":必须字段,这个名字可能在require()方法中被调用，所以应该尽可能短。但是很多插件在name这个字段，赋值都是插件id，所以现在我也赋值为插件id。

"cordova_name":在下面的package文档中，找不到该字段，后来想起来，该字段是从另一个插件的package.json文件拷贝过来的，作用尚且是告诉我们，这个插件的名字。

若插件需要加上其他字段,请点击
[官网文档](https://github.com/ericdum/mujiang.info/issues/6)
[中文文档1](https://github.com/ericdum/mujiang.info/issues/6)
[中文文档2](http://javascript.ruanyifeng.com/nodejs/packagejson.html)
[中文文档3](http://www.mujiang.info/translation/npmjs/files/package.json.html)

#####plugin.xml文件
plugin.xml 是插件的配置文件。
还是先给出例子:
 {% codeblock lang:xml %}
<?xml version="1.0" encoding="UTF-8"?>
<plugin xmlns="http://www.phonegap.com/ns/plugins/1.0"
   xmlns:android="http://schemas.android.com/apk/res/android"
   id="com.technologystudios.logout"
   version="0.0.1">
   
   <name>LogoutPlugin</name>
   
   <description>LogoutPlugin.</description>
   
   <engines>
       <engine name="cordova" version=">=3.0.0" />
   </engines>    
   
   <license>PHOENIX</license>
   
   <!-- ios -->
  <platform name="ios">
      <config-file target="config.xml" parent="/*">
          <feature name="LogoutPlugin">
               <param name="ios-package" value="CDVLogoutPlugin" />
           </feature>
      </config-file>
      
   <header-file src="src/ios/CDVLogoutPlugin.h"/>
   <source-file src="src/ios/CDVLogoutPlugin.m"/>
     
  <js-module src="www/logoutplugin.js" name="logoutplugin">
   <clobbers target="plugins.logoutplugin" />
  </js-module>

  </platform>
</plugin>
{% endcodeblock %}

* plugin:依次为命名空间、ID、版本，值得我们注意的是属性id。id的值为插件的id值:"com.technologystudios.logout"。
* name:插件的名称
* description:描述信息
* engines:Cordova版本 
* license:授权
* platform:支持的平台,iOS平台的话,属性name的值为iOS。
* config-file:封装feature 注入特定平台(在这里平台为iOS)的config.xml文件
	* feature:这个feature的名字
	* param:iOS平台，name的值就是"ios-package"；安卓平台，name的值就是"android-package"
	使用CLI添加插件后(后面会介绍CLI添加插件的方法)，在项目文件下的platforms/ios/项目名称/config.xml文件，会注入

		<feature name="LogoutPlugin">
		    <param name="ios-package" value="CDVLogoutPlugin" />
		</feature>
而又值得我们注意的是,feature元素的属性name的值"LogoutPlugin"，需要与logoutplugin.js文件的

		cordova.exec(successCallback, errorCallback, 'LogoutPlugin','executeLogout', [options]);
	这个方法的第三个参数的值一致。(在logoutplugin.js文件会详细说明)
	而param元素属性value="CDVLogoutPlugin","CDVLogoutPlugin"这个值是需要作为与src/ios目录下的文件名和类名一致(在CDVLogoutPlugin会详细说明)。
* header-file，source-file:在iOS平台下，指向头文件和实现文件的位置
* js-module:js文件地址，指向www/logoutplugin.js。
* clobbers:这个元素目前暂时不是很理解其作用,target的值笔者一般赋值"plugins.xxx",xxx的值就是js-module属性name的值一致。
* framework:插件依赖的framework。例子: `<framework src="libz.dylib" />`

[官网plugin.xml文档](https://cordova.apache.org/docs/en/4.0.0/plugin_ref_spec.md.html)是官网对plugin.xml字段的详细解释。

#####src/ios/CDVLogoutPlugin文件

该目录下的文件名以及类名需要与plugin.xml文件里面的param的value属性值一致，因为"CDVLogoutPlugin"表示名字为"LogoutPlugin"的feature，映射到名为"CDVLogoutPlugin"的服务，该服务的名称为"CDVLogoutPlugin"的类，在logoutplugin.js文件中，会使用到这个映射的关系。
{% codeblock lang:xml %}
<feature name="LogoutPlugin">
            <param name="ios-package" value="CDVLogoutPlugin" />
</feature>
{% endcodeblock %}

CDVLogoutPlugin.h文件的代码例子:

{% codeblock lang:objc %}
#import <Foundation/Foundation.h>
#import <Cordova/CDVPlugin.h>
@interface CDVLogoutPlugin : CDVPlugin 
- (void)executeLogout:(CDVInvokedUrlCommand*)command;
@end{% endcodeblock %}
CDVLogoutPlugin.m文件的代码例子:
{% codeblock lang:objc %}
#import "CDVLogoutPlugin.h"

@implementation CDVLogoutPlugin
@synthesize callbackId = _callbackId;

#pragma mark - public interface

- (void)executeLogout:(CDVInvokedUrlCommand*)command
{	    
    NSLog(@"CDVLogoutPlugin's function:executeLogout is called");
    
    NSDictionary * argums = command.arguments[0];
    
    if (argums !=nil)
    {
        CDVPluginResult* result = [CDVPluginResult resultWithStatus: CDVCommandStatus_OK messageAsDictionary:@{@"code":@(0),@"data":@{},@"message":@"登出成功!",@"success":@"1"}];
        [self.commandDelegate sendPluginResult:result callbackId:command.callbackId];
    }
    
    else
    {
        //return no type error
        CDVPluginResult* result = [CDVPluginResult resultWithStatus: CDVCommandStatus_ERROR messageAsString:@"no such service"];
        [self.commandDelegate sendPluginResult:result callbackId:command.callbackId];
    }
}

@end{% endcodeblock %}

CDVLogoutPlugin类需要继承CDVPlugin类，CDVLogoutPlugin的方法executeLogout:名称，需要与logoutplugin.js的cordova.exec方法第四个参数一致。executeLogout:方法默认的参数类型是CDVInvokedUrlCommand。

{% codeblock lang:javascript%}
cordova.exec(successCallback, errorCallback, 'LogoutPlugin', 'executeLogout', [options]);
{% endcodeblock %}

	
在CDVLogoutPlugin.m文件中，executeLogout:方法，command.arguments[0]表示的是cordova.exe函数中的最后一个参数[options]，参数的格式可以定义在README.md文件中。该例子的参数格式为{"type":"Logout"}，在javascript传递一个对象到native的插件。
{% codeblock lang:objc %}
[self.commandDelegate sendPluginResult:result callbackId:command.callbackId]
{% endcodeblock %}
这行代码是返回调用插件的结果给javascript回调函数。Status为CDVCommandStatus_OK的，回调successCallback函数，Status为CDVCommandStatus_ERROR回调errorCallback函数。

#####www/logoutplugin.js文件
还是直接上例子
{% codeblock lang:javascript%}
	 
var exec = require('cordova/exec'),
cordova = require('cordova');	
var LogoutPlugin = function () {
};

LogoutPlugin.prototype.executeLogout = function (successCallback, errorCallback,options) {
   if (errorCallback == null) {
       errorCallback = function () {
       }
   }

   if (typeof errorCallback != "function") {
       console.log("LogoutPlugin.executeLogout: failure: failure parameter not a function");
       return
   }

   if (typeof successCallback != "function") {
       console.log("LogoutPlugin.executeLogout: success callback parameter must be a function");
       return
   }

   cordova.exec(successCallback, errorCallback, 'LogoutPlugin', 'executeLogout', [options]);
};

var logoutplugin = new LogoutPlugin();
module.exports = logoutplugin;
{% endcodeblock %}
	
代码第2，3行，是加载cordova，exec对象；
{% codeblock lang:javascript%}
var exec = require('cordova/exec'),
cordova = require('cordova');	
{% endcodeblock %}

第4行，new一个LogoutPlugin对象。

第7行是定义LogoutPlugin对象的executeLogout方法。

{% codeblock lang:javascript%}
LogoutPlugin.prototype.executeLogout = function (successCallback, errorCallback,options)
{% endcodeblock %}

executeLogout方法需要定义两个callback的函数，这是javascript与native对接的回调函数，通过native的

{% codeblock lang:objc %}
CDVPluginResult* result = [CDVPluginResult resultWithStatus: CDVCommandStatus_OK messageAsDictionary:@{@"code":@(0),@"data":@{},@"message":@"登出成功!",@"success":@"1"}];
        [self.commandDelegate sendPluginResult:result callbackId:command.callbackId];
{% endcodeblock %}

CDVPluginResult对象的属性status(CDVCommandStatus_OK/CDVCommandStatus_ERROR)来决定调用对应的成功(successCallback)/失败函数(errorCallback)，而options为javascript到native的参数，参数类型为数组。

第23行
{% codeblock lang:javascript%}
cordova.exec(successCallback, errorCallback, 'LogoutPlugin', 'executeLogout', [options]);
{% endcodeblock %}

这行代码是executeLogout方法映射到插件类的作用。

* 第一第二个参数（successCallback/errorCallback）是根据native返回的成功/失败来回调对应的callback函数。

* 第三个参数（"LogoutPlugin"），为插件的服务名称，必须与plugin.xml的

{% codeblock lang:xml%}
<feature name="LogoutPlugin">
	 <param name="ios-package" value="CDVLogoutPlugin" />
</feature>
{% endcodeblock %}

feature元素name属性值"LogoutPlugin"保持一致，"LogoutPlugin" 通过元素param设置其映射的Objective-C的类的名字为"CDVLogoutPlugin"。这就是为什么src/ios下的文件名称以及类名需要与
{% codeblock lang:xml%}
<param name="ios-package" value="CDVLogoutPlugin" />
{% endcodeblock %}
元素param的属性value的值"CDVLogoutPlugin"相一致。

* 第四个参数（'executeLogout'），为"LogoutPlugin" 映射到Objective-C的"CDVLogoutPlugin"类的方法名字
* 第五个参数（options），是javascript调用native的插件传递的参数，参数格式为数组，通过被调用的本地方法executeLogout:传入的对象(CDVInvokedUrlCommand*)command，
command.arguments[0]来获取。

大家可以尝试快速新建插件，[目录结构](#file_structure)以及各文件内容可以直接拷贝粘贴，先写一个雏形插件。假设大家都写好了这个登出的插件。

####3.Cordova CLI安装插件

这里没有使用pluginman来对Cordova插件的管理。若需要使用pluginman来管理插件的话，请点击这里[这里](https://cordova.apache.org/docs/en/4.0.0/plugin_ref_plugman.md.html)

假设大家已经搭建好Ionic环境，以及拥有一个Ionic的项目，如果没有，请点击[这里](http://ionicframework.com/getting-started/)来进行环境的搭建以及新建项目，这里不再赘言。

* 打开CLI，进入项目的目录
* 输入指令 `cordova plugin -h` 会出现帮助信息，其中我们关注的是 


		add <pluginid>|<directory>|<giturl> [...] ..... add specified plugins
	
                                                    pluginid will be matched in --searchpath / registry
                                                    
                                                    directory is a directory containing a plugin
                                                    
                                                    giturl is a git repo containing a plugin
 
  * pluginid:根据插件的id来搜索插件并安装，该插件需要在Cordova插件库注册成功的。
  * directory:通过插件所在的本地路径来安装(LogoutPlugin就是通过这种方式来安装)
  * giturl:通过url进行插件的安装

	这是使用CLI安装插件前的最外层的plugins目录结构(并非/platforms/ios/www/plugins)
![preInstall_plgin](http://f.cl.ly/items/0i412R2T1K40181f221l/preInstall_plugin.png)

	这是/platforms/ios/www/plugins
![pre_install_plugin_pl](http://f.cl.ly/items/3e403l0O3U1H1M3D1l39/pre_install_plugin_pl.png)

* 在CLI输入指令 `cordova plugin add /Users/Tony/Desktop/plugin/com.technologystudios.logout `
	"/Users/Tony/Desktop/plugin/com.technologystudios.logout"是插件的路径，
	敲下回车键之后，出现 
	`Installing "com.technologystudios.logout" for ios`信息表示成功安装插件
	
* 安装成功后，最外层的plugins目录和/platforms/ios/www/plugins都会多一个刚才add的插件，其中平台里面的plugins是copy最外层的plugins的内容

 	![af_install_plugin](http://f.cl.ly/items/0o0R2v0O3L1a2v0T4130/af_install_plugin.png)
  
  
  
####4.HTML调用本地插件
 1.html代码
 
 {% codeblock lang:html%}
<ion-view >
	<ion-content class="padding">
		<button 
		class="button button-block button-balanced" 
		ng-click="logout()"
		>
  		登出
		</button>
	</ion-content>
</ion-view>
{% endcodeblock %}

第3-8行代码，描述的是名字为"登出"的按钮，ng-click事件为"logout()"。很容易理解。

2.javascript代码

{% codeblock lang:javascript%}
.controller('ChatDetailCtrl', function($scope) {

	$scope.logout = function() {
	
	    var newOptions = {
	              "type": "Logout"
	          };
	    var successCallback = function(result){
	        console.log("调用logout插件成功");
	        console.log(result);
	          };
	    var errorCallback = function(error){
	        console.log("调用logout插件失败");    
	          };
	
	   // 获取 plguin
	   var logoutPlugin = false;
	   if (typeof cordova !== 'undefined') {
	       logoutPlugin = cordova.require("com.technologystudios.logout.logoutplugin")
	   };
	
	   if (!logoutPlugin) {
	       if(angular.isFunction(options.errorCallback)) {
	           options.errorCallback({
	               code: 'no plguin',
	               message: '这个功能还没做'
	           });
	       };
	       return false;
	   };
	    // 尝试调用原生代码
	    try
	    {
	        logoutPlugin.executeLogout(successCallback, errorCallback, newOptions);
	    }
	    catch (error){
	        if(angular.isFunction(options.errorCallback)) {
	            options.errorCallback({
	                code: 'no plguin',
	                message: '这个功能还没做'
	            });
	        };
	    	};    
	 };
  })
 {% endcodeblock %}
 
 第3行代码，是登出按钮的logout()事件的定义。
 
 第8-13行代码，简单地输入控制台调用本地插件的情况，以及输入返回的参数。
 
 第5-13行代码，是options参数格式，以及成功回调函数和失败回调函数的定义。
 
 第18行代码，`cordova.require("com.technologystudios.logout.logoutplugin")` 加载logoutPlugin对象。
 Javascript端调用native端的插件，首先需要通过`cordova.require("插件id.xxx")`来获取得插件对象，其中"xxx"为plugin.xml的元素js-module属性name的值"logoutplugin"。
{% codeblock lang:xml%}
<js-module src="www/logoutplugin.js" name="logoutplugin">
 	<clobbers target="plugins.logoutplugin" />
</js-module> 
{% endcodeblock %}

第33行代码，获取logoutPlugin对象后，通过调用executeLogout(successCallback, errorCallback, newOptions)方法(方法的定义，在logoutplugin.js文件)，来调用映射到objective-c的CDVLogoutPlugin类的方法executeLogout:。


3.Objective-C 代码

{% codeblock lang:objc %}
#import "CDVLogoutPlugin.h"

@implementation CDVLogoutPlugin
@synthesize callbackId = _callbackId;

#pragma mark - public interface

- (void)executeLogout:(CDVInvokedUrlCommand*)command
{
    self.callbackId = command.callbackId;
    
    NSLog(@"CDVLogoutPlugin's function:executeLogout is called");
    
    NSDictionary * argums = command.arguments[0];
    
    if (argums !=nil)
    {
        CDVPluginResult* result = [CDVPluginResult resultWithStatus: CDVCommandStatus_OK messageAsDictionary:@{@"code":@(0),@"data":@{},@"message":@"登出成功!",@"success":@"1"}];
        [self.commandDelegate sendPluginResult:result callbackId:command.callbackId];
    }
    
    else
    {
        //return no type error
        CDVPluginResult* result = [CDVPluginResult resultWithStatus: CDVCommandStatus_ERROR messageAsString:@"no such service"];
        [self.commandDelegate sendPluginResult:result callbackId:command.callbackId];
    }
}
@end
{% endcodeblock %}

第12行代码，简单地在控制台输出native的插件被调用的信息。

第18行代码，组装CDVPluginResult类型的result对象，初始化status为CDVCommandStatus_OK，以及返回的参数:`@{@"code":@(0),@"data":@{},@"message":@"登出成功!",@"success":@"1"}`


####5.图片例子

![html_UI](http://f.cl.ly/items/1a1h3t2y0b1f322r2j3Z/html_UI.png)

点击HTML5页面的"登出"按钮，在xcode控制台会输出信息

`CDVLogoutPlugin's function:executeLogout is called`

表示消息成功地传递到native的插件。

然后顺序执行

{% codeblock lang:objc %}
CDVPluginResult* result = [CDVPluginResult resultWithStatus: CDVCommandStatus_ERROR messageAsString:@"no such service"];
        [self.commandDelegate sendPluginResult:result callbackId:command.callbackId];
{% endcodeblock %}

返回调用成功会HTML，在前端的控制台会输入下面的信息：

![callback_HTML](http://f.cl.ly/items/1e0g344747372H1t3w1c/callback_HTML.png)

返回的Object就是在native的插件组装的对象`@{@"code":@(0),@"data":@{},@"message":@"登出成功!",@"success":@"1"}`。

####6.Cordova更新插件

一般情况下，被添加的插件会被修改，而且被修改的文件位置在 项目名/platforms/ios/www/plugins/pluginid，这时候被修改的插件是不会自动更新到 项目名/plugins/pluginid的文件。笔者的做法是，每一次修改platform下的插件，都会拷贝一份到最外层的插件文件(项目名/plugins/pluginid的文件)。

那么，被添加的插件如何更新?

很遗憾，目前Cordova的CLI没有提供`Cordova plugin update`指令来直接更新插件。

笔者解决方案
这里分两种情况:

1.没有在ng-cordova注册的插件

笔者采用一个曲线救国的方法来更新插件。（该方法针对没有在ng-cordova注册的插件）

* 保存更新的插件文件，更新package.json，plugin.xml文件的version，拷贝PluginDemoTabs/plugins/com.technologystudios.logout文件夹到桌面或者你认为合适的地方。
* 进入到项目的路径下，输入指令

	`cordova plugin remove com.technologystudios.logout`
	
	CLI会输出这些信息
	
	`Uninstalling com.technologystudios.logout from ios
Removing "com.technologystudios.logout`

	这个指令的意思是移除plugin id为com.technologystudios.logout的插件。
	
	成功执行remove puglin之后，PluginDemoTabs/plugins/以及platform
	
*  

2.在ng-cordova注册的插件





