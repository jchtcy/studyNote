# 一、yaml配置文件
* 1、yaml的用法
````
Yet Another Markup Language
非常适合用来做以数据为中心的配置文件。
````
* 2、基本语法
````
key: value；kv之间有空格
大小写敏感
使用缩进表示层级关系
缩进不允许使用tab，只允许空格
缩进的空格数不重要，只要相同层级的元素左对齐即可
'#'表示注释
字符串无需加引号，如果要加，单引号''、双引号""表示字符串内容会被 转义、不转义
````
* 3、数据类型
````
字面量：单个的、不可再分的值。date、boolean、string、number、null
对象：键值对的集合。map、hash、set、object
数组：一组按次序排列的值。array、list、queue
````
* 4、自定义类绑定的配置提示
````
自定义的类和配置文件绑定一般没有提示。若要提示，添加如下依赖：

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>

<!-- 下面插件作用是工程打包时，不将spring-boot-configuration-processor打进包内，让其只在编码的时候有用 -->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludes>
                    <exclude>
                        <groupId>org.springframework.boot</groupId>
                        <artifactId>spring-boot-configuration-processor</artifactId>
                    </exclude>
                </excludes>
            </configuration>
        </plugin>
    </plugins>
</build>
````