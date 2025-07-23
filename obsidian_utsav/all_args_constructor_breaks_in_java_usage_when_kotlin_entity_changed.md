#kotlin #java #constructor

### Problem 
When kotlin DAO is used in a java codebase, there is chance of code breaking whenever there is a change in DAO structure since there we only create an all args contructor with nullable fields or non-nullable fields with default values.

### Solution
1. Explicitly expose a no args constructor
2. Use @JvmOverlaods annotation on constructor definition
[Official Documentation](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/-jvm-overloads/)
[Medium article](https://medium.com/android-news/demystifying-the-jvmoverloads-in-kotlin-10dd098e6f72)

>  jvm overloads will not cause any new inconsistencies. all new fields must be added at the end and must have a default value even if the default is null. so long as this rule is followed you should be avoiding this class of issues entirely