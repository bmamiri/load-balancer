# load-balancer

**Traditional server side load balancing** <br>
Server-side load balancing is involved in monolithic applications where we have a limited number of application instances behind the load balancer. We deploy our war/ear files into multiple server instances which are a pool of server having the same application deployed and we put a load balancer in front of it. <br>

The load balancer has a public IP and DNS. The client makes a request using that public IP/DNS. The load balancer decides to which internal application server request will be forwarded to. It mainly uses a round-robin or sticky session algorithm. We call it server-side load balancing. <br>

**Problems in a microservice's architecture** <br>
Mostly server-side load balancing is a manual effort, and we need to add/remove instances manually to the load balancer to work. So ideally we are losing today’s on-demand scalability to auto-discover and configure when any new instances will be spun off. <br>

Another problem is to have a fail-over policy to provide the client with a seamless experience. Finally we need a separate server to host the load balancer instance which has an impact on cost and maintenance. <br>

**Client side load balancing** <br>
To overcome the problems of traditional load balancing, client-side load balancing came into the picture. They reside in the application as an inbuilt component and bundled along with the application, so we don’t have to deploy them in separate servers. <br>

Now let’s visualize the big picture. In a microservice architecture, we will have to develop many microservices and each microservice may have multiple instances in the ecosystem. To overcome this complexity we have already one popular solution to use service discovery patterns. In spring boot applications, we have a couple of options in the service discovery space such as eureka, consul, zookeeper, etc. <br>

Now if one microservice wants to communicate with another microservice, it generally looks up the service registry using discovery client and the Eureka server returns all the instances of that target microservice to the caller service. Then it is the responsibility of the caller service to choose which instance to send the request. <br>

Here the client-side load balancing comes into the picture and automatically handles the complexities around this situation and delegates to a proper instance in load-balanced fashion. Note that we can specify the load balancing algorithm to use. <br>

**Netflix ribbon – Client-side load balancer** <br>
Netflix ribbon from Spring Cloud family provides such a facility to set up client-side load balancing along with the service registry component. Spring boot has a very nice way of configuring ribbon client-side load balancer with minimal effort. It provides the following features <br>

* Load balancing
* Fault tolerance
* Multiple protocol (HTTP, TCP, UDP) support in an asynchronous and reactive model
* Caching and batching

**Test the application** <br>
Eureka first, then the back-end microservice and finally the frontend microservice. <br>

**Deploy multiple instances of backend microservice** <br>
To do that we need to use different ports for this, to start service in a specific port, We will create 2 instances of this service in ports 9090 and 9092 ports.

**Verify eureka server**  <br>
Now go to http://localhost:8761/ in the browser and check eureka server is running with all microservices are registered with the desired number of instances <br>

**Check if client-side load balancing is working** <br>
In the frontend microservice, we are calling the backend microservice using RestTemplate. The rest template is enabled as a client-side load balancer using @LoadBalanced annotation. <br>

Now go to the browser and open the client microservice rest endpoint http://localhost:8888/client/frontend and see that response is coming from any one of the backend instances. <br>

To understand this backend server is returning its running port, and we are displaying that in client microservice response as well. Try refreshing this URL couple of times and notice that the port of the backend server keeps changing, which means client-side load balancing is working. Now try to add more instances of the backend server and check that is also registered in the eureka server and eventually considered in ribbon, as once that will be registered in eureka and ribbon automatically ribbon will send the request to the new instances as well. <br>

**Test with hard code backends without service discovery** <br>
Go the frontend microservice `application.properties` file and enable this. <br>

`server.ribbon.listOfServers=localhost:9090,localhost:9091` <br>
`server.ribbon.eureka.enabled=false` <br>

Now test the client URL. You will get a response from the registered instances only. Now if you start a new instance of backend microservice in a different port, Ribbon will not send requests to the new instance until we register that manually in the ribbon. <br>