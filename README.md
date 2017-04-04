<h1>1. Overview</h1><br/><br/>

In this tutorial, we’ll introduce client-side service discovery via “Spring Cloud Netflix Eureka“.

Client-side service discovery allows services to find and communicate with each other without hardcoding hostname and port. The only ‘fixed point’ in such an architecture consists of a service registry with which each service has to register.

A drawback is that all clients must implement a certain logic to interact with this fixed point. This assumes an additional network round trip prior to the actual request.

With Netflix Eureka each client is able to simultaneously act as a server, to replicate its status to a connected peer. In other words, a client retrieves a list of all connected peers of a service registry and makes all further requests to any other services through a load-balancing algorithm.

To be informed about the presence of a client, they have to send a heartbeat signal to the registry.

To achieve the goal of this write-up, we will implement 3 microservices : <br/>
    <li>a service registry (<b>Eureka Server</b>),</li>
    <li>a REST service which registers itself at the registry (<b>Eureka Client</b>) and</li>
    <li>a web-application, which is consuming the REST service as registry-aware client (<b>Spring Cloud Netflix Feign Client</b>).</li>

<h1>2. Eureka Server</h1><br/><br/>

To implement a Eureka Server for using as service registry is as easy as: adding <a href="http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka-server%22">spring-cloud-starter-eureka-server</a> to the dependencies, enable the Eureka Server in a <a>href="http://www.baeldung.com/spring-boot-application-configuration">@SpringBootApplication</a> per annotate it with @EnableEurekaServer and configure some properties. But we’ll do it step by step.

Firstly we’ll create a new Maven project and put the dependencies into it. You have to notice that we’re importing the <a href="http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-parent%22">spring-cloud-starter-parent</a> to all projects described in this tutorial:
<div class="container">
  <div class="line number1 index0 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number2 index1 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.springframework.cloud&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number3 index2 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;spring-cloud-starter-eureka-server&lt;/</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number4 index3 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;1.1.5.RELEASE&lt;/</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number5 index4 alt2">
    <code class="xml plain">&lt;/</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number6 index5 alt1">&nbsp;</div>
   <div class="line number7 index6 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependencyManagement</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number8 index7 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependencies</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number9 index8 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number10 index9 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.springframework.cloud&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
    </div>
    <div class="line number11 index10 alt2">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">artifactId</code>
      <code class="xml plain">&gt;spring-cloud-starter-parent&lt;/</code>
      <code class="xml keyword">artifactId</code><code class="xml plain">&gt;</code>
     </div>
     <div class="line number12 index11 alt1">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">version</code>
      <code class="xml plain">&gt;Brixton.SR4&lt;/</code>
      <code class="xml keyword">version</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number13 index12 alt2">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">type</code>
      <code class="xml plain">&gt;pom&lt;/</code>
      <code class="xml keyword">type</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number14 index13 alt1">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">scope</code>
      <code class="xml plain">&gt;import&lt;/</code>
      <code class="xml keyword">scope</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number15 index14 alt2">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;/</code>
      <code class="xml keyword">dependency</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number16 index15 alt1">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;/</code>
      <code class="xml keyword">dependencies</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number17 index16 alt2">
      <code class="xml plain">&lt;/</code>
      <code class="xml keyword">dependencyManagement</code>
      <code class="xml plain">&gt;</code>
     </div>
    </div>
    <br/>
    Next, we’re creating the main application class:<br/>
    <table border="0" cellpadding="0" cellspacing="0">
      <tbody>
        <tr>
          <td class="gutter">
            <div class="line number1 index0 alt2">1</div>
            <div class="line number2 index1 alt1">2</div>
            <div class="line number3 index2 alt2">3</div>
            <div class="line number4 index3 alt1">4</div>
            <div class="line number5 index4 alt2">5</div>
            <div class="line number6 index5 alt1">6</div>
            <div class="line number7 index6 alt2">7</div>
           </td>
           <td class="code">
            <div class="container">
              <div class="line number1 index0 alt2">
                <code class="java color1">@SpringBootApplication</code>
              </div>
              <div class="line number2 index1 alt1">
                <code class="java color1">@EnableEurekaServer</code>
              </div>
              <div class="line number3 index2 alt2">
                <code class="java keyword">public</code> 
                <code class="java keyword">class</code> 
                <code class="java plain">EurekaServerApplication {</code>
              </div>
              <div class="line number4 index3 alt1">
                <code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
                <code class="java keyword">public</code> 
                <code class="java keyword">static</code> 
                <code class="java keyword">void</code> 
                <code class="java plain">main(String[] args) {</code>
              </div>
              <div class="line number5 index4 alt2">
                <code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
                <code class="java plain">SpringApplication.run(EurekaServerApplication.</code>
                <code class="java keyword">class</code>
                <code class="java plain">, args);</code>
              </div>
              <div class="line number6 index5 alt1">
                <code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
                <code class="java plain">}</code>
              </div>
              <div class="line number7 index6 alt2">
                <code class="java plain">}</code>
              </div>
            </div>
          </td>
        </tr>
      </tbody>
    </table><br/>
    Finally, we’re configuring the properties in YAML format; so application.yml will be our configuration file:<br/>
    <table border="0" cellpadding="0" cellspacing="0">
  <tbody>
    <tr>
      <td class="gutter">
        <div class="line number1 index0 alt2">1</div>
        <div class="line number2 index1 alt1">2</div>
        <div class="line number3 index2 alt2">3</div>
        <div class="line number4 index3 alt1">4</div>
        <div class="line number5 index4 alt2">5</div>
        <div class="line number6 index5 alt1">6</div>
      </td>
      <td class="code">
        <div class="container">
          <div class="line number1 index0 alt2">
            <code class="text plain">server:</code>
          </div>
          <div class="line number2 index1 alt1">
            <code class="text spaces">&nbsp;&nbsp;</code>
            <code class="text plain">port: 8761</code>
          </div>
          <div class="line number3 index2 alt2">
            <code class="text plain">eureka:</code>
          </div>
          <div class="line number4 index3 alt1">
            <code class="text spaces">&nbsp;&nbsp;</code>
            <code class="text plain">client:</code>
          </div>
          <div class="line number5 index4 alt2">
            <code class="text spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
            <code class="text plain">registerWithEureka: false</code>
          </div>
          <div class="line number6 index5 alt1">
            <code class="text spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
            <code class="text plain">fetchRegistry: false</code>
          </div>
        </div>
      </td>
    </tr>
  </tbody>
</table><br/>
Here we’re configuring an application port – 8761 is the default one for Eureka servers. We are telling the built-in Eureka Client not to register with ‘itself’, because our application should be acting as a server.

Now we will point our browser to <a href="http://localhost:8761">http://localhost:8761</a> to view the Eureka dashboard, where we will later inspecting the registered instances.

At the moment, we see basic indicators such as status and health indicators.

<h1>3. Eureka Client</h1>
For a @SpringBootApplication to be discovery-aware, we have to include some Spring Discovery Client (for example: <a href="http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-eureka%22">spring-cloud-starter-eureka</a>) into our classpath.

Then we need to annotate a @Configuration with either @EnableDiscoveryClient or @EnableEurekaClient.

The latter tells Spring Boot to explicitly use Spring Netflix Eureka for service discovery. To fill our client application with some sample-life, we’ll also include the <a href="http://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-web%22">spring-boot-starter-web</a> package in the pom.xml and implement a REST controller.

But first, we will add the dependencies:<br/>
   
   
<div class="container">
  <div class="line number1 index0 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number2 index1 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.springframework.cloud&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number3 index2 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;spring-cloud-starter-eureka&lt;/</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number4 index3 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;1.1.5.RELEASE&lt;/</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number5 index4 alt2">
    <code class="xml plain">&lt;/</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number6 index5 alt1">&nbsp;</div>
   
   <div class="line number1 index0 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number2 index1 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.springframework.boot&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number3 index2 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;spring-boot-starter-web&lt;/</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number4 index3 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;1.4.0.RELEASE&lt;/</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number5 index4 alt2">
    <code class="xml plain">&lt;/</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number6 index5 alt1">&nbsp;</div>
   
   <div class="line number7 index6 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependencyManagement</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number8 index7 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependencies</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number9 index8 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number10 index9 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.springframework.cloud&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
    </div>
    <div class="line number11 index10 alt2">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">artifactId</code>
      <code class="xml plain">&gt;spring-cloud-starter-parent&lt;/</code>
      <code class="xml keyword">artifactId</code><code class="xml plain">&gt;</code>
     </div>
     <div class="line number12 index11 alt1">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">version</code>
      <code class="xml plain">&gt;Brixton.SR4&lt;/</code>
      <code class="xml keyword">version</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number13 index12 alt2">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">type</code>
      <code class="xml plain">&gt;pom&lt;/</code>
      <code class="xml keyword">type</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number14 index13 alt1">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">scope</code>
      <code class="xml plain">&gt;import&lt;/</code>
      <code class="xml keyword">scope</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number15 index14 alt2">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;/</code>
      <code class="xml keyword">dependency</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number16 index15 alt1">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;/</code>
      <code class="xml keyword">dependencies</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number17 index16 alt2">
      <code class="xml plain">&lt;/</code>
      <code class="xml keyword">dependencyManagement</code>
      <code class="xml plain">&gt;</code>
     </div>
    </div>
    <br/>
    Next, we have to set-up an application.yml with a configured Spring application name to uniquely identify our client in the list of registered applications.

We are able to let Spring Boot choose a random port for us, because later we are accessing this service with its name, and finally we have to tell our client, were it has to locate the registry:<br/>

<table border="0" cellpadding="0" cellspacing="0">
  <tbody>
    <tr>
      <td class="gutter">
        <div class="line number1 index0 alt2">1</div>
        <div class="line number2 index1 alt1">2</div>
        <div class="line number3 index2 alt2">3</div>
        <div class="line number4 index3 alt1">4</div>
        <div class="line number5 index4 alt2">5</div>
        <div class="line number6 index5 alt1">6</div>
        <div class="line number7 index6 alt2">7</div>
        <div class="line number8 index7 alt1">8</div>
        <div class="line number9 index8 alt2">9</div>
        <div class="line number10 index9 alt1">10</div>
        <div class="line number11 index10 alt2">11</div>
      </td>
      <td class="code">
        <div class="container">
          <div class="line number1 index0 alt2">
            <code class="text plain">spring:</code>
          </div>
          <div class="line number2 index1 alt1">
            <code class="text spaces">&nbsp;&nbsp;</code>
            <code class="text plain">application:</code>
          </div>
          <div class="line number3 index2 alt2">
            <code class="text spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
            <code class="text plain">name: spring-cloud-eureka-client</code>
          </div><div class="line number4 index3 alt1">
          <code class="text plain">server:</code>
          </div>
          <div class="line number5 index4 alt2">
            <code class="text spaces">&nbsp;&nbsp;</code>
            <code class="text plain">port: 0</code>
          </div>
          <div class="line number6 index5 alt1">
            <code class="text plain">eureka:</code>
          </div>
          <div class="line number7 index6 alt2">
            <code class="text spaces">&nbsp;&nbsp;</code>
            <code class="text plain">client:</code>
          </div>
          <div class="line number8 index7 alt1">
            <code class="text spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
            <code class="text plain">serviceUrl:</code>
          </div>
          <div class="line number9 index8 alt2">
            <code class="text spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
            <code class="text plain">defaultZone: ${EUREKA_URI:<a href="http://localhost:8761/eureka">http://localhost:8761/eureka</a>}</code>
          </div>
          <div class="line number10 index9 alt1">
            <code class="text spaces">&nbsp;&nbsp;</code>
            <code class="text plain">instance:</code>
          </div>
          <div class="line number11 index10 alt2">
            <code class="text spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
            <code class="text plain">preferIpAddress: true</code>
          </div>
        </div>
      </td>
    </tr>
  </tbody>
</table>
<br/>
When we decided to setup our Eureka Client this way, we had in mind that this kind of service should later be easily scalable.

Now we will run the client and point our browser to <a href="http://localhost:8761">http://localhost:8761</a> again, to see its registration status on the Eureka Dashboard. By using the Dashboard we can do further configuration e.g. link the homepage of a registered client with the Dashboard for administrative purposes. The configuration options however, are beyond the scope of this article.

<h1>4. Feign Client</h1>

To setup our Feign Client project, we’re going to add the following 4 dependencies to its pom.xml:<br/>

<div class="container">
  <div class="line number1 index0 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number2 index1 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.springframework.cloud&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number3 index2 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;spring-cloud-starter-feign&lt;/</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number4 index3 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;1.1.5.RELEASE&lt;/</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number6 index5 alt1">&nbsp;</div>
   
   <div class="line number1 index0 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number2 index1 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.springframework.cloud&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number3 index2 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;spring-cloud-starter-eureka&lt;/</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number4 index3 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;1.1.5.RELEASE&lt;/</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number6 index5 alt1">&nbsp;</div>
   
   <div class="line number1 index0 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number2 index1 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.springframework.boot&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number3 index2 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;spring-boot-starter-web&lt;/</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number4 index3 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;1.4.0.RELEASE&lt;/</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number6 index5 alt1">&nbsp;</div>
   
   <div class="line number1 index0 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number2 index1 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.springframework.boot&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number3 index2 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;spring-boot-starter-thymeleaf&lt;/</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number4 index3 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;1.4.0.RELEASE&lt;/</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number6 index5 alt1">&nbsp;</div>
   
   <div class="line number1 index0 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number2 index1 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.thymeleaf&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number3 index2 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;thymeleaf&lt;/</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number4 index3 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;2.1.4.RELEASE&lt;/</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number6 index5 alt1">&nbsp;</div>
   
   <div class="line number1 index0 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number2 index1 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.springframework.boot&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number3 index2 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;spring-boot-starter-thymeleaf&lt;/</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;</code>
   </div>
   
   <div class="line number6 index5 alt1">&nbsp;</div>
   
   <div class="line number1 index0 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number2 index1 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;net.sourceforge.nekohtml&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number3 index2 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;nekohtml&lt;/</code>
    <code class="xml keyword">artifactId</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number4 index3 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;1.9.22&lt;/</code>
    <code class="xml keyword">version</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number6 index5 alt1">&nbsp;</div>
   
   <div class="line number7 index6 alt2">
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependencyManagement</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number8 index7 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependencies</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number9 index8 alt2">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">dependency</code>
    <code class="xml plain">&gt;</code>
   </div>
   <div class="line number10 index9 alt1">
    <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="xml plain">&lt;</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;org.springframework.cloud&lt;/</code>
    <code class="xml keyword">groupId</code>
    <code class="xml plain">&gt;</code>
    </div>
    <div class="line number11 index10 alt2">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">artifactId</code>
      <code class="xml plain">&gt;spring-cloud-starter-parent&lt;/</code>
      <code class="xml keyword">artifactId</code><code class="xml plain">&gt;</code>
     </div>
     <div class="line number12 index11 alt1">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">version</code>
      <code class="xml plain">&gt;Brixton.SR4&lt;/</code>
      <code class="xml keyword">version</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number13 index12 alt2">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">type</code>
      <code class="xml plain">&gt;pom&lt;/</code>
      <code class="xml keyword">type</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number14 index13 alt1">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;</code>
      <code class="xml keyword">scope</code>
      <code class="xml plain">&gt;import&lt;/</code>
      <code class="xml keyword">scope</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number15 index14 alt2">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;/</code>
      <code class="xml keyword">dependency</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number16 index15 alt1">
      <code class="xml spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
      <code class="xml plain">&lt;/</code>
      <code class="xml keyword">dependencies</code>
      <code class="xml plain">&gt;</code>
     </div>
     <div class="line number17 index16 alt2">
      <code class="xml plain">&lt;/</code>
      <code class="xml keyword">dependencyManagement</code>
      <code class="xml plain">&gt;</code>
     </div>
    </div>
    <br/>
    
    The Feign Client is located in the spring-cloud-starter-feign package. To enable it, we have to annotate a @Configuration with @EnableFeignClients. To use it, we simply annotate an interface with @FeignClient(“service-name”) and auto-wire it into a controller.

A good method to create such Feign Clients is to create interfaces with @RequestMapping annotated methods and put them into a separate module. This way they can be shared between server and client. On server-side you can implement them as @Controller and on client-side they can be extended and annotated as @FeignClient.

Furthermore the spring-cloud-starter-eureka package needs to be included in the project and enabled by annotating the main application class with @EnableEurekaClient.

The spring-boot-starter-web and spring-boot-starter-thymeleaf dependencies are used to present a view, containing fetched data from our REST service.

This will be our Feign Client interface:<br/>

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number1 index0 alt2">1</div><div class="line number2 index1 alt1">2</div><div class="line number3 index2 alt2">3</div><div class="line number4 index3 alt1">4</div><div class="line number5 index4 alt2">5</div></td><td class="code"><div class="container"><div class="line number1 index0 alt2"><code class="java color1">@FeignClient</code><code class="java plain">(</code><code class="java string">"spring-cloud-eureka-client"</code><code class="java plain">)</code></div><div class="line number2 index1 alt1"><code class="java keyword">public</code> <code class="java keyword">interface</code> <code class="java plain">GreetingClient {</code></div><div class="line number3 index2 alt2"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java color1">@RequestMapping</code><code class="java plain">(</code><code class="java string">"/greeting"</code><code class="java plain">)</code></div><div class="line number4 index3 alt1"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java plain">String greeting();</code></div><div class="line number5 index4 alt2"><code class="java plain">}</code></div></div></td></tr></tbody></table>
<br/>
Here we will implement the main application class which simultaneously acts as controller: <br/>

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number1 index0 alt2">1</div><div class="line number2 index1 alt1">2</div><div class="line number3 index2 alt2">3</div><div class="line number4 index3 alt1">4</div><div class="line number5 index4 alt2">5</div><div class="line number6 index5 alt1">6</div><div class="line number7 index6 alt2">7</div><div class="line number8 index7 alt1">8</div><div class="line number9 index8 alt2">9</div><div class="line number10 index9 alt1">10</div><div class="line number11 index10 alt2">11</div><div class="line number12 index11 alt1">12</div><div class="line number13 index12 alt2">13</div><div class="line number14 index13 alt1">14</div><div class="line number15 index14 alt2">15</div><div class="line number16 index15 alt1">16</div><div class="line number17 index16 alt2">17</div><div class="line number18 index17 alt1">18</div></td><td class="code"><div class="container"><div class="line number1 index0 alt2"><code class="java color1">@SpringBootApplication</code></div><div class="line number2 index1 alt1"><code class="java color1">@EnableEurekaClient</code></div><div class="line number3 index2 alt2"><code class="java color1">@EnableFeignClients</code></div><div class="line number4 index3 alt1"><code class="java color1">@Controller</code></div><div class="line number5 index4 alt2"><code class="java keyword">public</code> <code class="java keyword">class</code> <code class="java plain">FeignClientApplication {</code></div><div class="line number6 index5 alt1"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java color1">@Autowired</code></div><div class="line number7 index6 alt2"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java keyword">private</code> <code class="java plain">GreetingClient greetingClient;</code></div><div class="line number8 index7 alt1">&nbsp;</div><div class="line number9 index8 alt2"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java keyword">public</code> <code class="java keyword">static</code> <code class="java keyword">void</code> <code class="java plain">main(String[] args) {</code></div><div class="line number10 index9 alt1"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java plain">SpringApplication.run(FeignClientApplication.</code><code class="java keyword">class</code><code class="java plain">, args);</code></div><div class="line number11 index10 alt2"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java plain">}</code></div><div class="line number12 index11 alt1">&nbsp;</div><div class="line number13 index12 alt2"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java color1">@RequestMapping</code><code class="java plain">(</code><code class="java string">"/get-greeting"</code><code class="java plain">)</code></div><div class="line number14 index13 alt1"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java keyword">public</code> <code class="java plain">String greeting(Model model) {</code></div><div class="line number15 index14 alt2"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java plain">model.addAttribute(</code><code class="java string">"greeting"</code><code class="java plain">, greetingClient.greeting());</code></div><div class="line number16 index15 alt1"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java keyword">return</code> <code class="java string">"greeting-view"</code><code class="java plain">;</code></div><div class="line number17 index16 alt2"><code class="java spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="java plain">}</code></div><div class="line number18 index17 alt1"><code class="java plain">}</code></div></div></td></tr></tbody></table>
<br/>

This will be the HTML template( classpath : /resources/templates/greeting-view.html) for our view:<br/>

<table border="0" cellpadding="0" cellspacing="0"><tbody><tr><td class="gutter"><div class="line number1 index0 alt2">1</div><div class="line number2 index1 alt1">2</div><div class="line number3 index2 alt2">3</div><div class="line number4 index3 alt1">4</div><div class="line number5 index4 alt2">5</div><div class="line number6 index5 alt1">6</div><div class="line number7 index6 alt2">7</div><div class="line number8 index7 alt1">8</div><div class="line number9 index8 alt2">9</div></td><td class="code"><div class="container"><div class="line number1 index0 alt2"><code class="html plain">&lt;!DOCTYPE html&gt;</code></div><div class="line number2 index1 alt1"><code class="html plain">&lt;</code><code class="html keyword">html</code> <code class="html color1">xmlns:th</code><code class="html plain">=</code><code class="html string">"<a href="http://www.thymeleaf.org">http://www.thymeleaf.org</a>"</code><code class="html plain">&gt;</code></div><div class="line number3 index2 alt2"><code class="html spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="html plain">&lt;</code><code class="html keyword">head</code><code class="html plain">&gt;</code></div><div class="line number4 index3 alt1"><code class="html spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="html plain">&lt;</code><code class="html keyword">title</code><code class="html plain">&gt;Greeting Page&lt;/</code><code class="html keyword">title</code><code class="html plain">&gt;</code></div><div class="line number5 index4 alt2"><code class="html spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="html plain">&lt;/</code><code class="html keyword">head</code><code class="html plain">&gt;</code></div><div class="line number6 index5 alt1"><code class="html spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="html plain">&lt;</code><code class="html keyword">body</code><code class="html plain">&gt;</code></div><div class="line number7 index6 alt2"><code class="html spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="html plain">&lt;</code><code class="html keyword">h2</code> <code class="html color1">th:text</code><code class="html plain">=</code><code class="html string">"${greeting}"</code><code class="html plain">/&gt;</code></div><div class="line number8 index7 alt1"><code class="html spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code><code class="html plain">&lt;/</code><code class="html keyword">body</code><code class="html plain">&gt;</code></div><div class="line number9 index8 alt2"><code class="html plain">&lt;/</code><code class="html keyword">html</code><code class="html plain">&gt;</code></div></div></td></tr></tbody></table>
<br/>
At least the application.yml configuration file is almost the same as in the previous step:<br/>

<div class="container">
  <div class="line number1 index0 alt2">
    <code class="text plain">spring:</code>
    </div>
  <div class="line number2 index1 alt1">
    <code class="text spaces">&nbsp;&nbsp;</code>
    <code class="text plain">application:</code>
  </div>
  <div class="line number3 index2 alt2">
    <code class="text spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="text plain">name: spring-cloud-eureka-feign-client</code>
  </div>
  <div class="line number4 index3 alt1">
    <code class="text plain">server:</code>
  </div>
  <div class="line number5 index4 alt2">
    <code class="text spaces">&nbsp;&nbsp;</code>
    <code class="text plain">port: 8080</code>
  </div>
  <div class="line number6 index5 alt1">
    <code class="text plain">eureka:</code>
  </div>
  <div class="line number7 index6 alt2">
    <code class="text spaces">&nbsp;&nbsp;</code>
    <code class="text plain">client:</code>
  </div>
  <div class="line number8 index7 alt1">
    <code class="text spaces">&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="text plain">serviceUrl:</code>
  </div>
  <div class="line number9 index8 alt2">
    <code class="text spaces">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</code>
    <code class="text plain">defaultZone: ${EUREKA_URI:<a href="http://localhost:8761/eureka">http://localhost:8761/eureka</a>}</code>
  </div>
  <div class="line number1 index0 alt2">
    <code class="text plain">thymeleaf:</code>
  </div>
  <div class="line number2 index1 alt1">
    <code class="text spaces">&nbsp;&nbsp;</code>
    <code class="text plain">check-template-location: true </code>
  </div>
  <div class="line number2 index1 alt1">
    <code class="text spaces">&nbsp;&nbsp;</code>
    <code class="text plain">prefix: classpath:/resources/templates/</code>
  </div>
  <div class="line number2 index1 alt1">
    <code class="text spaces">&nbsp;&nbsp;</code>
    <code class="text plain">suffix: .html</code>
  </div>
  <div class="line number2 index1 alt1">
    <code class="text spaces">&nbsp;&nbsp;</code>
    <code class="text plain">encoding: UTF-8</code>
  </div>
  <div class="line number2 index1 alt1">
    <code class="text spaces">&nbsp;&nbsp;</code>
    <code class="text plain">content-type: text/html </code>
  </div>
  <div class="line number2 index1 alt1">
    <code class="text spaces">&nbsp;&nbsp;</code>
    <code class="text plain">cache: true</code>
  </div>
</div>
<br/>
Now we are able to build and run this service. Finally we’ll point our browser to <a href="http://localhost:8080/get-greeting">http://localhost:8080/get-greeting</a> 

<h1>5. Conclusion</h1>

As we’ve seen, we’re now able to implement a service registry using Spring Netflix Eureka Server and register some Eureka Clients with it.

Because our Eureka Client from step 3 listens on a randomly chosen port, it doesn’t know its location without the information from the registry. With a Feign Client and our registry, we are able to locate and consume the REST service, even when the location changes.

Finally we’ve got a big picture about using service discovery in a microservice architecture.

The original articales for this setup is <a href="www.baeldung.com/spring-cloud-netflix-eureka">here</a> .
