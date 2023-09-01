**Auto Instrumentation**

Use the OpenTelemetry Java Agent JAR alongside the corresponding modules for your frameworks/libraries. These modules generally start with 
`opentelemetry-javaagent-instrumentation-` which injects bytecode (in your codebase) to capture telemetry at various points in your application or service. 

To automatically instrument a Java application with OpenTelemetry using Docker, you can utilize the `opentelemetry-javaagent` in combination with your Docker build process. Here are the steps to achieve this:

1. Update your Dockerfile
   
Ensure that you have a `Dockerfile` for your Java application. Add the following lines to your Dockerfile, it will download the OpenTelemetry Java Agent JAR file and set it as the application's Java agent. This JAR file contains the necessary agent and instrumentation libraries.

     ```dockerfile
     ARG OTEL_VERSION=1.7.0

     # Download and configure OpenTelemetry Java Agent
     RUN wget "https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v${OTEL_VERSION}/opentelemetry-javaagent-all.jar" -O /opentelemetry-javaagent.jar

     # Set the OpenTelemetry Java Agent as the application's agent
     ENV JAVA_TOOL_OPTIONS="-javaagent:/opentelemetry-javaagent.jar"
     ```
2. Build and Run your Container
   
Build your Docker image using the updated Dockerfile. Run the container, and the Java application will automatically be instrumented with OpenTelemetry. The telemetry data will then be sent to your configured exporters. To achieve that, add the following configuration to your machine startup arguments and launch your application:

     ```dockerfile
     docker run -e JAVA_TOOL_OPTIONS="-javaagent:/path/to/opentelemetry-javaagent.jar -Dotel.service.name=your-service-name" -v /path/to/myapp.jar:/myapp.jar openjdk:8 java -jar /myapp.jar
     ```

Replace `/path/to/opentelemetry-javaagent.jar` with the actual path to the Java agent JAR file. 

Replace `your-service-name` with the desired name for your service.

Replace `/path/to/myapp.jar` with the actual path to your application JAR file.

In this example, we use an OpenJDK 8 Docker image, but you can replace it with the appropriate image for your Java version.

3. Configure the agent
   
The agent provides various flexible configuration options.

Option 1: Pass configuration properties using the `-D` flag. For example, to configure a service name and a Zipkin exporter for traces:

   ```dockerfile
   docker run -e JAVA_TOOL_OPTIONS="-javaagent:/path/to/opentelemetry-javaagent.jar -Dotel.service.name=your-service-name -Dotel.traces.exporter=zipkin" -v /path/to/myapp.jar:/myapp.jar openjdk:8 java -jar /myapp.jar
   ```
Option 2: Use environment variables to configure the agent:

   ```dockerfile
   docker run -e JAVA_TOOL_OPTIONS="-javaagent:/path/to/opentelemetry-javaagent.jar" -e OTEL_SERVICE_NAME="your-service-name" -v /path/to/myapp.jar:/myapp.jar openjdk:8 java -jar /myapp.jar
   ```
Option 3: Supply a Java properties file to load configuration values:

   ```dockerfile
   docker run -e JAVA_TOOL_OPTIONS="-javaagent:/path/to/opentelemetry-javaagent.jar -Dotel.javaagent.configuration-file=/path/to/properties/file.properties" -v /path/to/myapp.jar:/myapp.jar openjdk:8 java -jar /myapp.jar
   ```

Replace `/path/to/properties/file.properties` with the actual path to your properties file.

Replace `/path/to/opentelemetry-javaagent.jar` with the actual path to the Java agent JAR file,

Replace `your-service-name` with your desired service name,

Replace `/path/to/myapp.jar` with the actual path to your application JAR file.

By automatically instrumenting your Java application with OpenTelemetry using the Java agent, you will easily gather telemetry data without modifying your application's code. This makes it convenient to add observability to your Java applications.

N.B. You can use OpenTelemetry Java with other web frameworks as well, such as Apache Wicket and Play.

**Example Application**

Let us assume that you want to auto-instrument a Java Spring application. First, we need to create the application. You can do this in two steps: 

Step 1: Upload your dependencies

Add the following dependencies to your project's pom.xml file:

```xml
<dependency>
  <groupId>io.opentelemetry</groupId>
  <artifactId>opentelemetry-api</artifactId>
  <version>1.7.0</version>
</dependency>
<dependency>
  <groupId>io.opentelemetry</groupId>
  <artifactId>opentelemetry-sdk</artifactId>
  <version>1.7.0</version>
</dependency>
<dependency>
  <groupId>io.opentelemetry</groupId>
  <artifactId>opentelemetry-javaagent-all</artifactId>
  <version>1.7.0</version>
</dependency>
<dependency>
  <groupId>io.opentelemetry</groupId>
  <artifactId>opentelemetry-javaagent-instrumentation-spring</artifactId>
  <version>1.7.0</version>
</dependency>
```
Step 2: Create and launch an HTTP Server

With the dependencies added, you can now create a simple Spring application with an HTTP server. Here's a basic example:

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
public class MyApp {

    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}

@RestController
class HelloController {

    @GetMapping("/")
    String hello() {
        return "Hello, World!";
    }
}
```
Having created your spring application, now, let us learn how to auto-instrument it with OTel. 

**Auto-instrumenting with OTel on Java Spring**

In the following steps, we assume you are using Docker.

1. **Set up a Dockerized Java Spring Application:**
   
Start by creating a new directory for your project: `mkdir my-app`. 

Navigate to the project directory: `cd my-app` and create a new file named `Dockerfile` with the following content:

     ```Dockerfile
     FROM adoptopenjdk/openjdk11:alpine-jre
     WORKDIR /app
     COPY target/my-app.jar .
     CMD ["java", "-javaagent:/opentelemetry-javaagent.jar", "-jar", "my-app.jar"]
     ```

Replace `my-app.jar` with the actual name of your Spring application JAR file, and Save the Dockerfile.

2. **Add OpenTelemetry Agent to the Docker image:**
Download the OpenTelemetry Java Agent JAR file by running the following command:

     ```bash
     curl -LJO https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent-all.jar
     ```

Rename the downloaded JAR file to `opentelemetry-javaagent.jar`: `mv opentelemetry-javaagent-all.jar opentelemetry-javaagent.jar`

Copy the `opentelemetry-javaagent.jar` into your project directory where the Dockerfile is located.

3. **Update your Maven dependencies:**
Add the OpenTelemetry Java instrumentation module for Spring to your pom.xml file by including the following dependency:

     ```xml
     <dependency>
       <groupId>io.opentelemetry</groupId>
       <artifactId>opentelemetry-javaagent-instrumentation-spring</artifactId>
       <version>1.7.0</version>
     </dependency>
     ```
Save your pom.xml file and let Maven download the required dependencies.

4. **Build and run your Dockerized applicatio****n:**
Build the Docker image using the command:

     ```bash
     docker build -t my-app .
     ```
Once the image is built, run the Docker container using:
     ```bash
     docker run -p 8080:8080 my-app
     ```
This will start your Spring application with OpenTelemetry instrumentation.

5. **Verify Instrumentation:**
Ensure your Spring application is up and running inside the Docker container. Open your web browser and visit `http://localhost:8080` to trigger requests. Verify that your application is now automatically instrumented with OpenTelemetry.

6. **Choose an Exporter:**
   
To export telemetry data to a backend, such as Jaeger or Zipkin, you can set up exporters in your application's configuration. Determine which backend or observability tool you want to export your telemetry data to.

i. Add the exporter dependency to your Maven pom.xml file. For example, if you choose the Jaeger exporter, add the following lines (make sure to update the version based on the latest release):

     ```xml
     <dependency>
       <groupId>io.opentelemetry.exporter</groupId>
       <artifactId>opentelemetry-exporter-jaeger</artifactId>
       <version>1.7.0</version>
     </dependency>
     ```

ii. Update your Spring application configuration to initialize and configure the exporter. For the Jaeger exporter, you can set the configuration through environment variables in your Dockerfile. Here's an example:

     ```dockerfile
     # Add environment variables to your Dockerfile
     ENV OTEL_EXPORTER_JAEGER_SERVICE_NAME=my-app
     ENV OTEL_EXPORTER_JAEGER_AGENT_HOST=jaeger-agent
     ENV OTEL_EXPORTER_JAEGER_AGENT_PORT=6831
     ```

Ensure that you set appropriate values for the service name, Jaeger agent host, and port.

iii. Build your Docker image with your Java application and run the container. The exporter should now be sending telemetry data to your chosen backend or observability tool.

7. **Analyze and Visualize**
 Use the chosen backend's user interface or APIs to inspect the data with whichyou will analyze and visualize the performance and behaviour of your application. 
