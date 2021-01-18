---
title: "Bootstrapping Hibernate 5 with Spring"
date: 2021-01-18 12:00:28 -0400
categories: spring
tags:
  - hibernate
  - spring
  - database
---
how to **bootstrap Hibernate 5 with Spring.**
## Spring Integration
Bootstrapping a SessionFactory with the native Hibernate API is a bit complicated and would task us quite a few lines of code.
Fortunately, **Spring supports bootstrapping the SessionFactory**.
## Dependencies
- org.hibernate.hibernate-core
- org.springframework.spring-orm
	- providing the Spring integration with Hibernate
## Configuration
As mentioned before, Spring supports us with bootstrapping the Hibernate *SessionFactory*.
All we have to do is to **define some beans as well as a few parameters.**
```java
@Configuration
@EnableTransactionManagerment
public class HibernateConf {
	@Bean
	public LocalSessionFactoryBean sessionFactory() {
		LocalSessionFactoryBean sessionFactory = new LocalSessionFactoryBean();
		sessionFactory.setDataSource(dataSource());
		sessionFactory.setPacakgesToScan({"com.test.hibernate.bootstrap.model"});
		sessionFactory.setHibernateProperties(hibernateProperties());
		
		return sessionFactory;
	}

	@Beam
	public DataSource dataSource() {
		BasicDataSource dataSource = new BasicDataSource();
		dataSource.setDriverClassName("org.h2.Driver");
		dataSource.setUrl("jdbc:h2:mem:db;DB_CLOSE_DELAY=-1");
		datasource.setUsername("sa");
		dataSource.setPassword("sa");
	
		return dataSource;
	}

	@Bean
	public PlatfromTransactionManager hibernateTransactionManager() {
		HbernateTransactionManager transactionManager = new HibernateTransactionManager();
		transactionManager.setSessionFactory(sessionFactory().getObject());
		return transactionMnanager;
	}

	private final Properties hibernateProperties() {
		Properties hibernateProperties = new Properties();
		hibernateProperties.setProperty("hibernate.hbm2ddl.auto", "create-drop");
		hibernateProperties.setProperty("hibernate.dialect", "org.hibernate.dialect.H2Dialect");

		return hibernateProperties;
	}
}
```
> Dialect?
> A database driver is a program for which implements a protocol (ODBC, JDBC) for connecting to a database. It is an Adaptor which  connects a generic interface to a specific vendors implementation, just like printer drivers etc.
> A database dialect is a configuration setting for platform independent software (JPA, Hibernate, etc) which allows such software to translate its generic SQL statements into verdor specific DDL, DML.
> 
> > 출처: https://stackoverflow.com/questions/2085368/difference-between-database-drivers-and-database-dialects

## Usage
```java
public abstract class TestHibernateDAO {
	@Autowired
	private SessionFactory sessionFactory;
	//...
}
```

> 출처: https://www.baeldung.com/hibernate-5-spring
