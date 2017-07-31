
当我们的程序需要执行一些好事操作的时候，比如发送一条网络请求时（最常见了吧），考虑到网速等其他原因，服务器未必会立即响应我们的请求，如果不将这类操作放在子线程里取运行，就会导致主线程被阻塞，从而影响用户对软件的正常使用。我们先来学习一下线程的基本用法。

>**注**：此博文的所有知识均是阅读了《第一行代码》（第二版）之后所学知识。
	
**目录**
 
[TOC]

---
#**一、线程的三种基本用法**
Android的多线程使用并不是很特殊，基本和java使用相同的语。比如说定义一个新的线程只需要新建一个类继承自Thread，然后重写父类的 *run()* 方法，并在里面编写耗时逻辑即可。示例：
```java
class MyThread extends Thread{
	@Override
	public void run(){
		//处理具体的逻辑
	}
}
```
当我们要启动这个线程时，就可以调用这行代码启动它了：
```java
new MyThread().start();
```

当然，使用继承的方式耦合性有点高，更多时候我们也可以选择使用实现Runnable接口的方式来定义一个线程：
```java
class MyRunnable implements Runnable{
	@Override
	public void run(){
		//处理具体逻辑
	}
}
```
如果用这种方式，启动线程的方法也对应的需要更改一下：
```java
MyRunnable myRunnable = new MyRunnable();
new Thread(myRunnable).start();
```
可以看到Thread构造函数接收一个Runnable参数，而我们的myRunnable正是一个实现了Runnable接口的对象，这样我们就可以在开启一个子线程并且传进我们自己定义的Runnable，*start()*方法调用就会让*run()*中的代码在子线程中运行了。

要定义一个新的类实在太麻烦？没问题的，我们可以使用匿名类的方式来使用而且这种方法更为常见：
```java
new Thread(new Runnable(){

	@Override
	public void run(){
		//处理具体逻辑
	}
}).start();
```
```java

```
不要忘记最后的.start()哦（也就我这种渣渣才会忘记吧[捂脸]）

上面我就介绍了3种使用子线程的方法，在不同的时候就拿合适的来用吧。

---

#**二、在子线程中更新UI**

和许多其他的[GUI库][1]一样，Android的UI也是线程不安全的。也就是说，如果想要更新应用程序里的UI元素（比如给一个TextView设置新的Text），则必须在主线程中进行，否则就会出现异常。但是有时候我们必须在子线程里去执行一些耗时任务，然后根据任务的执行结果来更新相应的UI控件（TextView等），这该如何操作？

对于这种情况，Android提供了一两套一步处理消息的机制，完美的解决了在子线程中进行UI操作的问题。下面就介绍一下异步处理消息的方法。

我们先假设一个场景，UI中有一个TextView和一个Button，实现点击button创建一个子线程来更新TextView的内容。如果直接new一个Thread在*run()*里面执行更新TextView的话会使程序崩溃（自己可以试一试这里就不贴例子了）。这个时候我们就要用到Android异步消息处理了。先介绍一下吧：

Android异步消息处理主要由4各部分组成：Message、Handler、MessageQueue和Looper。简要介绍：
###**1. Message：**

Message是在线程之间传递的消息，它可以在内部携带少量的信息，用于在不同的线程之间交换数据。可以携带好几种数据：

```java
Message msg = new Message();
 msg.arg1 = 1;
 msg.arg2 = 2;//携带int类型的数据，
 msg.obj = object;//object代指你要存放的Object对象
```
### **2.Handler：**
 
 Handler顾名思义就是处理者的意思，主要用于发送和处理消息。发送消息是使用Handler的*sendMessage(Message msg)*方法，而发送出的消息经过一系列的辗转处理后，最终会传递到Handler的*handleMessage(Message msg)*方法中。
 我们就可以在主线程中new一个Handler并重写*handleMessage(Message msg)*方法来进行由Handler在子线程中通过*sendMessage(Message msg)*方法发送的Message的处理并在主线程中更新UI控件，这样就不会报错了。
 
 ###**3.MessageQueue：**
 
 根据名字我们可以知道它是消息队列的意思，主要用于存放所有通过Handler发送的消息。这部分消息会一直存放在消息队列中，等待被处理。每个线程中只会有一个MessageQueue对象。

###**4.Looper：**

Looper是每个线程中MessageQueue的管家，调用Looper的*loop()*方法后，就会静茹到一个无限循环中，然后每当发现MessageQueue中存在一条消息，就会将它去除，并传递到Handler的*handleMessage(Message msg)*方法中。每个线程中也只会有一个Looper对象。

看一下主要的流程图吧：
```sequence
Note left of msg1\nmsg2\nmsg3\n…: MessageQueue
msg1\nmsg2\nmsg3\n…->Looper: 1、取出待处理消息
Looper->Handler: 2、回调dispatchMessage()方法
Handler->handleMessage():3、处理传进的Message、
Handler->msg1\nmsg2\nmsg3\n…:（随时）发送新消息
```

让我们梳理一下整个流程：
		
首先需要在主线程中创建一个Handler对象，并重写handlerMessage()方法。然后当子线程中需要进行UI操作时，就创建一个Message对象，并通过Handler将这条消息发送出去。之后这条消息就会被添加到MessageQueue中等待被处理，而Looper则会一直尝试从MessageQueue中取出待处理消息，最后分发回Handler的handleMessage()方法中。由于Handler是在主线程中创建的，所以此时handleMessage()方法中的代码也会在主线程中运行，这样的话我们就可以安心的进行UI操作了，呼~。
		
		下面看一看前面提到的子线程需要更新UI中的TextView的情况的解决方法吧(MainActivity):
```java
public class MainActivity extends AppCompatActivity implements View.OnclickListener{
	
	public static final int UPDATE_TEXT = 1;

	private TextView text;
	
	private Handler handler = new Handler(){
		
		public void handleMessage(Message msg){
			switch (msg.what){
				case UPDATE_TEXT:
					//在这里可以进行UI操作
					text/setText("Nice to meet you")
					break;
				default:
					break;
			}
		}
	};

	@Override
	protected void onCreate(Bundle savedInstanceState){
		super.onCreate(saveInstanceState);
		setContentView(R.layout.activity_main);
		text = (TextView)findViewById(R.id.text)//这里id为虚构id，代指在布局文件中已经定义的TextView的id,下面Button同
		Button changeText = (Button)findViewById(R.id.change_text);
		changeText.setOnclickListener(this);
	}

	@Override
		public void onClick(View v){
			switch (v.getId()){
				case R.id.change_text:
					new Thread(new Runnable){
						@Override
						public void run(){
							Message msg = new Message();
							message.what = UPDATE_TEXT;
							handler.sendMessage(msg);//将Message对象发送到它自己的MessageQueue中
						}
					}).start();//不要忘了…一般写完上面一大堆代码就老是忘记这里的start()...
					break;
				default:
					break;
			}
		}
	}
```
这里我们定义了一个整形常量UPDATE_TAXT,用来表示更新TextView这个动作。然后新增一个Handler对象，并重写handleMessage()方法，在这里对具体的Message进行处理，如果发现Message的what字段的值等于UPDATE_TEXT,就将TextView中的内容改成Nice to meet you。

而发送创建并发送Message的代码被写在了子线程的run()方法中，一般我们都是在进行了耗时操作后得到了结果，然后再让一个Message携带着结果被发送到主线程中的Handler的消息队列中，接着Handler就能在handleMessage()方法中取得Message携带的耗时操作的结果，并根据这个结果来进行一些子线程不能做的UI操作~~。

好了 ，关于Android多线程的基础知识整理大概就是这么多了，谢谢浏览~~ 
>**如果想要学习更详细，可以自行阅读《第一行代码》(第二版)，不过基本一样…只是我提取出来分篇比较方便看~~**
[1]: http://baike.sogou.com/v66355812.htm?fromTitle=gui%E5%BA%93
