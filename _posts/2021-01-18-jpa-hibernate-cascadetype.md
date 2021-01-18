---
title: "Overview of JPA/Hibernate Cascade Types"
date: 2021-01-18 16:23:28 -0400
categories: jpa
tags:
  - jpa
  - hibernate
  - database
---
## Entity의 상태
1. Transient: 객체를 생성하고, 값을 주어도 JPA나 hibernate가 그 객체에 관해 아무것도 모르는 상태. 즉, database와 mapping된 것이 아무것도 없다.
2. Persistent: 저장을 하고 나서, JPA가 아는 상태(관리하는 상태)가 된다. 그러나 save()를 했다고 바고 DB에 이 객체의 data가 들어가는 것은 아니다. JPA가 persistent 상태로 관리하고 있다가, 후에 data를 저장한다. (1차 캐시, Dirty Chechking(변경사항 감지), Write Behind(최대한 늦게, 필요한 시점에 DB에 적용) 등의 기능을 제공함)
3. Detached: JPA가 더이상 관리하지 않는 상태, JPA가 제공해주는 기능들을 사용하려면, 다시 persistent 상태로 돌아가야 한다.
4. Removed: JPA가 관리하는 상태이긴 하지만, 실제 commit이 일어날 때, 삭제가 일어난다. 

**Cascade는 이러한 상태변화를 전이시킨다.**

## What is Cascading?
Entity relationships often depend on the existence of another entity - for example, the *Person-Address* relationshp, Without the *Person*, the *Address* entity doesn't have any meaning of its own. When we delete the Person entity, our Address entity should also get deleted.
**When we perform some action on the target entity, the same action will be applied to the associated entity.**
### JPA Cascade Type
All JPA-specific cascade operations are represented by the *javax.persistence.CascadeType* enum containing entries:
- ALL
- PERSIST
- MERGE
- REMOVE
- REFRESH
- DETACH
### Hibernate Cascade Type
Hibernate supprts three additional Cascade Types along with those specified by JPA. These Hibernate-specific Cascade Types are available in org.hibernate.annotaions.CascadeType:
- REPLICATE
- SAVE_UPDATE
- LOCK
## Difference Between the Cascade Types
### CascadeType.ALL
*Cascade.ALL* **propagates all operations —  including Hibernate-specific ones — from a parent to a child entity.**
```java
@Entity
public class Person {
	@Id
	@GeneratedValue(strategy = GenerationType.AUTO)
	private int id;
	private String name;
	@OneToMany(mappedBy = "person", cascade = CascadeType.ALL)
	private List<Address> addresses;
}
```
```java
@Entity
public class Address {
	@Id
	@GeneratedValue(strategy = GenerationType.AUTO)
	private int id;
	private String street;
	private int houseNumber;
	private String city;
	private int zipCode;
	@ManyToOne(fetch = FetchType.LAZY)
	private Person person;
}
```
### CascadeType.PERSIST
The persist operation makes a transient instance persistent.
Person의 instance가 Transient에서 Persistent 상태로 넘어갈 때, child 객체(Address)도 같이 Persistent 상태가 되면서 같이 저장이 된다.
```java
@Test  
public void whenParentSavedThenChildSaved() { 
	Person person = new Person(); 
	Address address = new Address(); 
	address.setPerson(person); 
	person.setAddresses(Arrays.asList(address)); 
	session.persist(person); 
	session.flush(); 
	session.clear(); 
}
```
When we run the above test case, we'll see the following SQL:
```sql
Hibernate: insert into Person (name, id) values (?, ?) 
Hibernate: insert into Address ( city, houseNumber, person_id, street, zipCode, id) values (?, ?, ?, ?, ?, ?)
```
### CascadeType.MERGE
The merge operation copies the state of the given object onto the persistent object with the same identifier.
```java
@Test  
public void whenParentSavedThenMerged() { 
	int addressId; 
	Person person = buildPerson("devender"); 
	Address address = buildAddress(person); 
	person.setAddresses(Arrays.asList(address)); 
	session.persist(person); 
	session.flush(); 
	addressId = address.getId(); 
	session.clear();  

	Address savedAddressEntity = session.find(Address.class, addressId); 
	Person savedPersonEntity = savedAddressEntity.getPerson(); 
	savedPersonEntity.setName("devender kumar"); 
	savedAddressEntity.setHouseNumber(24); 
	session.merge(savedPersonEntity); 
	session.flush(); 
}
```
```sql
Hibernate: select address0_.id as id1_0_0_, address0_.city as city2_0_0_, address0_.houseNumber as houseNum3_0_0_, address0_.person_id as person_i6_0_0_, address0_.street as street4_0_0_, address0_.zipCode as zipCode5_0_0_ from Address address0_ where address0_.id=? 
Hibernate: select person0_.id as id1_1_0_, person0_.name as name2_1_0_ from Person person0_ where person0_.id=? 
Hibernate: update Address set city=?, houseNumber=?, person_id=?, street=?, zipCode=? where  id=? 
Hibernate: update Person set  name=? where  id=?
```
### CascadeType.REMOVE
**_CascadeType.REMOVE_  propagates the remove operation from parent to child entity.**  **Similar to JPA's _CascadeType.REMOVE,_  we have  _CascadeType.DELETE_, which is specific to Hibernate**.
### CascadeType.DETACH
The detach operation removes the entity from the persistent context.
```java
@Test  
public void whenParentDetachedThenChildDetached() { 
	Person person = buildPerson("devender"); 
	Address address = buildAddress(person); 
	person.setAddresses(Arrays.asList(address)); 
	session.persist(person); 
	session.flush();  
	
	assertThat(session.contains(person)).isTrue(); 
	assertThat(session.contains(address)).isTrue();  
	
	session.detach(person); 
	assertThat(session.contains(person)).isFalse(); 
	assertThat(session.contains(address)).isFalse(); 
}
```
### CascadeType.LOCK
**_CascadeType.LOCK_  re-attaches the entity and its associated child entity with the persistent context again.**
```java
@Test  public void whenDetachedAndLockedThenBothReattached() { 
	Person person = buildPerson("devender"); 
	Address address = buildAddress(person); 
	person.setAddresses(Arrays.asList(address)); 
	session.persist(person); 
	session.flush();  

	assertThat(session.contains(person)).isTrue(); 
	assertThat(session.contains(address)).isTrue();  

	session.detach(person); 
	assertThat(session.contains(person)).isFalse(); 
	assertThat(session.contains(address)).isFalse(); 
	session.unwrap(Session.class) 
		.buildLockRequest(new LockOptions(LockMode.NONE)) 
		.lock(person);  

	assertThat(session.contains(person)).isTrue(); 
	assertThat(session.contains(address)).isTrue(); 
}
```
### CascadeType.REFRESH
Refresh operations **re-read the value of a given instance from the database**.
```java
@Test  
public void whenParentRefreshedThenChildRefreshed() { 
	Person person = buildPerson("devender"); 
	Address address = buildAddress(person); 
	person.setAddresses(Arrays.asList(address)); 
	session.persist(person); 
	session.flush(); 
	person.setName("Devender Kumar"); 
	address.setHouseNumber(24); 
	session.refresh(person);  

	assertThat(person.getName()).isEqualTo("devender"); 
	assertThat(address.getHouseNumber()).isEqualTo(23); 
}
```
### CascadeType.REPLICATE
?
### CascadeType.SAVE_UPDATE
It's useful when we use **Hibernate-specific operations like  _save, update,_  and  _saveOrUpdate_**_._

> 출처: https://velog.io/@max9106/JPA%EC%97%94%ED%8B%B0%ED%8B%B0-%EC%83%81%ED%83%9C-Cascade

> 출처: https://www.baeldung.com/jpa-cascade-types
