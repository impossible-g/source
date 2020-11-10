#gunicorn 源码解析，以gevent为例
```python
# 启动gunicorn的命令行程序
WSGIApplication("%(prog)s [OPTIONS] [APP_MODULE]").run()

# 加载配置文件类
self.do_load_config()

# 加载wsgi app，以django为例app_uri 就是 profile.wsgi:application, 如果不写：，会自动去这个文件里面找application
util.import_app(self.app_uri)  # app_uri = model.wsgi

# 进程维护类启动，初始化master进程，同时监控worker进程，如果worker无响应时就杀死worker然后在启动一个新的worker进程
Arbiter(self).run()
self.start()
self.manage_workers()
self.spawn_workers()
worker = self.worker_class(self.worker_age, self.pid, self.LISTENERS,
                                   self.app, self.timeout / 2.0,
                                   self.cfg, self.log)

# 进入worker进程，会创建一个server类，等待和处理链接都是在该类下做的
pid = os.fork()  # 创建子进程
worker.init_process()
# worker每一秒就会给master发送一个信号，通知master自己还活着，防止被master干掉
while self.alive:
    self.notify()
    gevent.sleep(1.0)
server = self.server_class(s, application=self.wsgi, spawn=pool, log=self.log,
                    handler_class=self.wsgi_handler, environ=environ,
                    **ssl_args)
server.start()
# 开始监听链接，并读出数据调用do_handle去处理链接
self.start_accepting()
args = self.do_read()  # 读取数据
self.do_handle(*args)
self.pool = spawn  # spawn == Pool(self.worker_connections)
def spawn(self, *args, **kwargs):
    greenlet = self.greenlet_class(*args, **kwargs)
    self.start(greenlet)
    return greenlet
self._spawn = spawn.spawn  # == spawn
self.handle = handle
self._handle = self.handle # 也就是你的web服务
# 创建一个新的协程，并在其调用_handle_and_close_when_done这个方法
spawn(_handle_and_close_when_done, handle, close, args)
# 调用你自己的web服务
def _handle_and_close_when_done(handle, close, args_tuple):
    try:
        return handle(*args_tuple)
    finally:
        close(*args_tuple)
```
