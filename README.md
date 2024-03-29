[![Build Status][Travis badge]][Travis build]

CREATE TABLE `products` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `price` int NOT NULL,
  `discounted` int NOT NULL,
  PRIMARY KEY (`id`)
);
INSERT INTO `products` (name,price, discounted)
VALUES ('piyush', 400, 5);
<bean id="dataSource"
		class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
		<property name="driverClassName"
			value="com.mysql.cj.jdbc.Driver" />
		<property name="url"
			value="jdbc:mysql://localhost:3306/restservices" />
		<property name="username" value="root" />
		<property name="password" value="root" />
	</bean>
     <bean id="sqlComponent"
		class="org.apache.camel.component.sql.SqlComponent">
		<property name="dataSource" ref="dataSource" />
	</bean>
[Travis badge]: https://travis-ci.org/astefanutti/spring-boot-camel-rest-jpa.svg
[Travis build]: https://travis-ci.org/astefanutti/spring-boot-camel-rest-jpa

# Spring Boot Camel REST / JPA Example

  <plugin>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-codegen-plugin</artifactId>
        <version>3.5.0</version>
    <executions>
        <execution>
            <id>generate-sources</id>
            <phase>generate-sources</phase>
            <configuration>
                <sourceRoot>${basedir}/src/main/java/org/country</sourceRoot>
                <wsdlOptions>
                    <wsdlOption>
                        <wsdl>${basedir}/src/main/resources/wsdl/CountryInfo.wsdl</wsdl>
                    </wsdlOption>
                </wsdlOptions>
            </configuration>
            <goals>
                <goal>wsdl2java</goal>
            </goals>
        </execution>
    </executions>
</plugin>

This example demonstrates how to use JPA and Camel's REST DSL
to expose a RESTful API that performs CRUD operations on a database.

It generates orders for books referenced in database at a regular pace.
Orders are processed asynchronously by another Camel route. Books available
in database as well as the status of the generated orders can be retrieved
via the REST API.

This example relies on the [Fabric8 Maven plugin](https://maven.fabric8.io)
for its build configuration and uses the
[fabric8 Java base image](https://github.com/fabric8io-images/java).
It relies on Swagger to expose the API documentation of the REST service.

## Build

The example can be built with:

    $ mvn install

This automatically generates the application resource descriptors and builds
the Docker image against a running Docker daemon (which must be accessible either
via Unix Socket or with the URL set in `DOCKER_HOST`). Alternatively, when connected
to an OpenShift cluster, then a Docker build is performed on OpenShift which at the end
creates an [ImageStream](https://docs.openshift.com/container-platform/3.6/architecture/core_concepts/builds_and_image_streams.html).

## Run

### Locally

The example can be run locally using the following Maven goal:

    $ mvn spring-boot:run

Alternatively, you can run the application locally using the executable
JAR produced:

    $ java -jar -Dspring.profiles.active=dev target/spring-boot-camel-rest-jpa-${project.version}.jar

This uses an embedded in-memory HSQLDB database. You can use the default
Spring Boot profile in case you have a MySQL server available for you to test.

You can then access the REST API directly from your Web browser, e.g.:

- <http://localhost:8080/camel-rest-jpa/books>
- <http://localhost:8080/camel-rest-jpa/books/order/1>

### Kubernetes

#### Prerequisites

It is assumed a Kubernetes cluster is already running. The easiest way to setup
a local single-node Kubernetes cluster is to [install Minikube](https://github.com/kubernetes/minikube#installation) and run:

    $ minikube start

Besides, it is assumed that a MySQL service is already running on the platform.
You can deploy it using the provided deployment by executing:

    $ kubectl create -f https://raw.githubusercontent.com/astefanutti/spring-boot-camel-rest-jpa/master/src/test/fabric8/mysql-kubernetes.yaml

You'll need to update the `MYSQL_USER` and `MYSQL_PASSWORD` environment variables in the template.

#### Deployment

The example can be deployed by executing the following command:

    $ mvn fabric8:deploy -Dmysql-service-username=<username> -Dmysql-service-password=<password>

The `username` and `password` system properties correspond to the credentials
used when deploying the MySQL database service.

You can then inspect the status of the deployment, e.g.:

- To list all the running pods:
    ```
    $ kubectl get pods
    ```

- Then find the name of the pod that runs this example, and output the logs from the running pod with:
    ```
    $ kubectl logs <pod_name>
    ```

#### Runtime

:pencil2:

### OpenShift

#### Prerequisites

It is assumed an OpenShift platform is already running. The easiest way to setup
a local single-node OpenShift cluster is to [install Minishift](https://github.com/minishift/minishift#installation) and run:

    $ minishift start

Besides, it is assumed that a MySQL service is already running on the platform.
You can deploy it using the provided deployment by executing:

    $ oc create -f https://raw.githubusercontent.com/openshift/origin/master/examples/db-templates/mysql-ephemeral-template.json
    $ oc new-app --template=mysql-ephemeral -p MYSQL_VERSION=5.6 -p MYSQL_USER=<username> -p MYSQL_PASSWORD=<password>

More information can be found in [using the MySQL database image](https://docs.openshift.com/container-platform/3.6/using_images/db_images/mysql.html).
You may need to relax the security in your cluster as the MySQL container requires
the `setgid` access right permission. This can be achieved by running the following command:

    $ oc adm policy add-scc-to-group anyuid system:authenticated

That grants all authenticated users access to the `anyuid` SCC. You can find
more information in [Managing Security Context Constraints](https://docs.openshift.com/container-platform/3.6/admin_guide/manage_scc.html).
For this command to run successfully, you would need to be logged in with a user that has the
`cluster-admin` role bound, e.g. with the default system account:

    $ oc login https://$(minishift ip):8443 -u system:admin

In case you need / want to access the Web console with a privileged account, you can create
an admin user account with:

```sh
$ oc adm policy add-cluster-role-to-user cluster-admin admin
# oc login -u admin -p admin
```

#### Deployment

The example can be deployed by executing the following command:

    $ mvn fabric8:deploy -Dmysql-service-username=<username> -Dmysql-service-password=<password>

The `username` and `password` system properties correspond to the credentials
used when deploying the MySQL database service.

This streams the pod logs into the console. Alternatively, you can use the
OpenShift client tool to inspect the status, e.g.:

- To list all the running pods:
    ```
    $ oc get pods
    ```

- Then find the name of the pod that runs this example, and output the logs from the running pod with:
    ```
    $ oc logs <pod_name>
    ```

#### Runtime

##### REST service

When the example is running, a REST service is available to list the books
that can be ordered, and as well the order statuses.
As it depends on your OpenShift setup, the hostname for the route
may vary. You can retrieve it by running the following command:

    $ oc get routes -o jsonpath='{range .items[?(@.spec.to.name == "camel-rest-jpa")]}{.spec.host}{"\n"}{end}'

The actual endpoint is using the _context-path_ `camel-rest-jpa/books` and
the REST service provides two services:

- `books`: to list all the available books that can be ordered,
- `books/order/{id}`: to output order status for the given order `id`.

The example automatically creates new orders with a running order `id`
starting from 1.
You can then access these services from your Web browser, e.g.:

- [http://<route_hostname>/camel-rest-jpa/books](http://route_hostname/camel-rest-jpa/books)
- [http://<route_hostname>/camel-rest-jpa/books/order/1](http://route_hostname/camel-rest-jpa/books/order/1)

##### Swagger API

The example provides API documentation of the service using Swagger using
the _context-path_ `camel-rest-jpa/api-doc`. You can access the API documentation
from your Web browser at [http://<route_hostname>/camel-rest-jpa/api-doc](http://route_hostname/camel-rest-jpa/api-doc).

## Test

### Locally

The tests can be executed with:

    $ mvn surefire:test

This starts the application by picking an available port at random and executes the tests.

### Kubernetes

:pencil2:

### OpenShift

This requires to have an OpenShift environment running.
Depending on the authentication scheme of your environment, you may need to configure
the test client access.

Minishift relies on the default [identity provider](https://docs.openshift.com/container-platform/3.6/install_config/configuring_authentication.html#AllowAllPasswordIdentityProvider),
so that you can create a user for the test execution just by logging in, e.g.:

    $ oc login -u test -p test

And then execute the integration tests with:

    $ mvn failsafe:integration-test

Note that the test user requires to have the `basic-user` role bound, so that it can
create the project in which the application and the MySQL server get deployed prior
to the test execution.
Cluster roles can be viewed as documented in [Viewing Cluster Policy](https://docs.openshift.com/container-platform/3.6/admin_guide/manage_authorization_policy.html#viewing-cluster-policy).

Finally, it may be handy to keep the project created for the test execution.
This can be achieved by setting the `namespace.cleanup.enabled` system variable
to `false`, e.g.:

    $ mvn failsafe:integration-test -Dnamespace.cleanup.enabled=false
    
    
    package com.camel.soap;

import java.util.HashMap;
import java.util.Map;

import javax.xml.namespace.QName;

import org.apache.camel.CamelContext;
import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import org.apache.camel.builder.RouteBuilder;
import org.apache.camel.component.cxf.CxfComponent;
import org.apache.camel.component.cxf.CxfEndpoint;
import org.apache.camel.component.cxf.DataFormat;
import org.apache.camel.component.cxf.common.message.CxfConstants;
import org.apache.camel.model.dataformat.JaxbDataFormat;
import org.oorsprong.websamples.CountryInfoService;
import org.oorsprong.websamples.CountryName;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.stereotype.Component;
import org.springframework.ws.server.endpoint.annotation.Namespace;
import org.tempuri.AdditionalCxfPayloadConverters;

import ch.qos.logback.core.Context;


@Component
@Configuration
public class RestControler extends RouteBuilder {
	public static final String SERVICE_ADDRESS = "http://www.dneonline.com/calculator.asmx";
	public static final String ENDPOINT = "cxf:bean:cxfEndpoint";
	public static final String CountrySERVICE_ADDRESS = "https://www.w3schools.com/xml/tempconvert.asmx";
	public static final String countryENDPOINT = "cxf:bean:cxfcountryEndpoint";
	                    
	
	@Autowired
	private CamelContext context;
	 
	//Namespace ns=new Namesp
	@Override
	public void configure() throws Exception {
		JaxbDataFormat jaxb=new JaxbDataFormat();
		 jaxb.setContextPath("com.camel.soap");
		
		// TODO Auto-generated method stub
		from("timer:foo?period=50m")
		. log("timer started body ")
                //.log("request body A=${body.intA} and B=${body.intB}").
		       
				.setHeader(CxfConstants.OPERATION_NAME, constant("Add"))
				.setHeader(CxfConstants.OPERATION_NAMESPACE, constant("http://tempuri.org/"))
				//for payload format
				//.bean(AdditionalCxfPayloadConverters.class,"toCxfPayload(${body})")
				
				//for pojo mode
				.bean(GetCalculatorRequestBuilder.class).setBody(simple("${header.jaxb}")).convertBodyTo(Add.class).log("request body ${body}")
			    .setBody(simple("${exchangeProperty.param}"))
				.to("cxf://http://www.dneonline.com/calculator.asmx" +"?serviceClass=com.camel.soap.CalculatorSoap"+"&wsdlURL=/calculator.wsdl"+"&portName=CalculatorSoap"+"&dataFormat=POJO"+"&loggingFeatureEnabled=true")
				//.to(ENDPOINT)
				
				//.to(ENDPOINT)
				.log("reponse body ===${body} ");
   
		
		from("timer:coo?period=50m").autoStartup(false)
        .log("timer started body ")
		.setHeader("operationName", constant("CelsiusToFahrenheit"))
	  .setHeader("operationNamespace", constant("https://www.w3schools.com/xml/"))
		//for payload format
		.bean(AdditionalCxfPayloadConverters.class,"countryCxfPayload")
		
		//for pojo mode
		// .bean(GetCalculatorRequestBuilder.class)
		//
		//.to("cxf://http://www.dneonline.com/calculator.asmx" +"?serviceClass=com.camel.soap.CalculatorSoap"+"&wsdlURL=/calculator.wsdl"+"&portName=CalculatorSoap"+"&dataFormat=POJO"+"&loggingFeatureEnabled=true")
		//.to(ENDPOINT)
		
		.to(countryENDPOINT);
		//.bean(AdditionalCxfPayloadConverters.class,"cxfPayLoadToNode")
		
		

}
		
	
	

	
	  @Bean(name = {"cxfEndpoint"}) 
	  public CxfEndpoint customerServiceEndpoint() {
		    //CxfComponent cxfComponent = new CxfComponent(context);
		    CxfEndpoint cxfEndpoint = new CxfEndpoint();
		    cxfEndpoint.setAddress(SERVICE_ADDRESS);
		    //cxfEndpoint.setServiceClass(CalculatorSoap.class);
		    cxfEndpoint.setWsdlURL("/calculator.wsdl");
		    cxfEndpoint.setLoggingFeatureEnabled(true);
		    cxfEndpoint.setServiceName("{http://tempuri.org/}Calculator");
   		    cxfEndpoint.setPortName("{http://tempuri.org/}CalculatorSoap");
		     // in pojo mode only service class  and (service address or wsdl address)is needed 
     	     // cxfEndpoint.setDataFormat(DataFormat.POJO);
		     // in Payload mode only WSDL  and (service name and port name )is  needed
		    cxfEndpoint.setDataFormat(DataFormat.PAYLOAD);
		    cxfEndpoint.setLoggingFeatureEnabled(true);
		    Map<String, Object> properties = new HashMap<String, Object>();
		    properties.put("faultStackTraceEnabled", true);
		    cxfEndpoint.setProperties(properties);
	     return cxfEndpoint; 
	  }
	  
	  @Bean(name = {"cxfcountryEndpoint"}) 
	  public CxfEndpoint CountryServiceEndpoint() {
		    //CxfComponent cxfComponent = new CxfComponent(context);
		    CxfEndpoint cxfEndpoint1 = new CxfEndpoint();
		    cxfEndpoint1.setAddress(CountrySERVICE_ADDRESS);
		  //cxfEndpoint1.setServiceClass(CountryInfoService.class);
		    cxfEndpoint1.setWsdlURL("https://www.w3schools.com/xml/tempconvert.asmx?WSDL");
		    cxfEndpoint1.setLoggingFeatureEnabled(true);
		   
		     // in pojo mode only service class  and (service address or wsdl address)is needed 
     	     // cxfEndpoint.setDataFormat(DataFormat.POJO);
		     // in Payload mode only WSDL  and (service name and port name )is  needed
		    cxfEndpoint1.setDataFormat(DataFormat.PAYLOAD);
		    cxfEndpoint1.setLoggingFeatureEnabled(true);
		    Map<String, Object> properties = new HashMap<String, Object>();
		    properties.put("faultStackTraceEnabled", true);
		    cxfEndpoint1.setProperties(properties);
		     cxfEndpoint1.setPortName("{https://www.w3schools.com/xml/}TempConvertSoap");
		     cxfEndpoint1.setServiceName("{https://www.w3schools.com/xml/}TempConvert");
		     //cxfEndpoint1.setDefaultOperationNamespace("https://www.w3schools.com/xml/");
		    // cxfEndpoint1.setDefaultOperationName("{https://www.w3schools.com/xml/}CountryName");
		  //  QName qname = new QName("https://www.w3schools.com/xml/", "CountryInfoService");
		   // cxfEndpoint1.setServiceNameAsQName(qname);
	     return cxfEndpoint1; 
	  }
	 
}

