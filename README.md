<img src="docs/images/wawa.jpg" width="100" height="100"/>

[![Build Status](https://travis-ci.org/openmrs/openmrs-core.svg?branch=master)](https://google.com/) [![Coverage Status](https://coveralls.io/repos/github/openmrs/openmrs-core/badge.svg?branch=master)](https://google.com/) [![SonarQube Badge](https://api.codacy.com/project/badge/Grade/a51303ee46c34775a7c31c8d6016da6b)](https://codefresh.io/steps/)

# Domain Service - Reference Implementation

## Table of Contents

1. [Introduction](#1-introduction)
2. [Pre-requisites](#2-prerequisites)
3. [Environment Variables](#3-environment-variables)
4. [Consumed Services](#4-consumed-services)
5. [Event Produced & Consumed](#5-events-produced-and-events-consumed)
6. [Logging](#6-logging)
7. [Health Checks](#7-health-checks)
8. [Build & Deployment](#8-build-and-deployment)
9. [Testing Instructions](#9-testing-instructions)

## 1. Introduction

There are [three different types of microservices defined for Wawa](https://wawaappdev.atlassian.net/wiki/spaces/ENTERPRISE/pages/550633706/ABB+-+Microservices). This is a reference implementation of [Wawa Architecture defined business microservice](https://wawaappdev.atlassian.net/wiki/spaces/ENTERPRISE/pages/586843021/ABB+-+Business+Service) to achieve standardization and reduce duplication of effort.

This implementation considers Aggregate as an entity which could be replaced by the business entity intended for development. For example, there could be a order service which will act as a business service for order entity.

The domain service currently performs the following tasks:
1. Perform CRUD operations for the business entity using MongoDB or Postgres.
2. Publish events for CRUD operations of the entity using Spring Cloud Stream.
3. Provides a client for the exposed API end points of the domain service using Rest Template.

The microservice consists of the following modules:
* api - This module contains the exposed end points for the microservice.
* app - This module is used by Spring Boot to package everything as a deployable container.
* client - This module provides the client for consumers of the domain microservice.
* db - This module provides the dependency for MongoDB & Postgres.
* domain - This module contains the domain logic for the entity.
* event - This module provides the avro schema for the domain entities.
* postman - This module contains the postman collection which can be used to invoke the application flows. 

Also, the module makes use of spring factories to define the configurations. Example:
```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.wawa.platform.reference.api.ReferenceApiAutoConfiguration
```

### See Also
* [Design Reference](docs/DESIGN.md)
* [Support Reference](docs/SUPPORT.md)

### Supporting Team(s)
* <ins>Engineering Team 2</ins> is the owner of this Reference Implementation.

### Outstanding work to be done
* Add unit tests
* Add integration tests
* Add performance tests
* Extend the service for pagination and search
* Extend the service to use custom event bindings

## 2. Prerequisites

### Tools / Software

* [Java Build System](https://wawaappdev.atlassian.net/wiki/spaces/KM/pages/328830959/Java)
* [Approved IDE](https://wawaappdev.atlassian.net/wiki/spaces/KM/pages/329352164/IDE)
* [Local Integrated Environment](https://wawaappdev.atlassian.net/wiki/spaces/KM/pages/447155015/Local+Docker+Desktop+Kubernetes+Istio+Kafka+Mongo+Development)

### Wawa Build Time Dependencies

| Project Name         | Version       |  Project URL  |   
|:---------------------|:--------------|:--------------------------------------------------------|      
|  spring-boot-starter-parent  |  2.2.2.RELEASE   |   [spring-boot-starter-parent](https://github.com/wawa/reference-domain-service/blob/master/pom.xml)  |
|  spring-cloud-dependencies |  Hoxton.RELEASE |  [spring-boot-starter-data-mongodb](https://github.com/wawa/reference-domain-service/blob/master/pom.xml) |
| reference-event |  0.0.1-SNAPSHOT     | [reference-event](https://github.com/wawa/reference-domain-service/blob/master/db/pom.xml) |
| reference-api |  0.0.1-SNAPSHOT   |  [reference-api](https://github.com/wawa/reference-domain-service/blob/master/db/pom.xml)  |
| reference-api-client |   0.0.1-SNAPSHOT  |  [reference-api-client](https://github.com/wawa/reference-domain-service/blob/master/db/pom.xml)  |
| reference-db  |  0.0.1-SNAPSHOT    |   [ reference-db ](https://github.com/wawa/reference-domain-service/blob/master/pom.xml)  | 
|  reference-domain  |   0.0.1-SNAPSHOT     |   [ reference-domain](https://github.com/wawa/reference-domain-service/blob/master/pom.xml)  | 
|  reference-exception |    0.0.1-SNAPSHOT      |   [ reference-exception](https://github.com/wawa/reference-domain-service/blob/master/pom.xml)  |
| avro-maven-plugin  |  1.9.2 |   [avro-maven-plugin](https://github.com/wawa/reference-domain-service/blob/master/pom.xml)  |
|  Embedded postgresql|  2.10   |  [Embedded postgresql](https://github.com/wawa/reference-domain-service/blob/master/pom.xml) |
| mongodb-driver-core| 3.11.2 | [mongodb-driver-core](https://github.com/wawa/reference-domain-service/blob/master/db/pom.xml)|
| Core APAAS API Starter |  0.0.1-SNAPSHOT |[ Core APAAS API Starter](https://github.com/wawa/core-apaas-app-starters/tree/master/core-apaas-api-starter)|
| Core APAAS Event Publisher Starter|  0.0.1-SNAPSHOT  |[Core APAAS Event Publisher Starter](https://github.com/wawa/core-apaas-app-starters/tree/master/core-apaas-event-publisher-starter)|
| Core APAAS Event Subscriber Starter |  0.0.1-SNAPSHOT  | [Core APAAS Event Subscriber Starter](https://github.com/wawa/core-apaas-app-starters/tree/master/core-apaas-event-subscriber-starter) |
|spring-boot-starter-data-mongodb |2.2.2.RELEASE |[spring-boot-starter-data-mongodb](https://github.com/wawa/reference-domain-service/blob/master/db/pom.xml)|
| postgresql|42.2.8|[PostgreSQL JDBC Driver ](https://github.com/wawa/reference-domain-service/blob/master/db/pom.xml)|



### Infrastructure


|Software              | Version       | Comment(s)  |   
|:---------------------|:--------------|:--------------------------------------------------------|      
|  Mongodb          |  4.2.8    |    NoSQL Database  |
|  PostgreSQL       |  12.3  |   Relational database management system  |
| Docker Desktop | 2.2.0.0| Docker Desktop and enable Kubernetes|



## 3. Environment Variables

|Environment Variable Name | Type (Env or Secret)  |  Scope (Build or Runtime)    | Responsible Party for value  | Purpose | Comment(s)  |   
|:-------------------------|:----------------------|:-----------------------------|:-----------------------------|:--------|:------------|      
|  SERVICE_BASE_URL    |    Env        |    Build & Runtime       |  Integration Platform    |           |  Scope of this variable changes at run time|
 |  BUILD_NUMBER           |    Env        |    Build & Runtime       |  Integration Platform    |           |  It is pr_pull_number |         |  BUILD_HASH               |    Env        |    Build & Runtime       |  Integration Platform    |           |  It is github hash    |  
  |  APP_VERSION           |    Env        |    Build & Runtime       |  Application Developer   |           |  The app_version comes from              the helm chart.|
 |   GITHUB_TOKEN          |    Env        |    Build             |  Integration Platform    |This variable allows downloading npm packages published to the GitHub NPM Registry.   |  To be renamed to GIT_PACKAGE_MANAGER_TOKEN   |
|ENABLE_BUILD_DETAILS|Env| Build & Runtime|Application Developer|Control visibility of build and version number in UI application expected value:- true : to show details false : to hide details        |  Yet to be created. It will come from helm chart.|

## 4. Consumed Services
| Service             | Discovery Address       |   
|:--------------------|:------------------------|   
|  Service XX         |  \<Discovery Address>  |
|  Service XX         |  \<Discovery Address>  |

*Discovery address is the name in the Istio service mesh that is used to access a downstream service*

## 5. Events Produced And Events Consumed
| Event               |  Event Schema          |  Description           |
|:--------------------|------------------------|------------------------|
| AggregateCancelledEvent        | [AggregateCancelledEvent ](https://github.com/wawa/reference-domain-service/blob/master/event/src/main/avro/aggregate-cancelled-event.avsc)  | Event schema for  AggregateCancelledEvent |
|  AggregateCreatedEvent        | [AggregateCreatedEvent](https://github.com/wawa/reference-domain-service/blob/master/event/src/main/avro/aggregate-created-event.avsc)  | Event schema for  AggregateCreatedEvent  |
|AggregateNameUpdatedEvent  | [AggregateNameUpdatedEvent](https://github.com/wawa/reference-domain-service/blob/master/event/src/main/avro/aggregate-name-updated-event.avsc) | Event schema for AggregateNameUpdatedEvent |
|  SomeOtherAggregateEvent|  [SomeOtherAggregateEvent](https://github.com/wawa/reference-domain-service/blob/master/event/src/main/avro/some-other-aggregate-event.avsc) | Event schema for SomeOtherAggregateEvent |


## 6. Logging
Java Logging API is provided by Simple Logging Facade(SLF4J) 
This section should have critical logging messages that can be used to understand the health of this component or aid in the debugging of the component

*[Logging Standard](https://wawaappdev.atlassian.net/wiki/spaces/ENTERPRISE/pages/337412190/ST9.1-+Logging+Standard)*

## 7. Health Checks
| Endpoint             | Path               |   Content     |
|:--------------------|:--------------------|---------------|   
|  Endpoint Path      |  /healthz           |   OK          |

*Endpoints and the decoder for the current state of the service and any dependent components*

## 8. Build And Deployment
### Compilation
```bash
cd myapp
mvn clean package
```

### Run As Application
```bash
java -jar ./myapp.jar
```

### Build Container
```bash
docker build .
```

### Run In Local Kubernetes
```bash
export $appname=myapp
helm upgrade $appname . --install --recreate-pods --namespace $namespace --version $appversion --values $values
```

## 9. Testing Instructions 
### Unit test cases
There are multiple unit test cases written to cover the different components of the application. However there is a global application test suite file _**UnitTests.java**_     that combines all the test cases in a logical manner to create a complete suite.We can also use mocking frameworks like mockito for effective unit testing of JAVA applications. It can be run from command prompt using the following command -

Run all tests in a class
```
mvn clean test -Dtest=xxxxTest
```


Run an individual test
```
mvn clean test -Dtest=xxxxTest#testA
```
