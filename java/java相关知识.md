## JMM(java内存模型)：是种概念，规范，不是真实存在的

分为：主内存、线程工作内存
线程--> 工作内存--> 主内存

JMM关于同步的规定：
1. 线程解锁前，必须把工作内存中的数据写回主内存中
2. 线程加锁前，必须把主内存中的最新值拷贝到工作内存中
3. 解锁加锁是同一把锁，即是一个原子操作

- JVM运行程序工作的是线程，而每个线程创建时JVM都会为其创建一个工作内存（栈空间），工作内存是线程的私有数据区域。
- java内存模型规定：所有变量都要存储在主内存中，主内存是共享内存区域，所有线程都可以访问。
- 线程对变量的操作必须在工作内存中进行，不能直接操作主内存，所以要将主内存中的变量拷贝到工作内存中，操作完了再写回主内存中。
- 各个线程存储的是主内存的拷贝副本变量
- 各个线程无法访问对方的工作内存，线程间的通信必须通过主内存来完成

代码案例：验证volatile的可见性
``` java

	private volatile int num = 0;
    public void addNum(){
        num += 60;
    }
    public static void main(String[] args) {
        HelloController hello = new HelloController();
        new Thread(() -> {
            try {
                Thread.sleep(3000);
                hello.addNum();
                System.out.println(Thread.currentThread().getName() + "-num的值为："+hello.num);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"Thread1").start();

        while (hello.num == 0){}
        System.out.println("num的值为：" + hello.num);
    }

```

代码案例：验证volatile的非原子性
```java
	private volatile int num = 0;
    public void addOne(){
        num ++;
    }
    public static void main(String[] args) {
        HelloController hello = new HelloController();
        for(int i = 0; i < 10; i++){
            new Thread(() -> {
                for(int j = 0; j < 1000; j++){
                    hello.addOne();
                }
            },"Thread"+i).start();
        }
        while (Thread.activeCount() > 2){Thread.yield();}
        System.out.println("num的值为：" + hello.num);
	}
```


## JMH性能测试：JSON工具测试

JMH(Java Microbenchmark Harness): 是专门用于微基准测试的api工具，我们可以通过JMH来对热点函数进行定量的分析，然后进行优化。
比较典型的应用就是：
	* 精准知道某方法的执行时长，以及执行时间t和输入p之间的相关性；
	* 比较不同接口实现，查找最佳性能方法
[官方地址](http://hg.openjdk.java.net/code-tools/jmh/help)

先用maven将依赖引入：
```java
	<!--JMH性能测试的-->
	<dependency>
		<groupId>org.openjdk.jmh</groupId>
		<artifactId>jmh-core</artifactId>
		<version>${jmh.version}</version>
	</dependency>
	<dependency>
		<groupId>org.openjdk.jmh</groupId>
		<artifactId>jmh-generator-annprocess</artifactId>
		<version>${jmh.version}</version>
		<scope>provided</scope>
	</dependency>
```

我们先来一个简单的样例来了解下JMH：String相加和使用StringBuilder的append方式的性能比较测试。
```java

// BenchmarkMode表示基准测试的模型，Throughput是表示吞吐量，可以得到单位时间内的执行的次数数，另外的一些值：
// AverageTime：每次调用的平均消耗时间。
// SampleTime：随机取样的方式。
// SingleShotTime：只运行一次，往往同时把 warmup 次数设为0，用于测试冷启动时的性能。
// All：所有的模式
@BenchmarkMode(Mode.Throughput)
// 这个设置是为了预热代码，这是考虑了JIT编译机制，使得测试更加符合实际情形，iterations是迭代的次数，也就是执行次数，time是一次迭代需要执行的时间
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
// 这个是正常的代码测试，参数意义和上面一样
@Measurement(iterations = 5, time = 5, timeUnit = TimeUnit.SECONDS)
// 测试进程数
@Threads(5)
// 这个是fork的进程分支数
@Fork(2)
// 这个是基准测试的时间单位
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public class JSONBenchmarkTest {

    @Benchmark
    public void testString(){
        String str = "";
        for (int i = 0; i < 10; i++) {
            str += "a";
        }
    }

    @Benchmark
    public void testStringBuilder(){
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 10; i++) {
            sb.append("a");
        }
    }

    public static void main(String[] args) throws RunnerException {
        Options optional = new OptionsBuilder().include(JSONBenchmarkTest.class.getSimpleName()).build();
        new Runner(optional).run();
    }
}
```

测试运行的方式有多种，你也可以生成jar包运行，也可以自己写个测试单元，我们来看看运行结果：
```cmd
# JMH version: 1.21
# VM version: JDK 1.8.0_241, Java HotSpot(TM) 64-Bit Server VM, 25.241-b07
# VM invoker: C:\Program Files\Java\jdk1.8.0_241\jre\bin\java.exe
# VM options: -javaagent:D:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2019.3.4\lib\idea_rt.jar=56074:D:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2019.3.4\bin -Dfile.encoding=UTF-8
# Warmup: 5 iterations, 1 s each
# Measurement: 5 iterations, 5 s each
# Timeout: 10 min per iteration
# Threads: 5 threads, will synchronize iterations
# Benchmark mode: Throughput, ops/time
# Benchmark: com.ryoma.test.JSONBenchmarkTest.testString

# Run progress: 0.00% complete, ETA 00:02:00
# Fork: 1 of 2
# Warmup Iteration   1: 15483.501 ops/ms
# Warmup Iteration   2: 19191.194 ops/ms
# Warmup Iteration   3: 20264.030 ops/ms
# Warmup Iteration   4: 19480.834 ops/ms
# Warmup Iteration   5: 23256.853 ops/ms
Iteration   1: 15957.031 ops/ms
Iteration   2: 22213.562 ops/ms
Iteration   3: 18531.152 ops/ms
Iteration   4: 15090.877 ops/ms
Iteration   5: 14872.852 ops/ms

# Run progress: 25.00% complete, ETA 00:01:34
# Fork: 2 of 2
# Warmup Iteration   1: 9488.254 ops/ms
# Warmup Iteration   2: 14901.396 ops/ms
# Warmup Iteration   3: 15147.344 ops/ms
# Warmup Iteration   4: 14724.827 ops/ms
# Warmup Iteration   5: 16359.645 ops/ms
Iteration   1: 17793.775 ops/ms
Iteration   2: 15591.598 ops/ms
Iteration   3: 17155.395 ops/ms
Iteration   4: 20171.710 ops/ms
Iteration   5: 14919.348 ops/ms


Result "com.ryoma.test.JSONBenchmarkTest.testString":
  17229.730 ±(99.9%) 3746.976 ops/ms [Average]
  (min, avg, max) = (14872.852, 17229.730, 22213.562), stdev = 2478.393
  CI (99.9%): [13482.754, 20976.706] (assumes normal distribution)


# JMH version: 1.21
# VM version: JDK 1.8.0_241, Java HotSpot(TM) 64-Bit Server VM, 25.241-b07
# VM invoker: C:\Program Files\Java\jdk1.8.0_241\jre\bin\java.exe
# VM options: -javaagent:D:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2019.3.4\lib\idea_rt.jar=56074:D:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2019.3.4\bin -Dfile.encoding=UTF-8
# Warmup: 5 iterations, 1 s each
# Measurement: 5 iterations, 5 s each
# Timeout: 10 min per iteration
# Threads: 5 threads, will synchronize iterations
# Benchmark mode: Throughput, ops/time
# Benchmark: com.ryoma.test.JSONBenchmarkTest.testStringBuilder

# Run progress: 50.00% complete, ETA 00:01:03
# Fork: 1 of 2
# Warmup Iteration   1: 29986.558 ops/ms
# Warmup Iteration   2: 31278.352 ops/ms
# Warmup Iteration   3: 34750.060 ops/ms
# Warmup Iteration   4: 32943.155 ops/ms
# Warmup Iteration   5: 32914.852 ops/ms
Iteration   1: 34710.068 ops/ms
Iteration   2: 33898.339 ops/ms
Iteration   3: 35172.227 ops/ms
Iteration   4: 34691.384 ops/ms
Iteration   5: 34007.925 ops/ms

# Run progress: 75.00% complete, ETA 00:00:31
# Fork: 2 of 2
# Warmup Iteration   1: 29339.982 ops/ms
# Warmup Iteration   2: 29560.675 ops/ms
# Warmup Iteration   3: 65644.791 ops/ms
# Warmup Iteration   4: 76074.187 ops/ms
# Warmup Iteration   5: 75281.694 ops/ms
Iteration   1: 71302.248 ops/ms
Iteration   2: 76488.885 ops/ms
Iteration   3: 76566.340 ops/ms
Iteration   4: 76071.468 ops/ms
Iteration   5: 74394.781 ops/ms


Result "com.ryoma.test.JSONBenchmarkTest.testStringBuilder":
  54730.367 ±(99.9%) 32328.809 ops/ms [Average]
  (min, avg, max) = (33898.339, 54730.367, 76566.340), stdev = 21383.506
  CI (99.9%): [22401.557, 87059.176] (assumes normal distribution)


# Run complete. Total time: 00:02:07

REMEMBER: The numbers below are just data. To gain reusable insights, you need to follow up on
why the numbers are the way they are. Use profilers (see -prof, -lprof), design factorial
experiments, perform baseline and negative tests that provide experimental control, make sure
the benchmarking environment is safe on JVM/OS/HW level, ask for reviews from the domain experts.
Do not assume the numbers tell you what you want them to tell.

Benchmark                             Mode  Cnt      Score       Error   Units
JSONBenchmarkTest.testString         thrpt   10  17229.730 ±  3746.976  ops/ms
JSONBenchmarkTest.testStringBuilder  thrpt   10  54730.367 ± 32328.809  ops/ms

Process finished with exit code 0

```

当然你也可以把运行结果通过日志打印出来.output("D://text.log").build()；
通过结果我们可以看出来：会输出每条线程的详细信息，最后还有一个总结的输出信息，从最后的信息可以看出来，StringBuilder比String的效率还是
高很多的，这里输出的吞吐量，值越高越好。还有些方法属性级别的注解：
- @Benchmark：方法级别的注解，表示需要benchmark的对象，类似于单元测试的@Test。
- @Setup：方法级的注解，可以用于初始化一些数据操作。
- @TearDown：方法级的注解，可用于测试后的结束工作。
- @State：当使用@Setup和@TearDown注解时都需要使用这个注解，该注解是类注解，表示类的共享情况
	+ @State(Scope.Benchmark)：表示所有线程间共享
	+ @State(Scope.Group)：表示同一个组里面所有线程共享
	+ @State(Scope.Thread)：表示每个线程独享
- @Param：属性级的注解，可以用于测试不同参数的方法性能。

其他详细的注解推荐大家到这个博文查看：[JMH性能测试](https://www.jianshu.com/p/ad34c4c8a2a3)

进入正题测试，我们最常见的是4种JSON使用工具，如下：

* Gson：Google开源的解析json格式数据的工具，功能很全很强大，主要有toJson和fromJson两个函数，无其他的类库依赖，
	托管地址：[gson](https://github.com/google/gson)
* FastJson：阿里开源的解析json格式的数据工具，速度够快，无其他类库依赖，
	托管地址：[fastjson](https://github.com/alibaba/fastjson)
* Jackson：托管在github上的开源项目，Jackson社区相对比较活跃，更新速度也比较快，springMvc的默认json解析神器，解析大的json
	文件比较快，运行内存低，优点多多啊。
	托管地址：[jackson](https://github.com/FasterXML/jackson)
* Json-lib：是java的一个类库，项目地址：[json-lib](http://json-lib.sourceforge.net/index.html)

> 首先我们来添加maven的依赖：
```xml
	<!--json工具-->
	<!--阿里的开源json工具-->
	<dependency>
		<groupId>com.alibaba</groupId>
		<artifactId>fastjson</artifactId>
		<version>1.2.68</version>
	</dependency>
	<!--Google开源的json工具-->
	<dependency>
		<groupId>com.google.code.gson</groupId>
		<artifactId>gson</artifactId>
		<version>2.8.6</version>
	</dependency>
	<!--java类库提供的-->
	<dependency>
		<groupId>net.sf.json-lib</groupId>
		<artifactId>json-lib</artifactId>
		<version>2.4</version>
	</dependency>
	<!--开源的第三方库json工具-->
	<dependency>
		<groupId>com.fasterxml.jackson.core</groupId>
		<artifactId>jackson-databind</artifactId>
		<version>2.9.4</version>
	</dependency>
	<dependency>
		<groupId>com.fasterxml.jackson.core</groupId>
		<artifactId>jackson-annotations</artifactId>
		<version>2.9.4</version>
	</dependency>
```

> 先准备json的相关方法代码：
```java

	private static Gson gson = new GsonBuilder().create();
    private static ObjectMapper objectMapper = new ObjectMapper();
	
	/** gson的bean转为json */
    public String gsonBean2Json(Object object){
        return gson.toJson(object);
    }
    /** gson的json转为bean */
    public <T> T gsonJson2Bean(String json, Class<T> cls){
        return gson.fromJson(json, cls);
    }
    /** fastjson的bean转为json */
    public String fastjsonBean2Json(Object object){
        return JSON.toJSONString(object);
    }
    /** fastjson的json转为bean */
    public <T> T fastjsonJson2Bean(String json, Class<T> cls){
        return JSON.parseObject(json, cls);
    }
    /** jackson的bean转为json */
    public String jacksonBean2Json(Object object) throws JsonProcessingException {
        return objectMapper.writeValueAsString(object);
    }
    /** jackson的json转为bean */
    public <T> T jacksonJson2Bean(String json, Class<T> cls) throws JsonProcessingException {
        return objectMapper.readValue(json, cls);
    }
    /** jsonlib的bean转为json */
    public String jsonlibBean2Json(Object object){
        return JSONObject.fromObject(object).toString();
    }
    /** jsonlib的json转为bean */
    public <T> T jsonlibJson2Bean(String json, Class<T> cls){
        return (T) JSONObject.toBean(JSONObject.fromObject(json), cls);
    }
```

> 实体类代码：
```java
@Data
public class PersonEntity {
    private int age;
    private String name;
    private List<PersonEntity> friendList;
    private Map<String, Object> hobbiesMap;
}
```

> 初始化数据：
```java
	@Setup
    public void initData(){
        PersonEntity person = new PersonEntity();
        person.setAge(20);
        person.setName("序列化测试");
        List<PersonEntity> personEntities = Lists.newArrayList();
        PersonEntity p1 = new PersonEntity();
        p1.setAge(20);
        p1.setName("朋友1");
        PersonEntity p2 = new PersonEntity();
        p2.setAge(20);
        p2.setName("朋友2");
        personEntities.add(p1);
        personEntities.add(p2);
        person.setFriendList(personEntities);
        Map<String, Object> map = Maps.newHashMap();
        map.put("打篮球", "最爱");
        person.setHobbiesMap(map);
        personEntity = person;
    }
```

> 测试类注解：
```java
@BenchmarkMode(Mode.SingleShotTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
// 表示所有线程间共享
@State(Scope.Benchmark)
public class JSONBenchmarkTest {
	.....
}
```

> 序列化对象测试：
```java

	@Param({"1000", "10000", "100000"})
    private int count;
    private PersonEntity personEntity;
	
	@Benchmark
    public void gsonb2j(){
        for (int i = 0; i < count; i++) {
            gsonBean2Json(personEntity);
        }
    }
    @Benchmark
    public void fastJsonb2j(){
        for (int i = 0; i < count; i++) {
            fastjsonBean2Json(personEntity);
        }
    }
    @Benchmark
    public void jacksonb2j() throws JsonProcessingException {
        for (int i = 0; i < count; i++) {
            jacksonBean2Json(personEntity);
        }
    }
    @Benchmark
    public void jsonlibb2j(){
        for (int i = 0; i < count; i++) {
            jsonlibBean2Json(personEntity);
        }
    }
	
	public static void main(String[] args) throws RunnerException {
        Options optional = new OptionsBuilder().include(JSONBenchmarkTest.class.getSimpleName()).forks(1).warmupIterations(0).build();
        new Runner(optional).run();
    }
```

> 运行后的测试结果，我们直接来看看最后的总结数据：
```java
Benchmark                      (count)  Mode  Cnt     Score   Error  Units
JSONBenchmarkTest.fastJsonb2j     1000    ss        156.439          ms/op
JSONBenchmarkTest.fastJsonb2j    10000    ss        182.629          ms/op
JSONBenchmarkTest.fastJsonb2j   100000    ss        222.440          ms/op
JSONBenchmarkTest.gsonb2j         1000    ss         46.276          ms/op
JSONBenchmarkTest.gsonb2j        10000    ss        112.191          ms/op
JSONBenchmarkTest.gsonb2j       100000    ss        452.821          ms/op
JSONBenchmarkTest.jacksonb2j      1000    ss         82.042          ms/op
JSONBenchmarkTest.jacksonb2j     10000    ss        151.490          ms/op
JSONBenchmarkTest.jacksonb2j    100000    ss        308.667          ms/op
JSONBenchmarkTest.jsonlibb2j      1000    ss        343.709          ms/op
JSONBenchmarkTest.jsonlibb2j     10000    ss        917.391          ms/op
JSONBenchmarkTest.jsonlibb2j    100000    ss       3304.863          ms/op
```

数据越多的话fastjson的效率就越高，gson和jackson都是稳步上升，jsonlib则比较差。

我们来看看反序列的测试结果，以下有些改动：
初始化数据的时候，我们会把之前的person实例对象变成json格式的字符串：
```java
	...跟上面的一样...
	person.setHobbiesMap(map);
    personJson = JSON.toJSONString(person);
```
```java

	private String personJson;
	@Benchmark
    public void gsonj2b(){
        for (int i = 0; i < count; i++) {
            gsonJson2Bean(personJson, PersonEntity.class);
        }
    }
    @Benchmark
    public void fastJsonj2b(){
        for (int i = 0; i < count; i++) {
            fastjsonJson2Bean(personJson, PersonEntity.class);
        }
    }
    @Benchmark
    public void jacksonj2b() throws IOException {
        for (int i = 0; i < count; i++) {
            jacksonJson2Bean(personJson, PersonEntity.class);
        }
    }
    @Benchmark
    public void jsonlibj2b(){
        for (int i = 0; i < count; i++) {
            jsonlibJson2Bean(personJson, PersonEntity.class);
        }
    }
```

> 看看反序列化的测试结果：
```java

Benchmark                      (count)  Mode  Cnt     Score   Error  Units
JSONBenchmarkTest.fastJsonj2b     1000    ss         58.142          ms/op
JSONBenchmarkTest.fastJsonj2b    10000    ss         96.720          ms/op
JSONBenchmarkTest.fastJsonj2b   100000    ss        268.407          ms/op
JSONBenchmarkTest.gsonj2b         1000    ss         42.239          ms/op
JSONBenchmarkTest.gsonj2b        10000    ss         90.028          ms/op
JSONBenchmarkTest.gsonj2b       100000    ss        307.894          ms/op
JSONBenchmarkTest.jacksonj2b      1000    ss        107.127          ms/op
JSONBenchmarkTest.jacksonj2b     10000    ss        175.029          ms/op
JSONBenchmarkTest.jacksonj2b    100000    ss        333.688          ms/op
JSONBenchmarkTest.jsonlibj2b      1000    ss        498.235          ms/op
JSONBenchmarkTest.jsonlibj2b     10000    ss       1313.649          ms/op
JSONBenchmarkTest.jsonlibj2b    100000    ss       5141.867          ms/op
```

反序列的测试结果可以看出和序列化的结论是差不多的，最后补充下，虽然fastjson的速度很快，但是最近频频出bug，
还是得慎重考虑选择哪个。