---
title: "Spring Batch Intro"
date: 2021-01-05 19:22:28 -0400
categories: spring
tags:
  - java
  - spring
  - batch
---

## 1. Spring Batch?
Spring Batch project는 Accenture와 Spring Source의 공동 작업으로 2007년에 탄생했다.
### 1-1. Batch 처리에서 Spring Batch를 써야하는 이유?
- 효과적인 logging, statistics처리, transaction 관리 등 re-usable한 필수 기능을 support함.
- Exception한 사항과 unstable한 동작에 대한 protection 기능이 있다.
### 1-2. Spring boot batch 주의 사항
- batch 처리시 System I/O 사용을 최소화해야한다. 잦은 I/O로 DB Connection과 Network 비용이 커지면서 성능에 영향을 줄 수 있기 때문이다. 따라서 가능하면 한번에 데이터를 조회하여 memory에 저장해두고 처리를 한 다음, 그 결과를 한번에 db에 저장하는 것이 좋다.
- Spring boot batch는 scheduler를 제공하지 않는다. batch 처리 기능만 제공하며 scheduling 기능은 spring에서 제공하는 Quart z Framework를 이용해야 한다. linux crontab 명령은 가장 간단하게 사용할 수 있지만 이는 추천하지 않는다. crontab에 경우 각 서버마다 따로 scheduling을 관리해야 하기 때문이다.

## 2. Core component
![](https://media.vlpt.us/images/kihwanyu/post/8fbf64f0-9581-4cfa-bc5a-8ac5918c4926/image.png)
>Job과 Step 1:M
>Step과 ItemReader, ItemProcessor, ItemWriter 1:1

즉, Job 하나의 일을 두고 여러 Step을 가지고 있음.

### 2-1. Job
 개발자가 설계/작성하고, 한 번에 실행되기를 의도하는 작업의 집합이다.

Spring batch에서 Job 객체는 여러 Step Instance를 포함하는 container이다.

Job 객체를 만드는 builder는 여러 개 있으며, 여러 builder를 통합하여 처리하는 공장인 JobBuilderFactory로 원하는 Job을 쉽게 만들 수 있다.

##### JobBuilderFactory Example
```java
public class JobBuilderFactory {
	private JobRepository jobRepository;
	
	public JobBuilderFactory(JobRepository jobRepository) {
		this.jobRepository = jobRepository;
	}
	
	public JobBuilder get(String name){
		JobBuiler builder = new JobBuilder(name).repository(jobRepository);
		return builder;
	}
}
```
##### SimpleJobBuilder를 이용한 Job 생성 Example
```java
@Configuration
public class SimpleJobConfig{
	@Bean
	public Job simpleJob(JobBuilderFactory jobBuilderFactory, Step simpleJobStep) {
		return jobBuilderFactory.get("simpleJob")
		.preventRestart()
		.start(simpleJobStep)
		.build();
	}
}
```
#### 2-1-1. JobInstance
- JobInstance는 여러 개의 JobExecution를 가지고 있을 수 있다.
> - 오늘 Job을 실행했는데 실패했다면 다음날 동일한 JobInstance를 가지고 또 실행한다. Job 실행이 실패하면 JobInstance가 끝난 것으로 간주하지 않기 때문이다.
>- JobInstance는 어제의 실패한 JobExecution과 오늘의 성공한 JobExecution 두개를 가지게 된다.
#### 2-1-2. JobExecution
- JobInstance에 대한 한 번의 실행을 나타내는 객체
#### 2-1-3. JobParameters
- Job이 실행될 때 필요한 Parameters를 Map 타입으로 저장하는 객체
- 하나의 Job이 생성될 때, 시작 시간등의 정보를 Parameter로 해서 하나의 JobInstance를 생성해서 JobInstance와 JobParameters는 1:1 관계를 유지한다.
- 객체의 equality를 평가하는 기준이 된다.
- Parameter type은 String, Long, Date, Double을 사용할 수 있다.
#### 2-1-4. JobRepository
- batch 처리 정보를 담고 있는 mechanism이다. 어떤 Job이 실행되었으면 몇 번 실행되었고 언제 끝났는지 등 batch 처리에 대한 metadata를 저장한다.
- Job 하나가 실행되면 JobRepository에서는 batch 실행에 관련된 정보를 담고 있는 domain JobExecution을 생성한다.
- Step의 실행 정보를 담고 있는 StepExecution도 저장소에 저장하여 전체 metadata를 저장/관리하는 역할을 수행
#### 2-1-5. JobLauncher
Job, JobParameters와 함께 batch를 실행하는 interface이다.

### 2-2. Step
- 실질적인 batch 처리를 정의하고, 제어하는데 필요한 모든 정보가 들어 있는 domain 객체
#### 2-2-1. StepExecution
Job에는 JobExecution Job 실행 정보가 있다면, Step에는 StepExecution이라는 Step 실행 정보를 담는 객체가 있다.
#### 2-2-2. ItemReader
Step의 대상이 되는 batch data를 읽어오는 interface이다. File, Xml, DB등 여러 type의 data를 읽어올 수 있다.
#### 2-2-3. ItemProcessor
ItemReader로 읽어 온 batch data를 변환하는 역할을 수행한다. 비즈니스 로직의 분리를 위함이다.
#### 2-2-4. ItemWriter
batch data를 저장한다. 일반적으로 DB나 file에 저장한다. 

### 2-3. Chunk
 **한 번의 operation을 통해 다룰 data의 집합**이다. 각 step은 설정에 정의 된 chunk 단위에 따라 data를 읽어 들이고, 가공한 후 기록한다. 크기의 unit은 DB의 한 row일 수도 있고, CSV file의 한 줄 일 수도 있다.

DB의 작업을 수반하는 Step이라면, 이 chunk는 transaction의 관리단위가 되기도 한다. 즉, 하나의 chunk를 처리하는 동안 Spring Batch는 component 간에 계속하여 TX를 전파하고, 마지막 처리가 끝나는 시점에 TX를 commit하거나 rollback한다.

> 출처: https://velog.io/@kihwanyu/Spring-Batch-%EC%A0%95%EB%A6%AC
