#jpa #entity #java #table #join

## How to connect entities when they are not really related to each other ?

[JPA-Join-Column by Baeldung](https://www.baeldung.com/jpa-join-column)
[Query Unrelated entites by Baeldung](https://www.baeldung.com/jpa-query-unrelated-entities)

The domain of our example is a cocktail bar. Here we have two tables in the database:

-   The _menu_ table to store the cocktails that our bar sells and their prices, and
-   The _recipes_ table to store the instructions of creating a cocktail

```
menu
cocktail_name string,
price double
```

```
recipes
cocktail string
instructions string
```

These two tables are not strictly related to each other. A cocktail can be in our menu without keeping instructions for its recipe. Additionally, we could have available recipes for cocktails that we don't sell yet.

In our example, we are going to find all the cocktails on our menu that we have an available recipe.

We can easily create two JPA entities to represent our tables:

```java
@Entity
@Table(name = "menu")
public class Cocktail {
    @Id
    @Column(name = "cocktail_name")
    private String name;

    @Column
    private double price;

    // getters & setters
}
```

```java
@Entity
@Table(name="recipes")
public class Recipe {
    @Id
    @Column(name = "cocktail")
    private String cocktail;

    @Column
    private String instructions;
    
    // getters & setters
}
```

Between the _menu_ and _recipes_ tables, **there is an underlying one-to-one relationship without an explicit foreign key constraint**. For example, if we have a _menu_ record where its _cocktail_name_ column's value is “Mojito” and a _recipes_ record where its _cocktail_ column's value is “Mojito”, then the _menu_ record is associated with this _recipes_ record.

To represent this relationship in our _Cocktail_ entity, we add the _recipe_ field annotated with various annotations:

```java
@Entity
@Table(name = "menu")
public class Cocktail {
    // ...
 
    @OneToOne
    @NotFound(action = NotFoundAction.IGNORE)
    @JoinColumn(name = "cocktail_name", 
       referencedColumnName = "cocktail", 
       insertable = false, updatable = false, 
       foreignKey = @javax.persistence
         .ForeignKey(value = ConstraintMode.NO_CONSTRAINT))
    private Recipe recipe;
   
    // ...
}
```

The first annotation is _[@OneToOne](https://www.baeldung.com/jpa-one-to-one),_ which declares the underlying one-to-one relationship with the _Recipe_ entity.

Next, we annotate the _recipe_ field with the _@NotFound(action = NotFoundAction.IGNORE)_ Hibernate annotation. This tells our ORM to not throw an exception when there is a _recipe_ for a _cocktail_ that doesn't exist in our _menu_ table.

**The annotation that associates the _Cocktail_ with its associated _Recipe_ is [_@JoinColumn_](https://www.baeldung.com/jpa-join-column). By using this annotation, we define a pseudo foreign key relationship between the two entities.**

Finally, by setting the _foreignKey_ property to _@javax.persistence.ForeignKey(value = ConstraintMode.NO_CONSTRAINT),_ we instruct the JPA provider to not generate the foreign key constraint.

### The JPA and QueryDSL Queries[](https://www.baeldung.com/jpa-query-unrelated-entities#one-to-one-queries)

Since we are interested in retrieving the _Cocktail_ entities that are associated with a _Recipe,_ we can query the _Cocktail_ entity by joining it with its associated _Recipe_ entity.

One way we can construct the query is by using [JPQL](https://docs.oracle.com/html/E13946_04/ejb3_langref.html):

```java
entityManager.createQuery("select c from Cocktail c join c.recipe")
```

Or by using the QueryDSL framework:

```java
new JPAQuery<Cocktail>(entityManager)
  .from(QCocktail.cocktail)
  .join(QCocktail.cocktail.recipe)
```

Another way to get the desired results is to join the _Cocktail_ with the _Recipe_ entity and by using the _on_ clause to define the underlying relationship in the query directly.

We can do this using JPQL:

```java
entityManager.createQuery("select c from Cocktail c join Recipe r on c.name = r.cocktail")
```

or by using the QueryDSL framework:

```java
new JPAQuery(entityManager)
  .from(QCocktail.cocktail)
  .join(QRecipe.recipe)
  .on(QCocktail.cocktail.name.eq(QRecipe.recipe.cocktail))
```

### One-To-Many Underlying Relationship[](https://www.baeldung.com/jpa-query-unrelated-entities#one-to-many-underlying-relationship)

Let's change the domain of our example to show how we can **join two entities with a one-to-many underlying relationship**.

[![](https://www.baeldung.com/wp-content/uploads/2020/04/one-to-many.png)](https://www.baeldung.com/wp-content/uploads/2020/04/one-to-many.png)  
Instead of the _recipes_ table, we have the _multiple_recipes_ table, where we can store as many _recipes_ as we want for the same _cocktail_.

```java
@Entity
@Table(name = "multiple_recipes")
public class MultipleRecipe {
    @Id
    @Column(name = "id")
    private Long id;

    @Column(name = "cocktail")
    private String cocktail;

    @Column(name = "instructions")
    private String instructions;

    // getters & setters
}
```

Now, **the _Cocktail_ entity is associated with the _MultipleRecipe_ entity by a one-to-many underlying relationship** :

```java
@Entity
@Table(name = "cocktails")
public class Cocktail {    
    // ...

    @OneToMany
    @NotFound(action = NotFoundAction.IGNORE)
    @JoinColumn(
       name = "cocktail", 
       referencedColumnName = "cocktail_name", 
       insertable = false, 
       updatable = false, 
       foreignKey = @javax.persistence
         .ForeignKey(value = ConstraintMode.NO_CONSTRAINT))
    private List<MultipleRecipe> recipeList;

    // getters & setters
}
```

To find and get the _Cocktail_ entities for which we have at least one available _MultipleRecipe,_ we can query the _Cocktail_ entity by joining it with its associated _MultipleRecipe_ entities.

We can do this using JPQL:

```java
entityManager.createQuery("select c from Cocktail c join c.recipeList");
```

or by using the QueryDSL framework:
```java
new JPAQuery(entityManager).from(QCocktail.cocktail)
  .join(QCocktail.cocktail.recipeList);
```

There is also the option to not use the _recipeList_ field which defines the one-to-many relationship between the _Cocktail_ and _MultipleRecipe_ entities_._ Instead, we can write a join query for the two entities and determine their underlying relationship by using JPQL “on” clause:

```java
entityManager.createQuery("select c "
  + "from Cocktail c join MultipleRecipe mr "
  + "on mr.cocktail = c.name");
```

Finally, we can construct the same query by using the QueryDSL framework:

```java
new JPAQuery(entityManager).from(QCocktail.cocktail)
  .join(QMultipleRecipe.multipleRecipe)
  .on(QCocktail.cocktail.name.eq(QMultipleRecipe.multipleRecipe.cocktail));
```

### Many-To-Many Underlying Relationship[](https://www.baeldung.com/jpa-query-unrelated-entities#many-to-many-underlying-relationship)

In this section, we choose to categorize our cocktails in our menu by their base ingredient. For example, the base ingredient of the mojito cocktail is the rum, so the rum is a cocktail category in our menu.

To depict the above in our domain, we add the _category_ field into the _Cocktail_ entity:

```java
@Entity
@Table(name = "menu")
public class Cocktail {
    // ...

    @Column(name = "category")
    private String category;
    
     // ...
}
```

Also, we can add the _base_ingredient_ column to the _multiple_recipes_ table to be able to search for recipes based on a specific drink.

```java
@Entity
@Table(name = "multiple_recipes")
public class MultipleRecipe {
    // ...
    
    @Column(name = "base_ingredient")
    private String baseIngredient;
    
    // ...
}
```

After the above, here's our database schema:
[![](https://www.baeldung.com/wp-content/uploads/2020/04/many_to_many.png)](https://www.baeldung.com/wp-content/uploads/2020/04/many_to_many.png)

Now, **we have a many-to-many underlying relationship between _Cocktail_ and _MultipleRecipe_ entities**. Many _MultipleRecipe_ entities can be associated with many _Cocktail_ entities that their _category_ value is equal with the _baseIngredient_ value of the _MultipleRecipe_ entities.

To find and get the _MultipleRecipe_ entities that their _baseIngredient_ exists as a category in the _Cocktail_ entities, we can join these two entities by using JPQL:

```java
entityManager.createQuery("select distinct r " 
  + "from MultipleRecipe r " 
  + "join Cocktail c " 
  + "on r.baseIngredient = c.category", MultipleRecipe.class)
```

Or by using QueryDSL:

```java
QCocktail cocktail = QCocktail.cocktail; 
QMultipleRecipe multipleRecipe = QMultipleRecipe.multipleRecipe; 
new JPAQuery(entityManager).from(multipleRecipe)
  .join(cocktail)
  .on(multipleRecipe.baseIngredient.eq(cocktail.category))
  .fetch();
```