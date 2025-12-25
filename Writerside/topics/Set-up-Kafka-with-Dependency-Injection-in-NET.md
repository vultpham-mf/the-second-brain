# Set up Kafka with Dependency Injection in .NET
<primary-label ref="guide"/>
<secondary-label ref="dotnet"/>
<secondary-label ref="kafka"/>

<img src="kafka-brainless-meme.jpg" alt="Wow, my project must use Kafka and Microservices"/>

## Overview

- This guide will show you how to set up Kafka with Dependency Injection in .NET.
- It mostly focuses on Dependency Injection with ASP.NET Core.

## Install Kafka

- Package name: <code>Confluent.Kafka</code>
- Version (latest at the time of writing this): <code>2.12.0</code>
- Command: <code>dotnet add package Confluent.Kafka --version 2.12.0</code>

## Set up Producer

### 1. Setup <code>ProducerSettings</code>

#### Create <code>ProducerSettings</code> class

<p>This class stores the configuration settings required to establish a connection to your Kafka Cluster.</p>

<tabs>
    <tab title="ProducerSettings.cs">
<code-block lang="c#">
using System.ComponentModel.DataAnnotations;

namespace ...;

public record ProducerSettings
{
    [Required]
    public required string BootstrapServers { get; init; }

    [Required]
    public required string SaslUsername { get; init; }
    
    [Required]
    public required string SaslPassword { get; init; }
}
</code-block>
    </tab>
</tabs>

#### Add your credentials to appsettings.json

<p>
Go to <code>appsettings.json</code> and add your credentials, 
please note that the section and fields name must be exactly the same as in the class <code>ProducerSettings</code>.
</p>

<tabs>
    <tab title="appsettings.json">
        <code-block lang="json">
        {
            ...,
            "ProducerSettings": {
                "BootstrapServers": "your secret",
                "SaslUsername": "your secret",
                "SaslPassword": "your secret"
            }
        }
        </code-block>
    </tab>
</tabs>

#### Config ProducerSettings Dependency Injection

<tabs>
    <tab title="Program.cs">
        <code-block lang="c#">
        builder.Services
            .AddOptionsWithValidateOnStart&lt;ProducerSettings&gt;()
            .BindConfiguration(nameof(ProducerSettings))
            .ValidateDataAnnotations(); // Optional
        </code-block>
    </tab>
</tabs>

<note>
    Calling <code>AddOptionsWithValidateOnStart</code> is recommended. 
    This will validate the configuration object at startup using the DataAnnotations validation attributes defined in the class.
</note>

<warning>
    Calling <code>ValidateDataAnnotations</code> is optional. 
    It validates your configuration object each time it is injected into a service.
</warning>

### 2. Setup Serializer {id="2-setup-serializer"}

<p>
    To enable producing complex data types such as classes, lists ...,
    you need to provide a serialization method for your complex types. In this guide, I will create a custom serializer using JSON.
</p>

<tabs>
    <tab title="CustomJsonSerializer.cs">
<code-block lang="c#">
using System.Text.Json;
using Confluent.Kafka;

namespace ...;

public class CustomJsonSerializer&lt;T&gt; : ISerializer&lt;T&gt;
{
    public byte[] Serialize(T data, SerializationContext context)
        => JsonSerializer.SerializeToUtf8Bytes(data);
}
</code-block>
    </tab>
</tabs>

<tip>
    <p>Kafka provides built-in serialization support through:</p>
    <list>
        <li><code>Confluent.SchemaRegistry.Serdes.Avro</code></li>
        <li><code>Confluent.SchemaRegistry.Serdes.Json</code></li>
        <li><code>Confluent.SchemaRegistry.Serdes.Protobuf</code></li>
    </list>
    <p>
        However, configuring these serializers requires additional setup. 
        For a detailed implementation example, please refer to: <a href="https://github.com/confluentinc/confluent-kafka-dotnet/blob/master/examples/JsonSerialization/Program.cs">the official Kafka .NET JSON serialization example</a>.
    </p>
</tip>

### 3. Configure Producer Service Dependency Injection

<tabs>
    <tab title="Program.cs">
<code-block lang="c#">
builder.Services.AddSingleton&lt;IProducer&lt;string, Event&gt;&gt;(serviceProvider =&gt;
{
    // Get ProducerSettings
    var settings = serviceProvider.GetRequiredService&lt;IOptions&lt;ProducerSettings&gt;&gt;().Value;

    // Setup ProducerConfig
    var producerConfig = new ProducerConfig
    {
        BootstrapServers = settings.BootstrapServers,
        SaslUsername = settings.SaslUsername,
        SaslPassword = settings.SaslPassword,
        
        SecurityProtocol = SecurityProtocol.SaslSsl,
        SaslMechanism = SaslMechanism.Plain,
        Acks = Acks.All,
    };
    
    // Build Producer
    return new ProducerBuilder&lt;string, Event&gt;(producerConfig)
        .SetKeySerializer(Serializers.Utf8) // Serialize key as UTF-8 string
        .SetValueSerializer(new CustomJsonSerializer&lt;Event&gt;()) // Use CustomJsonSerializer for value
        .Build();
});
</code-block>
    </tab>
    <tab title="Event.cs">
<code-block lang="c#">
public record Event
{
    public required string Message { get; init; }
}
</code-block>
    </tab>
</tabs>

<p>
    The Producer is now ready to be injected anywhere.
</p>

<tip>
    <code>IProducer</code> implements <code>IDisposable</code> and should be properly disposed to release unmanaged resources. 
    When registered as a singleton in the DI container, ASP.NET Core will automatically dispose it when the application shuts down.
    However, if you manually create a producer instance outside of DI, ensure you dispose it using <code>using</code> statements or call <code>Dispose()</code> explicitly.
</tip>

### 4. Example Usage

<p>Here is a full example for your reference: <a href="https://github.com/vultpham-mf/KafkaPlayGround/tree/main/2.ProducerAndConsumerExample">ProducerAndConsumerExample</a></p>

<tabs>
    <tab title="Program.cs">
<code-block lang="c#">
app.MapGet("trigger-producer", (IProducer&lt;string, Event&gt; producer) =&gt;
{
    producer.Produce(
        "topic_0",
        new Message&lt;string, Event&gt;
        {
            Key = Guid.NewGuid().ToString(),
            Value = new Event
            {
                Message = "Hello MotherFather!"
            }
        }
    );

    producer.Flush();
    
    return "Hello World!";
});
</code-block>
    </tab>
</tabs>

## Set up Consumer

### 1. Setup <code>ConsumerSettings</code>

#### Create <code>ConsumerSettings</code> class

<tabs>
    <tab title="ConsumerSettings.cs">
<code-block lang="c#">
using System.ComponentModel.DataAnnotations;

namespace ...;

public record ConsumerSettings
{
    [Required]
    public required string BootstrapServers { get; init; }

    [Required]
    public required string SaslUsername { get; init; }
    
    [Required]
    public required string SaslPassword { get; init; }
}
</code-block>
    </tab>
</tabs>

#### Add your cluster credentials to appsettings.json {id="add-your-credentials-to-appsettings-json_1"}

<p>
Go to <code>appsettings.json</code> and add your credentials, 
please note that the section and fields name must be exactly the same as in the class <code>ConsumerSettings</code>.
</p>

<tabs>
    <tab title="appsettings.json">
        <code-block lang="json">
        {
            ...,
            "ConsumerSettings": {
                "BootstrapServers": "your secret",
                "SaslUsername": "your secret",
                "SaslPassword": "your secret"
            }
        }
        </code-block>
    </tab>
</tabs>

#### Config ConsumerSettings Dependency Injection

<tabs>
    <tab title="Program.cs">
        <code-block lang="c#">
        builder.Services
            .AddOptionsWithValidateOnStart&lt;ConsumerSettings&gt;()
            .BindConfiguration(nameof(ConsumerSettings))
            .ValidateDataAnnotations(); // Optional
        </code-block>
    </tab>
</tabs>

### 2. Setup Deserializer

<p>
    To enable consumption of complex data types such as classes and lists,
    you must provide a deserialization method that corresponds to your serializer. 
    The deserializer must be compatible with the serializer defined in <a href="#2-setup-serializer">Setup Serializer</a>. 
    Since a custom JSON serializer was implemented in the previous section, a JSON deserializer will be used here.
</p>

<tabs>
    <tab title="CustomJsonDeserializer.cs">
<code-block lang="c#">
using System.Text.Json;
using Confluent.Kafka;

namespace ...;

public class CustomJsonDeserializer&lt;T&gt; : IDeserializer&lt;T&gt;
{
    public T Deserialize(ReadOnlySpan&lt;byte&gt; data, bool isNull, SerializationContext context)
        =&gt; JsonSerializer.Deserialize&lt;T&gt;(data)!;
}
</code-block>
    </tab>
</tabs>

### 3. Configure Consumer Service Dependency Injection

<tabs>
    <tab title="Program.cs">
<code-block lang="c#">
builder.Services.AddSingleton&lt;IConsumer&lt;string, Event&gt;&gt;(serviceProvider =&gt;
{
    // Get ConsumerSettings
    var settings = serviceProvider.GetRequiredService&lt;IOptions&lt;ConsumerSettings&gt;&gt;().Value;

    // Setup ConsumerConfig
    var consumerConfig = new ConsumerConfig
    {
        BootstrapServers = settings.BootstrapServers,
        SaslUsername = settings.SaslUsername,
        SaslPassword = settings.SaslPassword,

        SecurityProtocol = SecurityProtocol.SaslSsl,
        SaslMechanism = SaslMechanism.Plain,
        GroupId = "group-id-1",

        AutoOffsetReset = AutoOffsetReset.Earliest
    };

    // Build Consumer
    return new ConsumerBuilder&lt;string, Event&gt;(consumerConfig)
        .SetKeyDeserializer(Deserializers.Utf8) // Deserialize key as UTF-8 string
        .SetValueDeserializer(new CustomJsonDeserializer&lt;Event&gt;()) // Deserialize value using CustomJsonDeserializer
        .Build();
});
</code-block>
    </tab>
    <tab title="Event.cs">
<code-block lang="c#">
public record Event
{
    public required string Message { get; init; }
}
</code-block>
    </tab>
</tabs>

<tip>
    <code>IConsumer</code> implements <code>IDisposable</code> and should be properly disposed to release unmanaged resources and close connections to the Kafka cluster. 
    When registered as a singleton in the DI container, ASP.NET Core will automatically dispose it when the application shuts down.
    However, if you manually create a consumer instance outside of DI, ensure you dispose it using <code>using</code> statements or call <code>Dispose()</code> explicitly.
</tip>


### 4. Create Background Service for Consumer

<tabs>
    <tab title="ConsumerBackgroundService.cs">
<code-block lang="c#">
using Confluent.Kafka;

namespace ...;

public class ConsumerBackgroundService(
    ILogger&lt;ConsumerBackgroundService&gt; logger,
    IConsumer&lt;string, Event&gt; consumer
) : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken cancellationToken)
    {
        consumer.Subscribe("topic_0");

        while (!cancellationToken.IsCancellationRequested)
        {
            var consumeResult = consumer.Consume(cancellationToken);
            
            // TODO: Call your async event handler here
            await Task.Delay(0, cancellationToken);
            logger.LogInformation($"Key {consumeResult.Message.Key ?? ""} - Value {consumeResult.Message.Value.Message}");
        }
    }
}
</code-block>
    </tab>
    <tab title="Program.cs">
<code-block lang="c#">
... 
builder.Services.AddHostedService&lt;ConsumerBackgroundService&gt;();
...
</code-block>
    </tab>
</tabs>

<warning>
Your event handler must be an asynchronous function. 
The <code>ConsumerBackgroundService.ExecuteAsync</code> method runs on the main thread of your application. 
If your event handler is synchronous, it will block the main thread, preventing the rest of the application from running.
</warning>