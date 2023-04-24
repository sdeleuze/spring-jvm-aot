# Using AOT optimizations in Spring Boot JVM applications

This repository shows how to use [Spring AOT optimizations](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#core.aot) to speedup startup on
cheap Cloud instances.

Be aware that AOT optimizations fix at build time the beans created in your application context and currently do not support changing Spring profiles at runtime, see [spring-framework#29844](https://github.com/spring-projects/spring-framework/issues/29844) related issue.

## Data points on small Cloud instances

Measures have been done on Azure Container Apps with containers using low resources.
Those are Kotlin projects but results are similar with Java.

Data points for a [Spring Boot microservice using the functional web router](https://github.com/sdeleuze/spring-kotlin-functional) (this applications starts in 0.6s with the JVM on my local machine).

![Functional data points](https://raw.githubusercontent.com/sdeleuze/spring-jvm-aot/main/functional-data-points.png) 

Data points for the [Spring Data JDBC variant of Petclinic](https://github.com/sdeleuze/spring-petclinic-data-jdbc/tree/kotlin) (includes H2 database creation, this applications starts in 1.6s with the JVM on my local machine):

![Petclinic data points](https://raw.githubusercontent.com/sdeleuze/spring-jvm-aot/main/petclinic-data-points.png)

## Configuration

### Maven

Add the following profile to your `pom.xml`:
```xml
<profiles>
	<profile>
		<id>aot</id>
		<build>
			<plugins>
				<plugin>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-maven-plugin</artifactId>
					<executions>
						<execution>
							<id>process-aot</id>
							<goals>
								<goal>process-aot</goal>
							</goals>
						</execution>
					</executions>
					<configuration>
						<image>
							<env>
								<BPE_DELIM_JAVA_TOOL_OPTIONS xml:space="preserve"> </BPE_DELIM_JAVA_TOOL_OPTIONS>
								<BPE_APPEND_JAVA_TOOL_OPTIONS>-Dspring.aot.enabled=true</BPE_APPEND_JAVA_TOOL_OPTIONS>
							</env>
						</image>
					</configuration>
				</plugin>
			</plugins>
		</build>
	</profile>
</profiles>
```

Build your container image with `mvn -Paot spring-boot:build-image`.
See `demo-maven-aot` related sample.

## Gradle Kotlin

Add `id("org.springframework.boot.aot")` plugin then configure:
```kotlin
tasks.named<BootBuildImage>("bootBuildImage") {
    environment.set(mapOf(
        "BPE_DELIM_JAVA_TOOL_OPTIONS" to " ",
        "BPE_APPEND_JAVA_TOOL_OPTIONS" to "-Dspring.aot.enabled=true"
    ))
}
```

Build your container image with `./gradlew bootBuildImage`.
See `demo-gradle-kts-aot` related sample.

## Gradle Groovy     

Add `id 'org.springframework.boot.aot'` plugin then configure:
```groovy
tasks.named("bootBuildImage") {
	environment["BPE_DELIM_JAVA_TOOL_OPTIONS"] = " "
	environment["BPE_APPEND_JAVA_TOOL_OPTIONS"] = "-Dspring.aot.enabled=true"
}
```                                               
    
Build your container image with `./gradlew bootBuildImage`.
See `demo-gradle-aot` related sample.
