# unity3D计网

## 网络模型

### osi七层参考模型

应用层->处理网络应用

表示层->数据表示

会话层->互联主机通信

传输层->端到端链接

网络层->寻址和最短路径

数据链路层->接入介质

物理层->二进制传输

### 各层作用

物理层->在物理媒介中传输原始的数据比特流

数据链路层->将数据分成一个个数据帧，以数据帧为单位传输。有应有答，遇错重发

网络层->将数据分成一定长度的分组，将分组穿过通信子网，从信源选择路径后传到信宿

传输层->提供不具体网络的高效经济透明的端到端数据传输服务

会话层->进程间的对话也称为会话，会话层管理不同主机上各进程间的对话

表示层->为应用层提供格式化的表示和转换数据服务

应用层->提供应用程序访问OSI环境的手段

## TCP/IP协议

TCP/IP不是一个协议，而是一个协议族的统称

TCP/IP模型

应用层->应用层 表示层 会话层

传输层->传输层

网际层->网络层

网络接口层->数据链路层 物理层

## socket

### socket介绍

socket是一种特殊的文件，一些socket函数就是对数据进行操作（读/写IO，打开，关闭）， 当我们调用socket创建一个socket时，返回的socket描述字它存在于协议族空间中，但没有一个具体地址。如果想要给他赋值一个地址，就必须调用bind（）函数，否则就当调用connect（），listen（）时系统会自动随机分配一个端口。

```
Socket socket = new Socket(AddressFamily, SocketType, ProtocolType); //地址族 连接类型 协议类型
AddressFamily.InterNetwork //ipv4
SocketType根据ProtocolType而改变
```

```
Socket socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
IPEndPoint endpoint = new IPEndPoint(IPAddress.Any, 9999);//创建一个n的point类型的终结点//ip地址+端口号
socket.Bind(endpoint); //绑定终结点
//连接到ip地址为any端口号为9999的进程
```



## TCP编程

### 服务器端

```
//当自身为服务器时监听客户端连接
IPAddress ip = IPAddress.Parse("127.0.0.1");
int port = 5566;
TcpListener server = new TcpListener(ip,port);//直接传入ip地址和端口号
server.Start(); //开启listener
server.AcceptTcpClient(); //监听客户端连接
```

### 客户端

```
IPAddress ip = IPAddress.Parse("127.0.0.1");
int port = 5566;
TcpClient client = new TcpClient();
client.Connect(ip, port);
```

### 数据流

```
NetworkStream stream = client.GetStream();//定义数据流//将从客户端读入的数据流存放到stream中
stream.Read();

stream.Write();
```

## TCP和UDP的区别

1.基于连接与无连接

2.对系统资源的要求（TCP较多，UFO少）

3.UDP程序结构较简单

4.流模式与数据报模式

5.tcp保证数据正确性，udp可能丢包，tcp保证数据顺序，udp不保证

## UDP编程

C#中没有标准输入输出关键字，要调用console类下的方法。

```
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading;

namespace SocketText
{
    class Program
    {
        static void Main(string[] args)
        {
            UDPText.Start();
        }
    }
    class UDPText {
        public static void Start()
        {
            UdpClient client = new UdpClient(new IPEndPoint(IPAddress.Any, 0)); //IPAddress.Any代表本机上的所有IP地址，MSDN上说是“一个 IP 地址，指示服务器应侦听所有网络接口上的客户端活动”，0就代表所有可用端口了。
            IPEndPoint endPoint = new IPEndPoint(IPAddress.Parse("255.255.255.255"), 7788);//你要发送的地址
            byte[] buf = Encoding.Default.GetBytes("hello world"); //字节流 //返回一个字节数组定义一个变量来接受 //要发送的话

            Thread t = new Thread(new ThreadStart(RecvtHread));//因为他是一个线程所以同时发送接受会阻塞 //调用另一个协程（方法）
            t.IsBackground = true; //确定是否为主线程
            t.Start();
            while (true)
            {
                client.Send(buf, buf.Length, endPoint); //传送的数组 长度 地址
                Thread.Sleep(1000); //毫秒
            }
        }
        static void RecvtHread()
        {
            UdpClient client = new UdpClient(new IPEndPoint(IPAddress.Any, 7788)); //服务器端接收开放的地址和端口
            IPEndPoint endpoint = new IPEndPoint(IPAddress.Any, 0); 
            while (true) {
                byte[] buf = client.Receive(ref endpoint);
                string mag = Encoding.Default.GetString(buf); //将byte转换为string
                Console.WriteLine(mag); //打印

            }
        }
    }
}
```

## 简易聊天室

```
//服务端主程序
using System;

namespace TalkingText
{
    class Program
    {
        static void Main(string[] args)
        {
            new TalkServer().Start(); //启动服务端
        }
    }
}

```

```
//服务端代码
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Sockets;
using System.Text;

namespace TalkingText
{
    class TalkServer
    {
        IPAddress address = IPAddress.Parse("127.0.0.1");
        int port = 5566; //tcp用到的端口和本机ip

        NetworkStream stream; //字节流
        string welcome = "你好， 我是{0}，我们可以开始通话了";//0占位符
        byte[] receByte = new byte[256];

        TcpListener server;
        TcpClient client;
        string name;
        string sendName;

        public void Start() {
            Console.WriteLine("请输入您的昵称:");
            name = Console.ReadLine();
            sendName = name + ":";

            server = new TcpListener(address, port);
            server.Start();

            client = server.AcceptTcpClient(); //接收
            stream = client.GetStream();
            Talk();
        }

        public void Talk() {
            welcome = string.Format(welcome, name);
            stream.Write(GetBytes(welcome), 0, GetBytes(welcome).Length); //从0开始写入写到字符串长度位置 
            while (true) {
                int length;
                while ((length = stream.Read(receByte, 0, receByte.Length)) > 0) {
                    string receive = ParseToString(receByte);
                    receive = receive.Replace("\0", ""); //将\0替换为空格
                    Console.WriteLine(receive); //表示向控制台写入字符串后换行。

                    string strSend = "";
                    while (strSend == "") {
                        Console.Write("Input>");
                        strSend = Console.ReadLine(); //表示从控制台读取字符串后进行换行。
                    }
                    if (strSend == "exit") //退出
                    {
                        client.Close();
                        break;
                    }
                    else { 
                        stream.Write(Encoding.UTF8.GetBytes(sendName + strSend), 0, Encoding.UTF8.GetBytes(sendName + strSend).Length); //打印
                    }
                    Array.Clear(receByte, 0, receByte.Length);//清除缓冲池
                 }
                break;
            }
            server.Stop();
        }

        public byte[] GetBytes(string parseString) //将字符串转换为字节流
        { 
            return Encoding.UTF8.GetBytes(parseString);
        }
        public string ParseToString(byte[] utf8Bytes) //将字节流转换为字符串
        {
            return Encoding.UTF8.GetString(utf8Bytes);
        }
    }
}

```

```
//客户端主程序
using System;
using System.Threading;

namespace TalkClient
{
    class Program
    {
        static void Main(string[] args)
        {
            new Client().Start();
        }
    }
}

```

```
//客户端代码
using System;
using System.Collections.Generic;
using System.Net;
using System.Net.Sockets;
using System.Text;

namespace TalkClient
{

    class Client
    {
        IPAddress address = IPAddress.Parse("127.0.0.1");
        int port = 5566; //tcp用到的端口和本机ip

        NetworkStream stream; //字节流
        byte[] receByte = new byte[256];

        TcpClient client;
        string name;
        string sendMsg;

        public void Start() {
            Console.WriteLine("请输入您的昵称");
            name = Console.ReadLine();
            sendMsg = name + ":";
            client = new TcpClient("127.0.0.1", port);
            stream = client.GetStream();

            int length = 0; //判断是否存在数据流
            while ((length = stream.Read(receByte, 0, receByte.Length)) != 0) {
                string receive = ParseToString(receByte);
                receive = receive.Replace("\0", ""); //将\0替换为空格
                Console.WriteLine(receive); //表示向控制台写入字符串后换行。

                string strSend = "";
                while (strSend == "")
                {
                    Console.Write("Input>");
                    strSend = Console.ReadLine(); //表示从控制台读取字符串后进行换行。
                }
                if (strSend == "exit") //退出
                {
                    client.Close();
                    break;
                }
                else
                {
                    stream.Write(Encoding.UTF8.GetBytes(sendMsg + strSend), 0, Encoding.UTF8.GetBytes(sendMsg + strSend).Length); //打印
                }
                Array.Clear(receByte, 0, receByte.Length);//清除缓冲池
            }
        }

        public byte[] GetBytes(string parseString) //将字符串转换为字节流
        {
            return Encoding.UTF8.GetBytes(parseString);
        }
        public string ParseToString(byte[] utf8Bytes) //将字节流转换为字符串
        {
            return Encoding.UTF8.GetString(utf8Bytes);
        }
    }
}

```





















