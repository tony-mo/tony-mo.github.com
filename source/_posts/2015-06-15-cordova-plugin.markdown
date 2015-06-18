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

在新建名称为com.technologystudios.logout的文件夹后，依次新建以下的文件(com.technologystudios.logout不必再新建，只需新建README.md，plugin.xml，src和www文件夹以及下级的文件，其中plugin.xml,src,www为必须的文件)

新建后的文件目录结构(这只是个人新建插件文件目录结构的习惯，并非标准。)

	com.technologystudios.logout
	├── README.md
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

#####plugin.xml文件
plugin.xml 是插件的配置文件。
还是先给出例子:
{% codeblock lang:xml plugin.xml %}<?xml version="1.0" encoding="UTF-8"?>
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
* config-file:封装feature元素注入特定平台(在这里平台为iOS)的config.xml文件
	* feature:属性name为插件的名字
	* param:iOS平台，name的值就是"ios-package"；安卓平台，name的值就是"android-package"
	使用CLI添加插件后(后面会介绍CLI添加插件的方法)，在项目名称/platforms/ios/项目名称/config.xml文件，会注入
	{% codeblock lang:xml%}
	<feature name="LogoutPlugin">
		    <param name="ios-package" value="CDVLogoutPlugin" />
	</feature>{% endcodeblock %}
而又值得我们注意的是,feature元素的属性name的值"LogoutPlugin"，是插件的服务名称，需要与logoutplugin.js文件的
	{% codeblock lang:javascript%}cordova.exec(successCallback, errorCallback, 'LogoutPlugin','executeLogout', [options]);{% endcodeblock %}
		
      这个方法的第三个参数的值一致，(在logoutplugin.js文件会详细说明)。
	  而param元素属性value="CDVLogoutPlugin","CDVLogoutPlugin"是native代码的类名，需要作为与src/ios目录下的文件名和类名一致(在CDVLogoutPlugin会详细说明)。
* header-file，source-file:在iOS平台下，指向iOS原生代码的头文件和实现文件的位置
* js-module:src指向插件的js文件地址www/logoutplugin.js，属性name的值"logoutplugin"，作为插件id的最后一部分。JS使用`cordova.require`加载logoutPlugin对象的时候，使用到的是插件id + "logoutplugin" ，`cordova.require("com.technologystudios.logout.logoutplugin")`，为什么通过这个指令就能够得到logoutPlugin对象？

当通过CLI安装插件后，logoutplugin.js被拷贝到：项目名称/platforms/ios/www/plugins/my.plugin.id/www/logoutplugin.js这个目录，同时，插件的id，js文件被拷贝到的目标目录，以及clobbers元素信息，会作为新增的条目被增加到：项目名称/platforms/ios/cordova_plugins.js文件。
	cordova_plugins.js文件，我对它的理解，是封装插件的信息，通过插件id加载对应的插件js文件，获得插件对象，而不需要调用端关心需要加载哪一个js文件。
{% codeblock lang:javascript  cordova_plugins.js %}
cordova.define('cordova/plugin_list', function(require, exports, module) {
module.exports = [
    {
        "file": "plugins/com.technologystudios.logout/www/logoutplugin.js",
        "id": "com.technologystudios.logout.logoutplugin",
        "clobbers": [
            "plugins.logoutplugin"
        ]
    }
];
module.exports.metadata = 
// TOP OF METADATA
{
    "com.technologystudios.logout": "0.0.1"
}
// BOTTOM OF METADATA
});
{% endcodeblock %}

cordova.define定义名字为`'cordova/plugin_list'`的模块，改模块向外声明返回的对象类型为数组，该数组对象封装了项目的插件列表。`cordova.require("com.technologystudios.logout.logoutplugin")`，通过获得`'cordova/plugin_list'`模块返回的数组对象，并根据id加载对应的js文件logoutplugin.js。而logoutplugin.js文件定义了`module.exports = logoutplugin;`模块并对外部声明模块返回logoutplugin对象(具体代码内容见logoutplugin.js文件)。所以通过`cordova.require(id)`能获取插件的对象。

<a name="mudule_define">涉及到的mudule.exports以及cordova.define,cordova.require资料</a>

[nodejs中exports与module.exports的区别详细介绍](http://www.jb51.net/article/33269.htm)

[Cordova 3.x 源码分析（3） -- cordova.js模块系统require/define](http://www.ylzx8.cn/web/javascript/953687.html)

* clobbers:官网API给出的解释是"指示`module.exports`作为`window.some.value`被插入到window对象"。目前对该元素没有很深的理解,target的值我一般赋值"plugins.xxx",xxx的值就是与js-module属性name的值一致。有对这个元素理解深刻的同学麻烦指教一下。

* framework:插件依赖的framework。例子: `<framework src="libz.dylib" />`

[官网plugin.xml文档](https://cordova.apache.org/docs/en/4.0.0/plugin_ref_spec.md.html)plugin.xml字段的详细解释。

#####src/ios/CDVLogoutPlugin文件

该目录下的文件名以及类名需要与plugin.xml文件里面的
{% codeblock lang:xml plugin.xml %}
<config-file target="config.xml" parent="/*">
          <feature name="LogoutPlugin">
               <param name="ios-package" value="CDVLogoutPlugin" />
           </feature>
      </config-file>
{% endcodeblock %}
param的value属性值一致。"CDVLogoutPlugin"表示插件名字为"LogoutPlugin"的feature，映射到名为"CDVLogoutPlugin"的服务，插件服务的类名称为"CDVLogoutPlugin"，在logoutplugin.js文件中，会使用到这个映射的关系。

CDVLogoutPlugin.h文件的代码例子:

{% codeblock lang:objc CDVLogoutPlugin.h %}
#import <Foundation/Foundation.h>
#import <Cordova/CDVPlugin.h>
@interface CDVLogoutPlugin : CDVPlugin 
- (void)executeLogout:(CDVInvokedUrlCommand*)command;
@end{% endcodeblock %}

CDVLogoutPlugin.m文件的代码例子:
{% codeblock lang:objc CDVLogoutPlugin.m%}
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
{% codeblock lang:javascript logoutplugin.js%}
	 
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

executeLogout方法接收三个参数，第一第二个参数，函数类型。successCallback函数，调用本地插件成功时回调的函数；errorCallback函数，调用本地插件失败时回调的函数。这是javascript与native对接的回调函数，通过native的
{% codeblock lang:objc %}
CDVPluginResult* result = [CDVPluginResult resultWithStatus: CDVCommandStatus_OK messageAsDictionary:@{@"code":@(0),@"data":@{},@"message":@"登出成功!",@"success":@"1"}];
[self.commandDelegate sendPluginResult:result callbackId:command.callbackId];
{% endcodeblock %}

CDVPluginResult对象的属性status(CDVCommandStatus_OK/CDVCommandStatus_ERROR)来返回调用对应的成功(successCallback)/失败函数(errorCallback)。

第三个参数，数组类型，为javascript到native的参数。通过

{% codeblock lang:objc%}
NSDictionary * argums = command.arguments[0];
{% endcodeblock %}
获得，其中`command`为`CDVInvokedUrlCommand*`类型。

第23行
{% codeblock lang:javascript%}
cordova.exec(successCallback, errorCallback, 'LogoutPlugin', 'executeLogout', [options]);
{% endcodeblock %}
执行cordova.exec方法，调用'LogoutPlugin'服务的'executeLogout'方法。本地Objective-C代码会执行CDVLogoutPlugin类的executeLogout:方法。

* 第一第二个参数（successCallback/errorCallback）为外部传递进来的成功/失败回调函数。

* 第三个参数（"LogoutPlugin"），为插件的服务名称，需要与plugin.xml文件的

{% codeblock lang:xml%}
<feature name="LogoutPlugin">
	 <param name="ios-package" value="CDVLogoutPlugin" />
</feature>
{% endcodeblock %}

feature元素name属性值"LogoutPlugin"保持一致，"LogoutPlugin" 通过元素param设置其映射的Objective-C的类的名字为"CDVLogoutPlugin"。

* 第四个参数（'executeLogout'），为"LogoutPlugin" 映射到Objective-C的"CDVLogoutPlugin"类的方法名字
* 第五个参数（options），是javascript调用native的插件传递的参数，参数格式为数组，通过被调用的本地方法executeLogout:传入的对象(CDVInvokedUrlCommand*)command，
command.arguments[0]来获取。

第27行
{% codeblock lang:javascript%}
module.exports = logoutplugin;
{% endcodeblock %}

创建模块logoutplugin，通过加载logoutplugin.js文件能获取logoutplugin对象。在[plugin.xml](#mudule_define)文件中会详细说明。

大家可以尝试快速新建插件，[目录结构](#file_structure)以及各文件内容可以直接拷贝粘贴，先写一个雏形插件。假设大家都写好了这个登出的插件。

####3.Cordova CLI安装插件

这里没有使用pluginman来对Cordova插件的管理。若需要使用pluginman来管理插件的话，请点击这里[这里](https://cordova.apache.org/docs/en/4.0.0/plugin_ref_plugman.md.html)

假设大家已经搭建好Ionic环境，以及已经新建好Ionic的项目，如果没有，请点击[这里](http://ionicframework.com/getting-started/)来进行环境的搭建以及新建项目，这里不再赘言。

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
{% codeblock lang:html chat-detail.html%}
 <ion-view >
	<ion-content class="padding">
		<button 
		class="button button-block button-balanced" 
		ng-click="logout()"
		>
  		登出
		</button>
	</ion-content>
</ion-view>{% endcodeblock %}

第3-8行代码，描述的是名字为"登出"的按钮，ng-click事件为"logout()"。很容易理解。

2.javascript代码

{% codeblock lang:javascript controllers.js%}
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

{% codeblock lang:objc CDVLogoutPlugin.m%}
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


####5.例子

![html_UI](http://f.cl.ly/items/1a1h3t2y0b1f322r2j3Z/html_UI.png)

点击HTML5页面的"登出"按钮，在xcode控制台会输出信息

`CDVLogoutPlugin's function:executeLogout is called`

表示消息成功地传递到native的插件。

然后顺序执行

{% codeblock lang:objc %}
CDVPluginResult* result = [CDVPluginResult resultWithStatus: CDVCommandStatus_OK messageAsDictionary:@{@"code":@(0),@"data":@{},@"message":@"登出成功!",@"success":@"1"}];
[self.commandDelegate sendPluginResult:result callbackId:command.callbackId];
{% endcodeblock %}

返回调用成功会HTML，在前端的控制台会输入下面的信息：

![callback_HTML](http://f.cl.ly/items/1e0g344747372H1t3w1c/callback_HTML.png)

返回的Object就是在native的插件组装的对象`@{@"code":@(0),@"data":@{},@"message":@"登出成功!",@"success":@"1"}`。

####6.Cordova更新插件

一般情况下，被添加的插件会被修改，而且被修改的文件位置在:项目名/platforms/ios/www/plugins/pluginid/src/ios/xxxx，或者是项目名/platforms/ios/www/plugins/pluginid/www/xxx.js，被修改的文件是不会自动更新到:项目名/plugins/pluginid的文件。

那么，被添加的插件如何更新?

很遗憾，目前Cordova的CLI没有提供`Cordova plugin update`指令来直接更新插件。

笔者解决方案
这里分两种情况:

1.没有在ng-cordova注册的插件

笔者采用一个曲线救国的方法来更新插件，当需要修改:项目名/platforms/ios/www/plugins/pluginid下的文件，我会拷贝修改的文件内容到:项目名称/plugins/对应需要被覆盖的文件。

当插件的修改内容达到需要更新版本号的时候，保存更新的插件文件，更新:项目名称/plugins/pluginid/plugin.xml文件的`version="x.x.x"`，拷贝:项目名称/plugins/pluginid文件夹到桌面或者你认为合适的地方。

进入到项目的路径下，输入指令(使用logout插件id作为例子)

	`cordova plugin remove com.technologystudios.logout`
	
CLI会输出这些信息
	
	`Uninstalling com.technologystudios.logout from ios Removing "com.technologystudios.logout`

指令的意思是移除plugin id为com.technologystudios.logout的插件。

成功执行remove puglin之后，项目名称/plugins/以及项目名称/platforms/ios/www/plugins/下的com.technologystudios.logout文件件会被自动删除。

在CLI输入指令

`ionic build ios`

`cordova plugin add /Users/Tony/Desktop/com.technologystudios.logout`

重新安装最新的插件到项目，完成一次插件的更新。

*注意*
若重新安装后，调用插件，xcode的控制台出现

	2015-06-18 18:57:28.677 PluginDemoTabs[18677:645115] ERROR: Plugin 'LogoutPlugin' not found, or is not a CDVPlugin. Check your plugin mapping in config.xml.
	2015-06-18 18:57:28.677 PluginDemoTabs[18677:645115] -[CDVCommandQueue executePending] [Line 159] FAILED pluginJSON = ["LogoutPlugin1500224491","LogoutPlugin","executeLogout",[{"type":"Logout"}]]

* 选中使用xcode打开项目，找到CDVLogoutPlugin.m文件，选中，然后在Target Membership，选中项目的target，打钩，然后使用CLI输入指令`ionic build ios`重新编译项目，问题解决。

![file_struture](http://f.cl.ly/items/2w150J2G0f1V0I3H2609/file_struture.png)
![target_member](http://f.cl.ly/items/112j1x252u1U0S1J2a0E/target_member.png)

2.在ng-cordova注册的插件
目前还没有在ng-cordova上传并成功注册过插件，在这里先挖个坑，往后的日子再填。


(完)

