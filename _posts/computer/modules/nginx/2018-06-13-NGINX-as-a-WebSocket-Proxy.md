---
layout: post
title: "NGINX as a WebSocket Proxy"
categories: computer
tags:  computer nginx copy
author: "web"
source: "https://www.nginx.com/blog/websocket-nginx/"
---

* content
{:toc}


  

The [WebSocket](http://www.websocket.org/ "websocket.org") protocol provides a way of creating web applications that support real‑time bidirectional communication between clients and servers. Part of HTML5, WebSocket makes it much easier to develop these types of applications than the methods previously available. Most modern browsers support WebSocket including Chrome, Firefox, Internet Explorer, Opera, and Safari, and more and more server application frameworks are now supporting WebSocket as well.

For enterprise production use, where multiple WebSocket servers are needed for performance and high availability, a load balancing layer that understands the WebSocket protocol is required, and NGINX has supported WebSocket since version 1.3 and can act as a [reverse proxy](https://www.nginx.com/resources/admin-guide/reverse-proxy "NGINX Reverse Proxy") and do [load balancing](https://www.nginx.com/products/application-load-balancing/ "Application Load Balancing with NGINX Plus") of WebSocket applications. (All releases of NGINX Plus also support WebSocket.)

Check out recent [performance tests](https://www.nginx.com/blog/nginx-websockets-performance/) on the scalability of NGINX to load balance WebSocket connections.

The WebSocket protocol is different from the HTTP protocol, but the WebSocket handshake is compatible with HTTP, using the HTTP Upgrade facility to upgrade the connection from HTTP to WebSocket. This allows WebSocket applications to more easily fit into existing infrastructures. For example, WebSocket applications can use the standard HTTP ports 80 and 443, thus allowing the use of existing firewall rules.

A WebSocket application keeps a long‑running connection open between the client and the server, facilitating the development of real‑time applications. The HTTP Upgrade mechanism used to upgrade the connection from HTTP to WebSocket uses the `Upgrade` and `Connection` headers. There are some challenges that a reverse proxy server faces in supporting WebSocket. One is that WebSocket is a hop‑by‑hop protocol, so when a proxy server intercepts an Upgrade request from a client it needs to send its own Upgrade request to the backend server, including the appropriate headers. Also, since WebSocket connections are long lived, as opposed to the typical short‑lived connections used by HTTP, the reverse proxy needs to allow these connections to remain open, rather than closing them because they seem to be idle.

NGINX supports WebSocket by allowing a tunnel to be set up between a client and a backend server. For NGINX to send the Upgrade request from the client to the backend server, the `Upgrade` and `Connection` headers must be set explicitly, as in this example:

    location /wsapp/ {
        proxy_pass http://wsbackend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }

Once this is done, NGINX deals with this as a WebSocket connection.
<!--more-->

### NGINX Websocket Example

Here is a live example to show NGINX working as a WebSocket proxy. This example uses [ws](https://github.com/einaros/ws), a WebSocket implementation built on [Node.js](http://nodejs.org/). NGINX acts as a reverse proxy for a simple WebSocket application utilizing ws and Node.js. These instructions have been tested with Ubuntu 13.10 and CentOS 6.5 but might need to be adjusted for other OSs and versions. For this example, the WebSocket server’s IP address is 192.168.100.10 and the NGINX server’s IP address is 192.168.100.20.

1.  If you don’t already have Node.js and npm installed, run the following command:
    
    *   For Debian and Ubuntu:
        
            $ sudo apt-get install nodejs npm
        
    *   For RHEL and CentOS:
        
            $ sudo yum install nodejs npm
        
2.  Node.js is installed as `nodejs` on Ubuntu and as `node` on CentOS. The example uses `node`, so on Ubuntu we need to create a symbolic link from `nodejs` to `node`:
    
        $ ln -s /usr/bin/nodejs /usr/local/bin/node
    
3.  To install ws, run the following command:
    
        $ sudo npm install ws
    
    Note: If you get the error message: “Error: failed to fetch from registry: ws”, run the following command to fix the problem:
    
        $ sudo npm config set registry http://registry.npmjs.org/
    
    Then run the `sudo npm install ws` command again.
    
4.  ws comes with the program /root/node_modules/ws/bin/wscat that we will use for our client, but we need to create a program to act as the server. Create a file called server.js with these contents:
    
        console.log("Server started");
        var Msg = '';
        var WebSocketServer = require('ws').Server
            , wss = new WebSocketServer({port: 8010});
            wss.on('connection', function(ws) {
                ws.on('message', function(message) {
                console.log('Received from client: %s', message);
                ws.send('Server received from client: ' + message);
            });
         });
    
5.  To execute the server program, run the following command:
    
        $ node server.js
    
6.  The server prints an initial `"Server started"` message and then listens on port 8010, waiting for a client to connect to it. When it receives a client request, it echoes it and sends a message back to the client containing the message it received. To have NGINX proxy these requests, we create the following configuration:
    
        http {
            map $http_upgrade $connection_upgrade {
                default upgrade;
                '' close;
            }
         
            upstream websocket {
                server 192.168.100.10:8010;
            }
         
            server {
                listen 8020;
                location / {
                    proxy_pass http://websocket;
                    proxy_http_version 1.1;
                    proxy_set_header Upgrade $http_upgrade;
                    proxy_set_header Connection $connection_upgrade;
                }
            }
        }
    
    NGINX listens on port 8020 and proxies requests to the backend WebSocket server. The `proxy_set_header` directives enable NGINX to properly handle the WebSocket protocol.
    
7.  To test the server, we run wscat as our client:
    
        $ /root/node_modules/ws/bin/wscat --connect ws://192.168.100.20:8020
    
    wscat connects to the WebSocket server through the NGINX proxy. When you type a message for wscat to send to the server, you see it echoed on the server and then a message from the server appears on the client. Here’s a sample interaction:
    
    Server:
    
    Client:
    
    `$` `node server.js `  
    `Server started`
    
       
     
    
       
       
     
    
    `wscat --connect ws://192.168.100.20:8020  
    Connected (press CTRL+C to quit)  
    > Hello`
    
    `Received from client: Hello`
    
    `< Server received from client: Hello`
    
    Here we see that the client and server are able to communicate through NGINX which is acting as a proxy and messages can continue to be sent back and forth until either the client or server disconnects. All that is needed to get NGINX to properly handle WebSocket is to set the headers correctly to handle the Upgrade request that upgrades the connection from HTTP to WebSocket.
    

### Further Reading

*   [WebSocket proxying](http://nginx.org/en/docs/http/websocket.html) at nginx.org
*   [NGINX and NGINX Plus Feature Matrix](https://www.nginx.com/products/feature-matrix/)
*   [NGINX Plus Technical Specifications](https://www.nginx.com/products/technical-specs/)

















