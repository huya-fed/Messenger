# MessengerJS

---

跨文档通信解决方案

https://github.com/biqing/MessengerJS

http://segmentfault.com/a/1190000000702539

---

##适用场景

此方案适用于以下跨域情形:

- 父窗口与iframe之间通信
- 多个iframe之间通信
*上述所有情况, 都需确保对不同域的页面有修改权限, 并同时加载MessengerJS

*IE下不支持跨窗口通信

常见跨源问题有:

- 跨子域
- 跨全域
- 跨协议(HTTP与HTTPS)


##理念: 关于"信使"的一切

理解设计理念对实际使用有帮助作用, 高手可以直接跳到下方使用说明 : )

在跨文档通信中, 一切消息都是以字符串形式存在, 可以视其为"报文", 因此负责派送和接受信件的角色, 我们称其为"信使"(Messenger).

Messenger的职责很简单, 主要分为 发送消息(send) 与 监听消息(listen), 而消息的内容都是字符串. 实际使用中, 最好不要直接使用简单的字符串, 而建议使用结构化的消息(JSON String). 具体逻辑请自行实现: 发送前将json内容stringify, 收到后进行parse, 以实现消息内容的扩展性.


## 使用方式

第一种
	
	直接在页面中引用messenger.js
	<script type="text/javascript" src="messenger.js"></script>

第二种

	兼容amd模式
	var Messenger = require('messenger');
	

##如何使用

1. 在需要通信的文档中(父窗口和iframe们), 都确保引入MessengerJS

2. 每一个文档(document)，都需要自己的Messenger与其他文档通信。即每一个window对象都对应着一个，且仅有一个Messenger对象，该Messenger对象会负责当前window的所有通信任务。每个Messenger对象都需要唯一的名字，这样它们才可以知道跟谁通信。另外，推荐指定项目名称（类似命名空间的作用），以增强代码健壮性与组件复用性，避免未来与其他项目冲突。(注意: 项目名称应使用 字符串类型 )


		// 父窗口中 - 初始化Messenger对象
		// 推荐指定项目名称, 避免Mashup类应用中, 多个开发商之间的冲突
		var messenger = new Messenger('Parent', 'projectName');
		
		// iframe中 - 初始化Messenger对象
		// 注意! Messenger之间必须保持项目名称一致, 否则无法匹配通信
		var messenger = new Messenger('iframe1', 'projectName');
		
		// 多个iframe, 使用不同的名字
		var messenger = new Messenger('iframe2', 'projectName');

3. 在发送消息前，确保目标文档已经监听了消息事件。


		// iframe中 - 监听消息
		// 回调函数按照监听的顺序执行
		messenger.listen(function(msg){
		    alert("收到消息: " + msg);
		});

4. 父窗口想给iframe发消息，它怎么知道iframe的存在呢？添加一个消息对象吧。


		// 父窗口中 - 添加消息对象, 明确告诉父窗口iframe的window引用与名字
		messenger.addTarget(iframe1.contentWindow, 'iframe1');
		
		// 父窗口中 - 可以添加多个消息对象
		messenger.addTarget(iframe2.contentWindow, 'iframe2');

5. 一切ready，发消息吧~发送消息有两种方式。 (以父窗口向iframe发消息为例)

		// 父窗口中 - 向单个iframe发消息
		messenger.targets['iframe1'].send(msg1);
		messenger.targets['iframe2'].send(msg2);
		
		// 父窗口中 - 向所有目标iframe广播消息
		messenger.send(msg);



#例子

##www.huya.com --- paren.html

	<!DOCTYPE html>
		<html>
		<head>
		    <meta charset="utf-8">
		    <meta name="viewport" content="width=device-width, initial-scale=1.0">
		    <title>Demo - MessengerJS</title>
		    <script src="messenger.js"></script>
		    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.2/css/bootstrap.min.css">
		    <style>
		        iframe {
		            margin: 10px;
		            margin-left: 0;
		        }
		    </style>
		</head>
		<body>
		<div class="container">
		    <h1>Demo of MessengerJS</h1>
		    <p class="lead"><a href="http://biqing.github.io/MessengerJS/">Github project page</a></p>
		    <p>
		        <span class="label label-danger">parent</span>
		        Domain of the parent page
		        <script>document.write('(' + location.protocol + '//' + location.host + ')');</script>
		    </p>
		    <p>
		        <iframe id="iframe1" src="http://hd.huya.com/child/iframe1.html" width="550px" height="300px"></iframe>
		    </p>
		
		    <p>
		        <input type="text" placeholder="输入消息" id="message" />
		        <button type="button" class="btn btn-primary" onclick="sendMessage('iframe1');">send to iframe1</button>
		        <button type="button" class="btn btn-success" onclick="sendAll();">send to all</button>
		        <button type="button" id="child" class="btn btn-info" onclick="openWindow();">open a window</button>
		    </p>
		    <pre id="output" class="alert alert-warning"></pre>
		</div>
		<script>
		    var messenger = new Messenger('parent', 'MessengerDemo'),
		        iframe1 = document.getElementById('iframe1'),
		        input = document.getElementById('message');
		
		    messenger.listen(function (msg) {
		        var newline = '\n';
		        var text = document.createTextNode(msg + newline);
		        document.getElementById('output').appendChild(text);
		    });
		
		    messenger.addTarget(iframe1.contentWindow, 'iframe1');
		
		    function sendMessage(name) {
		        var msg = input.value;
		        messenger.targets[name].send("message from parent: " + msg);
		        input.value = '';
		    }
		
		    function sendAll() {
		        var msg = input.value;
		        messenger.send("message from parent: " + msg);
		        input.value = '';
		    }
		
		    function openWindow() {
		        var outerWindow = window.open("http://xxx.huya.com/child.html", "_blank", "width=603,height=489,location=yes,status=yes,toolbar=no,resizable=no");
		        messenger.addTarget(outerWindow, 'childWindow');
		        document.getElementById("child").innerHTML = "send to child window";
		        document.getElementById("child").onclick = function(){
		            sendMessage('childWindow');
		        };
		    }
		</script>
		</body>
		</html>


##hd.huya.com ---- iframe1.html

	<!DOCTYPE html>
		<html>
		<head>
		    <meta charset="utf-8">
		    <meta name="viewport" content="width=device-width, initial-scale=1.0">
		    <title>iframe communication (iframe page)</title>
		    <script src="messenger.js"></script>
		    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.2/css/bootstrap.min.css">
		</head>
		<body>
		<div class="container">
		    <p>
		        <span class="label label-danger">iframe1</span>
		        This is a iframe under the domain: 
		        <script>document.write('(' + location.protocol + '//' + location.host + ')');</script>
		    </p>
		
		    <p>
		        <input type="text" placeholder="输入消息" id="message" />
		        <button type="button" class="btn btn-primary" onclick="sendMessage('parent');">send to parent</button>
		        <button type="button" class="btn btn-success" onclick="sendAll();">send to all</button>
		    </p>
		    <pre id="output" class="alert alert-warning"></pre>
		</div>
		
		<script>
		    var messenger = new Messenger('iframe1', 'MessengerDemo'),
		        input = document.getElementById('message');
		
		    messenger.listen(function (msg) {
		        var newline = '\n';
		        var text = document.createTextNode(msg + newline);
		        document.getElementById('output').appendChild(text);
		    });
		
		    messenger.addTarget(window.parent, 'parent');
		
		    function sendMessage(name) {
		        var msg = input.value;
		        messenger.targets[name].send("message from iframe1: " + msg);
		        input.value = '';
		    }
		
		    function sendAll() {
		        var msg = input.value;
		        messenger.send("message from iframe1: " + msg);
		        input.value = '';
		    }
		</script>
		</body>
		</html>

###xxx.huya.com --- child.html

	<!DOCTYPE html>
	<html>
	<head>
	    <meta charset="utf-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <title>iframe communication (child window page)</title>
	    <script src="messenger.js"></script>
	    <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.0.2/css/bootstrap.min.css">
	</head>
	<body>
	<div class="container">
	    <p>
	        <span class="label label-danger">child window</span>
	        This is a child window under the domain: 
	        <script>document.write('(' + location.protocol + '//' + location.host + ')');</script>
	    </p>
	
	    <p>
	        <input type="text" placeholder="输入消息" id="message" />
	        <button type="button" class="btn btn-info" onclick="sendMessage('parent');">send to parent</button>
	    </p>
	    <pre id="output" class="alert alert-warning"></pre>
	</div>
	
	<script>
	    var messenger = new Messenger('childWindow', 'MessengerDemo'),
	        input = document.getElementById('message');
	
	    messenger.listen(function (msg) {
	        var newline = '\n';
	        var text = document.createTextNode(msg + newline);
	        document.getElementById('output').appendChild(text);
	    });
	
	    messenger.addTarget(window.opener, 'parent');
	
	    function sendMessage(name) {
	        var msg = input.value;
	        messenger.targets[name].send("message from childWindow: " + msg);
	        input.value = '';
	    }
	</script>
	</body>
	</html>


