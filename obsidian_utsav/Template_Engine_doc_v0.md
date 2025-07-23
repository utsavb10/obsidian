## Introduction
Welcome to Project Neon or Template Engine or Black Panther or call whatever you feel like, since everyone associated is doing so.
If you're here, you're probably tempted or forced to use widgetization to create your next UI feature on your application.
This guide aims to help you set up your environment quickly. Let's start.

## Add your Application
The first step is to register your application to the Black-Panther portal. You need to have a keycloak account to access the portal (the same credentials you use to access ops-portal). 

{Add Home page image without any existing projects}

On the landing page, you'll see all the applications to which you are allowed to contribute into along with a card with which you can add/register your application to the portal.

Clicking on the card will open a pop-up, asking information related to your portal. Right now, you only have to enter the name of the application (recommended pattern is UPPER_CASE_WITH_UNDESCORE).

You can also add the keycloak usernames of the developers/stakeholders involved with your application and they will also be given access to your application and it's page configurations.

Completing that completes your application registration with Neon. 
User access modifications can be done later also using the admin dahboard.

## Adding dependencies in your repositories
### Frontend Setup
...
### Backend Setup
First you need to add the **api-registry** dependency to your project's BFF (since your UI application is only supposed to call a BFF and then the BFF orchetrates the calls to necessary downstream services),
```xml
<dependency>  
  <groupId>com.navi.medici</groupId>  
  <artifactId>api-registry</artifactId>  
  <version>${api-registry.version}</version>  
</dependency>
```
Use the latest api-registry version (check nexus or contact us).

**api-registry** is a module within wakanda repository will call wakanda service pods at runtime to fetch page configurations. You'll need to add some environment configs needed by api-registry such as and also base urls of other downstream services from which data will be fetched for your UI.
```
WAKANDA_BASE_URL: https://dev-wakanda.np.navi-tech.in
DOWNSTREAM_RESPONSE_MAX_BUFFER_SIZE: 10485760
```

Next and final setup is to add this controller in your application,
```java
package com.navi.medici.webbff.controllers.api;

import static org.springframework.http.MediaType.APPLICATION_JSON_VALUE;
import com.navi.medici.webbff.commons.RequestUtils;
import com.navi.sa.metric.MethodMetric;
import com.navi.sa.mjolnir.config.Authorized;
import com.navi.wakanda.apiregistry.service.PageBuilderService;
import java.util.Map;
import javax.servlet.http.HttpServletRequest;
import lombok.extern.log4j.Log4j2;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Log4j2
@RequestMapping("/configuration")
@RestController
public class PageConfigurationsController {

@Autowired
private PageBuilderService pageBuilderService;

	@PostMapping(value = "/uri/**", produces = APPLICATION_JSON_VALUE)
	@Authorized(value = {"ui-template.page.config.read"})
	@MethodMetric("bff_get_template_page_config")
	public ResponseEntity<Object> generatePage(HttpServletRequest httpServletRequest, @RequestBody String requestBody) {
	
		final HttpHeaders headers = RequestUtils.duplicateHeaders(httpServletRequest);
		
		Object res = pageBuilderService.sendPageResponse(httpServletRequest.getRequestURI(),
		headers, Map.of(), requestBody);
		
		return ResponseEntity.ok(res);
	}

}
```

With this controller,
- BFF will be able to receive page configuration requests from your UI application. This controller uses `PageBuilderService` from the **api-registry** dependency to fetch configurations, make calls to downstream services if needed, stitch the response data with configuration and return the kind of response needed by Shuri UI dependency.
- Add any necessary customisations like `@Authorized` security permissions or `@MethodMetric`  performance metrics needed for your applications' requirements.

**NOTE: Don't forget to add wakanda's outbound connectivity in your BFF's deployment portal manifests.**

## Create your page configuration
...

### How to onboard a new backend API to widgetization ?

Use the `api-registry`([GitHub](https://github.cmd.navi-tech.in/medici/wakanda/tree/master/api-registry "https://github.cmd.navi-tech.in/medici/wakanda/tree/master/api-registry")) to record a new API. Api registry module is a binder of different backend APIs from where we connect backend data models/DAOs to widget specific data.

Concluding everything from [Component Downstream Contract](https://navihq.atlassian.net/wiki/spaces/BNKP/pages/456327786 "/wiki/spaces/BNKP/pages/456327786"), these are the steps needed to be taken to integrate backend api with a UI widget through api-registry,

1.  Create a ResponseModelClass which implements component specific `ComponentDataAdpater` interface to convert data into specific `ComponentData`.  
    Here we create an adapter class type based on the downstream API response DAO which returns the widget specific data according to requirement.
    
2.  Add the REST API as `GenericApi` object in downstream specific `registry`. The Generic API contains all the information that is required to hit the API at runtime. Any external header will get added later at runtime. Here we also mention the adapter class typed in the previous step. If the registry doesn’t exist already, we just need to make a one time effort to create a new downstream specific registry service and add our APIs there.
    

After these steps, we declare a new version of `api-registry` and then use this version in our wakanda/application and bff.

## Configuration promotion by Import/Export
While you can create/modify page configurations in any of the three environments right now, we still recommend you to make your changes in dev environment only, test your changes and then use the import/export feature to promote your configuration changes from one env to other.

Import button will copy the configuration to your clipboard and then you can use the export button on your target enviromnent to export the configuration. Configuration can be a new configuration or a modification to previously added page.

## Links
Please ensure you have access to these,
https://dev-black-panther.np.navi-tech.in/
https://qa-black-panther.np.navi-tech.in/
https://black-panther.prod.navi-tech.in/
https://github.cmd.navi-tech.in/medici/wakanda/tree/master/api-registry/src/main/java/com/navi/wakanda/apiregistry
https://nexus.cmd.navi-tech.in/#browse/browse:maven-snapshots:com%2Fnavi%2Fmedici%2Fapi-registry