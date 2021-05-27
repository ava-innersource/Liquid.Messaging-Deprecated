Liquid Application Framework - Messaging
========================================

This repository is part of the [Liquid Application Framework](https://github.com/Avanade/Liquid-Application-Framework), a modern Dotnet Core Application Framework for building cloud native microservices.

The main repository contains the examples and documentation on how to use Liquid.

Liquid Messaging
----------------

This package contains the messaging subsystem of Liquid, for interacting with assynchronous services. It also features several implementations. In order to use it, add the main package (Liquid.Messaging) to your project, along with the specific implementation that you will need.

|Available Cartridges|Badges|
|:--|--|
|[Liquid.Messaging.Aws](https://github.com/Avanade/Liquid.Messaging/tree/main/src/Liquid.Messaging.Aws)|[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Avanade_Liquid.Messaging.Aws&metric=alert_status)](https://sonarcloud.io/dashboard?id=Avanade_Liquid.Messaging.Aws)|
|[Liquid.Messaging.Azure](https://github.com/Avanade/Liquid.Messaging/tree/main/src/Liquid.Messaging.Azure)|[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Avanade_Liquid.Messaging.Azure&metric=alert_status)](https://sonarcloud.io/dashboard?id=Avanade_Liquid.Messaging.Azure)|
|[Liquid.Messaging.Gcp](https://github.com/Avanade/Liquid.Messaging/tree/main/src/Liquid.Messaging.Gcp)|[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Avanade_Liquid.Messaging.Gcp&metric=alert_status)](https://sonarcloud.io/dashboard?id=Avanade_Liquid.Messaging.Gcp)|
|[Liquid.Messaging.Kafka](https://github.com/Avanade/Liquid.Messaging/tree/main/src/Liquid.Messaging.Kafka)|[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Avanade_Liquid.Messaging.Kafka&metric=alert_status)](https://sonarcloud.io/dashboard?id=Avanade_Liquid.Messaging.Kafka)|
|[Liquid.Messaging.RabbitMq](https://github.com/Avanade/Liquid.Messaging/tree/main/src/Liquid.Messaging.RabbitMq)|[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Avanade_Liquid.Messaging.RabbitMq&metric=alert_status)](https://sonarcloud.io/dashboard?id=Avanade_Liquid.Messaging.RabbitMq)|


# Getting Started
## This is a sample usage with Azure Service Bus cartridge

### To publish a message, you must:

Register a producer in service provider

```C#
services.AddServiceBusProducer<MySampleMessage>("TestAzureServiceBus", "TestMessageTopic", false);
```

Include dependency in domain class constructor, and invoke this send method
```C#
using Liquid.Messaging;
```

```C#
public class MySampleProducer 
{
    //Dependency must be a ILightProducer implementation 
    private ILightProducer<MySampleMessage> _producer;

    public MySampleProducer(ILightProducer<MySampleMessage> producer)
    {
        _producer = producer;
    }
    public async Task Handle()
    {
        // Create a instance of message object defined as the type of the producer
        var message = new MySampleMessage();

         // Just invoke the Send method
        await _producer.SendMessageAsync(message, new Dictionary<string, object> { { "headerTest", "value" } });

    }
}
```
### To consume messages, declare your subscriber :

```C#
public class MySampleConsumer : ServiceBusConsumer<MySampleMessage>
{
    public MySampleConsumer(IServiceProvider serviceProvider
        , IMediator mediator
        , IMapper mapper
        , ILightContextFactory contextFactory
        , ILightTelemetryFactory telemetryFactory
        , ILoggerFactory loggerFactory
        , ILightMessagingConfiguration<ServiceBusSettings> messagingConfiguration
        , ServiceBusConsumerParameter serviceBusConsumerParameter) 
        : base(serviceProvider, mediator, mapper, contextFactory, telemetryFactory, loggerFactory, messagingConfiguration, serviceBusConsumerParameter)
    {
    }

    //the consume async method is invoked when a message arrives
    public override async Task<bool> ConsumeAsync(MySampleMessage message, IDictionary<string, object> headers, CancellationToken cancellationToken)
    {
        // Do what you need with the message, usually translate to a command and send it to the Mediator Service
    }
}
```
Dependency Injection
```C#
//the value of the first parameter must be the name of the appsettings section where the connectionstring of this producer / consumer is defined.
services.AddServiceBusProducer<MySampleMessage>("TestAzureServiceBus", "TestMessageTopic", false);

services.AddServiceBusConsumer<MySampleConsumer, MySampleMessage>("TestAzureServiceBus", "TestMessageTopic", "TestMessageSubscription");
```
appsettings.json
```Json
 "liquid": {
    "messaging": {
      "azure": {
        "TestAzureServiceBus": {
          "connectionString": ""
        }
      }
    }
  }
```
