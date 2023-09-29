# 一、实现最基本的Web应用
* 1、简单页面
````
1、CATALINA_HOME\webapps目录下新建一个子目录，起名：test 这个目录名test就是你这个webapp的名字

2、在test目录下新建资源文件，
index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <h1>Hello World</h1>
</body>
</html>

3、启动Tomcat服务器,输入url:http://127.0.0.1:8080/test/index.html
````
* 2、登录页面
````
login.html
<!DOCTYPE html>
<html lang = "en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>login page</title>
</head>
<body>
    <form>
        <h2>user login</h2>
        username:<input type="text" name="username" /><br>
        password:<input type="password" name="password"/><br>
        <input type="submit" value="login"/>
    </form>
</body>
</html>
````
* 3、超链接
````
index.html
<!DOCTYPE html>
<html lang = "en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Hello World</h1>
    <!-- 超链接相当于在浏览器地址栏输入url -->
    <a href="http://127.0.0.1:8080/test/login.html">user login</a>
    <a href="/test/login.html">user login</a>
</body>
</html>
````