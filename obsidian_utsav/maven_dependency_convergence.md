maven enforcer plugin issue
springframework.roo.annotations:1.2.3_RELEASE 401

exclude this from spring-data-jpa
```
<dependency>  
 <groupId>org.springframework.boot</groupId>  
 <artifactId>spring-boot-starter-data-jpa</artifactId>  
 <exclusion> 
   <groupId>org.springframework.roo</groupId>  
   <artifactId>org.springframework.roo.annotations</artifactId>  
 </exclusion>
</dependency>
```

also bump enforcer version to 3.0.0 instead of 3.0.0-M3