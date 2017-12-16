---
layout: post
title: SignalR Core Alpha
comments: true
excerpt: Some months ago I introduced you the new amazing SignalR version for Asp.Net Core 2.0 and we could get to know how it works, the differences respect to SignalR for Asp.Net and the new architecture for SignalR Core. Also, we talk about the possible dates to release the preview and the release version. So Last September 14th the SignalR Core team announced an alpha release which is the official preview release to SignalR Asp.Net Core 2.0. Today we going to talk about the changes about this version which is the “stable” and official version of SignalR Core.
keywords: "asp.net core, signalR, signalR core, signalR core alpha, C#, c-sharp, entity framework core, .net core, dot net core, .net core 2.0, dot net core 2.0, .netcore2.0, asp.net core mvc, asp.net, entity framework, sqlDependency, SqlTableDependency, sql server, sql service broker"
---

Some months ago I introduced you the [new amazing SignalR version for Asp.Net Core 2.0](http://elvanydev.com/SignalR-Core-SqlDependency-part1/) and we could get to know [how it works](http://elvanydev.com/SignalR-Core-SqlDependency-part2/), the differences respect to SignalR for Asp.Net and the new architecture for SignalR Core. Also, we talk about the possible dates to release the preview and the release version. So [Last September 14th](https://blogs.msdn.microsoft.com/webdev/2017/09/14/announcing-signalr-for-asp-net-core-2-0/) the SignalR Core team announced an Alpha release and later on [October 9th](https://blogs.msdn.microsoft.com/webdev/2017/10/09/announcing-signalr-for-asp-net-core-alpha-2/) they announced Alpha2 which is the official preview release to SignalR Asp.Net Core 2.0. Today we going to talk about the changes about this version which is the last "stable" and official version of SignalR Core.

When I got to know about the new version immediately I went to my repo and first of all I tried out to build it, and as it was expected, it didn't build, that's the price you must pay when you work over something that is building, but at the same time it's very exciting because you can be aware of the evolution and the improvements and you can realized why the changes happened. Now I just going to tell you the changes what I faced, that by the way, they are very cool.

> New nuget package version: `1.0.0-alpha2-final`

## HubConnectionBuilder

In the previous version when we needed to connect with a some Hub from the server side, we just used the `HubConnection` class, just like this:

```c#
var connection = new HubConnection(new Uri(baseUrl), loggerFactory);
```

Now we can use the `HubConnectionBuilder` class to make the connection creation more extensible and avoid to use a constructor full of parameters (love this change):

```c#
var connection = new HubConnectionBuilder()
                    .WithUrl(baseUrl)
                    .WithConsoleLogger()
                    .Build();
```

## Connection server-side handlers

When you needed to listen a method from some Hub you used the `On` method, like on the client side, but you needed to specify into an array the parameters that the method received:

```c#
connection.On("UpdateCatalog", new[] { typeof(IEnumerable<dynamic>) }, a =>
{
    var products = a[0] as List<dynamic>;
    foreach (var item in products)
    {
        Console.WriteLine($"{item.name}: {item.quantity}");
    }
});
```

Now there is a new overload generic method and we can specify the parameters in a safe way avoiding casting and of course the code looks like nicer. (this is one of the coolest ones)

```c#
connection.On<List<dynamic>>("UpdateCatalog", data =>
{
    var products = data;
    foreach (var item in products)
    {
        Console.WriteLine($"{item.name}: {item.quantity}");
    }
});
```

## Naming convention

I noticed a name change in the method `Invoke`:

```c#
await connection.Invoke("RegisterProduct", cts.Token, product, quanity);
```

As you can see it's an async method, so now following a naming convention is called `InvokeAsync`, and also the parameters order is changed, the cancellation token in this overload it's the last one:

```c#
await connection.InvokeAsync("RegisterProduct", product, quanity, cts.Token);
```

Another change related to naming convention was about on the `MapEndpoint` method, now it's called `MapEndPoint`. As you can see it applies the [Pascal case](https://msdn.microsoft.com/en-us/library/x2dbyw72(v=vs.71).aspx) style.

**Before:**
```c#
app.UseSockets(routes =>
{
    routes.MapEndpoint<MessagesEndPoint>("/message");
});
```

**Now:**
```c#
app.UseSockets(routes =>
{
    routes.MapEndPoint<MessagesEndPoint>("message");
});
```

If you see, now you don't need the "/" on the beginning, is the same with `MapHub` method. Actually, that's an inconsistency, but for the next version it will work normally again, with the "/" just like others .Net Core API's. I made this contribution myself to SignalR Core. (you can make yours if you want!)

>The previous changes were about Hub's connections (except the last one), now we going to talk about the changes related to EndPoint connections.

## Naming changes

There were a couple of naming changes there, one of them over the `Connection` class, now it's called `ConnectionContext`.

**Before:**
```c#
public override async Task OnConnectedAsync(Connection connection)
```

**Now:**
```c#
public override async Task OnConnectedAsync(ConnectionContext connection)
```

The other naming change was about on the `Transport` object into `ConnectionContext` class. Before the properties to manage the Input and Output were called just `Input` and `Output` respectively, now they are called `In` and `Out`.

**Before:**
```c#
connection.Transport.Input.WaitToReadAsync()
connection.Transport.Output.WriteAsync()
```

**Now:**
```c#
connection.Transport.In.WaitToReadAsync()
connection.Transport.Out.WriteAsync()
```

## TryRead and WriteAsync

`TryRead` and `WriteAsync` methods were simplified, before they received a `Message` object like parameter.

**Before:**
```c#
Message message;
if (connection.Transport.Input.TryRead(out message))
{
    ...
}

connection.Transport.Output.WriteAsync(new Message(payload, format, endOfMessage));
```

**Now:**
```c#
// message is byte[]
if (connection.Transport.In.TryRead(out var message))
{
    ...
}

// payload is byte[]
connection.Transport.Out.WriteAsync(payload);
```

The reason about that change is because of the `Channel<byte[]>` is used by the Sockets layer (which is the type of the `Transport` property into the `ConnectionContext` class), which is a low-level networking abstraction designed to behave like a standard TCP socket. SignalR Core team used to have a low-level framing protocol they added on top of that raw socket which included this data (though they didn't use it on WebSockets, because it already has framing) but they decided to move it up to the SignalR layer to keep the Sockets layer cleaner.

Message Types, Framing, etc. are all handled in the `IHubProtocol` implementations now. EndPoints work with raw binary data (allowing the EndPoint to apply whatever protocol it wants). This means that EndPoints can easily be used to implement other protocols.

## Other changes

Also I took advantage to refactor some things, for example I'm taking advantage to install the `signalr-client` module directly from npm in order to don't have a static file referenced into the web site. So I added the `signalr-client` as a dependency into the `package.json` file:

```json
{
  "version": "1.0.0",
  "name": "asp.net",
  "private": true,
  "dependencies": {
    "@aspnet/signalr-client": "^1.0.0-alpha2-final",
    "jquery.tabulator": "^1.12.0"
  }
}
```
So this way, Visual Studio automatically install the packages when you build the solution. Also I took advantage of the new bundling feature built-in on .Net Core in order to copy the `signalr-client` files to the `wwwroot` folder, this way you don't need dealing with gulp, grunt or another task runner.

```json
[
    {
        "outputFileName": "wwwroot/lib/signalr/signalr-clientES5-1.0.0-alpha2-final.min.js",
        "inputFiles": [
            "node_modules/@aspnet/signalr-client/dist/browser/signalr-clientES5-1.0.0-alpha2-final.min.js"
        ],
        "minify": {
            "enabled": false
        }
    },
    {
        "outputFileName": "wwwroot/lib/signalr/signalr-client-1.0.0-alpha2-final.min.js",
        "inputFiles": [
            "node_modules/@aspnet/signalr-client/dist/browser/signalr-client-1.0.0-alpha2-final.min.js"
        ],
        "minify": {
            "enabled": false
        }
    }
]
```

Last but not least, I took advantage to use the [EFCore.DbContextFactory](http://elvanydev.com/EF-DbContextFactory/) to manage my DbContext factories!

```c#
services.AddSqlServerDbContextFactory<InventoryContext>(Configuration.GetConnectionString("DefaultConnection"));
```

Those were the changes that I faced with this new SignalR core version, I wanted to share them with you in order we can understand what were the new changes and the most important, why. I hope it will be helpful and I encourage you to test this awesome new SignalR Core version.

> There is a new very cool feature that SignalR didn't offer when I start to study it on the beginning of this year and it's that SignalR now supports for Streaming, I think we gonna love that feature!

Download the updated code from my GitHub repository: <https://github.com/vany0114/SignalR-Core-SqlTableDependency>
