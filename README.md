share
=====

#xml-rpc实现的文件共享
这个项目取材与http://www.the5fire.com/python-project8-xml-rpc.html 。是python基础教程中的十个例子中的第八个。主要的实现原来如下：
每一个客户端都是一个节点。每一个节点，都启动一个 xml-rpc服务器。在 xml-rpc服务器中，维护着一个所有节点的集合。原文的例子，功能太少，只能下载。后来我加了一个ls 命令，可以查看包括自己的和所有节点的文件。原项目中的节点列表，必须是手动给出的，相当麻烦，是通过一个叫urlfile的文件来维护的。在我的这个项目中，维护 节点的信息是通过程序自己学习到的。每当一个节点启动的时候，该节点就会把自己的xml-rpc服务器的访问url，通过udp广播的方式，广播给某一个端口。同时每一个节点，只要它启动后，会监听某一个端口上的，有关xml-rpc服务器的访问url的监听。只要收到信息，就把它写入到节点列表中。通过 fetch下载文件时，如果发现了异常，则从节点列表中删除它。
现在假如有两个节点（启动了client.py文件的机器） a和b，a中的节点列表中有b，同样b中也有，当a尝试着fetch 一个文件时，如果没有在a中查找到的话，则会去找b,但是b中的节点列表是a,b会去找a。。。。。这样就形成了阻塞。原项目中，是通过一个url列表来维护的。但是这个项目中，a机器对于自己的url是localhost,b也是localhost，但是对于a来讲b就不是localhost。所以我的项目中，是通过维护一个secret列表来判断，下一个要查找的节点是不是已经被查过了。但是同时得先知道下一个节点的secret值，但是如果下一节点就是上一个节点的话，还是会有阻塞，所以把xml-rpc做成多线程就很必要了。新构建一个类class ThreadRPC(ThreadingMixIn, SimpleXMLRPCServer)  。这样ThreadRPC就变成了多线程的SimpleXMLRPCServer。

#配置文件
[global]
> \# 监听节点的端口
> listen_port = 1111 

> \# 数据传送的端口
> data_port = 1234

> \# 要共享的目录
> share_dir = /tmp/a

#使用方法
> 启动节点 :
  python client.py
  
> 获取文件列表:
  ls
  
> 下载文件:
  fetch xxx
