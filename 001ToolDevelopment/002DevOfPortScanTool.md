```python
# coding=utf-8

# ---------------v1.0--------------------
"""
需求：
    1- 完成一个端口的扫描
    127.0.0.1
"""
# --------------------------------------


import socket

def scan_test1():
    sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sk.settimeout(0.5)
    ip = input('请输入ip>>>')
    port = int(input('请输入port>>>'))
    conn = sk.connect_ex((ip, port))
    if conn == 0:
        print(f'主机名{ip},{port}端口号已开放')
    else:
        print(f'主机名{ip},{port}端口号未开放')
    sk.close()

if __name__ == '__main__':
    scan_test1()
```

```python
# ---------------v2.0--------------------
"""
反馈：
    如果需要扫描服务器的话，有点麻烦，需要一直操作！
优化方案：
    1- 循环扫描
    127.0.0.1
"""
# --------------------------------------

# cmd命令窗口输入:netstat -ano查看端口号和主机名
import socket

def scan_test2():
    ip = input('请输入需要扫描的ip>>>')
    port = input('请输入需要扫描的端口号范围(0-65535)>>>')
    start_port, end_port = port.split('-')

    for one in range(int(start_port), int(end_port)+1):
        sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sk.settimeout(0.5)
        try:
            conn = sk.connect_ex((ip, one))
            if conn == 0:
                print(f'主机名{ip}, 端口号{one}已开放')
            else:
                pass
        except Exception as e:
            print(e)
        finally:
            sk.close()

if __name__ == '__main__':
    scan_test2()
```

```python
# ---------------v3.0--------------------
"""
测试反馈：
    1、只能输入ip 地址，域名怎么扫描！
    2、ip判断了没有 
优化方案：周期
    1- 解析域名
    2- 做ip有效值判断
"""
# --------------------------------------
import re
import socket


# ip的有效判断
def check_ip(ip):
    ip_address = re.compile('((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})(\.((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})){3}')
    if ip_address.match(ip) and len(ip) != 0:
        return True
    else:
        return False


# 端口扫描
def port_scan(ip):
    start_port, end_port = (8000, 8008)
    for one in range(int(start_port), int(end_port)+1):
        sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        sk.settimeout(0.5)
        conn = sk.connect_ex((ip, one))
        try:
            if conn == 0:
                print(f'{ip}主机名,{one}端口号已开放')
            else:
                pass
        except Exception as e:
            print(e)
        sk.close()

# ip扫描
def ip_scan():
    ip = input('请输入需要扫描的ip>>>')
    if check_ip(ip):
        port_scan(ip)
    else:
        print('ip格式有误~')

# 域名扫描
def domain_scan():
    domain_name = input('请输入域名>>>')
    if 'http://' in domain_name or 'https://' in domain_name:
        domain_name = domain_name[domain_name.find('://')+3:]
        # domain_name = re.split(r"http://|https://", domain_name)[1]
        print('解析的域名', domain_name)

    # 获取域名对应的ip
    server_ip = socket.gethostbyname(domain_name)
    print(f'该域名{domain_name}的ip>>>', server_ip)
    port_scan(server_ip)

# 主入口
def main():
    info = """
        1.ip扫描端口
        2.域名扫描端口
    """
    print(info)
    select = input('请输入选择>>>')
    if select == '1':
        ip_scan()
    elif select == '2':
        domain_scan()
    else:
        print('输入有误')


if __name__ == '__main__':
    main()
```

## 多线程
    1. 进程要分配一大部分内存，而线程只需要分配一部分栈就可以
    2. 一个程序至少有一个进程，一个进程至少有一个线程
    3. 进程是资源分配的最小单位，线程是程序执行的最小单位
    4. 一个线程可以创建和撤销另一个线程，同一个进程中的多个线程可以并发执行
    
    应用：
    1. 使用线程可以把占据长时间的程序中的任务放到后台去处理
    2. 用户界面可以更加吸引人，比如用户点击了一个按钮去触发某些事件的处理，可以弹出一个进度条来显示处理的进度
    3. 程序的运行速度可能加快
    4. 在一些等待的任务实现上如用户输入，文件读写和网络收发数据等，线程就比较有用了，在这种情况可以释放一些珍贵的资源如内存占用等
    
    注意：区分好串行、并行、并发的不同
    
    这里采用的是并发方式，不是并行
    
## Threading模块

    常用方法
    1. run()：表示线程活动的方法
    2. start()：启动线程活动
    3. join([time])：等待至线程终止。这阻塞调用线程直至线程的join()方法被调用中止-正常退出或抛出未处理异常-或是可选的超时未处理
    4. isAlive()：返回线程是否活动
    5. getName()：获取线程名
    6. setName()：设置线程名
    
```python
import time,os
def doing(something):

    print('正在做>>> ',something)
    time.sleep(2)

start_time = time.time()
doing('在上课')
doing('在加班')
end_time = time.time()
print('总共耗时>>> ',end_time-start_time)
os.getpid()  #pid号
```

```python
# -------------------------
# 需求：减少执行时间
# 方案：
# io密集型的场景
# ------------------------
import threading
import time,os
def doing(something):
    print('正在做>>> ',something)
    time.sleep(2)
start_time = time.time()

# 创建线程
"""
target  你这个线程是做什么---需要执行的函数名--
args  一般是一个元组 是你 target 函数x需要传递的实参
直接启动线程：主线程（main）不等待子线程(t1,t2)运行完就结束！

#需求：主线程退出前，子线程都需要做完事情！

"""
t1 = threading.Thread(target=doing,args=('我在上课！',))
t2 = threading.Thread(target=doing,args=('我在加班！',))

# 2- 启动线程
t1.start()
t2.start()

# 3- 阻塞主线程继续执行
t1.join()
t2.join()

end_time = time.time()
print('总共耗时>>> ',end_time-start_time)
```