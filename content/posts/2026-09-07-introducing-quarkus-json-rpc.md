---
title: "Introducing Quarkus JSON-RPC: expose your Java methods with one annotation"
slug: introducing-quarkus-json-rpc
description: "Quarkus JSON-RPC is a Quarkiverse extension that exposes the public methods of an annotated class over JSON-RPC 2.0, on WebSocket or a Unix domain socket, with streaming, server push and a generated JavaScript client"
thumb: site/quarkus.png
date: 2026-09-07
tags: ["Quarkus", "JSON-RPC", "Quarkiverse", "WebSocket"]
---

[Quarkus JSON-RPC](https://github.com/quarkiverse/quarkus-json-rpc) is a Quarkiverse extension that adds [JSON-RPC 2.0](https://www.jsonrpc.org/specification) support to a Quarkus application. You annotate a class with `@JsonRPCApi` and its public methods become callable over JSON-RPC. There is no routing to write and no protocol plumbing.

## Where it comes from

The [Dev UI](https://quarkus.io/guides/dev-ui) has used JSON-RPC for a long time. Extension pages in the Dev UI are web components, and they talk to the running application over JSON-RPC to fetch data and call methods. That part of the Dev UI only worked in dev mode, and it was tied to the Dev UI itself.

We wanted to extract the JSON-RPC bit so that anyone can use it in their own application, outside of dev mode. That is what this extension is.

## Getting started

The extension is split into a transport-agnostic core and pluggable transport modules. You add the transport you want and it pulls in the core. For WebSocket:

```xml
{|<dependency>
    <groupId>io.quarkiverse.json-rpc</groupId>
    <artifactId>quarkus-json-rpc-websocket</artifactId>
    <version>${quarkus-json-rpc.version}</version>
</dependency>|}
```

Then create your API:

```java
{|import io.quarkiverse.jsonrpc.api.JsonRPCApi;

@JsonRPCApi
public class GreetingService {

    public String hello(String name) {
        return "Hello " + name;
    }
}|}
```

Connect to `ws://localhost:8080/json-rpc` and send:

```json
{|{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "GreetingService#hello",
  "params": { "name": "World" }
}|}
```

You get back:

```json
{|{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "Hello World"
}|}
```

Methods are addressed as `ClassName#methodName`. If you would rather not tie the method name to the class name, give the annotation a scope, for example `@JsonRPCApi("greet")`, and the method becomes `greet#hello`. Parameters can be named (a JSON object) or positional (a JSON array), and complex types are serialised with Jackson.

All `@JsonRPCApi` classes are found at build time, so there is no runtime classpath scanning and native image works. They are registered as CDI beans, so you can inject whatever you need into them.

## Return types

The return type decides how a method is dispatched. A plain type is blocking, `Uni<T>` is asynchronous, and `Multi<T>` becomes a streaming subscription. A `void` method is fire-and-forget: the client sends a JSON-RPC notification without an `id` and the server sends no response. You can fine-tune the thread with `@Blocking`, `@NonBlocking` or `@RunOnVirtualThread`.

Streaming is where JSON-RPC gets interesting. Return a `Multi` and the client gets each item as a notification:

```java
{|@JsonRPCApi
public class TickerService {

    public Multi<String> ticker() {
        return Multi.createFrom().ticks().every(Duration.ofSeconds(1))
                .onItem().transform(n -> "tick " + n);
    }
}|}
```

The client calls `TickerService#ticker` and gets a subscription id back. From then on every item arrives as a `subscription` notification, and a final notification with `complete: true` marks the end of the stream. The client can cancel at any time by calling `unsubscribe` with the subscription id.

## Server push

Sometimes the server needs to talk first. Inject `JsonRPCBroadcaster` into any CDI bean, not only a `@JsonRPCApi` class, and push a notification to every connected client or to one session:

```java
{|@ApplicationScoped
public class EventProcessor {

    @Inject
    JsonRPCBroadcaster broadcaster;

    public void onOrderPlaced(@Observes OrderPlacedEvent event) {
        broadcaster.broadcast("orderPlaced", event);
    }
}|}
```

## Calling it from the browser

Writing the JSON by hand is fine for a quick test, but in a real front end you want a typed client. Set this in `application.properties`:

```properties
{|quarkus.json-rpc.js-client.enabled=true|}
```

The extension then generates a JavaScript client library and a typed proxy with one export per `@JsonRPCApi` scope. Both are registered in the import map, so [Quarkus Web Bundler](https://docs.quarkiverse.io/quarkus-web-bundler/dev/index.html) and [Web Dependency Locator](https://quarkus.io/guides/web-dependency-locator) resolve them by name:

```javascript
{|import { GreetingService, TickerService } from '@quarkiverse/json-rpc-api';

const greeting = await GreetingService.hello({ name: 'World' });

const sub = TickerService.ticker()
    .onItem(item => console.log('Received:', item))
    .onComplete(() => console.log('Stream completed'));

await sub.cancel();|}
```

Streaming methods come through as subscriptions and `void` methods as notifications, so the JavaScript side matches the Java side.

## Not only WebSocket

The second transport is a Unix domain socket using JSONL framing, where each JSON-RPC message is one line terminated by a newline. This is for local inter-process communication where you do not need a network connection: CLI tools, agent-to-agent communication, or MCP servers.

```xml
{|<dependency>
    <groupId>io.quarkiverse.json-rpc</groupId>
    <artifactId>quarkus-json-rpc-domain-socket</artifactId>
    <version>${quarkus-json-rpc.version}</version>
</dependency>|}
```

```properties
{|quarkus.json-rpc.domain-socket.enabled=true
quarkus.json-rpc.domain-socket.path=/tmp/my-app.sock|}
```

You can test it from the command line with `socat`:

```bash
{|(printf '{"jsonrpc":"2.0","id":1,"method":"GreetingService#hello","params":{"name":"World"}}\n'; sleep 3) \
  | socat - UNIX-CONNECT:/tmp/my-app.sock|}
```

Both transports can run in the same application, and the same `@JsonRPCApi` classes are available on both.

## The rest

There is more in the [documentation](https://docs.quarkiverse.io/quarkus-json-rpc/dev/index.html), including:

- Security with the standard Jakarta annotations such as `@RolesAllowed` and `@Authenticated`, or Quarkus HTTP auth policies. This is opt-in, so nothing changes if you do not need it.
- An [OpenRPC](https://docs.quarkiverse.io/quarkus-json-rpc/dev/guides-openrpc.html) document generated at build time and served at `/json-rpc/openrpc.json`, describing every method with JSON Schema.
- A readiness health check when SmallRye Health is on the classpath, and request timing with Micrometer.
- A global timeout or a per-method `@Timeout` through MicroProfile Fault Tolerance.
- Connection lifecycle events as CDI events, so you can react when a client connects or disconnects.
- A method browser and tester in the Dev UI. It felt right that the Dev UI gets a page for this.

Try it out and let me know what you think.

- [GitHub repository](https://github.com/quarkiverse/quarkus-json-rpc)
- [Documentation](https://docs.quarkiverse.io/quarkus-json-rpc/dev/index.html)
- [Sample application](https://github.com/quarkiverse/quarkus-json-rpc/tree/main/sample)
- [Report issues](https://github.com/quarkiverse/quarkus-json-rpc/issues)
