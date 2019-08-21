# jib-harbor-demo
Compile java projects using jib-maven-plugin as docker image, then pushing it to customized harbor registry

1. HelloWorld.java
```java
package com.beamofsoul.docker;

public class HelloWorld {

	public static void main(String[] args) {
		System.out.println("Hello World Docker!");
	}
}
```
2. pom.xml
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	
	<groupId>com.beamofsoul.docker</groupId>
	<artifactId>helloworld</artifactId>
	<version>v1.0.0</version>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<jib-maven-plugin.version>1.4.0</jib-maven-plugin.version>
		<maven-compiler-plugin.version>3.8.0</maven-compiler-plugin.version>
	</properties>

	<dependencies>
	</dependencies>
	
	<developers>
		<developer>
			<name>beamofsoul</name>
		</developer>
	</developers>
	
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>${maven-compiler-plugin.version}</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
			<plugin>
				<groupId>com.google.cloud.tools</groupId>
				<artifactId>jib-maven-plugin</artifactId>
				<version>${jib-maven-plugin.version}</version>
				<configuration>
					<from>
						<image>openjdk:8-jre-alpine</image>
					</from>
					<to>
						<image>registryAddress/imageProjectName/imageName:${project.version}</image>
						<auth>
							<username>username</username>
							<password>password</password>
						</auth>
					</to>
					<allowInsecureRegistries>true</allowInsecureRegistries>
					<container>
						<jvmFlags>-Xms512</jvmFlags>
						<jvmFlags>-Xdebug</jvmFlags>
					</container>
		        </configuration>
				<executions>
					<execution>
						<phase>package</phase>
						<goals>
							<goal>build</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
```
3. Compile ... -> Image

Generate image locally, using command 'docker images' to check (if registryAddress using HTTPS protocol, do not need to set sendCredentialsOverHttp)
```shell
mvn compile jib:dockerBuild -DsendCredentialsOverHttp=true
```
or

Generate image and push it to customized harbor registry, login customized harbor website to check (DO NOT GENERATE IMAGE LOCALLY, if registryAddress using HTTPS protocol, do not need to set sendCredentialsOverHttp)
```shell
mvn compile jib:build -DsendCredentialsOverHttp=true
```
4. Run local image
```shell
docker run --rm -p 8080:8080 registryAddress/imageProjectName/imageName:${project.version}
```

Compile spring boot projects using jib-maven-plugin as docker image, then run

1. HelloWorld.java
```java
package com.beamofsoul.docker;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class SpringBootDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootDemoApplication.class, args);
	}
	
	@GetMapping("/")
	public String index() {
		return "Hello World Docker!";
	}

}
```
2. pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.7.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>

	<groupId>com.beamofsoul.docker</groupId>
	<artifactId>spring-boot-demo</artifactId>
	<version>v1.0.1</version>
	<name>spring-boot-demo</name>
	<packaging>jar</packaging>

	<properties>
		<java.version>1.8</java.version>
		<jib-maven-plugin.version>1.4.0</jib-maven-plugin.version>
		<registry-address>registryAddress</registry-address>
		<registry-username>registryUsername</registry-username>
		<registry-password>registrayPassword</registry-password>
		<registry-project>registrayProjectName</registry-project>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<finalName>${project.name}</finalName>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
				</configuration>
			</plugin>
			<plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
			<plugin>
				<groupId>com.google.cloud.tools</groupId>
				<artifactId>jib-maven-plugin</artifactId>
				<version>${jib-maven-plugin.version}</version>
				<configuration>
					<containerizingMode>packaged</containerizingMode>
					<from>
						<image>openjdk:8-jre-alpine</image>
					</from>
					<to>
						<image>${registry-address}/${registry-project}/${project.name}:${project.version}</image>
						<auth>
							<username>${registry-username}</username>
							<password>${registry-password}</password>
						</auth>
					</to>
					<allowInsecureRegistries>true</allowInsecureRegistries>
					<container>
						<useCurrentTimestamp>true</useCurrentTimestamp>
						<entrypoint>
							<entrypoint>/bin/sh</entrypoint>
							<entrypoint>-c</entrypoint>
							<entrypoint>java ${JAVA_OPTS} -jar /app/classpath/${project.build.finalName}.jar --spring.profiles.active=${PROFILE} > ${project.build.finalName}.log.out</entrypoint>
						</entrypoint>
					</container>
				</configuration>
				<!-- build, then push image to registry, same as: mvn compile jib:build -DsendCredentialsOverHttp=true -DskipTests=true -->
				<executions>
			        <execution>
			            <phase>package</phase>
			            <goals>
			                <goal>build</goal>
			            </goals>
			        </execution>
			    </executions>
			</plugin>
		</plugins>
	</build>
</project>
```
3. Compile ... -> Image
```shell
mvn package jib:dockerBuild -DsendCredentialsOverHttp=true -DskipTests=true
```
4. Run local image (download from Harbor first if do not exist locally)
```shell
docker run -d -e JAVA_OPTS='-Xms128m -Xmx1024m' -e PROFILE='dev' --rm -p 8080:8081 registryAddress/imageProjectName/imageName:${project.version}
```

