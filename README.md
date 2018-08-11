# Micro-Services
Synechron Training for Java Microservices.

Day1:
For resources visit: http://resources.way2learnonline.com/micro/ (Commited lab-docs-pdf-jan2018.zip and micro-ws-may2018.zip)

Monolithic Applications VS Micro-services
Monolithic Applications: One single appn having everything is called as monolithic appn
Disadvantages:
  1. One single appn having everything is called as monolithic appn, hence this is the biggest disadv.
  2. Developers take time to understand the appn
  3. Tightly coupled with technology stack, a change in tech can cause serious issues.
  
Micro-services: One big application with small small services
Advantages:
  1. Dev can become productive quickly
  2. Scaling the services is easy
  3. Every service can be developed in language of choice : eg- Search service in scala, cart service in ruby, etc; its easy to leverage        and use new technolgies due to this.
  4. Utilization of hardware resources is good.
  
Disadvantage:
  1. All the problems of distributed systems are present as microservice communicate using distributed calls.
  
Netflix Eureka library: For discovery service
Netflix Ribbon library: For client side load-balancing

Zones: 
- having all the service instances and individual eureka instances
- Every Eureka instances should be having the rtegistry of their own services and sync them up with other eureka instances in different zones. (Instances of eureka should be static may 2 or 3 instances)
    Refer example.png in the workspace folder
