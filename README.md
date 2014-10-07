share
=====

#xml-rpc实现的文件共享
这个项目取材与http://www.the5fire.com/python-project8-xml-rpc.html 。是python基础教程中的十个例子中的第八个。主要的实现原理如下：

每一个客户端都是一个节点。每一个节点，都启动一个 xml-rpc服务器。在 xml-rpc服务器中，维护着一个所有节点的集合。原文的例子，功能太少，只能下载。后来我加了一个ls 命令，可以查看包括自己的和所有节点的文件。原项目中的节点列表，必须是手动给出的，相当麻烦，是通过一个叫urlfile的文件来维护的。在我的这个项目中，维护 节点的信息是通过程序自己学习到的。每当一个节点启动的时候，该节点就会把自己的xml-rpc服务器的访问url，通过udp广播的方式，广播给某一个端口。同时每一个节点，只要它启动后，会监听某一个端口上的，有关xml-rpc服务器的访问url的监听。只要收到信息，就把它写入到节点列表中。通过 fetch下载文件时，如果发现了异常，则从节点列表中删除它。

现在假如有两个节点（启动了client.py文件的机器） a和b，a中的节点列表中有b，同样b中也有，当a尝试着fetch 一个文件时，如果没有在a中查找到的话，则会去找b,但是b中的节点列表是a,b会去找a。。。。。这样就形成了阻塞。原项目中，是通过一个url列表来维护的。但是这个项目中，a机器对于自己的url是localhost,b也是localhost，但是对于a来讲b就不是localhost。所以我的项目中，是通过维护一个secret列表来判断，下一个要查找的节点是不是已经被查过了。但是同时得先知道下一个节点的secret值，但是如果下一节点就是上一个节点的话，还是会有阻塞，所以把xml-rpc做成多线程就很必要了。新构建一个类class ThreadRPC(ThreadingMixIn, SimpleXMLRPCServer)  。这样ThreadRPC就变成了多线程的SimpleXMLRPCServer。

本来文件的传输是使用了xml-rpc。后来我把它独立出来了。单独作为一个文件传输服务，TranServer。这个文件传输，不使用xml-rpc，而是直接用socket。用了SocketServer框架。本来是想用asynchat的。但是后来发现，这个异步框架，有点蛋疼。比如说它的push方法。是会把数据放到叫producer_fifo的fifo数据结构中。这个做会出现一个大问题。我读本地的文件速度远远快于发送的数度，当体积大的时候，更是如此。所以使用push传输，会发现内存占用越来越大，越来越大。。。如果不使用push,而是使用send，会发现数据不同步.鉴于这些问题，所以我没有采用异步，而是使用了多线程的SocketServer, ThreadingMixIn

#配置文件
> [global]

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

> 查看文件内容:
  cat xxx [要查看的前n位]

#更新记录
  2014-10-2:

  在源项目的基础上，增加列出共享文件列表功能，增加了节点自动学习功能，不需要手工指定节点。

  2014-10-3

  修改节点前的通信内容，本来是文件内容，现在改成目标地址，类似于递归查询变成了迭代查询，减少没必要的网络流量。增加支持非文本文件的共享
  
  增加了大文件分块传输

  2014-10-6

  修改了文件传输模式，本来是用xml-rpc来read，然后本地write。现在直接使用socket（使用了SocketServer框架）来实现，节约了大量质量（原来是用xml-rpc），速度也快喽。可控性也更强了。
  
  2014-10-7

  增加了cat命令 准确来说，更像是head命令
