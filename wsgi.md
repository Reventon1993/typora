  cgi:[公共网关接口](https://baike.baidu.com/item/公共网关接口/10911997)（Common Gateway Interface，CGI）

uWSGI&uwsgi&wsgi

**uWSGI** is a [software application](https://en.wikipedia.org/wiki/Software_application) that "aims at developing a full stack for building [hosting services](https://en.wikipedia.org/wiki/Hosting_services)".[[3\]](https://en.wikipedia.org/wiki/UWSGI#cite_note-uwsgi-3) It is named after the [Web Server Gateway Interface](https://en.wikipedia.org/wiki/Web_Server_Gateway_Interface) (WSGI), which was the first plugin supported by the project.[[3\]](https://en.wikipedia.org/wiki/UWSGI#cite_note-uwsgi-3)

**uwsgi** (all lowercase) is the native binary protocol that uWSGI uses to communicate with other servers.[[4\]](https://en.wikipedia.org/wiki/UWSGI#cite_note-4)

uWSGI is often used for serving [Python](https://en.wikipedia.org/wiki/Python_(programming_language)) [web applications](https://en.wikipedia.org/wiki/Web_applications) in conjunction with [web servers](https://en.wikipedia.org/wiki/Web_server) such as [Cherokee](https://en.wikipedia.org/wiki/Cherokee_(web_server)) and [Nginx](https://en.wikipedia.org/wiki/Nginx), which offer direct support for uWSGI's native uwsgi protocol.[[5\]](https://en.wikipedia.org/wiki/UWSGI#cite_note-digitalocean-5) For example, data may flow like this: HTTP client ↔ Nginx ↔ uWSGI ↔ Python app.[[6\]](https://en.wikipedia.org/wiki/UWSGI#cite_note-6)



wsgi app就是一个callable的python对象

```python
from wsgiref.simple_server import make_server

# application就是一个wsgi app
def application(environ, start_response):
    body = start_response('200 OK', [('Content-Type', 'text/html')])
    body('body_data')  # body是write_body的方法
    return '<h1>Hello, web!</h1>' # 返回数据也写入body中
    
httpd = make_server('', 9002, application)
httpd.serve_forever()
```



![企业微信截图_a1c53c42-808b-4205-a529-53f5397db1fb](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210917112104.png)



在WSGI框架下web server被分成了两部分：实现了uwsgi的web server和wsgi app

wsgi app可以把一些通用的逻辑分出去叫：wsgi middleware，比如路由功能(路由到后面具体的wsgi app)。

webob

对wsgi app进行一层封装

```python
from wsgiref.simple_server import make_server
import webob
import webob.dec

@webob.dec.wsgify
def application(request):
    res = webob.Response()
    res.status = '200 OK'
    res.headerlist.append(('Content-Type', 'text/html'))
    res.body = '<h1>Hello, web!</h1>'
    return res

# def application(environ, start_response):
#     body = start_response('200 OK', [('Content-Type', 'text/html')])
#     body('body_data')   
#     return '<h1>Hello, web!</h1>' 
    
httpd = make_server('', 9002, application)
httpd.serve_forever()
```

![企业微信截图_331ef8fb-11cb-498b-ae15-286cc9e2791c](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210918144022.png)



python paste

构建wsgi app的框架

具体可以参考nova-api的构建







