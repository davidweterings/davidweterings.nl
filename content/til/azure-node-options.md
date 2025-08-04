---
title: "NODE_OPTIONS in Azure functions"
date: 2025-08-01T15:30:00+02:00
draft: false
tags: ["opentelemetry", "javascript", "azure"]
---

It turns out NODE_OPTIONS does not support most options in Azure functions. I'm currently working
on a migration from AWS to Azure. While adding observability to Azure Functions (we use NODE_OPTIONS to
bootstrap OpenTelemetry so it monkeypatches http/https properly) our functions stopped 
working. Logs in Azure only showed it was preparing to start up and nothing else.

Using `func start --verbose` locally it finally said 'unsupported option for NODE_OPTIONS'.

The [Azure Function documentation](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-node?tabs=javascript%2Cwindows%2Cazure-cli&pivots=nodejs-model-v4#languageworkersnodearguments) shows you can use `languageWorkers__node__arguments` 
instead. I wish Azure made this more obvious by default!
