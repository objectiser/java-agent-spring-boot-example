# Spring Boot Example using OpenTracing Java Agent

NOTE: Use of this example currently requires building the agent - clone and `mvn clean install` 
[this project](https://github.com/objectiser/java-agent). Then obtain the `opentracing-agent.jar` from the
`opentracing-agent/target` folder.

NOTE: This example is currently dependent upon [this PR](https://github.com/opentracing-contrib/java-web-servlet-filter/pull/11) therefore this version will need to be cloned and built locally.

## The Example

The example services are based on [this example](http://github.com/obsidian-toaster/quick_rest_springboot-tomcat),
but modified to have two spring boot services, one calling the other.

The first service (Service A) is called by an external client (e.g. curl) using the `/greeting` endpoint, supplying
an optional `name` parameter. This service then calls Service B to obtain the message template to be used. Finally
the template, with the optional supplied `name` parameter is returned to the client.

To try out the example, without any instrumentation being added, build each service:

```
cd servicea
mvn clean install
cd ../serviceb
mvn clean install
cd ..
```

then run both services - you will need to start two separate command windows and then run the following
commands in each:

```
cd servicea
mvn spring-boot:run
```

```
cd serviceb
mvn spring-boot:run
```

Finally in a separate command window run:

```
curl http://localhost:8080/greeting -d name=Fred
```

The response from this command should be:

```json
{"id":2,"content":"Hello, Fred!"}
```

## Running the Instrumented Example

Before being able to try the OpenTracing Java Agent, it will be necessary to obtain the agent jar.

It will then be necessary to [startup a Hawkular APM server](https://hawkular.gitbooks.io/hawkular-apm-user-guide/content/quickstart/) and set up the `HAWKULAR_APM_USERNAME`, `HAWKULAR_APM_PASSWORD` and `HAWKULAR_APM_URI` environment
variables accordingly within each of the command windows that will run the services.

To run the instrumented version of the services, simply add the agent to the JVM args:

```
cd servicea
mvn spring-boot:run -Drun.jvmArguments=-javaagent:<path-to>/opentracing-agent.jar
```

```
cd serviceb
mvn spring-boot:run -Drun.jvmArguments=-javaagent:<path-to>/opentracing-agent.jar
```

Then re-run the client:

```
curl http://localhost:8080/greeting -d name=Fred
```

Finally, start up the Hawkular APM UI and you should see the interactions between the two services on the
Distributed Tracing page.


## How It Works

This section describes how to take a standard Spring Boot application and prepare it for instrumentation
using the OpenTracing Java Agent.


### Instrument the technologies and frameworks

The OpenTracing framework integrations and rules are added to the services as dependencies, e.g.

```xml
    <!-- OpenTracing Java Agent rule dependencies -->
    <dependency>
      <groupId>io.opentracing.contrib</groupId>
      <artifactId>opentracing-agent-rules-java-net</artifactId>
      <version>...</version>
    </dependency>
    <dependency>
      <groupId>io.opentracing.contrib</groupId>
      <artifactId>opentracing-agent-rules-java-web-servlet-filter</artifactId>
      <version>...</version>
    </dependency>

    <!-- OpenTracing Framework Integration dependencies -->
    <dependency>
      <groupId>io.opentracing.contrib</groupId>
      <artifactId>opentracing-web-servlet-filter</artifactId>
      <version>...</version>
    </dependency>
```

In these example services, we will use the
[servlet integration](https://github.com/opentracing-contrib/java-web-servlet-filter) and the direct
instrumentation rules for the Java `HttpURLConnection`.

NOTE: Currently the agent instrumentation rules for installing the servlet filter are provided as a
separate maven dependency. Eventually these rules will be bundled with the servlet integration project.

### Add an OpenTracing Compliant Tracer

This example has currently been configured to use the Hawkular APM OpenTracing compliant Tracer:

```xml
    <!-- OpenTracing compliant Tracer dependencies -->
    <dependency>
      <groupId>org.hawkular.apm</groupId>
      <artifactId>hawkular-apm-client-opentracing</artifactId>
      <version>...</version>
    </dependency>
    <dependency>
      <groupId>org.hawkular.apm</groupId>
      <artifactId>hawkular-apm-trace-publisher-rest-client</artifactId>
      <version>...</version>
    </dependency>
```

However any suitable `Tracer` implementation can be used, as long as it can be located using the
[Global Tracer](https://github.com/opentracing-contrib/java-globaltracer) utility.

Use of Hawkular APM's Tracer also requires a custom ByteMan rule to be used as a temporary measure,
to prevent the trace information being reported to the APM server being instrument itself. The `Tracer`
uses Java's `HttpURLConnection` to report the information to the server, and therefore must instruct the
agent to ignore these communications. This is achieved using the rule defined in the service's
`src/main/resources/otagent/hawkular-apm.btm` file, with content:

```
RULE Hawkular APM Ignore server communications
CLASS java.net.URL
METHOD openConnection
HELPER io.opentracing.contrib.agent.OpenTracingHelper
AT EXIT
IF $0.getPath().startsWith("/hawkular/apm")
DO
  $!.setRequestProperty("opentracing.ignore","true");
ENDRULE
```

NOTE: Once the OpenTracing Agent has been released, the Hawkular APM project will be updated to mark the
`HttpURLConnection` directly with this request property.


