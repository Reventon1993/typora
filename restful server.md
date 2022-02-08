python restful server

routers库（mvc模式）







wsgi server + wsgi app

wsgi app 是一个callable的python对象，只接受两个参数environ和start_response

environ是python字典，包含cgi中定义的变量

start_response指向一个回调函数

wsgi middleware:本质上来说是wsgi app。wsgi app可以分层成多个独立的wsgi app，wsgi middleware往往是

上层的通用的wsgi app



paste库：生成一个wsgi app(把每层的具体的wsgi app生成一个wsgi app)

openstack中对paste的使用很自由(很高级)，不太容易明白。在我看来各种type(app、filter等)最重要的就是第一行。

不同type第一行语法不同。



webob库

对wsgi的请求和响应进行封装，简化wsgi app的编写

    @webob.dec.wsgify(RequestClass=wsgi.Request)
    def __call__(self, req):


