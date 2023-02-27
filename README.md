## GRPC Example in Karaf
The repository contains  an example of how to generate stub code using maven, how to implement a client/server from generated stub code, and provides a karaf feature file allowing this to be deployed in the karaf container.

## Purpose of the fork
Aim is to debug/determine the root cause why gRPC server works in vanilla karaf and does not work in ONOS' karaf. gRPC client is disabled in this fork.

## Issue description 
- while the gRPC server works in vanilla karaf, it crashes in ONOS' karaf (same version of karaf 4.2.9) when attempt to connect is made with following exception:
```shell
2023-02-27T14:22:02,449 | WARN  | grpc-default-worker-ELG-3-1 | ChannelInitializer               | 137 - io.netty.common - 4.1.35.Final | Failed to initialize a channel. Closing: [id: 0x9572574e, L:/172.25.0.3:5000 - R:/172.25.0.1:37622]
java.lang.NoSuchMethodError: 'void io.netty.handler.codec.http2.DefaultHttp2HeadersDecoder.<init>(int, int, boolean)'
        at io.grpc.netty.NettyServerHandler.newHandler(NettyServerHandler.java:110) ~[?:?]
        at io.grpc.netty.NettyServerTransport.createHandler(NettyServerTransport.java:118) ~[?:?]
        at io.grpc.netty.NettyServerTransport.start(NettyServerTransport.java:76) ~[?:?]
        at io.grpc.netty.NettyServer$1.initChannel(NettyServer.java:136) ~[?:?]
        at io.netty.channel.ChannelInitializer.initChannel(ChannelInitializer.java:129) [!/:4.1.35.Final]
        at io.netty.channel.ChannelInitializer.handlerAdded(ChannelInitializer.java:112) [!/:4.1.35.Final]
        at io.netty.channel.AbstractChannelHandlerContext.callHandlerAdded(AbstractChannelHandlerContext.java:969) [!/:4.1.35.Final]
        at io.netty.channel.DefaultChannelPipeline.callHandlerAdded0(DefaultChannelPipeline.java:610) [!/:4.1.35.Final]
        at io.netty.channel.DefaultChannelPipeline.access$100(DefaultChannelPipeline.java:46) [!/:4.1.35.Final]
        at io.netty.channel.DefaultChannelPipeline$PendingHandlerAddedTask.execute(DefaultChannelPipeline.java:1461) [!/:4.1.35.Final]
        at io.netty.channel.DefaultChannelPipeline.callHandlerAddedForAllHandlers(DefaultChannelPipeline.java:1126) [!/:4.1.35.Final]
        at io.netty.channel.DefaultChannelPipeline.invokeHandlerAddedIfNeeded(DefaultChannelPipeline.java:651) [!/:4.1.35.Final]
        at io.netty.channel.AbstractChannel$AbstractUnsafe.register0(AbstractChannel.java:515) [!/:4.1.35.Final]
        at io.netty.channel.AbstractChannel$AbstractUnsafe.access$200(AbstractChannel.java:428) [!/:4.1.35.Final]
        at io.netty.channel.AbstractChannel$AbstractUnsafe$1.run(AbstractChannel.java:487) [!/:4.1.35.Final]
        at io.netty.util.concurrent.AbstractEventExecutor.safeExecute(AbstractEventExecutor.java:163) [!/:4.1.35.Final]
        at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:405) [!/:4.1.35.Final]
        at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:500) [!/:4.1.35.Final]
        at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:906) [!/:4.1.35.Final]
        at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) [!/:4.1.35.Final]
        at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30) [!/:4.1.35.Final]
        at java.lang.Thread.run(Thread.java:834) [?:?]
```
### How to replicate the issue

#### Pre-requirements
- docker [installed](https://docs.docker.com/get-docker/)
- docker-compose [installed](https://docs.docker.com/compose/install/other/)
- maven and java configured

#### Steps to replicate
- build and install the project to local maven repository
```shell
bpilka@bpilka-frequentis:~/work/karaf-grpc-fork$ mvn clean install
.
.
.
[INFO] ------------------------------------------------------------------------                                                                                                                                                                                                                                                                        
[INFO] Reactor Summary for com.stackleader.training.grpc 0.0.1:                                                                                                                                                                                                                                                                                        
[INFO]                                                                                                                                                                                                                                                                                                                                                 
[INFO] com.stackleader.training.grpc ...................... SUCCESS [  0.415 s]                                                                                                                                                                                                                                                                        
[INFO] com.stackleader.training.grpc.helloworld.api ....... SUCCESS [  4.007 s]                                                                                                                                                                                                                                                                        
[INFO] com.stackleader.training.grpc.helloworld.server .... SUCCESS [  1.527 s]                                                                                                                                                                                                                                                                        
[INFO] com.stackleader.training.grpc.helloworld.client .... SUCCESS [  0.652 s]                                                                                                                                                                                                                                                                        
[INFO] com.stackleader.training.grpc.helloworld.feature ... SUCCESS [  0.451 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS                                                                 
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  7.853 s                                                          
[INFO] Finished at: 2023-02-27T11:30:21+01:00
[INFO] ------------------------------------------------------------------------
bpilka@bpilka-frequentis:~/work/karaf-grpc-fork$
```
- bring the docker containers up
```
bpilka@bpilka-frequentis:~/work/karaf-grpc-fork$ docker-compose up -d
[+] Running 3/3
 ⠿ Network karaf-grpc-fork_default  Created                                                                                                                           0.4s
 ⠿ Container karaf                  Started                                                                                                                           2.3s
 ⠿ Container onos                   Started                                                                                                                           2.2s
bpilka@bpilka-frequentis:~/work/karaf-grpc-fork$ 
```

##### Start gRPC server in vanilla karaf 4.2.9 container
- add repo and install gRPC server feature
```shell
bpilka@bpilka-frequentis:~$ docker exec -it karaf /opt/apache-karaf/bin/client
client: Ignoring predefined value for KARAF_HOME
Logging in as karaf
        __ __                  ____      
       / //_/____ __________ _/ __/      
      / ,<  / __ `/ ___/ __ `/ /_        
     / /| |/ /_/ / /  / /_/ / __/        
    /_/ |_|\__,_/_/   \__,_/_/         

  Apache Karaf (4.2.9)

Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit 'system:shutdown' to shutdown Karaf.
Hit '<ctrl-d>' or type 'logout' to disconnect shell from current session.

karaf@root()> repo-add mvn:com.stackleader/com.stackleader.training.grpc.helloworld.feature/0.0.1/xml/features
Adding feature url mvn:com.stackleader/com.stackleader.training.grpc.helloworld.feature/0.0.1/xml/features
karaf@root()> feature:install com.stackleader.training.grpc.helloworld.feature                                                                                                                                                                     
```
- tail the karaf log of the karaf container and verify gRPC server was started 
```shell
bpilka@bpilka-frequentis:~$ docker exec karaf tail -f /opt/apache-karaf/data/log/karaf.log                                                                                                                                                                                                                                                             
2023-02-27T11:49:14,270 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   wrap_file__opt_karaf_.m2_repository_io_grpc_grpc-okhttp_0.14.0_grpc-okhttp-0.14.0.jar/0.0.0                                                                                                                 
2023-02-27T11:49:14,278 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   wrap_file__opt_karaf_.m2_repository_io_grpc_grpc-protobuf_0.14.0_grpc-protobuf-0.14.0.jar/0.0.0                                                                                                             
2023-02-27T11:49:14,286 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   com.stackleader.training.grpc.helloworld.api/0.0.1                                                                                                                                                          
2023-02-27T11:49:14,299 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   wrap_file__opt_karaf_.m2_repository_io_grpc_grpc-netty_0.14.0_grpc-netty-0.14.0.jar/0.0.0                                                                                                                   
2023-02-27T11:49:14,307 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   com.stackleader.training.grpc.helloworld.server/0.0.1                                                                                                                                                       
2023-02-27T11:49:14,398 | INFO  | features-3-thread-1 | PlatformDependent                | 57 - io.netty.common - 4.1.0.CR7 | Your platform does not provide complete low-level API for accessing direct buffers reliably. Unless explicitly requested, heap buffer will always be preferred to avoid potential system unstability.                    
2023-02-27T11:49:14,635 | INFO  | features-3-thread-1 | HelloWorldServer                 | 52 - com.stackleader.training.grpc.helloworld.server - 0.0.1 | Server started, listening on 5000                                                                                                                                                            
2023-02-27T11:49:14,646 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   wrap_file__opt_karaf_.m2_repository_io_grpc_grpc-protobuf-nano_0.14.0_grpc-protobuf-nano-0.14.0.jar/0.0.0
2023-02-27T11:49:14,650 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   wrap_file__opt_karaf_.m2_repository_com_google_auth_google-auth-library-oauth2-http_0.3.0_google-auth-library-oauth2-http-0.3.0.jar/0.0.0
2023-02-27T11:49:14,652 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 | Done.
```

- establish connection from host via telnet. Connection is opened and stays open. No exception in the karaf log
```shell
bpilka@bpilka-frequentis:~$ telnet localhost 5101
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
```

##### Start gRPC server in ONOS karaf 4.2.9 container
- add repo and install gRPC server feature (password is 'rocks')
```shell
bpilka@bpilka-frequentis:~/work/karaf-grpc-fork$ ssh onos@localhost -p 8102
The authenticity of host '[localhost]:8102 ([127.0.0.1]:8102)' can't be established.
RSA key fingerprint is SHA256:PSPyvytMVegYVj1UsE8CM38es5dzyDVmVsaH/tBjmsw.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[localhost]:8102' (RSA) to the list of known hosts.
Password authentication
(onos@localhost) Password: 
Welcome to Open Network Operating System (ONOS)!
     ____  _  ______  ____     
    / __ \/ |/ / __ \/ __/   
   / /_/ /    / /_/ /\ \     
   \____/_/|_/\____/___/     
                               
Documentation: wiki.onosproject.org      
Tutorials:     tutorials.onosproject.org 
Mailing lists: lists.onosproject.org     

Come help out! Find out how at: contribute.onosproject.org 

Hit '<tab>' for a list of available commands
and '[cmd] --help' for help on a specific command.
Hit '<ctrl-d>' or type 'logout' to exit ONOS session.

onos@root > repo-add mvn:com.stackleader/com.stackleader.training.grpc.helloworld.feature/0.0.1/xml/features                                                                                                                                                                                                                                   
Adding feature url mvn:com.stackleader/com.stackleader.training.grpc.helloworld.feature/0.0.1/xml/features
onos@root > feature:install com.stackleader.training.grpc.helloworld.feature                                                                                                                                                                                                                                                                   
```
- tail the karaf log of the karaf container and verify gRPC server was started
```shell
bpilka@bpilka-frequentis:~$ docker exec onos tail -f /root/onos/apache-karaf-4.2.9/data/log/karaf.log
2023-02-27T14:14:17,681 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   com.google.protobuf/3.0.0.alpha-5
2023-02-27T14:14:17,685 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   com.google.protobuf/3.0.0.beta-2
2023-02-27T14:14:17,687 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   com.google.protobuf.util/3.0.0.beta-2
2023-02-27T14:14:17,689 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   wrap_file__opt_karaf_.m2_repository_io_grpc_grpc-protobuf-lite_0.14.0_grpc-protobuf-lite-0.14.0.jar/0.0.0
2023-02-27T14:14:17,692 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   wrap_file__opt_karaf_.m2_repository_io_grpc_grpc-protobuf_0.14.0_grpc-protobuf-0.14.0.jar/0.0.0
2023-02-27T14:14:17,695 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   wrap_file__opt_karaf_.m2_repository_io_grpc_grpc-stub_0.14.0_grpc-stub-0.14.0.jar/0.0.0
2023-02-27T14:14:17,704 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   com.stackleader.training.grpc.helloworld.api/0.0.1
2023-02-27T14:14:17,707 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 |   com.stackleader.training.grpc.helloworld.server/0.0.1
2023-02-27T14:14:17,835 | INFO  | features-3-thread-1 | HelloWorldServer                 | 210 - com.stackleader.training.grpc.helloworld.server - 0.0.1 | Server started, listening on 5000
2023-02-27T14:14:17,841 | INFO  | features-3-thread-1 | FeaturesServiceImpl              | 11 - org.apache.karaf.features.core - 4.2.9 | Done.
```
- establish connection from host via telnet. Crash visible in the karaf logs. Connection closed immediately
```shell
bpilka@bpilka-frequentis:~/work/karaf-grpc-fork$ telnet localhost 5102                                                                                                                                                                                                                                                                                 
Trying 127.0.0.1...                                                                  
Connected to localhost.                                                              
Escape character is '^]'.                                                            
Connection closed by foreign host. 
```
- exception thrown:
```shell        
2023-02-27T14:30:07,587 | WARN  | grpc-default-worker-ELG-3-4 | ChannelInitializer               | 137 - io.netty.common - 4.1.35.Final | Failed to initialize a channel. Closing: [id: 0x29ea481d, L:/172.25.0.3:5000 - R:/172.25.0.1:45636]                                                                                                          
java.lang.NoSuchMethodError: 'void io.netty.handler.codec.http2.DefaultHttp2HeadersDecoder.<init>(int, int, boolean)'                                                                                                                                                                                                                                  
        at io.grpc.netty.NettyServerHandler.newHandler(NettyServerHandler.java:110) ~[?:?]                                                                                                                                                                                                                                                             
        at io.grpc.netty.NettyServerTransport.createHandler(NettyServerTransport.java:118) ~[?:?]
        at io.grpc.netty.NettyServerTransport.start(NettyServerTransport.java:76) ~[?:?]
        at io.grpc.netty.NettyServer$1.initChannel(NettyServer.java:136) ~[?:?]
        at io.netty.channel.ChannelInitializer.initChannel(ChannelInitializer.java:129) [!/:4.1.35.Final]
        at io.netty.channel.ChannelInitializer.handlerAdded(ChannelInitializer.java:112) [!/:4.1.35.Final]
        at io.netty.channel.AbstractChannelHandlerContext.callHandlerAdded(AbstractChannelHandlerContext.java:969) [!/:4.1.35.Final]
        at io.netty.channel.DefaultChannelPipeline.callHandlerAdded0(DefaultChannelPipeline.java:610) [!/:4.1.35.Final]
        at io.netty.channel.DefaultChannelPipeline.access$100(DefaultChannelPipeline.java:46) [!/:4.1.35.Final]
        at io.netty.channel.DefaultChannelPipeline$PendingHandlerAddedTask.execute(DefaultChannelPipeline.java:1461) [!/:4.1.35.Final]
        at io.netty.channel.DefaultChannelPipeline.callHandlerAddedForAllHandlers(DefaultChannelPipeline.java:1126) [!/:4.1.35.Final]
        at io.netty.channel.DefaultChannelPipeline.invokeHandlerAddedIfNeeded(DefaultChannelPipeline.java:651) [!/:4.1.35.Final]
        at io.netty.channel.AbstractChannel$AbstractUnsafe.register0(AbstractChannel.java:515) [!/:4.1.35.Final]
        at io.netty.channel.AbstractChannel$AbstractUnsafe.access$200(AbstractChannel.java:428) [!/:4.1.35.Final]
        at io.netty.channel.AbstractChannel$AbstractUnsafe$1.run(AbstractChannel.java:487) [!/:4.1.35.Final]
        at io.netty.util.concurrent.AbstractEventExecutor.safeExecute(AbstractEventExecutor.java:163) [!/:4.1.35.Final]
        at io.netty.util.concurrent.SingleThreadEventExecutor.runAllTasks(SingleThreadEventExecutor.java:405) [!/:4.1.35.Final]
        at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:500) [!/:4.1.35.Final]
        at io.netty.util.concurrent.SingleThreadEventExecutor$5.run(SingleThreadEventExecutor.java:906) [!/:4.1.35.Final]
        at io.netty.util.internal.ThreadExecutorMap$2.run(ThreadExecutorMap.java:74) [!/:4.1.35.Final]
        at io.netty.util.concurrent.FastThreadLocalRunnable.run(FastThreadLocalRunnable.java:30) [!/:4.1.35.Final]
        at java.lang.Thread.run(Thread.java:834) [?:?]                                                  
```