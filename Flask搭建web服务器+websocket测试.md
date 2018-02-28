###Flask搭建web服务器+websocket测试

####名词解释
>**Flask** 是一个用于 Python 的微型网络开发框架。

####上手

```python
from flask import Flask
app = Flask(__name__)

@(马克飞象)app.route('/')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
	#默认5000端口，0.0.0.0表示系统监听一个公开IP地址，局域网可见
    app.run(debug=True,host='0.0.0.0') 
```
>使用**Command+B**运行，可以在浏览器中访问http://127.0.0.1:5000/
你会看到一个简单的显示"Hello World"文本的页面。

**route()**装饰器用于把一个函数绑定到一个 URL 。 下面是一些基本的例子:
```python
@app.route('/')
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello World'
```
HTTP方法
```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        do_the_login()
    else:
        show_the_login_form()
```
####渲染网页模板
>使用 render_template() 方法可以渲染模板，你只要提供模板名称和需要 作为参数传递给模板的变量就行了。下面是一个简单的模板渲染例子:
>```python
>from flask import render_template
>
>@app.route('/hello/')
>@app.route('/hello/<name>')
>def hello(name=None):
>    return render_template('Hom.html', name=name)
>```

>html文件示例
>```html
><!DOCTYPE html>
<html lang="cn">
<head>
	<meta charset="UTF-8">
	<title>Hello from Flask</title>
	{% if name %}
		<h1>Hello {{ name }}!</h1>
	{% else %}
		<h1>Hello World!</h1>
	{% endif %}
</head>
<body>
	<h1 style="font-style:italic">Hello</h1>
</body>
</html>
>```
>尝试访问http://127.0.0.1:5000/hello/Tim

####WebSocket
>使用WebSocket通信，这里使用了**flask-socketio**包。
>```bash
>pip install flask-socketio
>```
>Flask-SocketIO提供给Flask服务器send()和emit()两个函数来发送事件。send()函数发送一个字符串或JSON类型的标准消息给客户端，emit()函数发送一个在自定义事件名之下的消息给客户端
>
>前端Html5原生支持websocket，需要加入**js引用**即可
>```html
><script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/socket.io/1.3.6/socket.io.min.js"></script>
>```

**服务端Python代码修改如下：**
```python
#导入flask_socketio
from flask_socketio import SocketIO,emit
app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret!'
async_mode = 'eventlet'
socketio = SocketIO(app,async_mode=async_mode)

# 主页模板
@app.route('/')
def home():
	return render_template("home.html")

# form模板
@app.route('/build',methods=['POST', 'GET'])
def build():
	if request.method == 'POST':
		return render_template("form.html")
	else :
		return render_template("form.html")

@socketio.on('connect')
def test_connect():
    print('\nClient connected\n')

@socketio.on('disconnect')
def test_disconnect():
    print('\nClient disconnected\n')

@socketio.on('message')
def handle_message(message):
    socketio.send(message)

@socketio.on('json')
def handle_json(json):
    socketio.send(json, json=True)

# 自定义响应编译事件
@socketio.on('build event')
def handle_build_event(json):
	print(json)
	buildIPA()
    
if __name__ == "__main__":
	app.debug = True
	socketio.run(app)
```
**前端Html代码修改如下：**
```html
<!DOCTYPE html>
<html lang="cn">
<head>
	<meta charset="UTF-8">
	<title>Form</title>
</head>
<script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/socket.io/1.3.6/socket.io.min.js"></script>
<script type="text/javascript" charset="utf-8">
var socket = io.connect('http://' + document.domain + ':' + location.port);
    socket.on('connect', function() {
        socket.emit('my event', {'data': 'I\'m connected!'});

    });
    socket.on('disconnect', function() {
        document.getElementById('ws_state').innerHTML = "disconnect";
    });
    function build() {
        socket.emit('build event', {'data': 'send build event'}); 
    }
    socket.on('Event_ServerState', function(msg){
        document.getElementById('ws_state').innerHTML = msg;
    });
    socket.on('Event_Build', function(msg) {       
        try
        {
            var jsonData = JSON.parse(msg);
            var isJSON = isJson(msg);

            if (isJSON && jsonData.rtcode == 0 && jsonData.data.url != null) {
                self.location = jsonData.data.url; 
            }
            else
                document.getElementById('build_state').innerHTML = msg;
        }
        catch(e)
        {
            document.getElementById('build_state').innerHTML = msg;
        }    	
    });
    //判断obj是否为json对象
    function isJson(obj){
        var isjson = typeof(obj) == "object" && Object.prototype.toString.call(obj).toLowerCase() == "[object object]" && !obj.length; 
        return isjson;
    }
    
</script>
<body>
	<p><label id='ws_state' style="color:red">ws_state</label></p>
	<p><label id='build_state' style="color:red">build_state</label></p>
    <p><input type="button" onclick="build()"></input></p>
</body>
</html>
```
>运行服务器后可以实现websocket连接通信