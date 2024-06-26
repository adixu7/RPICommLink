RPICommLink 是一个用于在不同设备之间建立通信连接的 Python 类。提供了一系列方法使设备能够通过 TCP/IP 协议进行数据交换。  
该库用于我的毕业设计，本意是用于树莓派于电脑之间的数据传输。因为树莓派的算力不够无法在树莓派进行模型验证与图像识别，所以将数据以局域网的方式进行传输。  
该项目已上传pypi，地址为：https://pypi.org/project/RPICommLink/  
可以有python环境的终端中通过以下方式下载：

```pip install RPICommLink```

它不仅仅只能在电脑于树莓派之间进行数据传输，电脑于电脑之间的传输也是可以的。目前功能还未完善。  
## 服务器端：
```python
import RPICommLink
server = RPICommLink.RPICommLink()  #实例化
server.open()  #打开服务器
print(server.recv())  #等待接收
```

将库实例化，它会设置一些必要参数。  
端口rpi_port与密码password，它们默认的端口为11451与88888888，一般可不做修改。接收端连接的端口与密码必须与服务器保持一致。
open函数会打开本机IP地址监听连接，它们有以下三个参数：  
```
device_name：服务器的设备名称，默认设备名称为‘default’ 。
max_connect：最大的客户端连接数量，如果它为0，则不作限制。默认为0。  
ip: ip地址，如果无法正确打开IP地址，请自行查看本地IP地址设置。一般来说未联网的设备会有一定几率无法获取到当前的IP地址
```
RPICommLink.recv()是一个阻塞线程，它会一直等待直到信息发送过来  

## 客户端：
```python
import RPICommLink
client = RPICommLink.RPICommLink()  #实例化
client.connect()  #连接服务器（一般来说，如果服务器的端口，密码，设备名称都是默认的，则不作修改）
client.send('hello world!')  #发送
```
将库实例化，它会设置一些必要参数。
端口rpi_port与密码password，它们默认的端口为11451与88888888，一般可不做修改。接收端连接的端口与密码必须与服务器保持一致。
connect函数会尝试连接当前局域网子网的256个IP地址，它们有以下两个参数：  
```
subnet_ip:子网，IP地址前三个数，格式为str‘xxx.xxx.xx’。
target_name: 指定的设备名字。
```
send函数会向已连接的服务器发送数据，该数据已在函数内进行utf-8编码。  
示例函数的运行结果为： 


服务端：  
![image](https://github.com/adixu7/RPICommLink/assets/169100555/c2c5184c-d4fd-4ad3-bbc4-efc893816257)   
客户端：  
![image](https://github.com/adixu7/RPICommLink/assets/169100555/d2c8477a-1992-4d13-9b59-c0c3e7566849)  

****
## 1.1.1

1.1.1的版本中增加了依赖opencv-python库的图像发送与摄像头帧的发送，下载RPICommLink库并不会通过依赖直接下载opencv-python，但可以通过以下方式在终端下载opencv-python：
```pip install opencv-python```
由于目前还未优化，所以建议只在服务端进行发送，由客户端进行接收。  
可以使用auto_frame()在服务端进行图像传输，它会自动打开摄像头并向客户端发送摄像头的当前帧，注意：该函数不是一个阻塞函数，请不要在循环中使用该函数。
可以使用send_frame(frame)发送单张cv2格式的图片。  

在客户端中，使用recv_frame来接收从服务器发送的图片。  
可以使用以下两端代码来尝试对图像的发送与接收
#### 服务端：
```python
import RPICommLink
server = RPICommLink.RPICommLink()
server.open()
server.auto_frame()
while True:
    print(server.recv())
```

#### 客户端：
```python
import RPICommLink
import cv2
server = RPICommLink.RPICommLink()
server.connect()
server.send('hello world!')
while True:
    frame = server.recv_frame()
    cv2.imshow('show', frame)
    if cv2.waitKey(1) == ord('q'):
        break
```


****
## 1.1.2

1.1.2之后就可以进行双向通信了，注意，recv()是一个阻塞线程，它会一直等待直到收到信息，可以使用线程来处理recv()收到的信息。
下面是更新后的示例代码
#### 服务端
```python
import RPICommLink
server = RPICommLink.RPICommLink()
server.open()
print('client:', server.recv())
server.send('Roger that')
print('client:', server.recv())
server.send('Server end the connection')
```
运行结果：  
![image](https://github.com/adixu7/RPICommLink/assets/169100555/bbd953e4-a192-4470-b590-f6f336950777)

#### 客户端
```python
import RPICommLink
client = RPICommLink.RPICommLink()
client.connect()
client.send('hello server')
print('server:', client.recv())
client.send('Client end the connection')
print('server:', client.recv())
```
运行结果：  
![image](https://github.com/adixu7/RPICommLink/assets/169100555/2338c498-d69d-4919-b1c4-71b0c622253a)

****
## 1.1.3

1.1.3更新了装饰器的用法，它能在不影响主线程的情况下处理接收到的信息。
下面是更新后的示例代码
#### 服务端
```python
import RPICommLink
server = RPICommLink.RPICommLink()
server.open()


@server.recv_msg  # 使用recv_msg
def handle_msg(msg):
    print('recv:', msg)  # 可以在这里处理接收到的数据，它不会阻塞主线程，当接收到数据时，它会运行一次并传参给msg


while True:
    server.send(input())

```
#### 客户端
```python
import RPICommLink
client = RPICommLink.RPICommLink()
client.connect()


@client.recv_msg  # 使用recv_msg
def handle_msg(msg):
    print('recv:', msg)  # 可以在这里处理接收到的数据，它不会阻塞主线程，当接收到数据时，它会运行一次并传参给msg


while True:
    client.send(input())

```
运行结果：  
![0fba9b30aae0d0ccd7833c162cba5b6](https://github.com/adixu7/RPICommLink/assets/169100555/b0b37fbb-60d2-4444-a962-24856316c8d2)

****
## 1.1.4
1.1.4可以选择传输时的摄像头，如果你想在一台设备上同时发送两个摄像头的影像，甚至更多，可以按照以下方法开启不同的端口
#### 服务器
```python
import RPICommLink
Camera_one = RPICommLink.RPICommLink(rpi_port=11111)
Camera_two = RPICommLink.RPICommLink(rpi_port=22222)
Camera_two.open()
Camera_one.open()
Camera_two.auto_frame()
Camera_one.auto_frame(cap=1)

@Camera_one.recv_msg
def one_msg(msg):  # 这里接收第一个摄像头客户端发来的数据
    print(msg)

@Camera_two.recv_msg
def two_msg(msg):  # 这里接收第二个摄像头客户端发来的数据
    print(msg)
```
#### 客户端
```python
import RPICommLink
import cv2

Camera_one = RPICommLink.RPICommLink(rpi_port=11111)
Camera_one.connect()
Camera_two = RPICommLink.RPICommLink(rpi_port=22222)
Camera_two.connect()

# 使用装饰器可以使两个摄像头相互不影响
@Camera_one.recv_frame_test
def Camera_one_frame(recv_frame):
    global frame_one
    frame_one = recv_frame


@Camera_two.recv_frame_test
def jin_frame(frame):
    global frame_two
    frame_two = frame


while True:
    try:
        cv2.imshow('0', frame_one)
        cv2.imshow('1', frame_two)
        if cv2.waitKey(1) == ord('q'):
            break
    except Exception as e:
        print(e)
        pass

```
当然，客户端也可以接收不同设备发来的不同图片数据，只需要实例化并在open()与connect()中设置不同的设备名称，用以上的方法即可！

历代版本更新：

	1.0.2 -优化了函数的命名，发送模块不再进行死循环。
 
	1.0.3 -修复了潜在bug，优化了获取IP的方式，现在可以不命名而使用默认设备名称了。添加了中文介绍。
 
	1.0.4 -重新了上传中文介绍。
 
	1.0.5 -修复了无法在无互联网下获取IP的报错，修复了无法同时连接两个服务器的bug。
 
	1.0.6 -修复了潜在bug
 
	1.0.7 -轻量化
 
	1.1   -增加了传输摄像头帧的函数，需要自行下载opencv-python库。注意：该版本为测试版，将会有许多未发现的报错，建议在在服务器端发送摄像头帧而不是客户端。
 
	1.1.1 -修复了几个bug。注意：该版本为测试版，将会有许多未发现的报错，建议在在服务器端发送摄像头帧而不是客户端。

 	1.1.2 -现在服务端与客户端都可以进行发送或接收了。

  	1.1.3 -添加了装饰器的用法，使代码更加简介。

   	1.1.4 -给图像接收添加了装饰器的用法；auto_frame现在可以选择不同的相机了

 对代码有任何问题，可以通过邮箱adixu7@gmail.com联系我
