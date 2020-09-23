### BIO

```java
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
// 服务端
public class BIOServer {
    ServerSocket serverSocket;

    public BIOServer(int port) {
        try {
            this.serverSocket = new ServerSocket(port);
            System.out.println("服务启动, 监听端口:" + port);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void listen() throws IOException {
        while(true) {
            System.out.println("循环一次");
            Socket client = serverSocket.accept();
            InputStream inputStream = client.getInputStream();
            System.out.println("循环一次2");
            byte [] buff = new byte[1024];
            int len = inputStream.read(buff);
            System.out.println("循环一次3");
            if (len > 0) {
                String msg = new String(buff, 0,len);
                System.out.println("服务端收到:" + msg);
            }
        }
    }

    public static void main(String[] args) throws IOException {
        new BIOServer(8000).listen();
    }
}
```

### NIO

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;
import java.util.Set;

public class NIOServer {
    private int port = 8000;
    private Selector selector;
    private ByteBuffer buffer = ByteBuffer.allocate(1024);

    public NIOServer(int port) {
        this.port = port;
        try {
            ServerSocketChannel server = ServerSocketChannel.open();
            server.bind(new InetSocketAddress(this.port));
            server.configureBlocking(false);

            selector = Selector.open();
            server.register(selector, SelectionKey.OP_ACCEPT);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void listen() {
        while(true) {
            try {
                selector.select();
                Set<SelectionKey> keys = selector.selectedKeys();
                Iterator<SelectionKey> iterator = keys.iterator();
                while(iterator.hasNext()) {
                    SelectionKey key = iterator.next();
                    iterator.remove();
                    process(key);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private void process(SelectionKey key) throws IOException {
        if (key.isAcceptable()) {
            ServerSocketChannel server = (ServerSocketChannel) key.channel();
            SocketChannel channel = server.accept();
            channel.configureBlocking(false);
            key = channel.register(selector, SelectionKey.OP_READ);
        } else if (key.isReadable()) {
            SocketChannel channel = (SocketChannel) key.channel();
            int len = channel.read(buffer);
            if (len > 0) {
                buffer.flip();
                String content = new String(buffer.array(), 0, len);
                key = channel.register(selector, SelectionKey.OP_WRITE);
                key.attach(content);
                System.out.println("读取到的内容: " + content);
            }
        } else if (key.isWritable()) {
            SocketChannel channel = (SocketChannel) key.channel();
            String content = (String) key.attachment();

            channel.write(ByteBuffer.wrap(("输出:" + content).getBytes()));
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            channel.close();
        }
    }


    public static void main(String[] args) {
        new NIOServer(8000).listen();
    }
}

```



### 客户端通用

```java

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;
import java.util.UUID;

public class BIOClient {
    public static void main(String[] args) throws IOException {
        Socket client = new Socket("localhost", 8000);

        OutputStream outputStream = client.getOutputStream();
        InputStream inputStream = client.getInputStream();

        String name = UUID.randomUUID().toString();

        System.out.println("客户端发送:" + name);

        outputStream.write(name.getBytes());

//        byte [] bytes = new byte[1024];
//        int len = inputStream.read(bytes);
//
//        if (len > 0) {
//            System.out.println("客户端收到的内容:" + new String(bytes, 0, len));
//        }


        outputStream.close();
        client.close();
    }
}
```

