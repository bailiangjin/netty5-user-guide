# Netty-5-用户指南-中文版
### The Problem 问题
如今，我们使用通用的应用程序或类库库相互通信。例如，我们经常使用HTTP客户端库从Web服务器获取信息，并通过Web服务来执行一个远程的调用。
但是，通用协议或其实现有时不能很好地满足需求。就像我们不使用通用的HTTP服务器来交换巨大的文件，电子邮件和近乎实时的消息，例如财务信息和多人游戏数据。所需要的是专门用于特殊目的的高度优化的协议实现。例如，你可能想要实现一个HTTP服务器，该服务器针对基于AJAX的聊天应用程序，媒体流或大文件传输进行了优化。你甚至可能想要设计和实现完全适合你的需求的全新协议。
另一个不可避免的情况是，你必须处理旧的专有协议以确保与旧系统的兼容性。在这种情况下，重要的是我们如何才能快速实现协议而不牺牲最终应用程序的稳定性和性能。
### The Solution 解决方案
Netty项目致力于提供异步事件驱动的网络应用程序框架和工具，以快速开发可维护的高性能，高可扩展性协议服务器和客户端。
换句话说，Netty是一个NIO客户端服务器框架，可以快速轻松地开发网络应用程序，例如协议服务器和客户端。它极大地简化和简化了网络编程，例如TCP和UDP套接字服务器开发。
“快速简便”并不意味着最终的应用程序会有低性能和难维护的问题。 Netty是一个精心设计的框架，在其设计过程中，吸取了很多其他协议（例如FTP，SMTP，HTTP以及各种基于二进制和文本的旧式协议）的实践经验。最终，Netty成功地找到了一种无需妥协即可轻松实现开发，高性能，高稳定性和灵活性的方法。
有些用户可能已经发现其他声称具有相同优势的网络应用程序框架，并且你可能想问一下Netty与他们有何不同。答案是建立在其基础上的哲学。 Netty旨在从第一天开始就API和实施方面为你提供最舒适的体验。这不是有形的东西，但Netty的这种设计理念，会让你在阅读本指南和使用Netty的过程中更加轻松。
### Getting Started 开始

本章围绕Netty的核心架构，通过简单的示例带你快速入门。读完本章，你将可以使用Netty编写写出一个客户端和服务器。
如果你喜欢自上而下的学习方法，那你可能要从第2章体系结构概述开始，然后回到此处。
### Before Getting Started 开始之前

运行本章介绍的示例的最低要求只有两个； Netty和JDK 1.6或更高版本的最新版本。项目下载页面中提供了Netty的最新版本。请访问你所有使用的JDK供应商的网站，自行下载相应版本的JDK。
在阅读时，你可能对本章介绍的类有更多疑问。如果你想进一步了解它们，请参考API的说明文档。为了方便起见，本文档中的所有类名都链接到在线API参考。另外，如果发现有任何不正确的信息，语法和错字错误，或者其他改进文档的建议，请联系Netty项目社区。
### Writing a Discard Server 写一个抛弃服务器

世界上最简单的协议不是“ Hello，World！”，而是DISCARD（抛弃服务）。这是一个协议，它丢弃任何接收到的数据而没有任何响应。
要实施DISCARD协议，你唯一要做的就是忽略所有接收到的数据。让我们直接从handler（处理器）的实现开始，handler用来处理Netty生成的I/O事件。

```
package io.netty.example.discard;

import io.netty.buffer.ByteBuf;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelHandlerAdapter;

/**
 * Handles a server-side channel.
 */
public class DiscardServerHandler extends ChannelHandlerAdapter { // (1)

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) { // (2)
        // Discard the received data silently.
        ((ByteBuf) msg).release(); // (3)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
        // Close the connection when an exception is raised.
        cause.printStackTrace();
        ctx.close();
    }
}
``` 
 1. DiscardServerHandler继承自ChannelHandlerAdapter，它是ChannelHandler的实现类。 ChannelHandler提供了可以覆盖的各种事件处理程序方法。目前，只需要继承ChannelHandlerAdapter即可，而不必自己去实现处理程序接口。 
 
2.在这里我们覆盖了channelRead（）事件处理程序方法。每当从客户端接收到新数据时，就使用接收到的消息调用此方法。在此示例中，接收到的消息的类型为ByteBuf。

3.要实现DISCARD协议，处理程序必须忽略收到的消息。 ByteBuf是一个引用计数对象，必须通过release（）方法显式释放。请记住处理程序的责任是释放任何传递给处理程序的引用计数对象。通常，channelRead（）处理程序方法的实现如下：

```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    try {
        // Do something with msg
    } finally {
        ReferenceCountUtil.release(msg);
    }
}
```


1.当Netty因I/O错误而引发异常或因处理事件时引发异常而由处理程序实现引发异常时，将使用Throwable调用exceptionCaught（）事件处理程序方法。在大多数情况下，应记录捕获的异常并在此处关闭其关联的通道，尽管此方法的实现可能会有所不同，具体取决于你要处理特殊情况时要采取的措施。例如，你可能想在关闭连接之前发送带有错误代码的响应消息。

到目前为止一切都很好。我们已经实现了DISCARD服务器的前半部分。现在剩下的是编写main（）方法来启动服务端的DiscardServerHandler。

```
package io.netty.example.discard;
    
import io.netty.bootstrap.ServerBootstrap;

import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
    
/**
 * Discards any incoming data.
 */
public class DiscardServer {
    
    private int port;
    
    public DiscardServer(int port) {
        this.port = port;
    }
    
    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); // (1)
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap(); // (2)
            b.group(bossGroup, workerGroup)
             .channel(NioServerSocketChannel.class) // (3)
             .childHandler(new ChannelInitializer<SocketChannel>() { // (4)
                 @Override
                 public void initChannel(SocketChannel ch) throws Exception {
                     ch.pipeline().addLast(new DiscardServerHandler());
                 }
             })
             .option(ChannelOption.SO_BACKLOG, 128)          // (5)
             .childOption(ChannelOption.SO_KEEPALIVE, true); // (6)
    
            // Bind and start to accept incoming connections.
            ChannelFuture f = b.bind(port).sync(); // (7)
    
            // Wait until the server socket is closed.
            // In this example, this does not happen, but you can do that to gracefully
            // shut down your server.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
    
    public static void main(String[] args) throws Exception {
        int port;
        if (args.length > 0) {
            port = Integer.parseInt(args[0]);
        } else {
            port = 8080;
        }
        new DiscardServer(port).run();
    }
}
```

1. NioEventLoopGroup是一个处理I/O操作的多线程事件循环。 Netty为不同类型的传输提供了各种EventLoopGroup实现。我们在此示例中实现服务器端应用程序，因此将使用两个NioEventLoopGroup。第一个通常称为“boss”，接受传入的连接。第二个通常称为“worker”，一旦boss接受连接并将注册的连接注册给worker，便处理已接受连接的流量。使用多少个线程以及如何将它们映射到创建的通道取决于EventLoopGroup的实现，并且可以通过构造函数进行配置。
2. ServerBootstrap是启动NIO服务的辅助启动类。可以直接使用Channel启动一个服务。但这是一个单调乏味的过程，很多情况下你并不需要这样做。 
3. 在这里我们指定使用NioServerSocketChannel类来举例说明一个新的Channel如何接收进来的连接。
4. 此处指定的处理程序将始终由新接受的Channel评估。 ChannelInitializer是一个特殊的处理程序，旨在帮助用户配置新的Channel。你很可能希望通过添加一些处理程序（例如DiscardServerHandler）来配置新Channel的ChannelPipeline来实现你的网络应用程序。随着应用程序变得复杂，你可能会向管道添加更多处理程序，并最终将此匿名类提取到顶级类中。
5. 你还可以设置某个特定Channel实现的配置参数。我们正在编写一个TCP/IP服务器，因此我们可以设置socket的参数选项，例如tcpNoDelay和keepAlive。请参考ChannelOption的apidocs和特定的ChannelConfig实现以对ChannelOptions有一个大体的认识。
6. 你是否关注过option（）和childOption（）？ option（）用于接受传入连接的NioServerSocketChannel。 childOption（）提供给父管道ServerChannel接收到的连接，在这个例子中，父管道为NioServerSocketChannel。
7. 我们继续。剩下的就是绑定端口并启动服务器。在这里，我们绑定到机器中所有网卡的8080端口。现在你可以根据需要，多次调用bind（）方法（使用不同的绑定地址）。

恭喜！你已经完成了一个基于Netty的服务端。

### Looking into the Received Data 查看接收到的数据
现在，我们已经编写了第一个服务端，我们需要测试它能否正常运行。最简单的测试方法是使用telnet命令。例如，你可以在命令行中输入telnet localhost 8080并输入一些内容。
但是，我们可以说这个服务端工作能正常运行吗？事实上，我们也不知道。因为它是一个discard服务，你根本不会得到任何回应。为了证明它确实有效，让我们修改服务端的应用程序，将其收到的内容打印出来。
我们已经知道，每当收到数据时都会调用channelRead（）方法。让我们将一些代码放入DiscardServerHandler的channelRead（）方法中：

```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    ByteBuf in = (ByteBuf) msg;
    try {
        while (in.isReadable()) { // (1)
            System.out.print((char) in.readByte());
            System.out.flush();
        }
    } finally {
        ReferenceCountUtil.release(msg); // (2)
    }
}
```
1.这个低效的循环实际上可以简化为：System.out.println（in.toString（io.netty.util.CharsetUtil.US_ASCII））
2.或者，你可以在这里调用in.release（）。
如果再次运行telnet命令，你将看到服务端打印收到的内容。
discard服务端的完整源代码位于发行版的io.netty.example.discard包下。

### Writing an Echo Server 编写一个Echo服务端
到目前为止，我们虽然收到了数据，却没有做任何响应。然而，通常一个服务端需要对请求作出响应。让我们学习如何通过实现ECHO协议将响应消息给客户端，在该协议中，服务端接收到任何的数据都将被回写给客户端。
与前面几节中实现的discard服务端的唯一区别在于，它会将接收到的数据发回，而不是将接收到的数据打印到控制台。因此，需要把channelRead（）方法修改如下：
```
 @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ctx.write(msg); // (1)
        ctx.flush(); // (2)
    }
```

1. ChannelHandlerContext对象提供了很多操作，使你能够触发各种I / O事件和操作。在这里，我们调用write（Object）以逐字记录接收到的消息。请注意，我们没有像在DISCARD示例中那样释放收到的消息。这是因为Netty在将其写到网络时会为你释放它。
2. ctx.write（Object）不会将消息写出到通道中。它在内部进行缓冲，然后通过ctx.flush（）将数据输出到通道中。或者，你可以用更加简洁的方式，调用ctx.writeAndFlush（msg）达到相同的目的。
如果再次运行telnet命令，你将发现，无论你给服务端发送什么，服务端都会讲你发送的内容给返回回来。

echo服务端的完整源代码位于发行版的io.netty.example.echo包下。

### Writing a Time Server 写个时间服务器

这部分实现的协议是TIME协议。它与前面的示例不同之处在于，它不包含任何请求就发送包含32位整数的消息，并在发送消息后关闭连接。在此示例中，你将学习如何构造和发送消息以及如何在完成时关闭连接。
由于我们将忽略任何接收到的数据，而是在建立连接后立即发送消息，因此这次我们无法使用channelRead（）方法。相反，我们应该重写channelActive（）方法。以下是具体实现：
```
package io.netty.example.time;

public class TimeServerHandler extends ChannelHandlerAdapter {

    @Override
    public void channelActive(final ChannelHandlerContext ctx) { // (1)
        final ByteBuf time = ctx.alloc().buffer(4); // (2)
        time.writeInt((int) (System.currentTimeMillis() / 1000L + 2208988800L));
        
        final ChannelFuture f = ctx.writeAndFlush(time); // (3)
        f.addListener(new ChannelFutureListener() {

            @Override
            public void operationComplete(ChannelFuture future) {
                assert f == future;
                ctx.close();
            }
        }); // (4)
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

1. 如前所述，建立连接并准备进行通信时，将调用channelActive（）方法。让我们写一个代表该方法当前时间的32位整数。
2. 要发送新消息，我们需要分配一个包含这个消息的新缓冲区。我们将要写入一个32位整数，因此我们需要一个容量至少为4个字节的ByteBuf。通过ChannelHandlerContext.alloc（）获取当前的ByteBufAllocator并分配一个新的缓冲区。
3. 与往常一样，我们编写一个构造好的消息。但是等一等，flip在哪里？在NIO中发送消息之前，我们不是曾经调用过java.nio.ByteBuffer.flip（）吗？ ByteBuf没有这个方法，因为它有两个指针;一个用于读取操作，另一个用于写入操作。当你向ByteBuf中写入内容时，写指针的索引会增加，而读指针索引不会改变。写指针索引和写指针索引分别表示消息的开始和结束位置。    
相较而言，NIO缓冲区没有提供一种简洁的的方法来确定消息内容的开始和结束位置，除非你调用flip方法。当你忘记调用flip方法时，会遇到没有发送任何束缚或发送数据异常的麻烦。而Netty中不会发生这样的错误，因为我们对不同的操作类型有不同的指针。当你习惯了这种不需调用flip的方式，你会发现这种调用方式会让你的使用变的更加的便利！  
还要注意的一点是ChannelHandlerContext.write（）（和writeAndFlush（））方法返回ChannelFuture。 ChannelFuture表示尚未发生的I / O操作。这意味着，任何一个请求操作都不会立即执行，因为Netty中所有的操作都是异步的。例如，以下示例代码中的消息，在被发送之前，channel可能已经关闭：
```
Channel ch = ...;
ch.writeAndFlush(message);
ch.close();
```
因此，你需要在write（）返回的ChannelFuture完成之后调用close（）方法，并在完成写操作后通知其侦听器。请注意，close（）可能也不会立即关闭连接，它也会返回ChannelFuture。


4. 当一个写请求完成时，我们如何得到通知？只需在往返的ChannelFuture上添加一个ChannelFutureListener。这里我们创建了一个匿名ChannelFutureListener类，用来在操作完成时关闭Channel。  
或者，你可以使用预定义侦听器方式，更简单地完成上述操作：
```
f.addListener(ChannelFutureListener.CLOSE);
```  
要测试我们的时间服务端能否像预期的一样工作，可以使用UNIX 的 rdate命令：   
```
$ rdate -o <port> -p <host>
```    
其中<port>是你在main（）方法中指定的端口号，而<host>使用localhost即可

### Writing a Time Client 编写时间客户端
与DISCARD和ECHO服务端不同，对于TIME协议，我们需要一个客户端，因为人类无法将32位二进制数据直接理解为日历中的具体日期。在本节中，我们讨论如何确保服务端正常工作，并学习如何使用Netty编写一个客户端。
Netty中服务端与客户端之间最大且唯一的区别是使用了不同的Bootstrap和Channel实现。请看下面的代码：
```
package io.netty.example.time;

public class TimeClient {
    public static void main(String[] args) throws Exception {
        String host = args[0];
        int port = Integer.parseInt(args[1]);
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        
        try {
            Bootstrap b = new Bootstrap(); // (1)
            b.group(workerGroup); // (2)
            b.channel(NioSocketChannel.class); // (3)
            b.option(ChannelOption.SO_KEEPALIVE, true); // (4)
            b.handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new TimeClientHandler());
                }
            });
            
            // Start the client.
            ChannelFuture f = b.connect(host, port).sync(); // (5)

            // Wait until the connection is closed.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
        }
    }
}
```

1. Bootstrap与ServerBootstrap相似，但它是用于非服务端channel，例如客户端或无连接传输模式的channel。
2. 如果只指定一个EventLoopGroup，它将同时用作boss group和worker group。尽管boss    worker并不用于客户端。
3. 客户端创建channel时，使用 NioSocketChannel代替了NioServerSocketChannel。
4. 请注意，与我们对ServerBootstrap所做的不同，这里我们不使用childOption（），因为客户端SocketChannel没有父类。
5. 我们应该调用connect（）方法而不是调用bind（）方法。

如你所见，客户端与与服务端代码并没有真正的区别。那ChannelHandler的实现呢？它应该从服务端接收一个32位整数，将其翻译为人们可以读懂的格式，打印翻译后的时间，然后关闭连接：
```
package io.netty.example.time;

import java.util.Date;

public class TimeClientHandler extends ChannelHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg; // (1)
        try {
            long currentTimeMillis = (m.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        } finally {
            m.release();
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}

```

1.在TCP/IP中，Netty把接收到的数据读取到ByteBuf中。
它看起来非常简单，并且与服务器端示例没有任何不同。但是，此处理程序有时会因抛出IndexOutOfBoundsException而拒绝工作。我们将在下一节讨论为什么会发生这种情况。

### Dealing with a Stream-based Transport 处理基于流的传输
#### One Small Caveat of Socket Buffer 关于Socket Buffer的一个小警告
在基于流的传输（例如TCP/IP）中，接收到的数据是存储在Socket Buffer中的。不幸的是，基于流的传输的缓冲区不是数据包队列而是字节队列。这意味着，即使你将两个消息作为两个独立的数据包发送，操作系统也不会将它们视为两个消息，而只是一堆字节。因此，不能保证你接收到的内容与你的远程对等方写的完全一样。例如，让我们假设操作系统的TCP/IP堆栈已收到3个数据包：


由于基于流传输的协议的这种基本的性质，因此在你的应用程序中读取数据时，数据很有可能会被分成下面的片段：

因此，一个接收端，无论是服务器端还是客户端，都应将接收到的数据整理到一个或多个有意义的帧中，以使应用程序逻辑易于理解。在上面的示例中，接收到的数据应构造成以下格式：

#### The First Solution 第一个解决方案
现在让我们回到TIME客户示例。我们在这里有同样的问题。 32位整数是非常少量的数据，并且不太可能经常被拆分。但是，问题在于它可以被拆分，并且被拆分的可能性会随着通信量的增加而增加。
一种简单的解决方案是创建一个内部累积缓冲区，然后等待直到所有4个字节都被接收到内部缓冲区中为止。以下是修改后的TimeClientHandler实现，可解决上述问题：
```
package io.netty.example.time;

import java.util.Date;

public class TimeClientHandler extends ChannelHandlerAdapter {
    private ByteBuf buf;
    
    @Override
    public void handlerAdded(ChannelHandlerContext ctx) {
        buf = ctx.alloc().buffer(4); // (1)
    }
    
    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) {
        buf.release(); // (1)
        buf = null;
    }
    
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) {
        ByteBuf m = (ByteBuf) msg;
        buf.writeBytes(m); // (2)
        m.release();
        
        if (buf.readableBytes() >= 4) { // (3)
            long currentTimeMillis = (buf.readUnsignedInt() - 2208988800L) * 1000L;
            System.out.println(new Date(currentTimeMillis));
            ctx.close();
        }
    }
    
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```


1. ChannelHandler具有两个生命周期侦听方法：handlerAdded（）和handlerRemoved（）。你可以执行初始化或取消初始化任务，只要它不会长时间被阻塞即可。
2. 首先，所有接收到的数据应累计到Buf中。
3. 然后，处理程序必须检查buf是否有足够的数据（在此示例中为4字节），然后继续进行实际的业务逻辑。否则，当更多数据到达时Netty将再次调用channelRead（）方法，直到最终累计够4个字节数据。

#### The Second Solution 第二种解决方案
尽管第一个解决方案已经解决了TIME客户端的问题，但修改后的处理程序看起来并不简洁。想象一下，当遇到一个由多个字段（例如可变长度字段）组成的更复杂的协议时。你的ChannelHandler实现将很快变得难以维护。
你可能已经注意到，你可以在一个ChannelPipeline中添加多个ChannelHandler，因此可以将一个整个ChannelHandler拆分为多个模块化的模块，以降低应用程序的复杂性。例如，你可以将TimeClientHandler分为两个处理器：
•TimeDecoder，处理数据的拆分问题
•TimeClientHandler作为原始版本的实现。
幸运的是，Netty提供了一个可扩展的类，帮助你实现TimeDecoder：
```
package io.netty.example.time;

public class TimeDecoder extends ByteToMessageDecoder { // (1)
    @Override
    protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) { // (2)
        if (in.readableBytes() < 4) {
            return; // (3)
        }
        
        out.add(in.readBytes(4)); // (4)
    }
}
```

1. ByteToMessageDecoder是ChannelHandler的实现，可以轻松处理数据拆分问题。
2. 每当接收到新数据时，ByteToMessageDecoder就会decode（）方法来处理内部缓存区缓存的数据。
3. 当累积缓冲区中没有足够的数据时，decode（）可以决定不输出任何内容。直到收到更多数据时，ByteToMessageDecoder将再次调用decode（）对数据进行解析。
4. 如果decode（）将对象添加到输出中，则表示解码器成功解码了一条消息。 ByteToMessageDecoder将丢弃累积缓冲区的读取部分。请记住，你不需要解码多条消息。 ByteToMessageDecoder将继续调用decode（）方法，直到不输出任何内容为止。
现在我们有另一个处理器要插入ChannelPipeline中，我们应该在TimeClient中修改ChannelInitializer实现：

```
b.handler(new ChannelInitializer<SocketChannel>() {
    @Override
    public void initChannel(SocketChannel ch) throws Exception {
        ch.pipeline().addLast(new TimeDecoder(), new TimeClientHandler());
    }
});
```

如果你是一个喜欢冒险的人，则可能需要尝试使用ReplayingDecoder来进一步简化解码器。不过，你将需要查阅API参考以获取更多信息。
```
public class TimeDecoder extends ReplayingDecoder<Void> {
    @Override
    protected void decode(
            ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
        out.add(in.readBytes(4));
    }
}

```

此外，Netty提供了开箱即用的解码器，使你能够非常轻松地实现大多数协议，并避免最终以一个庞大、不可维护的handler实现而告终。请参考下面的包以获取更多详细示例：
•对于二进制协议请参考：io.netty.example.factorial 
•对于文本协议请参考：io.netty.example.telnet 
### Speaking in POJO instead of ByteBuf 用POJO代替ByteBuf
到目前为止，回顾我们上述所有示例都用ByteBuf作为协议消息的主要数据结构。在本节中，我们将改进TIME协议客户端和服务端的示例，使用POJO代替ByteBuf。

在ChannelHandlers中使用POJO的优势显而易见，通过将解析ByteBuf的代码从ChannelHandler中抽取出来，会使ChannelHandler更容易维护和复用。在TIME客户端和服务器示例中，我们仅读取一个32位整数，并且直接使用ByteBuf并不是主要问题。但是，你会发现在实现实际协议时有必要进行分离。

首先，让我们定义一个新的类型叫做UnixTime。
```
package io.netty.example.time;

import java.util.Date;

public class UnixTime {

    private final long value;
    
    public UnixTime() {
        this(System.currentTimeMillis() / 1000L + 2208988800L);
    }
    
    public UnixTime(long value) {
        this.value = value;
    }
        
    public long value() {
        return value;
    }
        
    @Override
    public String toString() {
        return new Date((value() - 2208988800L) * 1000L).toString();
    }
}

```
我们现在可以修改TimeDecoder来返回UnixTime替代ByteBuf。
```
@Override
protected void decode(ChannelHandlerContext ctx, ByteBuf in, List<Object> out) {
    if (in.readableBytes() < 4) {
        return;
    }

    out.add(new UnixTime(in.readUnsignedInt()));
}
```
下面是修改后的解码器，TimeClientHandler 不再任何的 ByteBuf 代码了。
```
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) {
    UnixTime m = (UnixTime) msg;
    System.out.println(m);
    ctx.close();
}
```
是不是变得更加简单和优雅了？相同的技术也可以被运用到服务端。让我们首先更新一下TimeServerHandler的代码。
```
@Override
public void channelActive(ChannelHandlerContext ctx) {
    ChannelFuture f = ctx.writeAndFlush(new UnixTime());
    f.addListener(ChannelFutureListener.CLOSE);
}
```
现在，唯一缺少的部分是编码器，是ChannelHandler的实现，可以将UnixTime转换为ByteBuf。这比编写解码器要简单得多，因为在编码消息时无需处理数据包编码消息时拆分和组装。
```
package io.netty.example.time;

public class TimeEncoder extends ChannelHandlerAdapter {
    @Override
    public void write(ChannelHandlerContext ctx, Object msg, ChannelPromise promise) {
        UnixTime m = (UnixTime) msg;
        ByteBuf encoded = ctx.alloc().buffer(4);
        encoded.writeInt((int) m.value());
        ctx.write(encoded, promise); // (1)
    }
}
```

1. 在上面这个handler方法中，有几件事情需要注意：  
 第一，我们按原样传递原始的ChannelPromise，以便在将编码数据实际写到线路时，Netty将其标记为成功或失败。  
 第二，我们不再需要调用ctx.flush（）。因为处理有一个单独的处理程序方法void flush（ChannelHandlerContext ctx），覆盖了flush（）方法，帮我们完成了这些。

为了进一步简化，你可以使用MessageToByteEncoder：
```
public class TimeEncoder extends MessageToByteEncoder<UnixTime> {
    @Override
    protected void encode(ChannelHandlerContext ctx, UnixTime msg, ByteBuf out) {
        out.writeInt((int) msg.value());
    }
}
```

剩下的最后一项任务是在TimeServerHandler之前将TimeEncoder插入到服务器端的ChannelPipeline中。但这是不那么重要的工作。

### Shutting Down Your Application 关闭你的应用程序

关闭Netty应用往往只需要通过调用ShutdownGracefully（）便可优雅地关闭你所构建的所有EventLoopGroup。当EventLoopGroup被完全终止，相应的所有channel均已关闭时，Netty 会返回一个Future对象来通知你。
### Summary 总结
在本章中，我们快速浏览了Netty，并演示了如何在Netty之上编写功能全面的网络应用程序。
在接下来的章节中将有关于Netty的更多详细信息。我们也鼓励你查看io.netty.example包中的Netty示例。
另请注意，Netty社区一直期待你你提出问题和想法来帮助你，并根据你的反馈不断改进Netty及Netty的相关文档。

### 英文原文文章来源
Netty官网：
https://netty.io/wiki/user-guide-for-5.x.html


