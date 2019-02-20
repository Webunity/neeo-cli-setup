# neeo-cli-webunity
Helper files for the Webunity Neeo drivers and instructions how to setup the @neeo/cli for the Neeo remote

## Prerequisites
In order to use Neeo drivers you need an 'always-on' device capable of running NodeJS >= v6. I recommend a Raspberry Pi which is connected via LAN and connected to the same network as the devices you want to control via custom drivers, to ensure speed and stability.

This page describes how to setup the [neeo-cli](https://github.com/NEEOInc/neeo-sdk-toolkit/tree/master/cli) to install and run custom drivers for NodeJS. Basically you can have 1 server running, with multiple 'custom drivers' enabled for it.

## Setting up the Neeo/CLI
After setting up the hardware, you can set up a new 'neeo-server' which is able to run your custom drivers. Custom drivers are named "Neeo-driver-..." and are packaged as NPM packages. This way they end up in the `node_modules` directory and the Neeo CLI will automatically detect them and load them.

On the computer which you want to run your neeo-server, execute the following commands:
```
cd /opt
mkdir neeo-server
cd neeo-server
npm init -y
```

## How to run your Neeo Server at boot / deamonize it:
Create a new file in the root folder where you installed the Neeo Server called `run-server.sh`.

Copy these contents into it:
```
#!/bin/bash

if [[ $@ == 'debug' ]]
then
        export DEBUG="webunity:neeo-driver:*"
fi

npx neeo-cli start
```

And make it executable `chmod +x run-server.sh`. This makes it possible to start the Neeo server by running `./run-server.sh` and to give debug output by running `./run-server.sh debug`. This script is also needed if you want to run the script at boot. Note that the `export DEBUG` part is specific for drivers created by me.

Then we are going to create the file `neeo-server.service` for systemd. This is the way to run a service at boot on the Raspberry Pi. More info on systemd [can be found here](https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units).

```
nano /etc/systemd/system/neeo-server.service
```

Add the following contents to this file:
```
[Unit]
Description=My Neeo Server
Wants=network-online.target
After=network-online.target

[Service]
WorkingDirectory=/opt/neeo-server
ExecStart=/opt/neeo-server/run-server.sh
ExecReload=/bin/kill -HUP $MAINPID
KillMode=control-group
Restart=always
RestartSec=30

[Install]
WantedBy=multi-user.target
```

**Please make sure you update the paths in the above code to reflect your own setup.**

After you've saved this file, make sure `systemd` knows about:

```
systemctl daemon-reload
systemctl enable neeo-server.service
```

In order to start it, type:
```
systemctl start neeo-server.service
```

If you have installed new drivers, you can restart it like this:
```
systemctl restart neeo-server.service
```
