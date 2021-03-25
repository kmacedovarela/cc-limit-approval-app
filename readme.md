Credit Card Limit Approval sample application
=======================

This business application demonstrates the basics of how you can use Kafka + processes in jBPM (a.k.a. RHPAM). 

This application was tested on jBPM 7.51.0. 

Preparing the environment to use it:

- Download the [jbpm-event-emitters-kafka-7.51.0.Final.jar](https://search.maven.org/remotecontent?filepath=org/jbpm/jbpm-event-emitters-kafka/7.51.0.Final/jbpm-event-emitters-kafka-7.51.0.Final.jar) file and place it inside the folder `$JBPM_INSTALL_DIR/standalone/deployments/kie-server.war/WEB-INF/lib/`.
- Add the system property below to your `$JBPM_INSTALL_DIR/standalone/configuration/standalone.xml` file: 
```
<property name="org.kie.kafka.server.ext.disabled" value="false"/>
```

Kafka environment:
By default, jBPM will look for a kafka server running on localhost:9092. The topics will be automatically created for you. 

## Using the project

#### Automatic Approval scenario

To start processes, you need to emit events to the topic `incoming-requests` with a high credit score. See examples below:

```
$ bin/kafka-console-producer.sh --topic incoming-requests --bootstrap-server localhost:9092
{"data" : {"customerId": 1, "customerScore": 250, "requestedValue":1500}}
```

Monitor the topic `requests-approved` to see the process result. 
```
$ bin/kafka-console-consumer.sh --topic requests-approved  --bootstrap-server localhost:9092
{
  "specversion" : "1.0",
  "time" : "2021-03-24T11:57:13.731-0300",
  "id" : "f312807c-a8ef-42fc-8312-064ae31fb2de",
  "type" : "java.lang.String",
  "source" : "/process/cc-limit-approval-app.cc-limit-raise-approval-with-end-events/30",
  "data" : "Automatically Approved"
}
```

Monitor the topic `jbpm-processes-events` to check the auditing for the commited transaction:

```
$ bin/kafka-console-consumer.sh --topic jbpm-processes-events  --bootstrap-server localhost:9092
{"specversion":"1.0","time":"2021-03-24T23:57:13.743-0300","id":"c0d06513-f138-4cc2-812f-af1df52fcbaf","type":"process","source":"/process/cc-limit-approval-app.cc-limit-raise-approval-with-end-events/30","data":{"compositeId":"sample-server_30","id":30,"processId":"cc-limit-approval-app.cc-limit-raise-approval-with-end-events","processName":"cc-limit-raise-approval-with-events","processVersion":"1.0","state":2,"containerId":"cc-limit-approval-app_1.0.0-SNAPSHOT","initiator":"unknown","date":"2021-03-24T23:57:13.741-0300","processInstanceDescription":"cc-limit-raise-approval-with-events","correlationKey":"30","parentId":-1,"variables":{"request":{"customerId":1,"requestedValue":1500,"customerScore":250},"reason":"Automatically Approved","approval":true,"initiator":"unknown"}}}
```

![automatic-approval-process-instance-diagram](https://user-images.githubusercontent.com/253186/112412372-2dec2f80-8cfd-11eb-80cb-88f429b88e4f.png)

#### Manual Approval scenario


To start processes, you need to emit events to the topic `incoming-requests` with a low credit score. See examples below:

```
$ bin/kafka-console-producer.sh --topic incoming-requests --bootstrap-server localhost:9092
{"data" : {"customerId": 1, "customerScore": 100, "requestedValue":1500}}
```

Monitor the topic `jbpm-tasks-events` to check the auditing for the commited transaction for tasks:

```
$ bin/kafka-console-consumer.sh --topic jbpm-tasks-events  --bootstrap-server localhost:9092
{"specversion":"1.0","time":"2021-03-25T00:03:01.533-0300","id":"f78ba29f-a4a6-47e4-bd10-6330a25098bf","type":"task","source":"/process/cc-limit-approval-app.cc-limit-raise-approval-with-end-events/34","data":{"compositeId":"sample-server_13","id":13,"priority":0,"name":"Analyst validation","subject":"","description":"","taskType":null,"formName":"Task","status":"Ready","actualOwner":null,"createdBy":null,"createdOn":"2021-03-25T00:03:01.509-0300","activationTime":"2021-03-25T00:03:01.509-0300","expirationDate":null,"skipable":false,"workItemId":40,"processInstanceId":34,"parentId":-1,"processId":"cc-limit-approval-app.cc-limit-raise-approval-with-end-events","containerId":"cc-limit-approval-app_1.0.0-SNAPSHOT","potentialOwners":["kie-server"],"excludedOwners":[],"businessAdmins":["Administrator","process-admin"],"inputData":{"Skippable":"false","request":{"customerId":1,"requestedValue":1500,"customerScore":100},"TaskName":"Task","NodeName":"Analyst validation","GroupId":"kie-server"},"outputData":null}}
```

In Business Central you can claim and start the task, and input a reason for denial.

![manual-approval-human-task](https://user-images.githubusercontent.com/253186/112412813-e3b77e00-8cfd-11eb-8480-9454db4b33cf.png)

Monitor the topic `requests-denied` to check new events for denied requests:

```
$ bin/kafka-console-consumer.sh --topic requests-denied --bootstrap-server localhost:9092
{
  "specversion" : "1.0",
  "time" : "2021-03-25T12:04:53.891-0300",
  "id" : "ad8184ea-5034-4775-b94e-2159bd0cad73",
  "type" : "java.lang.String",
  "source" : "/process/cc-limit-approval-app.cc-limit-raise-approval-with-end-events/34",
  "data" : "This customer doesn't have a good payment reputation."
}
```

The process instance should be completed: 

![manual-approval-process-instance-diagram](https://user-images.githubusercontent.com/253186/112413039-4872d880-8cfe-11eb-9999-90c853057aac.png)

