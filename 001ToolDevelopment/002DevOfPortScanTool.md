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