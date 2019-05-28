# Docker系列学习文章 - 专业化定制镜像dockerfile（六）

[原文地址](https://cloud.tencent.com/developer/article/1116794)

## 一、什么是dockerfile

dockerfile 大家第一眼看它名字的时候就感觉到它就是一个file。没错，它就是一个简简单单的文本文件，但是它里面的内容对于镜像来说却不是简简单单。这些内容代表着一个镜像如何诞生，好比镜像的"基因"。我们可以看一段简单的dockerfile内容，先认识下它基本的面貌：

	# This dockerfile uses the ubuntu image
	# VERSION 2 - EDITION 1
	# Author: docker_user
	# Command format: Instruction [arguments / command] ..
	 
	# Base image to use, this must be set as the first line
	FROM ubuntu
	 
	# Maintainer: docker_user <docker_user at email.com> (@docker_user)
	MAINTAINER docker_user docker_user@email.com
	 
	# Commands to update the image
	RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
	RUN apt-get update && apt-get install -y nginx
	RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
	 
	# Commands when creating a new container
	CMD /usr/sbin/nginx

我们从 #的 注释看起，第一句话表明了这个dockerfile用的ubuntu的镜像；然后是注明了版本 EDITION 1；接着是镜像的作者写明了叫docker_user；最后是说明了下命令的格式。

前面四句注释写完后就开始真正编写dockerfile的命令了，FROM ubuntu 意思是这个镜像是基于ubuntu这个base镜像创建的。这个很重要，一般的dockerfile开始都是基于某个基本镜像去构建的，所以开始就得写这个FROM，注明镜像的底层来源是什么。我们之前也说过，镜像是一层层叠加的，总有一个初始化镜像在最底层。

说明完基于什么base镜像后，这里的MAINTAINER意思是这个镜像的创始人和维护者信息。相当于给了这个镜像标记了”爹妈“，让大家知道，这是谁生的，要是以后有问题可以找作者咨询或者提供相关建议。

第四段内容是dockerfile最重要的内容了，一个镜像有什么特性就是通过这段内容来去实现的。这段代码展示的是这个镜像生成的时候需要做哪些操作，这些操作一般都是一些命令。比如常见的shell命令，你可以把它理解为一段面向过程的脚本吧（但是严格意义上来说，也不是什么脚本）。通过这些命令，一步步实现你想在镜像中完成的事。注意最前面红色的关键字RUN，这是dockerfile里特有的语法标识，前面我们提到的FROM和MAINTAINER也是dockerfile的语法。这些语法我们接下来会详细介绍下。

最后一段内容表示的是镜像做好后，镜像变成容器需要执行的命令是什么。这里一般是一个服务的启动命令，比如上面示例中表示的就是启动nginx服务。到这，大家看这个dockerfile估计就明白了，前面所有编写的都是为了最后这一句 /usr/sbin/nginx 命令启动而做的准备。想要在一个空白的ubuntu镜像里运行nginx服务，那么首先得把ubuntu的apt源配置好，接着是apt-get install nginx包安装，最后是配置nginx.conf文件。只有完成了这三步，nginx才能跑起来。所以，以后大家编写dockerfile也是这样的思路，考虑清楚你做镜像的目的，然后分解成每一小步，然后一层层写dockerfile语句实现。

## 二、常用的dockerfile指令和语法

前面我们讲了关于dockerfile的介绍，基本上有了一个dockerfile编写的思路。但是，真正要下手编写估计你还不会。因为，具体的dockerfile语法你现在还不清楚。那么接下来我们来讲讲dockerfile的编写语法，掌握了这个，基本的套路你就明白了。

### 1. RUN指令

这个指令是dockerfile里用的最多的指令之一，它的作用就是执行一条命令。类似于linux的shell脚本里的命令一样，写一个RUN，后面跟着命令就执行一次。

比如上面的那个示例dockerfile里，就有那么一段关于RUN命令的集合：

	# Commands to update the image
	RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
	RUN apt-get update && apt-get install -y nginx
	RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf

三个RUN，就代表了执行了三步命令操作。第一步是配置Ubuntu源，第二步是执行apt-get更新，第三步是编辑nginx.conf文件。RUN后面的命令我们看着像是Linux的命令，对，其实就是！RUN后面可以跟shell格式的命令。当然，RUN后面还可以跟exec格式的命令，方式如：RUN ["可执行文件", "参数1", "参数2"]，不过这个用得比较少。

既然RUN后面可以跟shell命令，那么假如我要做的镜像要运行很多个命令才能完成那要怎么办？是写多个RUN吗？比如像下面这样的：

	RUN apt-get update
	RUN apt-get install -y gcc libc6-dev make
	RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz"
	RUN mkdir -p /usr/src/redis
	RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
	RUN make -C /usr/src/redis
	RUN make -C /usr/src/redis install

一共7行，每一行都是RUN。

其实这样写语法上没什么错，能运行成功。但是从优雅度和专业度上来说这样写很不合适。因为每写一个RUN命令就等于增加一层镜像。你写N个，那就是N层。而docker镜像的层数目前是有限制的，大概100多层。所以，咱们能尽量一个RUN命令搞定的就一个RUN搞定，让镜像的层数简化降低。上面这段dockerfile命令，其实可以简化成如下方式：

	RUN buildDeps='gcc libc6-dev make' \
	    && apt-get update \
	    && apt-get install -y $buildDeps \
	    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz" \
	    && mkdir -p /usr/src/redis \
	    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
	    && make -C /usr/src/redis \
	    && make -C /usr/src/redis install \
	    && rm -rf /var/lib/apt/lists/* \
	    && rm redis.tar.gz \
	    && rm -r /usr/src/redis \
	    && apt-get purge -y --auto-remove $buildDeps

一看你就懂了，用 && 把多个命令连上。

### 2. CMD指令

CMD 指令是用于指定启动容器默认的主进程命令的。因为容器其实就是进程，它不像虚拟机那样启动后不运行任何东西也能一直静默运行。所以，容器需要有主进程一直持续，不然就会退出。你可以想象，容器就是包着一个主进程在那跑... 主进程就是容器的灵魂，灵魂没有了，容器这一“肉身”在那也会消失。

那么用什么指令启动这一灵魂呢？那就可以用CMD指令。CMD指令也有两种格式：

- shell 格式：CMD <命令>
- exec 格式：CMD ["可执行文件", "参数1", "参数2"...]

也就是说，CMD后面也能跟shell命令，跟RUN一样，比如：

CMD cat /etc/redhat-release

查看系统类型版本。

如果换算成exec格式，上面那命令就等于CMD [ "sh", "-c", "cat /etc/redhat-release" ] 。也就是说，CMD后面如果是跟的shell命令，那么实际底层执行是用exc的sh -c的方式。再比如，你CMD后面写的是 systemctl start mysqld ，那么就等于CMD [ "sh", "-c", "systemctl start mysqld" ] 。 

注意，exec命令格式里的第一小段才是主进程，上面的那两个例子命令，主进程就是 sh ，而不是cat /etc/redhat-release 和 systemctl start mysqld。cat /etc/redhat-release 和 systemctl start mysqld这两个shell命令 有个特点，执行完后就会返回结果退出。因此，sh 到时候也会退出。sh主进程退出了，那么容器的灵魂就没有了，那么容器也不再会运行了。换句话说，灵魂也就持续存在了一两秒... 

你想要让容器一直运行，那么CMD就得写对。最好是用exec的命令格式。比如，启动运行nginx、mysql等，应该是类似这样写：

CMD ["nginx", "-g", "daemon off;"]

CMD ["/usr/bin/mysqld_safe"]

只要容器里主进程能一直运行，那么容器就不会退出。

这个CMD命令一般是写完dockerfile最后才写，dockerfile前面的内容都是配置环境做一些准备，等都做得差不多了，那么最后一句就是CMD启动容器主进程的指令，其实就类似docker的开机启动项。

### 3. ENTRYPOINT指令

ENTRYPOINT一般跟CMD配合起来一起使用。因为CMD里的内容能作为参数传到ENTRYPOINT里使用。官网里是这么介绍的：An ENTRYPOINT allows you to configure a container that will run as an executable. 换成中文意思是说ENTRYPOINT可以让你的容器功能表现得像一个可执行程序一样。

让容器表现得像一个可执行程序？这个要怎么理解呢? 我们看下下面的例子。

	FROM centos:7.2 
	ENTRYPOINT ["/bin/cat"]

假如我们写了一个上面这样简单的dockerfile，那么这个镜像做成后运行将带有cat的功能。我们在运行这个镜像的时候跟上一个文件路径，那么就会返回输出这个文件内容：

	# docker run -it image_test_entrypoint /etc/fstab    (注：image_test_entrypoint 是假设做好的image名字）

运行这个命令后，结果将是输出/etc/fstab文件的内容。

所以，到这你应该明白了，ENTRYPOINT能定义一些初始化命令在里面。

前面提到的，还能接收CMD的参数内容，比如你的dockfile这样写：

	FROM centos:7.2 
	ENTRYPOINT ["vmstat","3"]
	CMD ["5']

ENTRYPOINT里原本是执行vmstat每空3秒不停循环输出vmstat监控信息，然后有了CMD参数后，传入了一个5，那么vmstat结果就只能输出5次了。同理的，还可以看看下面这个top命令的dockerfile，道理都差不多：

	FROM centos:7.2
	ENTRYPOINT ["top", "-b"]   
	CMD ["-c"]

上面两个例子是命令的，ENTRYPOINT还可以带脚本，比如官方mysql5.6的dockerfile就在ENTRYPOINT使用了脚本：

	...
	COPY docker-entrypoint.sh /entrypoint.sh
	ENTRYPOINT ["/entrypoint.sh"]
	
	EXPOSE 3306
	CMD ["mysqld"]

这个entrypoint.sh就是自己定义好的shell脚本，完成一些初始化、逻辑判断的操作，毕竟有时候一些前提操作比较复杂，需要通过一些脚本才能完成。

总结来说，就是ENTRYPOINT可以定义一些初始化的命令、参数甚至脚本，然后做成的镜像更像一个可执行程序，你可以把它当作工具反复使用。所以，有些场景如果想把容器做成工具，可以使用ENTRYPOINT试试。不过得注意，整个dockerfile里ENTRYPOINT只能使用一次，如果你写了多个，那么生效的是最后一个。

### 4. COPY指令

在构建docker镜像的时候，肯定涉及到某个文件、脚本从某个路径拷贝到另外一个路径下。那么我们此时就可以用COPY命令去做这一操作。

- COPY <源路径>... <目标路径>
- COPY ["<源路径1>",... "<目标路径>"]

比如我们拷贝install.sh 这个脚本到/opt/shell下，那么就是这样写：

COPY install.sh /opt/shell

而且这个命令也支持通配符，比如用*和？，跟linux命令一样：

COPY install* /opt/shell

COPY in?tall /opt/shell

注意，这里的目标路径就是容器里面的目标路径，如果事先没创建也没事，到时候执行的时候会自动创建。

### 5. ADD指令

ADD指令和COPY指令有点类似，但是ADD相对高级些。高级在哪呢？高级在ADD不仅仅能复制，还能自动解压缩。比如你想COPY一个mysql.tar.gz 到/opt下面，如果使用COPY那么就是单纯的把mysql.tar.gz复制到/opt下。如果使用ADD，那么不仅仅是复制过去了，同时还会解压缩这个tar包。

另外，ADD还有下载的功能，如果源地址是个URL，那么将下载这个URL的目录或者文件到目标路径：

ADD http://example.com/foobar.py /opt

一般在写dockerfile时候用ADD命令都是看中它的自动解压缩功能，而不是拷贝功能。如果是单纯的拷贝还是建议使用COPY命令。这样你的dockerfile才比较直观，ADD的话有隐藏的高级功能，所以不大建议使用。

### 6. ENV指令

大家看到ENV这个词应该差不多能明白它是什么意思了，ENV就是环境变量单词的缩写。在dockerfile里，我们也经常得定义一些环境变量。语法如下：

单个变量：ENV <key> <value>        
多个变量：ENV <key1>=<value1> <key2>=<value2>...

我们来看个例子，比如官方mysql的dockerfile开头就用了ENV设置了环境变量：

	FROM oraclelinux:7-slim
	ENV PACKAGE_URL https://repo.mysql.com/yum/mysql-8.0-community/docker/x86_64/mysql-community-server-minimal-8.0.2-0.1.dmr.el7.x86_64.rpm
	
	# Install server
	RUN rpmkeys --import http://repo.mysql.com/RPM-GPG-KEY-mysql \
	  && yum install -y $PACKAGE_URL \
	  && yum install -y libpwquality \
	  && rm -rf /var/cache/yum/*
	RUN mkdir /docker-entrypoint-initdb.d
	.....

这里的 ENV 就指定了Mysql RPM包的下载路径，然后赋值给了PACKAGE_URL这个Key。

那么，后面我们可以看到RUN指令里引用了PACKAGE_URL这个值，用了 $PACKAGE_URL 这样的方式

所以，有了ENV，那么你可以想象成它就类似于编程里的定义全局变量，开头定义好了，后面就可以复用。如果后续万一参数有变化，只要改前面的ENV内容值即可，非常方便！

### 7. ARG指令

ARG指令就是用来传递变量用的，它一般结合docker build命令中的--build-arg一起使用。也就是说，ARG是dockerfile里声明一个变量值，然后使用--build-arg来传递值给ARG。

ARG的写法很简单，方式如下：

	ARG <name> 或者 ARG <name>=<default>

例如：

	ARG user1
	USER $user1

通过使用--build-arg 指定好user1的值是root用户。

	# docker build --build-arg user1=root ./opt/mysql

这里要注意的是，不能在ARG里指定一些密码和机密信息，因为使用docker history将显示出所有信息，很不安全；另外，ENV指令也是用$来引用值的，ARG也是，所以这里会有冲突，如果都使用，那么ENV的值会覆盖ARG的值，这样ARG就不生效了。所以，上面例子中假如你这样写是会有问题的：

ENV user1 root
ARG user1
USER $user1

到时候你无论怎么使用--build-arg，user1的值都是root。

所以，应该这么写就没问题了：

	ARG user1
	ENV user1 $user1
	USER $user1

在ENV里，值直接引用ARG里的user1即可，其实这个就是做了一个间接的传递。

最后，在docker里有几个变量是预设好了的，大家可以直接用--build-arg使用它们，无需使用ARG设置：

HTTP_PROXY、http_proxy、HTTPS_PROXY、https_proxy、FTP_PROXY、ftp_proxy、NO_PROXY、no_proxy    都是设置代理proxy的。

### 8. LABEL指令

WORKDIR也很好理解，就是在dockerfile指定工作路径。RUN, CMD, ENTRYPOINT, COPY和ADD指令都将用WORKDIR定义好的路径值。如果你指定的路径不存在，那么到时候会自动创建。

	WORKDIR /home/test1

比如上面这行就指定了WORKDIR路径为：/home/test1

WORKDIR能在一个dockerfile里使用多次，比如你开始定义了路径为/home/test1 ，那么后面的WORKDIR你可以写成相对路径，这个相对路径是基于最开始的绝对路径的，比如：

	WORKDIR /home/test1
	WORKDIR test2
	WORKDIR test3
	RUN pwd

那么结果将是 /home/test1/test2/test3 ,不过一般也不会这么用，有点混淆。咱们写dockerfile的时候还是得遵循结构清晰、内容易懂、简约优化等准则。

另外，WORKDIR也能引用ENV里的值，例：

	ENV DIRPATH /home
	WORKDIR $DIRPATH/test
	RUN pwd

结果就是 /home/test 

### 10. VOLUME指令

VOLUME的作用就是指定数据的存储挂载点。有的容器涉及到一些数据的持久化，比如mysql这样的容器，它就需要定义一个数据卷路径存储数据文件。我们看官方mysql的dockerfile，里面就有VOLUME的定义：

	FROM oraclelinux:7-slim
	ENV PACKAGE_URL https://repo.mysql.com/yum/mysql-8.0-community/docker/x86_64/mysql-community-server-minimal-8.0.2-0.1.dmr.el7.x86_64.rpm
	
	# Install server
	RUN rpmkeys --import http://repo.mysql.com/RPM-GPG-KEY-mysql \
	  && yum install -y $PACKAGE_URL \
	  && yum install -y libpwquality \
	  && rm -rf /var/cache/yum/*
	RUN mkdir /docker-entrypoint-initdb.d
	
	VOLUME /var/lib/mysql
	...

最后一行 VOLUME /var/lib/mysql就是。

当然，你也可以定义多个路径，只要空格隔开即可：

VOLUME /var/lib/mysql /var/log/mysql

### 11. ONBUILD指令

ONBUILD指令比较特殊，我们把这个单词拆分就是on build... 意思就是“在创建的路上...”的意思。但是，不是现在创建。这个怎么理解？既然不是现在创建，那意思就跟当前的镜像没关系，不会影响当前的环境，而是会在下一个镜像环境里有操作、有影响。

那么这个指令有什么作用呢？这个指令的作用其实就是以当前镜像为基础，在下一个镜像构建的时候去运行一些命令。简单的说就是为下一个镜像做准备，相当于下个镜像制作的“触发器”。

	ONBUILD ADD . /app/src
	ONBUILD RUN /usr/local/bin/python-build --dir /app/src

上面这个例子就是当下个镜像要制作的时候，会先把当前路径下的一些文件拷贝到/app/src目录下，然后再引用/app/src内容执行python-build命令。

总的来说，ONBUILD命令就是下个镜像制作构建的触发器。但是要注意的是，这个触发器只在“子辈”的镜像里有效果，在“孙辈”的镜像里没效果，隔一代，继承效果就消失了。

### 12. EXPOSE指令

EXPOSE作用就是声明容器要使用的端口。注意，这里用的是声明这个词而不是定义。因此，在容器启动后并不是就立即使用EXPOSE声明的端口，这只是在dockerfile里跟大家说明下，这个镜像做好后打算使用什么端口。

### 13. USER指令

USER指令很简单，就是在dockerfile里设置执行用户的。

	USER root

这个命令很简单，但是要注意的是，使用USER指令，会影响RUN、CMD、ENTRYPOINT等指令，同时也会影响容器中主进程运行的用户。

大家在使用dockerfile里指令的时候一定要注意指令的影响范围，不然处理不好会影响一些递归式的错误。

## 三、案例-用dockerfile定制一个MYSQL镜像

其实学会了上面的指令，你差不多就可以自己写dockerfile了。写dockerfile的时候注意秉着 “结构清晰”、“内容易懂”、“简约优化” 三个原则即可。

我们可以看下完整的官方mysql镜像dockerfile写法，参考几个，你自然就熟悉怎么写了。

	FROM oraclelinux:7-slim
	ENV PACKAGE_URL https://repo.mysql.com/yum/mysql-8.0-community/docker/x86_64/mysql-community-server-minimal-8.0.2-0.1.dmr.el7.x86_64.rpm
	
	# Install server
	RUN rpmkeys --import http://repo.mysql.com/RPM-GPG-KEY-mysql \
	  && yum install -y $PACKAGE_URL \
	  && yum install -y libpwquality \
	  && rm -rf /var/cache/yum/*
	RUN mkdir /docker-entrypoint-initdb.d
	
	VOLUME /var/lib/mysql
	
	COPY docker-entrypoint.sh /entrypoint.sh
	ENTRYPOINT ["/entrypoint.sh"]
	
	EXPOSE 3306 33060
	CMD ["mysqld"]

这个例子中，大部分指令都用到了。我们其实也可以做个总结，把dockerfile的指令做下归类，这样在日后的使用当中，你也就很清晰了。

一般地，Dockerfile 分为五部分：来源环境配置、维护者信息、镜像操作、配置和容器启动时执行指令。

|类别|指令|
|---|---|
|来源、环境设置|FROM、ENV、LABEL|
|维护者信息|MAINTAINER|
|镜像操作|RUN、COPY、ADD、VOLUME、ONBUILD|
|配置|EXPOSE、ARG、WORKDIR、USER|
|容器启动|CMD、ENTRYPOINT|

总结：Dockerfile的指令很多，上面介绍了比较常用、实用的几个指令。大家只要在日常工作当中多用一两次，基本上就能写出很专业实用的dockerfile。这些指令我想也不是死的，随着docker的发展，我想未来肯定会出一些新的指令。但是目前这些指令，基本上是够用了。除非容器有大的架构变动，不然这些指令以及他们的作用是不会轻易变化的。
