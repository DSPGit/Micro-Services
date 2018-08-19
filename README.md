# Micro-Services
Synechron Training for Java Microservices.

Day1:
For resources visit: 
* [LAB RESOURCES](http://resources.way2learnonline.com/micro/) (Commited lab-docs-pdf-jan2018.zip and micro-ws-may2018.zip)

## Monolithic Applications VS Micro-services
### Monolithic Applications: One single appn having everything is called as monolithic appn
Disadvantages:
  1. One single appn having everything is called as monolithic appn, hence this is the biggest disadv.
  2. Developers take time to understand the appn
  3. Tightly coupled with technology stack, a change in tech can cause serious issues.
  
### Micro-services: One big application with small small services
**Advantages:**
  1. Dev can become productive quickly
  2. Scaling the services is easy
  3. Every service can be developed in language of choice : eg- Search service in scala, cart service in ruby, etc; its easy to leverage        and use new technolgies due to this.
  4. Utilization of hardware resources is good.
  
**Disadvantage:**
  1. All the problems of distributed systems are present as microservice communicate using distributed calls.
  
Netflix Eureka library: For discovery service
Netflix Ribbon library: For client side load-balancing

  Zones: 
  - having all the service instances and individual eureka instances
  - Every Eureka instances should be having the rtegistry of their own services and sync them up with other eureka instances in different zones. (Instances of eureka should be static may 2 or 3 instances)
      Refer example.png in the workspace folder

### Circuit issue in micro-services:
````
Example:
If service 
  s1 is dependent on s2
  s2 is dependent on s3, s4
  s3 is dependent on s4
````
This is above where services create a circuit in within, so if S4 is down then we have a downfall in network then we should have a FALLBACK mechanism. However one failure should never be treated as circuit, there should be failure and success counters and if the % of failure count is more than specific then only it should be treated as circuit.
For this issue Netflix already has a library namely ***NETFLIX HYSTRIX***
>>>This library has to be present in the all the services as we have to check for circuit before making any request to other service.
````
Example :
S1 wants to make a call to S2:
S1 ---> Eureka (DS) 
  Eureka (DS) ---> S1
  S1 ---> Ribbon (LB)
  S1 ---> HYSTRIX --> S2
````

**Exposing internal services to external services**
No external service should be able to access my service architecture hence we should be having a proxy interface which will call the service on behalf of us. To acheive this there is a third party service provided by netflix **ZUUL**
````
Example:
ES (external service) wants to make a call to (your internal service) S2:
ES --> Netflix Zuul ---> Eureka (DS) 
  Eureka (DS) ---> Zuul
  Zuul ---> Ribbon (LB)
  Zuul ---> HYSTRIX --> S2
So here Zuul will be the interface which will handle all the responsibilities i.e. LB, DS and calling the target service.
````
**Having configuration of all the services on single server**
  
## SPRING BOOT for implementing microservices
  1. Automatic configuration
  2. If you just add module in the classpath, then all the beans required for added module will be created by boot.
    Eg: if you add JPA in classpath then all beans related to JPA will be created.
  3. No unnecessary JAR should be added in classpath or else unecessary beans will be created.
  4. In order to use boot then in pom.xml you need to add below code: (this code is must)
          <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.0.0.RC1</version>
            <relativePath />
          </parent>
  5. you can then go ahead and provide the required artifacts in the dependencies.
  6. once you add the required dependencies you have to use ***@AutoConfiguration*** annotation in the class
  
  ### How spring boot works:
   1. Read bean post processor (life cycle of spring bean)
   
# DAY 2 : REST/EUREKA/RIBBON
## CLOUD
  1. What is CLoud?
    - **SaaS**
      . SAP
      . Source Fourge
    - **PaaS**
      . Pivotal Cloud foundry
      . Heroku
    - **IaaS**
      . AWS
      . Google Cloud Platform
      . Azure
  - Whatever is not yours is with cloud. 
  - If we use IaaS : we just get the plain OS infrastructure and if I want to run a WAR on it we have to configure all the pre-requisite
  - If we use PaaS : we just have to say we want to run WAR then all the pre-requisite are handled by Platform
  - In SaaS : all the code is ready we just have to configure.
  
  **Problem statement:**  I want to write an appn which is cloud independent: i.e I can switch within clouds without changing code?
  **Solution:** ***Spring cloud*** provides set of interfaces which have implementors of Heruko, Cloud platform etc. Just change the dependency in spring boot and the required beans will be created.
  
# SPRING CLOUD
  - Its a library which can be deployed on PaaS
  - Its a big umbrella which has different projects like Eureka, ribbons, etc.
  
## How to start Eureka:
  - Eureka is just an REST api.
  - Spring cloud eureka is developed on top of spring boot, hence boot will add all the beans for you once you add the dependency
  - Spring boot parent doesnt have dependecy versions for spirng cloud api's hence we need to have different parent for spring cloud but we cannot have multiple parents in one pom so we add the cloud parent in <dependencyManagement> like below: 
* [Spring cloud](http://projects.spring.io/spring-cloud/)

````xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.1.RELEASE</version>
</parent>
<dependencyManagement>
    ***<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR1</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>***
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
</dependencies>
````
***Note:*** Please make sure you have correct compatibilty version of sprint boot parent and spring cloud.

- After which just use ***@EnableEurekaServer*** annotation in main class and run, the annotation enables all eureka rest controllers
- this above annotation will bring all the required beans on client side
- **default port of eureka is 8761**
- By default behavior of eureka is that it will first fetch the registry from other eureka server, if you have only one ES then you need to override the default behavior by adding a property in application.yml
````yaml
eureka:
  client:
    fetch-registry: false // no other ES hence value false
    register-with-eureka: false //there is no other ES to register with hence false; even if you have multiple ES but dont want to register
````
- every service needs to register with the ES hence we need to add below code in application.yml of your service
````yaml
eureka:
  client:
    service-url:
      z1: http://localhost:5001/eureka
````
- how many eureka are there with the zones? configure this.
````yaml
eureka:
  client:
    service-url:
      z1: http://localhost:5001/eureka
````
- If we have no zones then use below:
````yaml
eureka:
  client:
    service-url: 
      defaultZone: http://localhost:5001/eureka
````
- By default the service will start on port 8080, but if the port is not available then we can add below code in yaml
````yaml
server:
  port: 0 //0 means any random port available
````
- if you want to have multiple instance of services then the application id is generated based on combination of
  **host:service-name:port**
- this will make the id of the 2 service instances same, and eureka will show only one service in dashboard; hence you need to have different ids we need to make below changes: this code will generate random ids for each instance of service
````yaml
eureka:
  instance:
    instance-id:${spring.application.name}:${random.value}
````
**Note:** To monitor the id of the services once you start the service check the (TCP/IP Monitor) in eclipse

- In lease renewal: if the ES doesnt get the service reneval after its lease is expired it keeps on checking service and if 70% of the services dont renew (or dont provide heartbeat)then ES will go in **self preservation mode** and it will wake up automatically once it starts getting heartbeats.
- This threshold of 70% is set by Netflix and is not configurable.
- we can disable self preservation mode by using:
````yaml
eureka:
  server:
    enable-self-preservation: false
````
- **eviction thread** runs every 60 seconds
- when the service isnt giving heartbeat then ES wont remove it immediately from registry but will be marked for **eviction** then the eviction thread which runs after 60s will remove it in the next round of eviction. so the service gets max of 150s (90s of heartbeat and 60s of eviction thread)
***Open Q***  after eviction if the service tries to send the ES its heartbeat what happens, as the srvice is removed from registry ??

### Steps to start/configure the eureka client
- Step 1: Add dependency of eureka-client
- Step 2: Add annotation @EnableEurekaServer in the main class

# DAY 3 : RIBBON
- Ribbon is used as load balancer
- Ribbon by default uses DynamicServerListLoadBalancer for load balancing which can be configured
- To simplify code we can use restTemplate with **interceptors**
````java
Use @LoadBalancer annotation
````
- if one of the service replica is down then the loadbalancer keeps on hitting the lost one and the alive ones alternatively and the response we get from lost service is 404, to avoid this our loadbalancer should keep on retrying and not show 404, ***Spring provides one api called SPRING RETRY*** 
````xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
    <version>1.1.5.RELEASE</version>
</dependency>
````
- Spring retry api inbuilt uses DynamicServerListLoadBalancer (which is configurable) and if the service is down then it will lookup the registry and hit the live service. The retry and timeout is configurable as below:
````yaml
spring:
  cloud:
    loadbalancer:
      retry:
        enabled: true
````
-  you can configure the no. of retry's for a specifc service using below config:
````yaml
quotes-service: //this is the name of the service
  ribbon:
    ReadTimeout: 1000 // server is up, connection is successful but server is not responding in specified time, in our case its 1sec
    ConnectTimeout: 1000 // htpp request connection timeout
    OkToRetryOnAllOperations: true
    MaxAutoRetriesNextServer: 2
    MaxAutoRetries: 2 // max no. of autotries on particular service instance
````
### Algorithms in load balancing
- **ILoadBalancer >> Interface**: [Link](https://github.com/Netflix/ribbon/blob/master/ribbon-loadbalancer/src/main/java/com/netflix/loadbalancer/ILoadBalancer.java)  
- one of the implementor of this interface is **DynamicServerListLoadBalancer** which has a method ***getServerListImpl*** this will return list of all the services irrespective of the zones, to get only the services present in its zones we use another implementor of this interface which is **ZoneAwareLoadBalancer**

- Another interface is there which is **IRule >> Interface**: [Link](https://netflix.github.io/ribbon/ribbon-core-javadoc/com/netflix/loadbalancer/IRule.html)
- IRule intern uses ILoadBalancer : check method ***setLoadBalancer(ILoadBalancer lb)***
  - There is an implementor of this interface namely **RoundRobinRule** this uses round robin algorthm to choose the service
  - There is a child of **RoundRobinRule** namely **WeightedResponseTimeRule** this implementor will assign weights to the services based on the avg response time and based on that round-robin logic will run to choose the service.

- By default spring boot uses **DynamicServerListLoadBalancer** if we dont want this then we can write our own child class and override the implementation 
  
### Hystrix (for Circuit breaking)
- Spring has ***spring-cloud-hystrix*** a wrapper around Netflix Hystrix
  - Step 1: just add the dependency in pom
  - Step 2: Annotation ***@EnableHystrix***
  - Step 3: Annotation ***@HystrixCommand***
- We need to manage the endpoints for this use this config in yaml: (note below setting is not present in labs document hence dont forget to add this in portfolio service)
````yaml
management:
  endpoints:
    web:
      expose: 
       beans,metrics
````
- Spring Boot - Actuator. Spring Boot Actuator includes a number of additional features to help you monitor and manage your application when it's pushed to production. You can choose to manage and monitor your application using HTTP or JMX endpoints.

## FEIGN
- we can configure declarative restful client i.e we dont need to write any logic to consume any rest api but just write an interface with required configuration/
- Step 1 : add dependency
- Step 2 : Write interface as belows for eg:
````java
@FeignClient(name="quotes-service",fallback=QuoteServiceCallback.class)
public interface QuoteService {

  @RequestMapping(value = "/quotes", method = RequestMethod.GET)
	public ResponseEntity<List<Quote>> getQuotes(@RequestParam(value="q", required=false) String query)  ;
	
}
```` 
## Config Server
- If we have multiple services then we can have a common configuration repo (source control), condition is we should have the yml file with the service name i.e if service name is XYZservice then the config name will be XYZservice.yml
- This repo can have a common application.yml which is common to all the services
- We can have profile based configs as well like I want config for dev envt. and my microservice name is A then we can have 
    - A.yml, (config specific to A microservice)
    - A-dev.yml, (config specific to dev envt.)    
    - application.yml (common config to all the microservices)
- **Disadvantage** if the config server is down then our microservices wont be started ; pre-reqiusite is we need to have config server up-and-running before starting any microservice

# DAY 4: OAuth2/Zuul (gateway)
## Zuul
- Zuul is an edge service that provides dynamic routing, monitoring, resiliency, security, and more. 
- Routing is an integral part of a microservice architecture. For example, / may be mapped to your web application, /api/users is mapped to the user service and /api/shop is mapped to the shop service. Zuul is a JVM-based router and server-side load balancer from Netflix.
- [Zuul Wiki](https://github.com/Netflix/zuul/wiki)
- To expose an internal services, we use a proxy where 3rd party will hit the proxy(which will lookup eureka, and hit the service using ribbon lb).
- this proxy should also have config for blacklist and whitelist services and when request to blackliosted service comes it should block the request.
- this proxy acts as gateway btwn 3rd part and your services.
- We can then have another service which is exposed to the proxy and that service can internally call the services for request made by proxy.
- this service is called as orchestrated service.
- for this gateway we can use **Netflix Zuul!**.
- zuul acts as service side LB where only external request will be managed by zuul and not the internal request.
- While accessing the internal services zuul wil have to know the service name, however we dont want to expose the real name of service hence we need to configure alias for the service using below code in bootstrap.yml
````yaml
zuul:
 routes:
    quotes-service: /qs/*
    portfolio-service: /ps/** //double ** is for multiple elements
````
- in above config when the external service hits url with alias **/qs/** then it will be routed to **quote-service** automatically.
- we can ignore or blacklist the services from zuul for this we have to use below config: below config will ignore requests for quotes service
````yaml
zuul:
  prefix: /api
  ignored-services:
    quotes-service //(comma seperated service names) 
````
### Zuul Filters
- Just write a class which extends **ZuulFilter** and override ***run()***
````java
package com.way2learnonline;

import javax.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Component;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;

@Component
public class LoggingZuulFilter extends ZuulFilter {

	@Override
	public Object run() {
		 RequestContext ctx = RequestContext.getCurrentContext();
	        HttpServletRequest request = ctx.getRequest();

	       System.err.println(String.format("%s request to %s", request.getMethod(), request.getRequestURL().toString()));
	        return null;
	}

	@Override
	public boolean shouldFilter() {
		// TODO Auto-generated method stub
		return true;
	}

	@Override
	public int filterOrder() {
		// TODO Auto-generated method stub
		return 0;
	}

	@Override
	public String filterType() {
		// TODO Auto-generated method stub
		return "pre";
	}

}
````
### OAuth2 (Spring Security)
- Authentication
- Authorization
- Session
- Cross-Site Request Forgery (CSRF)
- Cross-origin resource sharing (CORS)
- Logout
- In spring security we need to do below config in security.xml in web-app/WEB-INF
````xml
<security:authentication-manager>
	<security:authentication-provider>	
			<security:user-service>				
			<security:user name="siva" password="secret" authorities="ROLE_ADMIN"/>
		</security:user-service>
	</security:authentication-provider>	
</security:authentication-manager> 
````
- and for security filters add below snippet
````xml
<security:http auto-config="true" use-expressions="true">
	<security:intercept-url pattern="/**" access="hasRole('ROLE_ADMIN')"/>
</security:http>
````
- after adding above config for any page access user will be prompted for login as we have mentioned ***pattern="/**"***
- 

#### Deligating filter proxy (DFP)
- Servlet filter need to configure in web.xml
- [AuthenticationManagerBuilder](https://docs.spring.io/spring-security/site/docs/4.2.5.RELEASE/apidocs/org/springframework/security/config/annotation/authentication/builders/AuthenticationManagerBuilder.html)

