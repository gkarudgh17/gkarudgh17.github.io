```java
/*
* thread Pool config 설정
* 
*/
@Configuration
@EnableAsync
public class ThreadPoolInitializer {
	
	@Bean(name = "executor")
	public TaskExecutor taskExecutor() {
		ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
		taskExecutor.setCorePoolSize(10);
		taskExecutor.setMaxPoolSize(20);
		taskExecutor.setQueueCapacity(200);
		taskExecutor.setThreadNamePrefix("executor-");
		taskExecutor.setWaitForTasksToCompleteOnShutdown(true);		// ThreadPool에 속한 Thread의 작업이 마칠때 까지 대기, 기본 값 false
		taskExecutor.setAwaitTerminationSeconds(WEEK_SECOND);		  // setWaitForTasksToCompleteOnShutdown과 같이 사용, 대기시간이 지나면 main thread 종료
		taskExecutor.initialize();
		return taskExecutor;
	}
}
```

```java
@SpringBootApplication
public class ECrawlingApplication implements CommandLineRunner {

	@Autowired
	private MultiThreadTest multiThreadTest;
	
	public static void main(String[] args) throws Exception {
		ConfigurableApplicationContext ctx = null;
		try {
			ctx = SpringApplication.run(ECrawlingApplication.class, args);
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			ctx.close();  // 작업이 완료되면 main thread 종료
		}
	}

	@Override
	public void run(String... args) throws Exception {
    for(int i=0; i<100; i++) {
      multiThreadTest.print(i);
    }
  }
}
```

```java
@Component
public class MultiThreadTest {
	
	@Async("executor")
	public void print(int index) throws Exception {
		System.out.println(index + " : " + Thread.currentThread().getName());
		Thread.sleep(100);
	}
	
}
```

```
실행결과
0 : executor-1
1 : executor-2
2 : executor-3
3 : executor-4
4 : executor-5
5 : executor-6
6 : executor-7
7 : executor-8
8 : executor-9
9 : executor-10
10 : executor-1
11 : executor-2
12 : executor-3
13 : executor-4
14 : executor-5
15 : executor-6
16 : executor-8
17 : executor-7
18 : executor-10
19 : executor-9
20 : executor-1

```
