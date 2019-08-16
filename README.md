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
