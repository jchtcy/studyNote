# 一、Maven的功能
* 1、Maven 作为依赖管理工具
````
随着我们使用越来越多的框架，或者框架封装程度越来越高，项目中使用的jar包也越来越多。项目中，一个模块里面用到上百个jar包是非常正常的。jar包所属技术的官网通常是英文界面，网站的结构又不尽相同，甚至找到下载链接还发现需要通过特殊的工具下载。框架中使用的 jar 包，不仅数量庞大，而且彼此之间存在错综复杂的依赖关系。jar 包之间有可能产生冲突。进一步增加了我们在 jar 包使用过程中的难度。

使用 Maven 后，依赖对应的 jar 包能够自动下载，方便、快捷又规范。使用 Maven 则几乎不需要管理 jar 包彼此之间存在错综复杂的依赖关系，极个别的地方调整一下即可，极大的减轻了我们的工作量。
````
* 2、Maven 作为构建管理工具
````
脱离 IDE 环境仍需构建
````
# 二、Maven简介
````
Maven 是 Apache 软件基金会组织维护的一款专门为 Java 项目提供构建和依赖管理支持的工具。
````
* 1、构建
````
Java 项目开发过程中，构建指的是使用『原材料生产产品』的过程。
    原材料:Java 源代码、基于 HTML 的 Thymeleaf 文件、图片、配置文件
    产品:一个可以在服务器上运行的项目
构建过程包含的主要的环节
    清理：删除上一次构建的结果，为下一次构建做好准备
    编译：Java 源程序编译成 *.class 字节码文件
    测试：运行提前准备好的测试程序
    报告：针对刚才测试的结果生成一个全面的信息
    打包:
        Java工程：jar包
        Web工程：war包
    安装：把一个 Maven 工程经过打包操作生成的 jar 包或 war 包存入Maven的本地仓库
    部署:
        部署 jar 包：把一个 jar 包部署到 Nexus 私服服务器上
        部署 war 包：借助相关 Maven 插件（例如 cargo），将 war 包部署到 Tomcat 服务器上
````
* 2、依赖
````
如果 A 工程里面用到了 B 工程的类、接口、配置文件等等这样的资源，那么我们就可以说 A 依赖 B。

依赖管理中要解决的具体问题：
    jar 包的下载：使用 Maven 之后，jar 包会从规范的远程仓库下载到本地
    jar 包之间的依赖：通过依赖的传递性自动完成
    jar 包之间的冲突：通过对依赖的配置进行调整，让某些jar包不会被导入
````
* 3、Maven 的工作机制
![本地路径](img/Maven的工作机制.png)