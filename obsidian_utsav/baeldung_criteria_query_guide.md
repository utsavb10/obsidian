#java #persistence #criteria #query 

# JPA Criteria Queries

Last modified: February 11, 2022

by [baeldung](https://www.baeldung.com/author/baeldung/ "Posts by baeldung")

-   [Persistence](https://www.baeldung.com/category/persistence/)

-   [Hibernate](https://www.baeldung.com/tag/hibernate/)

### **Get started with Spring Data JPA through the reference _Learn Spring Data JPA_ course:**


## **1. Overview**[](https://www.baeldung.com/hibernate-criteria-queries#Overview)

In this tutorial, we'll discuss a very useful JPA feature — Criteria Queries.

It enables us to write queries without doing raw SQL as well as gives us some object-oriented control over the queries, which is one of the main features of Hibernate. The Criteria API allows us to build up a criteria query object programmatically, where we can apply different kinds of filtration rules and logical conditions.

**Since Hibernate 5.2, the Hibernate Criteria API is deprecated, and new development is focused on the JPA Criteria API.** We'll explore how to use Hibernate and JPA to build Criteria Queries.

## Further reading:

## [Spring Data JPA @Query](https://www.baeldung.com/spring-data-jpa-query)

Learn how to use the @Query annotation in Spring Data JPA to define custom queries using JPQL and native SQL.

[Read more](https://www.baeldung.com/spring-data-jpa-query) →

## [Introduction to Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)

Introduction to Spring Data JPA with Spring 4 - the Spring config, the DAO, manual and generated queries and transaction management.

[Read more](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) →

## [A Guide to Querydsl with JPA](https://www.baeldung.com/querydsl-with-jpa-tutorial)

A quick guide to using Querydsl with the Java Persistence API.

[Read more](https://www.baeldung.com/querydsl-with-jpa-tutorial) →

## **2. Maven Dependencies**[](https://www.baeldung.com/hibernate-criteria-queries#Dependencies)

To illustrate the API, we'll use the reference JPA implementation Hibernate.

To use Hibernate, we'll make sure to add the latest version of it to our _pom.xml_ file:

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>   
    <version>5.3.2.Final</version>
</dependency>
```

We can find the latest version of Hibernate [here.](https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-core%22)

## **3. Simple Example Using Criteria**[](https://www.baeldung.com/hibernate-criteria-queries#Criteria)

Let's start by looking at how to retrieve data using Criteria Queries. We'll look at how to get all the instances of a particular class from the database.

We have an _Item_ class that represents the tuple _“ITEM”_ in the database:

```java
public class Item implements Serializable {

    private Integer itemId;
    private String itemName;
    private String itemDescription;
    private Integer itemPrice;

   // standard setters and getters
}
```

Let's look at a simple criteria query that will retrieve all the rows of _“ITEM”_ from the database:

```java
Session session = HibernateUtil.getHibernateSession();
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<Item> cr = cb.createQuery(Item.class);
Root<Item> root = cr.from(Item.class);
cr.select(root);

Query<Item> query = session.createQuery(cr);
List<Item> results = query.getResultList();
```

The above query is a simple demonstration of how to get all the items. Let's see it step by step:


1.  Create an instance of _Session_ from the _SessionFactory_ object
2.  Create an instance of C_riteriaBuilder_ by calling the _getCriteriaBuilder()_ method
3.  Create an instance of _CriteriaQuery_ by calling the _CriteriaBuilder_ _createQuery()_ method
4.  Create an instance of _Query_ by calling the _Session_ _createQuery()_ method
5.  Call the _getResultList()_ method of the _query_ object, which gives us the results

Now that we've covered the basics, let's move on to some of the features of criteria query.

### **3.1. Using _Expressions_**[](https://www.baeldung.com/hibernate-criteria-queries#1-using-expressions)

**The _CriteriaBuilder_ can be used to restrict query results based on specific conditions**, by using _CriteriaQuery where()_ method and providing _Expressions_ created by _CriteriaBuilder_.

Let's see some examples of commonly used _Expressions_.

In order to get items having a price of more than 1000:

```java
cr.select(root).where(cb.gt(root.get("itemPrice"), 1000));
```

Next, getting items having _itemPrice_ less than 1000:


```java
cr.select(root).where(cb.lt(root.get("itemPrice"), 1000));
```

Items having _itemName_ contain _Chair_:

```java
cr.select(root).where(cb.like(root.get("itemName"), "%chair%"));
```

Records having _itemPrice_ between 100 and 200:

```java
cr.select(root).where(cb.between(root.get("itemPrice"), 100, 200));
```

Items having _itemName_ in _Skate Board_, _Paint_ and _Glue_:

```java
cr.select(root).where(root.get("itemName").in("Skate Board", "Paint", "Glue"));
```

To check if the given property is null:

```java
cr.select(root).where(cb.isNull(root.get("itemDescription")));
```

To check if the given property is not null:

```java
cr.select(root).where(cb.isNotNull(root.get("itemDescription")));
```

We can also use the methods _isEmpty()_ and _isNotEmpty()_ to test if a _List_ within a class is empty or not.

Additionally, we can combine two or more of the above comparisons. **T****he Criteria API allows us to easily chain expressions**:

```java
Predicate[] predicates = new Predicate[2];
predicates[0] = cb.isNull(root.get("itemDescription"));
predicates[1] = cb.like(root.get("itemName"), "chair%");
cr.select(root).where(predicates);
```

To add two expressions with logical operations:


```java
Predicate greaterThanPrice = cb.gt(root.get("itemPrice"), 1000);
Predicate chairItems = cb.like(root.get("itemName"), "Chair%");
```

Items with the above-defined conditions joined with _Logical OR_:

```java
cr.select(root).where(cb.or(greaterThanPrice, chairItems));
```

To get items matching with the above-defined conditions joined with _Logical AND_:

```java
cr.select(root).where(cb.and(greaterThanPrice, chairItems));
```

### **3.2. Sorting**[](https://www.baeldung.com/hibernate-criteria-queries#2-sorting)

Now that we know the basic usage of _Criteria_, let's look at the sorting functionalities of _Criteria_.

In the following example, we order the list in ascending order of the name and then in descending order of the price:

```java
cr.orderBy(
  cb.asc(root.get("itemName")), 
  cb.desc(root.get("itemPrice")));
```

In the next section, we will have a look at how to do aggregate functions.

### **3.3. Projections, Aggregates and Grouping Functions**[](https://www.baeldung.com/hibernate-criteria-queries#3-projections-aggregates-and-grouping-functions)

Now let's see the different aggregate functions.

Get row count:

```java
CriteriaQuery<Long> cr = cb.createQuery(Long.class);
Root<Item> root = cr.from(Item.class);
cr.select(cb.count(root));
Query<Long> query = session.createQuery(cr);
List<Long> itemProjected = query.getResultList();
```

The following is an example of aggregate functions — _Aggregate_ function for _Average_:

```java
CriteriaQuery<Double> cr = cb.createQuery(Double.class);
Root<Item> root = cr.from(Item.class);
cr.select(cb.avg(root.get("itemPrice")));
Query<Double> query = session.createQuery(cr);
List avgItemPriceList = query.getResultList();
```

Other useful aggregate methods are _sum()_, _max()_, _min()_, _count()_, etc.

### **3.4. _CriteriaUpdate_**[](https://www.baeldung.com/hibernate-criteria-queries#4-criteriaupdate)

**Starting from JPA 2.1, there's support for performing database updates using the _Criteria_ API.**

_CriteriaUpdate_ has a _set()_ method that can be used to provide new values for database records:

```java
CriteriaUpdate<Item> criteriaUpdate = cb.createCriteriaUpdate(Item.class);
Root<Item> root = criteriaUpdate.from(Item.class);
criteriaUpdate.set("itemPrice", newPrice);
criteriaUpdate.where(cb.equal(root.get("itemPrice"), oldPrice));

Transaction transaction = session.beginTransaction();
session.createQuery(criteriaUpdate).executeUpdate();
transaction.commit();
```

In the above snippet, we create an instance of _CriteriaUpdate<Item>_ from the _CriteriaBuilder_ and use its _set()_ method to provide new values for the _itemPrice_. In order to update multiple properties, we just need to call the _set()_ method multiple times.

### **3.5. _CriteriaDelete_**[](https://www.baeldung.com/hibernate-criteria-queries#5-criteriadelete)

_CriteriaDelete_ enables a delete operation using the _Criteria_ API.

We just need to create an instance of _CriteriaDelete_ and use the _where()_ method to apply restrictions:

```java
CriteriaDelete<Item> criteriaDelete = cb.createCriteriaDelete(Item.class);
Root<Item> root = criteriaDelete.from(Item.class);
criteriaDelete.where(cb.greaterThan(root.get("itemPrice"), targetPrice));

Transaction transaction = session.beginTransaction();
session.createQuery(criteriaDelete).executeUpdate();
transaction.commit();
```

## **4. Advantage Over HQL**[](https://www.baeldung.com/hibernate-criteria-queries#Advantage)

In the previous sections, we covered how to use Criteria Queries.

Clearly, **the main and most hard-hitting advantage of Criteria Queries over HQL is the nice, clean, object-oriented API.**

We can simply write more flexible, dynamic queries compared to plain HQL. The logic can be refactored with the IDE and has all the type-safety benefits of the Java language itself.

Of course, there are some disadvantages as well, especially around more complex joins.

So, we generally have to use the best tool for the job — that can be the Criteria API in most cases, but there are definitely cases where we'll have to go lower level.

## **5. Conclusion**[](https://www.baeldung.com/hibernate-criteria-queries#Conclusion)

In this article, we focused on the basics of Criteria Queries in Hibernate and JPA as well as on some of the advanced features of the API.

The code discussed here is available in the [GitHub repository](https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-hibernate-5).