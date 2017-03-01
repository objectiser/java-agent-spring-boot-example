# Introduction

This project exposes a simple REST endpoint where the service `greeting` is available at this address `http://hostname:port/greeting` and returns a json Greeting message

```json
{
    "content": "Hello, World!",
    "id": 1
}

```

The id of the message is incremented for each request. 
To customize the message, you can pass as parameter the name of the person that you want to send your greeting.

You can perform this task in three different ways:

1. Build and launch using Spring Boot.
1. Build and deploy using OpenShift.
1. Build, deploy, and authenticate using OpenShift Online.

# Prerequisites

To get started with these quickstarts you'll need the following prerequisites:

Name | Description | Version
--- | --- | ---
[java][1] | Java JDK | 8
[maven][2] | Apache Maven | 3.2.x 
[oc][3] | OpenShift Client | v3.3.x
[git][4] | Git version management | 2.x 

[1]: http://www.oracle.com/technetwork/java/javase/downloads/
[2]: https://maven.apache.org/download.cgi?Preferred=ftp://mirror.reverse.net/pub/apache/
[3]: https://docs.openshift.com/enterprise/3.2/cli_reference/get_started_cli.html
[4]: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git

In order to build and deploy this project, you must have an account on an OpenShift Online (OSO): https://console.dev-preview-int.openshift.com/ instance.

# Build the Project

The project bundles the Apache Tomcat 8.0.36 artifacts with SpringBoot 1.4.1.RELEASE.

Execute the following maven command:

```
mvn clean install
```

# Launch and test

1. Run the following command to start the maven goal of Spring Boot:

    ```
    mvn spring-boot:run
    ```

1. If the application launched without error, use the following command to access the REST endpoint exposed using curl or httpie tool:

    ```
    http http://localhost:8080/greeting
    curl http://localhost:8080/greeting
    ```

1. To pass a parameter for the Greeting Service, use the following HTTP request:

    ```
    http http://localhost:8080/greeting name==Charles
    curl http://localhost:8080/greeting -d name=Bruno
    ```

