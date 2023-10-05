# 一、Jedis（Java程序操作Redis的工具）
* 1、导入Jedis依赖坐标
````
<dependency>
<groupId>redis.clients</groupId>
<artifactId>jedis</artifactId>
<version>3.2.0</version>
</dependency>
````
* 2、连接Redis注意事项
````
1.禁用Linux的防火墙：Linux(CentOS7)里执行命
systemctl stop/disable firewalld.service

2.redis.conf中注释掉bind 127.0.0.1 ,然后 protected-mode no

3.需要先开启服务端
redis-server /etc/redis.conf
不开服务端也会超时
````
* 3、Jedis常用操作
````
1、创建动态的工程
2、创建测试程序
package com.qxy;
import redis.clients.jedis.Jedis;
public class Demo01 {
	public static void main(String[] args) {
		Jedis jedis = new Jedis("192.168.137.3",6379);
		String pong = jedis.ping();
		System.out.println("连接成功："+pong);
		jedis.close();
	}
}
````
* 4、测试相关数据类型
````
1、Jedis-API: Key
jedis.set("k1", "v1");
jedis.set("k2", "v2");
jedis.set("k3", "v3");
Set<String> keys = jedis.keys("*");
System.out.println(keys.size());
for (String key : keys) {
    System.out.println(key);
}
System.out.println(jedis.exists("k1"));
System.out.println(jedis.ttl("k1"));                
System.out.println(jedis.get("k1"));

2、Jedis-API: String
jedis.mset("str1","v1","str2","v2","str3","v3");
System.out.println(jedis.mget("str1","str2","str3"));

3、Jedis-API: List
List<String> list = jedis.lrange("mylist",0,-1);
for (String element : list) {
    System.out.println(element);
}

4、Jedis-API: set
jedis.sadd("orders", "order01");
jedis.sadd("orders", "order02");
jedis.sadd("orders", "order03");
jedis.sadd("orders", "order04");
Set<String> smembers = jedis.smembers("orders");
for (String order : smembers) {
    System.out.println(order);
}
jedis.srem("orders", "order02");

5、Jedis-API: hash
jedis.hset("hash1","userName","lisi");
System.out.println(jedis.hget("hash1","userName"));
Map<String,String> map = new HashMap<String,String>();
map.put("telphone","13810169999");
map.put("address","atguigu");
map.put("email","abc@163.com");
jedis.hmset("hash2",map);
List<String> result = jedis.hmget("hash2", "telphone","email");
for (String element : result) {
    System.out.println(element);
}

6、Jedis-API: zset
jedis.zadd("zset01", 100d, "z3");
jedis.zadd("zset01", 90d, "l4");
jedis.zadd("zset01", 80d, "w5");
jedis.zadd("zset01", 70d, "z6");
 
Set<String> zrange = jedis.zrange("zset01", 0, -1);
for (String e : zrange) {
    System.out.println(e);
}
````
# 二、Jedis实例：手机验证码功能
````
要求：
1、输入手机号，点击发送后随机生成6位数字码，2分钟有效
2、输入验证码，点击验证，返回成功或失败
3、每个手机号每天只能输入3次
````
````
import redis.clients.jedis.Jedis;

import java.util.Random;

public class PhoneCode {
    public static void main(String[] args) {
        //模拟验证码发送
        //verifyCode("137000000");
        
        //校验发送的验证码
        //getRedisCode("137000000","验证码");
    }

    //1.随即生成一个六位数的验证码
    public static String getCode(){
        Random random = new Random();
        String code = "";
        for (int i = 0; i < 6; i++) {
            int rand = random.nextInt(10);
            code += rand;
        }
        return code;
    }

    //2.每个手机每天只能发送三次，验证码放到redis中，设置过期时间
    public static void verifyCode(String phone){
        //连接redis
        Jedis jedis = new Jedis("192.168.50.128", 6379);
        //输入redis连接密码
        jedis.auth("root");

        //拼接key
        //手机发送次数key
        String countKey = "VerifyCode" + phone + ":count";
        //验证码key
        String codeKey = "VerifyCode" + phone + ":code";

        //每个手机每天只能发送3次
        String count = jedis.get(countKey);
        if (count == null){
            //没有发送次数，第一次发送
            //设置发送次数是1
            jedis.setex(countKey,24 * 60 * 60,"1");
        }else if (Integer.parseInt(count) <= 2){
            //发送次数+1
            jedis.incr(countKey);
        }else if (Integer.parseInt(count) > 2){
            //发送三次了，不能再发送了
            System.out.println("今天的发送次数已经超过3次了");
            //关闭连接
            jedis.close();
            return;
        }

        //发送的验证码要放到redis中,设置过期时间为120s
        String vcode = getCode();
        jedis.setex(codeKey,120,vcode);
        jedis.close();
    }

    //3.验证码校验
    public static void getRedisCode(String phone,String code){
        //从redis中获取验证码
        Jedis jedis = new Jedis("192.168.50.128", 6379);
        jedis.auth("root");

        //验证码key
        String codeKey = "VerifyCode" + phone + ":code";
        String redisCode = jedis.get(codeKey);

        //判断
        if (redisCode.equals(code)){
            System.out.println("成功");
        }else {
            System.out.println("失败");
        }
        jedis.close();
    }
}
````