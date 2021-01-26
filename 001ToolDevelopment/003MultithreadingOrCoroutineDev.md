## 多线程编程

### 计算密集型
    
    串行运行计算密集型逻辑耗时时间
    注意：IO密集型是指存在输入输出逻辑的功能，运行时需要耗时
    
```python
# coding=utf-8

import time


def doing():
    data = 0
    for one in range(10000000):
        data += 1

start_time = time.time()
# 串行运行
doing()
doing()
end_time = time.time()

# 串行耗时时间:0.943469762802124
print('串行运行时间是>>>', end_time - start_time)
```

    线程并发运行计算密集型逻辑耗时时间
    
```python
# coding=utf-8

import time
import threading


def doing():
    data = 0
    for one in range(10000000):
        data += 1

start_time = time.time()
t1 = threading.Thread(target=doing)
t2 = threading.Thread(target=doing)

t1.start()
t2.start()

t1.join()
t2.join()
end_time = time.time()

# 线程并发耗时时间:0.9455926418304443
print('多线程并发运行时间是>>>', end_time - start_time)
```

    上面串行和多线程运行计算密集型逻辑耗时时间基本一致，因为是多线程并发运行而不是并行，因为CPU在多线程里
    的cPython解释器不管是多少核的，一个时间点只有一个线程在里面运行，是交叉运行，可能做更大数据的计算时，
    多线程运行的时间比串行耗时更久，因为来回切换也需要时间
    
    cPyhotn自带的全局解释器锁GIL造成的
    socket的TCP协议属于IO密集型的：也就是做完一件事需要等待的
    
### 守护线程

    应用场景：多线程操作情况下，子线程继续存在，主线程先退出
    
```python
# coding=utf-8
# 以下场景会进入死循环，子线程一直执行，主线程永远无法退出

import time
import threading


def doing():
    while True:
        print('I am doing...')
        time.sleep(1)

start_time = time.time()
"""
    主线程想满足一个条件就退出，使用多线程不能直接退出主线程
    解决方式：添加守护线程>>>主线程想退出，子线程也直接退出了~
"""
t1 = threading.Thread(target=doing)
t2 = threading.Thread(target=doing)

# 启动线程
t1.start()
t2.start()

# 阻塞主线程继续执行
t1.join()
t2.join()
end_time = time.time()
for one in range(3):
    print('主线程运行中...')
print('主线程退出...')

print('总共耗时>>>', end_time - start_time)
```
    
    守护线程作用：在主线程想要退出的时候，不需要等待自己运行结束，直接退出就可以了
    
```python
# coding=utf-8

import time
import threading


def doing():  # 不断获取传感器数据，只有按下按钮才停止所有子线程
    while True:
        print('I am doing...')
        time.sleep(1)

start_time = time.time()
t1 = threading.Thread(target=doing)
t2 = threading.Thread(target=doing)

# 添加守护线程，取消join()方法，一旦主线程执行完毕不管子线程是否执行完毕都会退出
t1.setDaemon(True)
t1.setDaemon(True)

t1.start()
t2.start()

end_time = time.time()
for one in range(3):
    print('主线程运行中...')
print('主线程退出...')

print('总共耗时>>>', end_time - start_time)

if __name__ == '__main__':
    doing()
```

    死锁：在线程间共享多个资源的时候，如果两个线程分别占有一部分资源并且同时等待对方资源就会造成死锁
    
    递归锁：为了支持在同一线程多次请求同一资源，python提供了可重入锁：threading.RLock。
    RLock内部维护着一个Lock和一个counter变量，counter记录了acquire的次数，从而使得资源可以被多次acquire
    

### 协程

```python
# coding=utf-8

import time
import gevent
from gevent import monkey

"""
    因为time是默认阻塞模式的
    要想在操作IO的时候遇到等待，就立即切换到其他协程运行就需要修改为非阻塞模式
    所以需要导入monkey模块使用patch_all方法
"""
monkey.patch_all()
def f1():
    for one in range(5):
        print(f'f1在运行---{one}')
        time.sleep(1)

def f2():
    for two in range(5):
        print(f'f2在运行---{two}')
        time.sleep(1)

# 创建协程对象
g1 = gevent.spawn(f1)
g2 = gevent.spawn(f2)

# 运行
gevent.joinall([g1, g2])

"""
运行结果
f1在运行---0
f2在运行---0
f1在运行---1
f2在运行---1
f1在运行---2
f2在运行---2
f1在运行---3
f2在运行---3
f1在运行---4
f2在运行---4
"""
```

    实例应用:tcp请求、socket、其他请求 request get
    
```python
# coding=utf-8

import requests
import gevent
from gevent import monkey
from datetime import datetime

monkey.patch_all()
def get_data(url):
    print(f'{datetime.now()},开始GET获取数据---')
    requests.get(url)
    print(f'{datetime.now()},结束GET获取数据---')

# 创建协程并运行
gevent.joinall(
    [gevent.spawn(get_data, 'http://vip.ytesting.com/'),
     gevent.spawn(get_data, 'http://www.songqinnet.com/')]
)
print('执行结束~')
```


### 多线程引入端口扫描工具

```python
# ---------------v4.0--------------------
# coding=utf-8

# ---------------v4.0--------------------
"""
测试反馈：
    1、是否可以自动识别ip或者域名！---在输入的时候判断！
    2、如果全扫描--发现很慢！---------因为单线程的跑，效率低---建议使用多线程/协程！
    3、如果出现扫描结果不一样，可以使用取并集  set1 |set 2----一个思路！
优化方案：
    1- 多线程---并发操作

"""
# --------------------------------------

import re
import time
import socket
import threading


# ip判断
def check_ip(ip):
    ip_address = re.compile('((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})(\.((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})){3}')
    if ip_address.match(ip) and len(ip) != 0:
        return True
    else:
        return False

# 域名判断
def check_domain(hostname):
    domain_address = re.compile('[a-zA-Z0-9][-a-zA-Z0-9]{0,62}(\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+\.?')
    if domain_address.match(hostname) and len(hostname) != 0:
        return True
    else:
        return False

# 单个端口扫描
def scan_port(ip, port):
    sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sk.settimeout(0.5)
    try:
        conn = sk.connect_ex((ip, port))
        if conn == 0:
            print(f'{ip}主机名,{port}端口号已开放')
    except Exception as e:
        print(e)
    sk.close()

# 多线程扫描
def thread_scan_port(ip):
    start_time = time.time()
    # 存放线程组
    threads = []
    # 创建线程
    for one in range(1, 65535+1):
        t = threading.Thread(target=scan_port, args=(ip, one))
        threads.append(t)

    # 启动线程
    for one in range(0, 65535):
        threads[one].start()

    # 阻塞主线程
    for one in range(0, 65535):
        threads[one].join()

    end_time = time.time()
    print('端口扫描总耗时>>>', end_time - start_time)

# ip扫描
def scan_ip(ip):
    thread_scan_port(ip)

# 域名扫描
def domain_name_scan(domain_name):
    # 域名判断解析
    if 'http://' in domain_name or 'https://' in domain_name:
        domain_name = domain_name[domain_name.find('://')+3:]
        print('解析的域名为>>>', domain_name)

    # 获取域名ip地址
    server_ip = socket.gethostbyname(domain_name)
    print(f'该域名{domain_name}的ip>>>{server_ip}')

    # 传入域名ip进行扫描
    thread_scan_port(server_ip)

def main():
    info = input("请输入需要扫描的域名或ip>>> ")
    if check_ip(info):
        scan_ip(info)
    elif check_domain(info):
        domain_name_scan(info)
    else:
        print('域名或ip输入有误~')


if __name__ == '__main__':
    main()
# --------------------------------------
```

### 协程引入端口扫描工具


```python
# coding=utf-8

# ---------------v5.0--------------------
"""
测试反馈：
    1、是否可以自动识别ip或者域名！---在输入的时候判断！
    2、如果全扫描--发现很慢！---------因为单线程的跑，效率低---建议使用多线程/协程！
    3、如果出现扫描结果不一样，可以使用取并集  set1 |set 2----一个思路！
优化方案：
    1- 协程---并发操作

"""
# --------------------------------------

import re
import time
import socket
import gevent.pool
import gevent.monkey


# ip判断
def check_ip(ip):
    ip_address = re.compile('((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})(\.((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})){3}')
    if ip_address.match(ip) and len(ip) != 0:
        return True
    else:
        return False

# 域名判断
def check_domain(hostname):
    domain_address = re.compile('[a-zA-Z0-9][-a-zA-Z0-9]{0,62}(\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+\.?')
    if domain_address.match(hostname) and len(hostname) != 0:
        return True
    else:
        return False

# 单个端口扫描
def scan_port(ip, port):
    sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sk.settimeout(0.5)
    try:
        conn = sk.connect_ex((ip, port))
        if conn == 0:
            print(f'{ip}主机名,{port}端口号已开放')
    except Exception as e:
        print(e)
    sk.close()

# 协程扫描
gevent.monkey.patch_all()
def gevent_scan_port(ip):
    start_time = time.time()
    # 创建线程池，限制协程并发数量，200个左右即可
    g = gevent.pool.Pool(200)
    # 存放运行的协程
    run_gevent_list = []
    for one in range(1, 65535+1):
        g_pool = g.spawn(scan_port, ip, one)
        run_gevent_list.append(g_pool)

    # 运行完，主线程退出 阻塞
    gevent.joinall(run_gevent_list)
    end_time = time.time()
    print('扫描结束总共耗时>>>', end_time - start_time)


# ip扫描
def scan_ip(ip):
    gevent_scan_port(ip)

# 域名扫描
def domain_name_scan(domain_name):
    # 域名判断解析
    if 'http://' in domain_name or 'https://' in domain_name:
        domain_name = domain_name[domain_name.find('://')+3:]
        print('解析的域名为>>>', domain_name)

    # 获取域名ip地址
    server_ip = socket.gethostbyname(domain_name)
    print(f'该域名{domain_name}的ip>>>{server_ip}')

    # 传入域名ip进行扫描
    gevent_scan_port(server_ip)

def main():
    info = input("请输入需要扫描的域名或ip>>> ")
    print('-----')
    print(info)
    print('----')
    if check_ip(info):
        scan_ip(info)
    elif check_domain(info):
        domain_name_scan(info)
    else:
        print('域名或ip输入有误~')


if __name__ == '__main__':
    main()
```

### 协程并发操作——增加导出结果文件

```python

```