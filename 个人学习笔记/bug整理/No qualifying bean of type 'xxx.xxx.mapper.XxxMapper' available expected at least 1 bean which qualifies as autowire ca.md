# No qualifying bean of type 'xxx.xxx.mapper.XxxMapper' available: expected at least 1 bean which qualifies as autowire candidate. 



今天在搭建环境的时候被这个bug坑了一个多小时，索性最后找到了问题



**原因就是mapper没有注入！**

我是初次使用模块化搭建项目的，结构如下

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210520090705180.png" alt="image-20210520090705180" style="zoom:50%;" />



主启动类写在了web模块

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210520091556355.png" alt="image-20210520091556355" style="zoom:50%;" />

mapper接口卸载了service模块

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210520091637639.png" alt="image-20210520091637639" style="zoom:50%;" />

我在主启动类上设置自动扫描mapper包

<img src="https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210520091501735.png" alt="image-20210520091501735" style="zoom:50%;" />

这么一看没啥不对的，可是一启动就报这个错

![image-20210520092331849](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210520092331849.png)



**原因分析：**

主启动类只能扫描其所在包及其子包下面的类！

主启动类所在包：web模块下的com.xxx

mapper接口所在包：service模块下的com.xxx.service.mapper

这样必然扫不到

**解决方案：**

在@springbootApplication中指定扫描包路径

![image-20210520092825697](https://figure-bed-liqun.oss-cn-beijing.aliyuncs.com/uPic/image-20210520092825697.png)

大功告成！