#kotlin #springboot #mapstruct

[document reference](https://medium.com/@muhammadusama_43306/how-to-use-mapstruct-with-spring-boot-kotlin-painlessly-55184765ace4)

# How to use MapStruct with Spring Boot (Kotlin) painlessly

## MapStruct’s whys, whens and hows?

Whether you’re a beginner or experienced spring boot engineer, with your brains intact 😉, of course, then you might’ve asked yourself a question why would I keep writing this **Boiler Plate Code** to convert **Models** to **DTO’s.**

Here [MapStruct](https://mapstruct.org/) comes into picture to let the compiler do this job without needing from developer.

If you’re a Java nerd, you might get away easily while configuring MapStruct for your **Spring Boot Project,** but what if you’re using **Kotlin** for your project? Then you might be in some trouble here, I’m like, **_been there done that,_** and I’m here to help you, to get you up to speed.

Complete [POM.xml](https://gist.github.com/the-mgi/df6fd9fd68d8ff4a7ce8b1b022cc28ba)

# **Steps to configure MapStruct for Kotlin**

1.  Add the MapStruct version variable in the **_pom.xml_** file, (variable created, will be used multiple times.)

<properties>  
    ...  
    <org.mapstruct.version>1.5.0.Beta1</org.mapstruct.version>  
</properties>

2. Add typical **_pom.xml_** dependency in the dependencies array, (to add MapStruct lib to the project)

<dependecies>  
    ...  
    <dependency>  
        <groupId>org.mapstruct</groupId>  
        <artifactId>mapstruct</artifactId>  
        <version>${org.mapstruct.version}</version>  
    </dependency>  
</dependecies>

3. Now we need to add the **_execution_** in the _org.jetbrains.kotlin:kotlin-maven-plugin,_ to execute the **_KAPT (K_**otlin annotation processor Tool**_)_** to generate the concrete **_Implementation_** of the interface (you’ll understand what I’m talking about).

<executions>  
    <execution>  
        <id>kapt</id>  
        <goals>  
            <goal>kapt</goal>  
        </goals>  
        <configuration>  
            <sourceDirs>  
                <sourceDir>src/main/kotlin</sourceDir>  
                <sourceDir>src/main/java</sourceDir>  
            </sourceDirs>  
            <annotationProcessorPaths>  
                <annotationProcessorPath>  
                    <groupId>org.mapstruct</groupId>  
                    <artifactId>mapstruct-processor</artifactId>  
                    <version>${org.mapstruct.version}</version>  
                </annotationProcessorPath>  
            </annotationProcessorPaths>  
        </configuration>  
    </execution>  
    <execution>  
        <id>compile</id>  
        <phase>compile</phase>  
        <goals>  
            <goal>compile</goal>  
        </goals>  
        <configuration>  
            <sourceDirs>  
                <sourceDir>${project.basedir}/src/main/kotlin</sourceDir>  
                <sourceDir>${project.basedir}/src/main/java</sourceDir>  
            </sourceDirs>  
        </configuration>  
    </execution>  
    <execution>  
        <id>test-compile</id>  
        <phase>test-compile</phase>  
        <goals>  
            <goal>test-compile</goal>  
        </goals>  
        <configuration>  
            <sourceDirs>  
                <sourceDir>${project.basedir}/src/test/kotlin</sourceDir>  
                <sourceDir>${project.basedir}/src/test/java</sourceDir>  
            </sourceDirs>  
        </configuration>  
    </execution>  
</executions>

Also add the MapStruct Processor dependency in the plugin

<dependencies>  
    <dependency>  
        <groupId>org.jetbrains.kotlin</groupId>  
        <artifactId>kotlin-maven-allopen</artifactId>  
        <version>${kotlin.version}</version>  
    </dependency>  
    <dependency>  
        <groupId>org.jetbrains.kotlin</groupId>  
        <artifactId>kotlin-maven-noarg</artifactId>  
        <version>${kotlin.version}</version>  
    </dependency>  
    <dependency>  
        <groupId>org.mapstruct</groupId>  
        <artifactId>mapstruct-processor</artifactId>  
        <version>${org.mapstruct.version}</version>  
    </dependency>  
</dependencies>

4. Now, there is a problem, that in default configuration, **Java Compiler** is invoked first, and the **Kotlin** compiler is invoked, but here we need to invoke the **Kotlin(**kotlin-maven-plugin**)** compiler first (to generate the concrete implementation) then the **Java(**maven-compiler-plugin**)** compiler to let it know that everything you want is present there already dude :). To do this add the following **plugin** to build configuration.

<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-compiler-plugin</artifactId>  
    <version>3.5.1</version>  
    <executions>  
        <!-- Replacing default-compile as it is treated specially by maven -->  
        <execution>  
            <id>default-compile</id>  
            <phase>none</phase>  
        </execution>  
        <!-- Replacing default-testCompile as it is treated specially by maven -->  
        <execution>  
            <id>default-testCompile</id>  
            <phase>none</phase>  
        </execution>  
        <execution>  
            <id>java-compile</id>  
            <phase>compile</phase>  
            <goals>  
                <goal>compile</goal>  
            </goals>  
        </execution>  
        <execution>  
            <id>java-test-compile</id>  
            <phase>test-compile</phase>  
            <goals>  
                <goal>testCompile</goal>  
            </goals>  
        </execution>  
    </executions>  
</plugin>

Configuration has been done, now let’s verify everything is as intended.

# Testing and Verifying

1.  Create mock Model and DTO classes, and Mapper interface which will do the magic 😉. Reference [MapStruct Example](https://github.com/mapstruct/mapstruct-examples/tree/master/mapstruct-kotlin).

data class Person(var firstName: String?, var lastName: String?, var phoneNumber: String?, var birthdate: LocalDate?)data class PersonDto(var firstName: String?, var lastName: String?, var phone: String?, var birthdate: LocalDate?)@Mapper  
interface PersonConverter {  
  
    @Mapping(source = "phoneNumber", target = "phone")  
    fun convertToDto(person: Person) : PersonDto  
  
    @InheritInverseConfiguration  
    fun convertToModel(personDto: PersonDto) : Person  
  
}

2. Now you need to do **mvn clean** and then **mvn install,** everything will be well and you’ll something like this in the target folder

![[images/Pasted_image_20220311152815.png]]

3. This is how you’ll use it then, Reference [MapStruct Example](https://github.com/mapstruct/mapstruct-examples/tree/master/mapstruct-kotlin).

val _converter_ = Mappers.getMapper(PersonConverter::class.java) // or PersonConverterImpl()  
  
val _person_ = Person("Samuel", "Jackson", "0123 334466", LocalDate.of(1948, 12, 21))  
  
val _personDto_ = _converter_.convertToDto(_person_)  
println(personDto)  
  
val _personModel_ = _converter_.convertToModel(_personDto_)  
println(personModel)

# Somethings to keep in mind

1.  Do turn on **Enable Annotation Processing,** if you’re using IntelliJ, if you’re not then you should ;)

![[images/Pasted_image_20220311152830.png]]

2. After adding a new **Mapper Interface** for any of your conversions, please do **mvn install,** simply again running the project from IntelliJ wouldn’t do the trick