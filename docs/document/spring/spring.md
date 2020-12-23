### spring

--------------

~~删除线~~

<u>下划线</u>

### 无序

* 无序1
  1. abc
* 无序2

+ 加号

1. abc
   * 1
   * 2

2. edf

#### 区块

> 列表
>
> 1. abc
>
> 2. def
>
>    - abd
>
>    - adc
>
>      

* 第一项

  > hahh
  >
  > hah

* 第二项

  > addf
  >
  > cdasf

`print(123)`函数

>    String str = new String();



```java
@Slf4j
@RestController
@RequestMapping("/nodeStatistics")
@Api(value = "WEB - NodeStatisticsController", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
```

```java
List<Map<String,Object>> nodeList = bizProcNodeInfoService.queryNodeAvgCostTime((EntityWrapper<BizProcNodeInfo>) new EntityWrapper<BizProcNodeInfo>()
.eq("def.proc_def_code",procDefCode)
.andNew()
.eq("DATE_FORMAT(info.start_time,'%Y%m')",time)
.or()
.eq("DATE_FORMAT(info.finish_Time,'%Y%m')",time)
.groupBy("info.proc_info_code"));
```

<http://baidu.com>

<u>abc</u>

~~adsf~~

<u>asdbc</u>

~~abc~~

[]: 



## springboot

1. 自动装配原理`@SpringbootApplication`

2. springboot官网jar包命名格式`spring-boot-starter-x`,非官方jar命名方式`x-spring-boot-starter`

   

| 表格编号 | 表格标题 | 表格选项 |
| :------: | :------: | :------: |
|    1     |  标题1   |  选项1   |
|    2     |  标题2   |  选项2   |
|    3     |  标题2   |  选项3   |

$$
∫tanxdx=−lncosx+C
$$

<!--asdfac-->
$$
∫ 
x 
2
 −a 
2
 
​	
 dx= 
2
x
​	
  
x 
2
 −a 
2
 
​	
 − 
2
a 
2
 
​	
 ln(x+ 
x 
2
 −a 
2
 
​	
 )+C
$$

### Dubbo 线程派发模型

* Dispather:`all`,`direct`,`message`,`excution`,`connection`
* ThreadPool:`fixed`,`cache`





### 自适应扩展机制

在 Dubbo 中，很多拓展都是通过 SPI 机制进行加载的，比如 Protocol、Cluster、LoadBalance 等。有时，有些拓展并不想在框架启动阶段被加载，而是希望在拓展方法被调用时，根据运行时参数进行加载。这听起来有些矛盾。拓展未被加载，那么拓展方法就无法被调用（静态方法除外）。拓展方法未被调用，拓展就无法被加载。对于这个矛盾的问题，Dubbo 通过自适应拓展机制很好的解决了。自适应拓展机制的实现逻辑比较复杂，首先 Dubbo 会为拓展接口生成具有代理功能的代码。然后通过 javassist 或 jdk 编译这段代码，得到 Class 类。最后再通过反射创建代理类，整个过程比较复杂。为了让大家对自适应拓展有一个感性的认识，下面我们通过一个示例进行演示。这是一个与汽车相关的例子，我们有一个车轮制造厂接口 WheelMaker：

```java
public interface WheelMaker {
    Wheel makeWheel(URL url);
}
```

WheelMaker 接口的自适应实现类如下：

```java
public class AdaptiveWheelMaker implements WheelMaker {
    public Wheel makeWheel(URL url) {
        if (url == null) {
            throw new IllegalArgumentException("url == null");
        }
        
    	// 1.从 URL 中获取 WheelMaker 名称
        String wheelMakerName = url.getParameter("Wheel.maker");
        if (wheelMakerName == null) {
            throw new IllegalArgumentException("wheelMakerName == null");
        }
        
        // 2.通过 SPI 加载具体的 WheelMaker
        WheelMaker wheelMaker = ExtensionLoader
            .getExtensionLoader(WheelMaker.class).getExtension(wheelMakerName);
        
        // 3.调用目标方法
        return wheelMaker.makeWheel(url);
    }
}
```





