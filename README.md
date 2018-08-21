# Maven Springboot Angular 6 Boilerplate

This project was generated with the guide of [dsyer/spring-boot-angular](https://github.com/dsyer/spring-boot-angular). I will use most of the instruction from there.<br/><br/>
For short it ables to build Angluar (not AngularJS, after Angular 2) for client side and SpringBoot for server side together with maven project build. 

## Basic Setup (Make a project from nothing)

* This setup is done with..
- Mac OS
- IntelliJ IDEA


      
    
![Welcome_IntelliJ](readme_img/0_Welcome_IntelliJ.png?raw=true "Title")

1.  Create New Project

![Spring_Initializr](readme_img/1_0_Spring_Initializr.png?raw=true "Title")

![Project_Metadata](readme_img/1_1_Project_Metadata.png?raw=true "Title")

![Dependencies](readme_img/1_2_Dependencies.png?raw=true "Title")

I just choosed [Web] for the basic setup.

2.  Npm Install (locally)
We will use [Maven Frontend Plugin](https://github.com/eirslett/frontend-maven-plugin) from Eirik Sletteberg. Now we will add it to out `pom.xml`:
<br/><br/>
pom.xml <br/>

```
<project ...>
	...
	<build>
	    <plugins>
	        ...
	        <plugin>
	            <groupId>com.github.eirslett</groupId>
	            <artifactId>frontend-maven-plugin</artifactId>
	            <version>1.6</version>
	            <configuration>
	                <nodeVersion>v9.2.0</nodeVersion>
	            </configuration>
	            <executions>
	                <execution>
	                    <id>install-npm</id>
	                    <goals>
	                        <goal>install-node-and-npm</goal>
	                    </goals>
	                </execution>
	            </executions>
	        </plugin>
	    </plugins>
	</build>
</project>
```

After that
```
$ ./mvnw install
```

It will install all the setups related to maven(pom.xml).

```
$ ls node*
```

you will see node and node_module installed in your project root directory.

3.  Install Angular CLI

First create a convinient script to run ```npm``` from the local installation. 

```
$ cat > npm
#!/bin/sh
cd $(dirname $0)
PATH="$PWD/node/":$PATH
node "./node/node_modules/npm/bin/npm-cli.js" "$@"

$ chmod +x npm
```

After typing this two command in the terminal, npm script file will be created.

and then run it to install the CLI:

```
$ ./npm install @angular/cli
```

Then create a similar wrapper for the CLI itself, and test it quickly:

```
$ cat > ng
#!/bin/sh
cd $(dirname $0)
PATH="$PWD/node/":"$PWD":$PATH
node_modules/@angular/cli/bin/ng "$@"
$ chmod +x ng
$ ./ng version
```

Running these 3 command, you will see that Angular CLI is successfully installed.

4.  Create an Angular App

Now we will create an Angular app, move the Angular app into the top level directory which can be build with Maven.

```
$ ./ng new client
```

and then move it to root directory:

```
$ cat ./client/.gitignore >> .gitignore
$ rm -rf ./client/node* ./client/src/favicon.ico ./client/.gitignore ./client/.git
$ sed -i '/node_/anode/' .gitignore
$ cp -rf ./client/* .
$ cp ./client/.??* .
$ rm -rf ./client
$ sed -i -e 's,dist/client,target/classes/static,' angular.json
```

5.  Building

Add an execution to install the modules used in the application:

```
<execution>
    <id>npm-install</id>
    <goals>
        <goal>npm</goal>
    </goals>
</execution>
```

Install the modules using `./mvnw generate-resources` and check the result.

```
$ ./ng version
```

and if you also add this as well:
```
<execution>
    <id>npm-build</id>
    <goals>
        <goal>npm</goal>
    </goals>
    <configuration>
        <arguments>run-script build</arguments>
    </configuration>
</execution>
```
then the client app will be compiled during the Maven build.

## Development server

**[Client side development use]**

```
$ ./ng serve 

```

For a client local server. Navigate to `http://localhost:4200/`. The app will automatically reload if you change any of the source files. 

**[Server side development use]**

First, we will create a Spring Boot WAR to run a local tomcat server.

1 ) We need to package a WAR application. Change pom.xml 
```
<packaging>war</packaging>
```

2 ) Modify the final WAR file name to avoid including version numbers:
```
<build>
    <finalName>${artifactId}</finalName>
    ... 
</build>
```

3 ) Add Tomcat dependency:
```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <scope>provided</scope>
</dependency>
```

4 ) Finally, we initialize the Servlet context required by Tomacat.<br/>
Inherit SpringBootServletInitializer class
```
@SpringBootApplication
public class SpringBootTomcatApplication extends SpringBootServletInitializer {
}
```

To build out Tomcat depolyable WAR application, we run :
```
$ ./mvnw clean package
```
WAR file will be generated at target/${artifactId}.war

5 ) Download Apache Tomcat and unpack into tomcat folder

6 ) Run configuration setup

![Run_Configuration](readme_img/2_1_Run_Configuration.png?raw=true "Title")

[Server] tab
Name the configuration, Configure to select the Tomcat version you downloaded.

![Deployment_Configuration](readme_img/2_2_Deployment.png?raw=true "Title")

[Deployment] tab
Add ${artifactId}:war exploded for server startup and click OK.
