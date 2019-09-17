# TranverseWall

这段时间需要查一些国外的资料，总结一下正确上网的教程。


基本步骤：

1. 购买VPS

    VPS商家有很多，目前基本上便宜的套餐都卖完了，这里博主老左总结了部分商家，大家可以参考下：[国内外VPS推荐](http://www.laozuo.org/myvps)，我使用的是Vultr，每月$5，买了两个月共学习使用。

2. 配置VPS

    我本地使用的是Ubuntu 16.04，使用ssh登陆到VPS，也可以在VPS官网管理界面，直接进入VPS Terminal。
    ```
    ssh 用户名@主机IP -p 端口号
    ```

    ssh端口号默认一般是22。

3. 更新VPS软件源

    对于非香港VPS，可以跳过此步。找一个USTC或者其他的源放上去：
    ```
    vi /etc/apt/source.list
    
    # 加入以下内容
    deb http://debian.ustc.edu.cn/ubuntu/ trusty main multiverse restricted universe 
    deb http://debian.ustc.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe 
    deb http://debian.ustc.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe 
    deb http://debian.ustc.edu.cn/ubuntu/ trusty-security main multiverse restricted universe 
    deb http://debian.ustc.edu.cn/ubuntu/ trusty-updates main multiverse restricted universe 
    deb-src http://debian.ustc.edu.cn/ubuntu/ trusty main multiverse restricted universe 
    deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-backports main multiverse restricted universe 
    deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-proposed main multiverse restricted universe 
    deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-security main multiverse restricted universe 
    deb-src http://debian.ustc.edu.cn/ubuntu/ trusty-updates main multiverse restricted universe

    # 更新
    apt-get update

    ```
    
4. 安装shadowsocks

    ```
    apt-get install python-pip
    pip install shadowsocks
    ```

5. 配置shadowsocks服务端

    ```
    {
      "server":"1.1.1.1",
      "server_port":"8000",
      "password":"111111",
      "timeout":300,
      "method":"aes-256-cfb"
    }
    ```
    server: VPS公网IPV4的IP地址 
    server_port: 服务器端口，可以自定义其他端口
    password: 自定义的密码 
    method: 信道加密方式

    保存成shadowsocks_server.json即可，名字可以自定义，文件格式需要为json。

    也可以开放多个端口：
    ```
    {
      "server":"1.1.1.1",
      "port_password": {
        "8000":"111111",
        "8001":"111111"
      },
      "timeout":300,
      "method":"aes-256-cfb"
    }
    ```
    
6. 启动服务端

      短连接：
      ```
      ssserver  -c ./shadowsocks_server.json
      ```
      
      后台启动：
      ```
      ssserver  -c ./shadowsocks_server.json -d start
      ```
      对于长短连接的区别，可自行百度，我这里使用的是长连接。
 
7. 配置shadowsocks客户端

    Windows，Android，Mac OS可以下载shadowsocks客户端连接，使用VPN. 

    [shadowsocks客户端](https://github.com/shadowsocks/)

    Windows: shadowsocks-windows

    Android: shadowsocks-android

    Mac OS: ShadowsocksX-NG

    IOS: 可以在app store里下载一个LiFi，然后配置即可

    Linux：Linux下麻烦一点


    Linux：以Ubuntu 16.04 为例：

      1）通过shadowsocks客户端软件连接

        ```
        sudo add-apt-repository ppa:hzwhuang/ss-qt5
        sudo apt-get update
        sudo apt-get install shadowsocks-qt5
        ```
        
    在命令行执行上面的三行命令，注意：仅支持Ubuntu 14.04或更高版本。如果不是Ubutun或者其他的系统，详细的安装方式请参考 [安装方式](https://github.com/shadowsocks/shadowsocks-qt5/wiki/%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97)，因为shadowsocks-qt5是跨平台的版本，所以安装这个也不要惊讶。如果安装没有问题的话，在应用程序中可以找到shadowsocks-qt5。

    打开后，在配置的时候，手动方式添加一个新的连接，输入服务端对应的信息，本地地址何端口可以使用127.0.0.1和1080，日志级别和自动化可以不勾选。配置好后，重启软件，使配置生效。

    配置完成后，还需要打开系统代理，在设置-->网络-->网络代理-->手动，然后在HTTP代理、HTTPS代理、Sockers主机都填上地址和端口，分别是127.0.0.1和1080，然后应该可以FQ了。

      2）通过命令行执行shadowsocks脚本连接

      首次需要先打开系统代理，具体方式如上

      编写客户端配置，保存为shadowsocks_client.json：
      ```
      { 
        “server”:”1.1.1.1”, 
        “server_port”:8000, 
        “local_address”:”127.0.0.1”, 
        “local_port”:1080, 
        “password”:”111111”, 
        “timeout”:300, 
        “method”:”aes-256-cfb” 
      } 
      ```
      
      运行客户端的配置信息，运行方式需要和服务端对应起来：

      短连接：
      ```
      sslocal -c shadowsocks_client.json
      ```
      长连接：
      ```
      sslocal -c shadowsocks_client.json -d start
      ```
      
8. 浏览器代理配置

    浏览器代理的配置网上教程很多，大家可以自行百度。总的思想就是，使用之前配置的shadowsocks 客户端来代理浏览器的请求等，我们之前本地配置的为127.0.0.1，端口号为1080。



9. 特殊配置

    当我们在终端执行get、curl等跑在http协议上的命令时，会发现终端输入的命令无效。由于我们之前本地配置的服务器代理类型为socks5，所以当我们需要执行这些命令时，需要将代理类型改为http类型，这个在我们的客户端软件中可以通过勾选的方式进行更改，比较简单，但是在Linux下，还需要更改浏览器的代理配置中对应的这个地方，然后即可在终端中执行这些命令。

    我们会发现在终端使用ping 命令也没有反应，因为ping命令走的是icmp协议，ICMP在OSI模型的第3层（网络层）工作，SOCKS在第5层（会话层）工作，httpp在第7层（应用层）工作，因此ping消息不能通过SOCKS传递，但httping消息可以。所以可以在http代理下，使用httpping测试下。
