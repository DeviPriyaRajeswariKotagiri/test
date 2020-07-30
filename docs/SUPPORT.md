# Domain Service - Support

## Table of Contents
1. [Deployment Status](#1-deployment-status)
2. [Health Statistics](#2-health-statistics)
3. [How to view Logs](#3-how-to-view-logs)
4. [FAQ](#4-faq)

## 1. Deployment Status
One of the ways to check Deployment status is via kubectl or CodeFresh .
Steps involved in Deploying an application in local docker-kubernetes environment
1. Package the code -> Go to the project location where project exist and open CLI:
         
```
mvn clean package
```
            
2. Build the docker image using below command:


         docker build -t reference-domain-service:v1 -f Dockerfile-local .
         
3. Install the application in kubernetes environment using helm install command:

         helm install --name reference-domain-service ./reference-domain-service --values ./reference-domain-service/values-local.yaml --namespace enterprise

4. Using below command developer can see the application status:
         
            $ kubectl get pods -A
   
   
## 2. Health Statistics
One of the way to view Health Statistics of your application is by adding Spring Boot Actuator module ,it helps you monitor and manage your Spring Boot application by providing production-ready features like health check-up, auditing, metrics gathering, HTTP tracing etc. Please check references for more information .[How to view Health statistics of a Microservice ](https://www.callicoder.com/spring-boot-actuator/)
  
## 3. How to view Logs
Developer can see the application logs using below commands:

      kubectl logs reference-domain-service-c5f5b7cfc-lwrmf -c reference-domain-service -n enterprise
   
## 4. FAQ

### Question 1
Answer to question 1

### Question 2
Answer to question 2
