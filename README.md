# fishbus
Tiny tiny library for receiving azure service bus messages

### Install
dotnet add package fishbus

### Declare a message type (an Event or a Command)
```c#
public class SomethingExcitingJustHappened
{
    public string Somedata { get; set; }
}
```

### Write a handler for the message
```c#
public class DeleteUserHandler : IHandleMessage<SomethingExcitingJustHappened>
{
    public async Task Handle(SomethingExcitingJustHappened message, Func<Task> markAsCompleted)
    {
        Log.Information("Received SomethingExcitingJustHappened");
        await markAsCompleted();
    }
}
```

### Configure the host for your handlers 
Here we just have a console app, with a Program.cs, but this could just as well be a web app.
We just need a host that can host a Microsoft.Extensions.Hosting.IHostedService implementation.

```c#
public static async Task<int> Main(string[] args)
{
    try
    {
        await new HostBuilder()
            .ConfigureHostConfiguration(configHost =>
            {
                configHost.SetBasePath(Directory.GetCurrentDirectory());
                configHost.AddEnvironmentVariables(prefix: "ASPNETCORE_");
            })
            .ConfigureServices((hostContext, services) =>
            {
                services
                    .ConfigureMessaging() //will register messagehandlers from current assembly
                    .Configure<MessageSources>(configuration.GetSection("MessageSources")); //register the MessageSources
            })
            .RunConsoleAsync();

        return 0;
    }
    catch (Exception ex)
    {
        Log.Fatal(ex, "Host terminated unexpectedly");
        return 1;
    }    
}
```

### Configuration - appsettings.json
The following appsettings.json file could be used to configure the messagsources that the application will listen to

```json
{
  "MessageSources": {
    "Subscriptions": [{
        "ConnectionString": "",
        "Name": "<name of topic>"
      }
    ],
    "Queues": [{
        "ConnectionString": ""
      }
    ]
  }  
}
```