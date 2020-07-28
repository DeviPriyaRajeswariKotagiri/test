# Domain Service - Detailed Design

## Table of Contents
1. [UML Diagrams](#1-uml-diagrams)
2. [Dependent Downstream Services](#2-dependent-downstream-services)
3. [Understanding the Code](#3-understanding-the-code)
4. [References](#4-references)

## 1. UML Diagrams
### Entity Relationship Diagram

![your-UML-diagram-name](https://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/DeviPriyaRajeswariKotagiri/Testing/master/docs/puml/class-diagram-01.puml)

### Entity State Machine Diagram

![your-UML-diagram-name](https://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/DeviPriyaRajeswariKotagiri/Testing/master/docs/puml/state-diagram-01.puml)

## 2. Dependent Downstream Services
Describe the dependent services in your application.

## 3. Understanding the Code
### Application contains 4 configuration files in reference-domain module

* JpaConfiguration - For managing the JPA configuration
* MongoConfiguration - For managing the MongoDB configuration
* ReferenceDomainAutoConfiguration - For managing the common application configuration
* SecretsConfiguration - For managing the secrets configuration. Developer must create blank “secrets.properties” file in “c:\\Temp” folder.

###  Application Profiling

Application Environment property to specify which profiles are active. You can specify the property in any of the usual ways, for example you could include it in your application.properties. Here you can use mongodb or jpa. To run application with MongoDB, we need to put active profiles as "mongodb" in the reference-app-> application.properties file. If we need to work with Postgres DB then we need need to put active profiles as "jpa" in the "reference-app > application.properties" file.

###  Code walk through

* The owner of the domain microservice is responsible for providing a client for their consumers as an independent module. This module also contains the VO objects used for the entity while communicating through client.
Reference for create aggregate API client call:

```java
public class ReferenceApiClient {

    public String createAggregate(String aggregateId, String aggregateName) {
        template.postForObject(
                "/aggregate/{aggregateId}/create/{aggregateName}",
                null,
                Object.class,
                aggregateId,
                aggregateName
        );
        return aggregateId;
    }

}
```

* The api module exposes the web logic with the API end points for the domain service. The classes call the command handlers in the domain module.
Reference for create aggregate API:

```java
@RestController
public class CreateAggregateApi {

    @PostMapping("/aggregate/{aggregateId}/create/{aggregateName}")
    public void createAggregate(@PathVariable String aggregateId,
                                      @PathVariable String aggregateName) {
        handler.createAggregate(aggregateId, aggregateName);
    }
}
```

* The domain module implements command design pattern and calls the respective handlers directed by api module. The handler then performs the CRUD operation on the entity.
Reference for create aggregate request:

```java
@Component
public class CreateAggregateCommandHandler {

  @Transactional
  @EventPublisher(keyExpression = "aggregateId")
  public AggregateCreatedEvent createAggregate(String aggregateId, String aggregateName) {
    return Optional
        .of(new Aggregate(aggregateId, aggregateName))
        .map(repository::save)
        .map(a -> AggregateCreatedEvent.newBuilder()
            .setAggregateId(aggregateId)
            .setAggregateName(aggregateName)
            .build())
        .orElseThrow();
  }

}
```

* The handler makes use of the methods exposed by CrudRepository to connect with MongoDB or Postgres and execute the queries:

```java
public interface AggregateRepository extends CrudRepository<Aggregate, String> {
}
```

* The handler methods perform the CRUD operation, maps the response to the avro created events through events module.
This avro object is sent on the Kafka topics through the use of EventPublisher custom annotation.
The current implementation uses the default output topic provided by Spring Cloud stream to send the messages.
This is leveraged through the event publisher starter. For more details about publishing events, refer to the starter documentation.

* The events module contain the avsc files based on the entity objects defined for the business.
These files generate class files at compile time using the following plugin:

```xml
<plugin>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-maven-plugin</artifactId>
    <version>${avro.version}</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>schema</goal>
                <goal>protocol</goal>
                <goal>idl-protocol</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

* The generated class files are referenced by the domain module to send the events on the topic.


[Please refer SBB for more details on JPA reference implementation as follow:](https://wawaappdev.atlassian.net/wiki/spaces/EE/pages/804096109/SBB+-+JPA+reference+implementation+for+Postgres)
1. Run Application on IDE or local docker kubernetes or IP environments.
2. How to work Application with mongodb or PostgreSQL.

## 4. References
| Link | Description | 
| :---- | :----------- |
| [UML Component Design](https://wawaappdev.atlassian.net/wiki/spaces/ENTERPRISE/pages/586843021/ABB+-+Business+Service) |  Confluence page|
