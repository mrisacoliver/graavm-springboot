# graavm-springboot


How to Use GraalVM Native Image with Spring Boot to Reduce Memory Usage in Kubernetes?


GraalVM Native Image compiles your Spring Boot application directly into a native binary executable (Ahead-Of-Time—AOT), eliminating the need to run it on the traditional JVM inside the container. This reduces memory consumption (RSS) by up to 60–80% and eliminates warmup time.


1. How Memory Reduction Works
   
Métrika Spring Boot (JVM) Spring Boot (GraalVM Native)
Memory Footprint (Initial RSS) ~250 MB – 500 MB ~30 MB – 80 MB
Startup Time 2.5s – 6.0s0.0 5s – 0.1s
Off-Heap RAM Usage (Metaspace/Threads)
Requires large amounts of RAM
Without Metaspace, uses little native RAM

2.Step-by-Step Guide to Building the Native Image
Option A: Using Spring Boot Buildpacks (Recommended)
If you don't have GraalVM installed locally, you can use Docker to build the native image using the Maven or Gradle plugin.


1. Configure the pom.xml (Maven)
Make sure the Spring Boot plugin is configured with AOT support:

XML
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <image>
                    <builder>paketobuildpacks/builder-jammy-tiny:latest</builder>
                </image>
            </configuration>
        </plugin>
    </plugins>
</build>



2.Run the compilation command
Run the command below in the terminal (requires Docker to be running):

./mvnw spring-boot:build-image -Pnative

In Gradle: ./gradlew bootBuildImage


Note: AOT compilation analyzes all the code at build time and can take 3 to 7 minutes.

3. Kubernetes Manifest Optimized for GraalVM
Since the native application consumes very little memory and doesn't have the overhead of the JVM's JIT compiler, you can set extremely low limits in Kubernetes:


YAML

apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-native-app
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: app
        image: sua-imagem-native:latest
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "250m"
        ports:
        - containerPort: 8080

4. Best Practices and Reflection (Hints)
Native compilation removes dead code and, by default, does not know what will be loaded dynamically via reflection.


Compatible libraries: Spring Boot 3+ provides native support for almost all official libraries (Spring Data, Spring Security, Jackson, etc.).


Custom Reflection: If you use older third-party libraries or perform manual reflection, create a hints class:


import org.springframework.aot.hint.RuntimeHints;
import org.springframework.aot.hint.RuntimeHintsRegistrar;

public class MeuDTOHints implements RuntimeHintsRegistrar {
    @Override
    public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
        hints.reflection().registerType(MeuDTO.class);
    }
}







 
