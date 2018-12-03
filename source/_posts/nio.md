---
title: 深入理解Java NIO
date: 2018-09-26 14:33:01
tags: IO
---

### All is blocking	
从严格意义上说，所有的请求都是阻塞的，原因是所有的请求发出去之后都会等待，只是等待时长多少而已，这点希望大家明白。
阻塞与非阻塞实际上就是阻塞的时间长短不同。
### Blocking vs Non-Blocking
两者不同之处在于请求（或方法调用）无法正常处理时的行为:
> 
* Blocking-mode：客户端线程会一直阻塞等待返回；
* Non-Blocking: 服务端会立即返回，通知客户端线程无法处理，客户端线程不会阻塞；

<!-- more -->
时序图如下：

  ![](http://blog.lovezhangli.top/pics/blocking.png)
  ![](http://blog.lovezhangli.top/pics/non-blocking.png)


### BIO vs NIO

BIO，就是传统的基于流的IO操作，所有基于流的操作都是阻塞的：
>
* 使用server socket启动一个服务后，服务器是阻塞的，等待客户端连接；
* 客户端服务端分别基于socket新建输入输出流，基于流的读写也是阻塞的，如果流中没有可读数据，读操作阻塞，写操作阻塞等待数据写完。

关键代码如下：

```
	public class Server {
	    public static void main(String[] args) {
	        try {
	            ServerSocket serverSocket = new ServerSocket(8888);
	            Socket socket = serverSocket.accept(); // 阻塞，等待client连接
	            System.out.println("a client connected");
	            DataOutputStream dos = new DataOutputStream(socket.getOutputStream());
	            Thread.sleep(10000l); // 阻塞client
	            dos.writeUTF("hello client");
	            dos.writeUTF("I am server"); // 写也阻塞，好像是异步(macOS)
	            System.out.println("do next thing");
	            dos.flush();
	            dos.close();
	            socket.close();
	            serverSocket.close();
	
	        } catch (IOException e) {
	            e.printStackTrace();
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	    }
	}
	
	public class Client {
	    public static void main(String[] args) {
	        Socket socket = null;
	        try {
	            socket = new Socket("localhost", 8888);
	            DataInputStream dis = new DataInputStream(socket.getInputStream());
	            System.out.println(dis.readUTF()); //阻塞，等待有数据写入
	            Thread.sleep(10000l);
	            System.out.println(dis.readUTF()); //阻塞，等待有数据写入
	            System.out.println("do next thing");
	        } catch (IOException e) {
	            e.printStackTrace();
	        } catch (InterruptedException e) {
	            e.printStackTrace();
	        }
	
	
	    }
	}
	
```

NIO，new IO，但更确切的称呼是 non-block IO，基于channel数据读写都是非阻塞的(blocking-mode:false）：
> * 服务端采用selector(多路复用器)轮训每个channel的事件；
> * 当channel中无可读数据时，读操作立刻返回空，不会阻塞;

关键代码如下：

```
    private static void query(String host) throws IOException {
        InetSocketAddress isa
                = new InetSocketAddress(InetAddress.getByName(host), port);
        SocketChannel sc = null;
        // Direct byte buffer for reading
        ByteBuffer dbuf = ByteBuffer.allocateDirect(1024);
        try {

            // Connect
            sc = SocketChannel.open();
            sc.configureBlocking(false); //
            sc.connect(isa); // non-blocking: connect() return right now, may fail to connect, must call next step
            sc.finishConnect();
            System.out.println(sc.isBlocking());


            // Read the time from the remote host.  For simplicity we assume
            // that the time comes back to us in a single packet, so that we
            // only need to read once.
            dbuf.clear();
            sc.read(dbuf); // non-block: 立即返回， block: 阻塞直到读取到内容
            // Print the remote address and the received time
            dbuf.flip();  // 复位，读操作需要复位
            CharBuffer cb = decoder.decode(dbuf);
            System.out.println(Thread.currentThread().getName() + " : " + cb);

            System.out.println(Thread.currentThread().getName() + ": Do next thing");



        } finally {
            // Make sure we close the channel (and hence the socket)
            if (sc != null)
                sc.close();
        }
    }
    
        private static void acceptConnections(int port) throws Exception {
        // Selector for incoming time requests
        Selector acceptSelector = SelectorProvider.provider().openSelector();

        // Create a new server socket and set to non blocking mode
        ServerSocketChannel ssc = ServerSocketChannel.open();
        ssc.configureBlocking(false);

        // todo: multiply serverSocketChannel ?
        ServerSocketChannel ssc1 = ServerSocketChannel.open();
        ssc1.configureBlocking(false);
        // 只有一个ServerSocketChannel
        System.out.println("ssc: " + ssc);
        System.out.println("ssc1: " + ssc1);
        // Bind the server socket to the local host and port

        InetAddress lh = InetAddress.getLocalHost();
        InetSocketAddress isa = new InetSocketAddress(lh, port);
        ssc.socket().bind(isa);

        // Register accepts on the server socket with the selector. This
        // step tells the selector that the socket wants to be put on the
        // ready list when accept operations occur, so allowing multiplexed
        // non-blocking I/O to take place.
        SelectionKey acceptKey = ssc.register(acceptSelector,
                SelectionKey.OP_ACCEPT);

        // selectionKey represent the registration of a selectable channel
        System.out.println(acceptKey);

        ssc1.register(acceptSelector, SelectionKey.OP_ACCEPT);

        int keysAdded = 0;
        System.out.println("waiting connecting....");
        // Here's where everything happens. The select method will
        // return when any operations registered above have occurred, the
        // thread has been interrupted, etc.

        while ((keysAdded = acceptSelector.select()) > 0) {
            System.out.println(keysAdded);
            // Someone is ready for I/O, get the ready keys
            Set readyKeys = acceptSelector.selectedKeys();
            Iterator i = readyKeys.iterator();

            // Walk through the ready keys collection and process date requests.
            while (i.hasNext()) {
                SelectionKey sk = (SelectionKey)i.next();
                System.out.println(sk); // equal to acceptKey
                i.remove();
                // The key indexes into the selector so you
                // can retrieve the socket that's ready for I/O
                ServerSocketChannel nextReady =
                        (ServerSocketChannel)sk.channel();
                System.out.println("nextReady: " + nextReady);
                // Accept the date request and send back the date string
                Date now = new Date();
                // write the current timestamp to buffer
                ByteBuffer writeBuf = ByteBuffer.allocateDirect(1024);
                writeBuf.putLong(now.getTime());

                SocketChannel socketChannel = nextReady.accept();
                socketChannel.configureBlocking(false);
               // Thread.sleep(10000l); // block client, client won't block
                // socketChannel.write(encoder.encode(CharBuffer.wrap(now.toString() + "\r\n")));
                StringBuffer sb = new StringBuffer(now.toString());
                for (int j = 0; j < 1000; j++) {
                    sb.append(now.toString());
                }
                int written = socketChannel.write(encoder.encode(CharBuffer.wrap(sb.toString() + "\r\n")));
                System.out.println("written: " + written);
                System.out.println("Do next thing");


            }
        }

    }
    
```
