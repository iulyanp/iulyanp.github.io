---
layout: post
title:  "Runing Node JS Server In Background"
slug: 'running-node-js-server-in-background'
date:   2021-06-04 12:01:06 +0300
category: nodejs
tags: [nodejs, pm2, systemd, server]
icon: terminal
---

When you use Node JS to implement an HTTP / HTTPS web server, in general for development you're using the basic node or nodemon command to run the webserver. But if the terminal closes or the node process gets killed, the HTTP web server will stop also. 
This article will show a few options we can use in order to avoid such an issue and make the node HTTP web server run in background even if we close the terminal.

#### Create a server file to start the HTTP web server

```
var http = require('http');
var httpServer = http.createServer(function (req, resp) {
    resp.writeHead(200, {'Access-Control-Allow-Origin':'*','Content-Type': 'text/plain'});
    resp.write("Welcome to www.emag.ro");
    resp.end();
});

// Start http server listen on port 3000.
httpServer.listen(3000);
console.log("Use browser to get url 'http://localhost:3000/server.js'");
```

#### Open a terminal and start the HTTP web server

```
node server.js
```

Open a web browser and browse URL http://localhost:3000, that means the HTTP web server has been started successfully.

### Run Node In Background - Nohup Command

In the above method, the Node JS server will always occupy the terminal until the terminal is closed. But when you close the terminal, the Node JS server will also stop. To fix this issue, you can start the Node JS server with nohup command as below.

```
nohup node server.js > prod.log &

ps aux | grep node
```

The `nohup` command will run commands in background, in this case, it will start our Node JS HTTP web server in background and display the node server process id. 

All the server-side output will be saved in prod.log file, but when the node process gets killed, the HTTP server stops also.

### Run Node In Background - Use Node Forever Package

Forever is a Node JS package that can make a node script execute forever even after the node script process is killed. It will execute the node js script in a new process when the old process has been stopped suddenly.

1. Install Node forever package. Below command will install the forever package globally.

```
npm install forever -g
```

2. After installation, run npm list command to see the forever package installation path. Generally npm package is installed in /usr/local/lib/node_modules folder.

```
npm list forever -g
```

3. Start node js HTTP web server with `forever start` command. Forever starts the HTTP web server in new process with an id, but if the process gets killed, forever start another process automatically to run the node HTTP web server for us. So the HTTP web server will run in background "forever".

```
forever start server.js
```

4. You can use `forever list` command to list all forever running processes.

5. `forever stop` command can stop the running node script.

```
forever stop server.js
```

You can find all the other options forever has on it’s official [github page](https://github.com/foreversd/forever)

### Run Node In Background - Use Linux Systemd

All the Linux OS contains the systemd daemon which is running in the background when Linux OS startup.
First, go to `/etc/systemd/system` directory and create a `node-app-service.service` file in this directory.

```
[Unit]
Description=Run my node js app in background

[Service]

# The path to node app file that will be executed in background when system startup.
ExecStart=/var/www/node-app/app.js
Restart=always
User=nobody

# For Debian/Ubuntu Linux OS uses 'nogroup', for RHEL/Fedora Linux OS uses 'nobody' 
Group=nogroup
Environment=PATH=/usr/bin:/usr/local/bin
Environment=NODE_ENV=production
WorkingDirectory=/var/www/node-app

[Install]
WantedBy=multi-user.target
```

Edit the `/var/www/node-app/app.js` file in vim, add text `#!/usr/bin/env node` at the beginning of the file content.

Add execute permission to the target node js app file `chmod +x /var/www/node-app/app.js`.

Execute `systemctl daemon-reload` in a terminal to let systemd load our new service `node-app-service.service`.

Start the service `systemctl start node-app-service`, the `node-app-service` is the service file name as above.

We can use the command `systemctl enable node-app-service` in order to run node in background at system startup.

The execution logs can be seen running `journalctl -u node-app-service` for our node js systemd service.

### Run Node In Background - Use PM2

PM2 is an open source and free to use process manager / monitoring tool. You can run PM2 on all Linux, Windows & macOS.

This process manager has a built-in load balancer which allow the node js app keep alive forever without downtime even if the server restart.

To install PM2 globally run the command `npm install pm2 -g` in a terminal.

If you did not install Node js on your server, you can find [here](https://www.geeksforgeeks.org/installation-of-node-js-on-linux/) more detailed.

We can start the nodejs application as a deamon service with `pm2 start app.js`, and in order to monitor logs you cam use `pm2 monit`.

##### Other usefull commands 

The pm2 command provides several options for you to manage the processes. The most popular are 

```
pm2 list
pm2 stop
pm2 restart
pm2 delete
```

You can visit the pm2 [official website](https://pm2.keymetrics.io/) if you want to learn more about it.