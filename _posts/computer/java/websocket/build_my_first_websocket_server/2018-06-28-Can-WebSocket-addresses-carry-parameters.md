---
layout: post
title:  "Can WebSocket addresses carry parameters"
categories: computer
tags:  computer copy stackoverflow java websocket
author: "web"
source: "https://stackoverflow.com/questions/17301269/can-websocket-addresses-carry-parameters"
---

* content
{:toc}


`ws://myserver.com/path?param=1` is a valid WebSocket URI. However, the way that your WebSocket server application can access the path and query string will differ depending on what WebSocket server framework you are using.

If you are using the Node.js [`einaros/ws`](https://github.com/websockets/ws/) library, then in your websocket connection object will have the full path with the query string at `upgradeReq.url`.

For example this:

    wss.on('connection', function(ws) {
        console.log("url: ", ws.upgradeReq.url);
    };

will print `url: /path?param=1` when you connect to `ws://myserver.com/path?param=1`.










