#kotlin #lombok 
> Lombok annotations doesn't work with kotlin. There was a #compile time #error when building the service and error was like this ->
```
java: cannot find symbol
symbol: method getCustomerReferenceId()
location: variable requestCallback of type com.navi.medici.homeloanorigination.client.contract.journey.request.RequestCallback
```
RequestCallback was a java pojo with lombok annotations,
[Solution ->](https://stackoverflow.com/questions/65128763/java-you-arent-using-a-compiler-supported-by-lombok-so-lombok-will-not-work-a)  I added the argument below in the build process VM options in

```
-Djps.track.ap.dependencies=false
```

Setting:-

`Build, Execution, Deployment -> Compiler -> Shared build process VM options`