---
title: "Introducing the Quarkus JSON-RPC extension"
slug: introducing-quarkus-json-rpc
description: "A short introduction to quarkus-json-rpc, a Quarkiverse extension I have worked on, and where JSON-RPC already shows up in the Quarkus tooling"
thumb: site/quarkus.png
date: 2026-09-07
tags: ["Quarkus", "JSON-RPC", "Quarkiverse"]
---

[Quarkus JSON-RPC](https://github.com/quarkiverse/quarkus-json-rpc) is a Quarkiverse extension I have worked on. This post is a short introduction to it and to where JSON-RPC already sits in the Quarkus tooling I spend my days on.

## Where JSON-RPC shows up in Quarkus

If you have used the [Dev UI](https://quarkus.io/guides/dev-ui), you have already used JSON-RPC. Extensions add their own pages to the Dev UI using web components, and those pages talk to the running application over JSON-RPC. That is how the Dev UI can show the extensions, configuration and runtime data of the application and let you act on it without a restart.

[Dev MCP](https://quarkus.io/guides/dev-mcp) uses JSON-RPC as well. Because the Dev UI and MCP share the same protocol, the tools and data the Dev UI shows can be offered to an AI agent as MCP tools and resources. Extension authors mark the methods they want to expose, and the agent can then inspect the running application or change things like log levels.

I wrote more about both of these on the <a href="{=site.url('about')}">about page</a>.

## The extension

The quarkus-json-rpc extension lives in the Quarkiverse, next to [Chappie](https://github.com/quarkiverse/quarkus-chappie). The source is on [GitHub](https://github.com/quarkiverse/quarkus-json-rpc). Have a look at the repository if you want to see how it works, try it out or report an issue.
