# MOBA手游客户端制作

## 主体程序

### 脚本

```
using UnityEngine;
using System.Collections;
using System.Net;
using System.Net.Sockets;
using System;
using System.IO;
using System.Collections.Generic;

public class NetIO {

	public static NetIO instance;//单例

	public List<SocketModel> messagesList = new List<SocketModel>();

	private Socket socket;
	private string ip = "127.0.0.1";
	private int port = 6650;
	private  byte[] readbuff=new byte[1024];
	private bool isReading = false;
	List<byte> cache = new List<byte>();


	//单例对象
	public static NetIO Instance
	{
		get
		{
			if (instance == null)
			{
				instance = new NetIO();
			}
			return instance;
		}
	}

	private NetIO()//构造方法：创建客户端连接服务器的方法
	{
		try  
		{
			//创建客户端链接
			socket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
			//链接服务器
			socket.Connect(ip, port);
			// 开启异步消息接收（因为不知道什么时候连接上）,消息到达后会直接写入缓冲区 readbuff，刚到达消息肯定是字节形式的；
			socket.BeginReceive(readbuff, 0, 1024, SocketFlags.None, ReceivecallBall,readbuff);
			//0-1024缓冲区大小  SocketFlags是枚举类型是套接字发送和接收的行为有按位组合的特性
			//SocketFlags.None表明对接收行为没有任何指示
			//回调函数一旦受到信息就对回调函数ReceivecallBall进行调用
		}
		catch(Exception e)//没有连接上服务器会打印异常消息
		{
			Debug.Log(e.Message);
		}     
	}

	//收到消息后回调（主要作用就是异步消息接收）
	//结束消息接收、消息拷贝、再次开启消息接收
	private void ReceivecallBall(IAsyncResult ar)
	{
		try //保证接收的时候与服务器是正常连接的
		{
			//收到消息之后结束掉异步的接收,返回当前收到的消息的长度
			int length= socket.EndReceive(ar);
			byte[] message = new byte[length];
			//接收到的数据Copy到了新的数组message当中
			//BlockCopy参数：第一个0表示从redbuff的第一位开始copy，
			//第二个0表示在message的第一位开始写入；
			//length 表示copy的总长度
			Buffer.BlockCopy(readbuff ,0,message,0,length);
			// List<byte> cache = new List<byte>();
			cache.AddRange(message);//cache是byte类型的集合list，缓冲层（因为会有很多消息）
			//以上只是从数据流中获取的byte数组，下面要对其进行解码
			if (!isReading)//判断数据是否在读取中
			{
				isReading = true;
				OnDate();//调用数据处理方法对读取的数据进行解析，if语句确保解码时，没有在读取消息
			}
			//尾递归  读取之后，再次开启异步消息接收，防止有消息漏掉,消息到达后会直接写入缓冲区 readbuff；
			socket.BeginReceive(readbuff, 0, 1024, SocketFlags.None, ReceivecallBall, readbuff);
		}
		catch(Exception e)
		{
			Debug.Log("远程服务器主动断开连接"+e.Message);
			socket.Close();//连接异常，关闭Socket，打印异常消息
		}

	}

	//缓存中数据都是字节数组类型的，是0101形式的
	//缓存中数据处理，对消息进行解码（先是长度解码然后是消息解码）
	public void OnDate()
	{

		//有数据的话就调用长度解码
		byte[] result = decode(ref cache);

		//长度解码结果返回空的话，说明消息体不全，就不能继续往下读了，等待下条消息过来补全
		if (result==null)
		{
			isReading = false;
			return;
		}
		//消息体解码：（mdecode(result)是将长度解码的结果传到消息解码）
		//因为发送消息的时候是经过封装的，封装成了SocketModel形式的
		//解码完成之后就是SocketModel类型，所以用SocketModel来接收保存
		SocketModel message = mdecode(result);

		if (message==null)
		{
			isReading = false;
			return;    
		}
		//将消息体存储下来，等待Unity来调用。
		messagesList.Add(message);
		//尾递归，防止在消息处理过程中有其他消息到达而没有经过处理，继续处理消息
		OnDate();
	}

	//对消息体长度解码（读取消息的总长度）
	public byte[] decode(ref  List<byte> cache)
	{
		//消息体的长度识别数据都不够，肯定没数据
		if (cache.Count < 4)   return null ;

		//创建内存流对象，是将缓存数据转换成Stream类型，方便下面获取长度
		MemoryStream ms = new MemoryStream(cache.ToArray());
		//定义二进制读取流的类，读取消息的长度
		BinaryReader br = new BinaryReader(ms);//BinaryReader(Stream input)
		//获取了数据的长度
		int  length=br.ReadInt32();
		//如果获得的消息体长度大于缓存中数据长度（内存流中消息体长度），说明消息没有读取完成 ，等待下次消息到达后再次处理。
		if (length>ms.Length-ms.Position)//ms.Position表示此时读取到的位置
		{
			return null;
		}
		//读取正确数据的长度
		byte[] result = br.ReadBytes(length);
		//清空缓存
		cache.Clear();
		//前面把消息体的数据读取出来了，将读取后其他剩余的数据写入缓存
		cache.AddRange(br.ReadBytes((int)(ms.Length-ms.Position)));
		br.Close();
		ms.Close();
		return result;
	}

	//消息体解码
	public SocketModel mdecode(byte[] value)
	{
		//如何把二进制转换为SocketModel里对应的相应的类型呢，通过ByteArray类来实现
		ByteArray ba = new ByteArray(value);
		//因为发送消息传参数的时候都是SocketModel类里面的属性，所以定义它SocketModel model，然后写入到model中
		SocketModel model = new SocketModel();

		byte type;
		int area;
		int command;
		//从数据中读取三层协议，读取数据顺序必须和写入顺序保持一致，
		ba.read(out type);
		ba.read(out area);
		ba.read(out command);  
		//读取之后变量写入到model中
		model.type = type;
		model.area = area;
		model.command = command;

		//判断读取完SocketModel三层协议后，还要进行消息体读取，消息体是
		//消息体是object类型，如何将字节数组转换为object类型（用序列化反序列化）
		//序列化是把对象类型序列化为一些字节数组，字节流形式的
		if (ba.Readnable)//判断当前数据是否还有数据要读取
		{

			byte[] message;
			//将剩余的数据ba.Length-ba.Position全部读取出来存在message中
			ba.read(out message ,ba.Length-ba.Position);
			//反序列化对象（解码） model.message是object类型，message是byte[]数组
			model.message = SerializeUtil.decode(message);
		}
		ba.Close();
		return model;
	}
		
	//发送消息 调用的时候 NetIO.instance.write()
	/// <param name="type">一级协议 用于区分所属模块（比如开发过程中有战斗模块。登录模块。选择房间模块儿等）</param>
	/// <param name="area">区域：二级协议 用于区分 模块下所属子模块（场景中想要做什么功能）</param>
	/// <param name="command">命令：三级协议  用于区分当前处理逻辑功能（比如升级命令）能</param>
	/// <param name="message">消息内容：消息体 当前需要处理的主体数据</param>
	public void Write(byte type, int area, int command, object message)
	{
		#region MyRegion  //发送消息时先是消息体编码。消息写入数据流进行封装
		ByteArray ba = new ByteArray();//消息转换为二进制
		ba.write(type);
		ba.write(area);
		ba.write(command);
		if (message!=null)
		{
			ba.write(SerializeUtil.encode(message));
		}
		#endregion

		#region MyRegion   // 再长度编码
		ByteArray  arr1 = new ByteArray();
		arr1.write(ba.Length);//数据长度写入
		arr1.write(ba.getBuff());//数据写入，再次封装
		#endregion
		//发送
		try
		{
			socket.Send(arr1.getBuff());
		}
		catch (Exception e)
		{
			Debug.Log("网络错误，请重新登录"+e.Message);
		}
	}
}
```

## 单例模式

创建类NetIO.cs是单例对象，单例是为了保证它是场景中的唯一一个，单例模式方便外部调用它的方法属性等等。

### 脚本

```
public static NetIO Instance
{
    get
    {
        if (instance == null)
        {
            instance = new NetIO();
        }
        return instance;
    }
}
```

## 二进制转换

连接上服务器之后，在发送消息给服务器的时候肯定不是字符串类型或者是int类型，肯定是一个字节流类型的，所以要把我们的消息转换为二进制的结构流。将我们的字符串类型或者是int类型，或者object类型等转换成一个二进制然后写入到字节流当中。

### 脚本

```
using UnityEngine;
using System.Collections;
using System.IO;
using System;


//将消息体中信息转换为二进制存入到字节流中
//将int 、字符串类型等转换成二进制流，或者把二进制转换成int 、字符串类型等
public class ByteArray  {

	MemoryStream ms = new MemoryStream();  //创建内存流对象，并将缓存数据写进去
	BinaryWriter bw;//写入数据流的类 //写入二进制数据
	BinaryReader br;//读取数据流的类

	//关闭所有数据流
	public void Close()
	{
		bw.Close();
		br.Close();
		ms.Close();
	}
		
	//支持传入初始数据的构造
	public ByteArray(byte[] buff)
	{
		ms = new MemoryStream(buff);
		bw = new BinaryWriter(ms);
		br = new BinaryReader(ms);
	}


	//获取当前数据 读取到的下标位置，返回数据读到哪里的方法
	public int Position
	{
		get { return (int)ms.Position; }
	}
		
	//获取当前数据长度
	public int Length
	{
		get { return (int)ms.Length; }
	}

	//当前是否还有数据可以读取
	//内存中数据流的长度比此时读取到的位置要大，说明还有数据要读取。相等表示读取完毕
	public bool Readnable
	{
		get { return ms.Length > ms.Position; }
	}


	//默认构造
	public ByteArray()
	{
		bw = new BinaryWriter(ms);
		br = new BinaryReader(ms);
	}
	//写入不同的数据会调用不同的方法
	public void write(int value)
	{
		bw.Write(value);
	}
	public void write(byte value)
	{
		bw.Write(value);
	}
	public void write(bool value)
	{
		bw.Write(value);
	}
	public void write(string value)
	{
		bw.Write(value);
	}
	public void write(byte[] value)
	{
		bw.Write(value);
	}

	public void write(double value)
	{
		bw.Write(value);
	}
	public void write(float value)
	{
		bw.Write(value);
	}
	public void write(long value)
	{
		bw.Write(value);
	}

	//读取数据之后转换的类型
	public void read(out int value)
	{
		value = br.ReadInt32();//将传过来的二进制数据解析成int类型传出
	}
	public void read(out byte value)
	{
		value = br.ReadByte();
	}
	public void read(out bool value)
	{
		value = br.ReadBoolean();
	}
	public void read(out string value)
	{
		value = br.ReadString();
	}
	public void read(out byte[] value, int length)
	{
		value = br.ReadBytes(length);
	}

	public void read(out double value)
	{
		value = br.ReadDouble();
	}
	public void read(out float value)
	{
		value = br.ReadSingle();
	}
	public void read(out long value)
	{
		value = br.ReadInt64();
	}

	public void reposition()
	{
		ms.Position = 0;
	}
		
	//获取数据从字节流取出数据
	//获取字节流数据的方法，获取数据之后还要转换成字符串操作
	public byte[] getBuff()
	{
		byte[] result = new byte[ms.Length];
		Buffer.BlockCopy(ms.GetBuffer(), 0, result, 0, (int)ms.Length);
		return result;
	}

}
```

## 序列化

为了消息的安全性我们要进行编码和解码

### 脚本

```
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.Serialization.Formatters.Binary;
using System.Text;
public class SerializeUtil
{
	//对象序列化（对象转换为字节数组）
	public static byte[] encode(object value)
	{
		MemoryStream ms = new MemoryStream();//创建编码解码的内存流对象
		BinaryFormatter bw = new BinaryFormatter();//二进制流序列化对象
		//将obj对象序列化成二进制数据 写入到 内存流
		bw.Serialize(ms, value);+
		byte[] result = new byte[ms.Length];
		//将流数据拷贝到结果数组，ms.GetBuffer()是获取数据流，拷贝到result中
		Buffer.BlockCopy(ms.GetBuffer(), 0, result, 0, (int)ms.Length);
		ms.Close();//关闭二进制流序列化对象
		return result;//把消息进行了序列化
	}

	//反序列化对象（将字节流对象转换为object类型）
	public static object decode(byte[] value)
	{ 
		MemoryStream ms = new MemoryStream(value);//创建编码解码的内存流对象 并将需要反序列化的数据写入其中
		BinaryFormatter bw = new BinaryFormatter();//二进制流序列化对象
		//将流数据反序列化为obj对象
		object result = bw.Deserialize(ms);
		ms.Close();
		return result;
	}
}
```

## 消息体模型

发送消息时会进行编码， 同样接收消息时也要进行解码，相反的是我们先进行长度解码，然后是消息体解码，我们把发送的消息封装成了SocketModel形式的，解码完成之后返回的也是SocketModel类型。

### 脚本

```
using UnityEngine;
using System.Collections;
//发送接收消息，传数据用到的类型
public class SocketModel  {
	//一级协议 用于区分所属模块（比如开发过程中有战斗模块。登录模块。选择房间模块儿等）
	public byte type { get; set; }

	//二级协议 用于区分模块下所属子模块（模块中中想要做什么功能）
	public int area { get; set; }

	//三级协议  用于区分当前处理逻辑功能（比如升级命令）
	public int command { get; set; }

	//消息体 当前需要处理的主体数据（假设要传英雄模型传过来）
	public object message { get; set; } 

	public SocketModel() { }
	//构造函数
	public SocketModel(byte t,int a,int c,object o)
	{
		this.type = t;
		this.area = a;
		this.command = c;
		this.message = o;
	}

	public T GetMessage<T>()
	{
		return (T) message;
	}
}
```









