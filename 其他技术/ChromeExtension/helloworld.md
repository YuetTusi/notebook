## manifest.json 文件

开发Chrome扩展，项目的根目录中必须有一个manifest.json（清单）文件。

格式如下：

```json
{  
	"manifest_version": 3,  
	"name": "Hello Extensions",  
	"description": "Base Level Extension",  
	"version": "1.0",  
	"action": 
	{
		"default_popup": "hello.html",
		"default_icon": "hello_extensions.png" 
	}
}
```

其中`manifest_version`，`name`和`version`字段必须存在，manifest_version的值为“3”，这是目前Chrome扩展的最新版本（从Chrome88开始），后续版本升级可能有变化。

action字段设置了扩展程序在浏览器右上角的弹出页，和图标。

在项目根目录创建一个hello.html页和hello_extensions.png图片。

其中，hello.html页面中写上一句话：Hello Extendsion。

在Chrome浏览器中，地址栏输入[chrome://extensions/](chrome://extensions/)

然后选择项目目录并加载，如果文件没有缺失且无误，则会加载到扩展程序列表里：

![](../../images/1e5d58903540.png)

同时，右上角出会出现对应的图标，点击弹出popup菜单，显示出hello.html网页的内容：

![](../../images/2c0894fa8670.png)

这样一个最简单的Chrome扩展就写好了。

后续在修改了代码后，需要重新加载扩展程序，则点击已加载的程序名称右下的刷新即可。


## 扩展程序功能


一个Chrome Extension扩展程序可以做以下事情：

- 自定义扩展界面
- 构建扩展工具
- 修改和监听浏览器
- 扩展开发控制台
- 修改和监听网页

