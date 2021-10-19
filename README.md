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
|[Liquid.Messaging.ServiceBus](https://github.com/Avanade/Liquid.Messaging/tree/main/src/Liquid.Messaging.Azure)|[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Avanade_Liquid.Messaging.ServiceBus&metric=alert_status)](https://sonarcloud.io/dashboard?id=Avanade_Liquid.Messaging.ServiceBus)|
|[Liquid.Messaging.Gcp](https://github.com/Avanade/Liquid.Messaging/tree/main/src/Liquid.Messaging.Gcp)|[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Avanade_Liquid.Messaging.Gcp&metric=alert_status)](https://sonarcloud.io/dashboard?id=Avanade_Liquid.Messaging.Gcp)|
|[Liquid.Messaging.Kafka](https://github.com/Avanade/Liquid.Messaging/tree/main/src/Liquid.Messaging.Kafka)|[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Avanade_Liquid.Messaging.Kafka&metric=alert_status)](https://sonarcloud.io/dashboard?id=Avanade_Liquid.Messaging.Kafka)|
|[Liquid.Messaging.RabbitMq](https://github.com/Avanade/Liquid.Messaging/tree/main/src/Liquid.Messaging.RabbitMq)|[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=Avanade_Liquid.Messaging.RabbitMq&metric=alert_status)](https://sonarcloud.io/dashboard?id=Avanade_Liquid.Messaging.RabbitMq)|


# Getting Started
## This is a sample usage with Azure Service Bus cartridge

### To publish a message, you must:

Register a producer in service provider

```C#
//the value of the parameter must be the name of the appsettings section where the configuration of your producer is defined.
services.AddServiceBusProducer<SampleMessageEntity>("Liquid:Messaging:ServiceBus:SampleProducer");
```

Include dependency in domain class constructor, and invoke this send method
```C#
using Liquid.Messaging;
```

```C#
public class MySampleProducer 
{
    //Dependency must be a ILiquidProducer implementation 
    private ILiquidProducer<MySampleMessage> _producer;

    public MySampleProducer(ILiquidProducer<MySampleMessage> producer)
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
### To consume messages, implement LiquidWorker:
```C#
 public class Worker : ILiquidWorker<SampleMessageEntity>
    {;
        private readonly IMediator _mediator;

        public Worker(IMediator mediator)
        {
            _mediator = mediator;
        }
        //the ProcessMessageAsync method is invoked when a message arrives
        public async Task ProcessMessageAsync(ProcessMessageEventArgs<SampleMessageEntity> args, CancellationToken cancellationToken)
        {
             // Do what you need with the message, usually translate to a command and send it to the Mediator Service
            await _mediator.Send(new PutCommandRequest(args.Data));
        }
    }
```
### The dependency injection method register the LiquidBackgroundService{TMessage}, that is the Liquid implementation of IHostedService and work as the host of Consumer.
```C#
//the value of the first parameter must be the name of the appsettings section where the configuration of your producer is defined, and second parameter is an array of assemblies where domain handlers are difined.
services.AddLiquidServiceBusConsumer<Worker, SampleMessageEntity>("Liquid:Messaging:ServiceBus:SampleConsumer", typeof(PutCommandRequest).Assembly);
```
### Configurations sample:
```Json
 "liquid": {
    "messaging": {
      "serviceBus": {
        "sampleProducer": {
          "ConnectionString": "",
          "EntityPath": "sampleentity"
        },
        "sampleConsumer": {
          "ConnectionString": "",
          "EntityPath": "samplemessage"
        }
      }
    }
  }
```
