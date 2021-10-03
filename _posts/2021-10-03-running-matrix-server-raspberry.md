---
layout: post
title: Starting a Synapse/Matrix server on Raspberry Pi
---

Synapse is a reference “homeserver” implementation of Matrix. In this article, I will show step by step how to install a Matrix server on a Raspberry Pi. In my setup, the username of the Raspberry Pi user is **pi**, and the home directory is **/home/pi**.

For more information, see [Matrix-Element : How to install the Synapse-Matrix - Raspberry PI - Rock 64 Server](https://www.mytinydc.com/en/blog/matrix-synapse-installationserveur-clientelement/).

## Synapse/Matrix service installation

### Install the necessary packages:

```
sudo apt -y install build-essential make python3 python3-dev python3-dev python3-virtualenv python3-pip python3-setuptools libffi-dev libpq-dev  python3-cffi zlib1g-dev libxml2-dev libxml2-dev libxslt1-dev libssl-dev libjpeg-dev python3-lxml virtualenv libopenjp2-7 libtiff5
```

### Project compilation:

```
mkdir -p ~/synapse
virtualenv -p python3 ~/synapse/env
cd ~/synapse/
source ~/synapse/env/bin/activate
pip install --upgrade pip
pip install --upgrade setuptools
pip install matrix-synapse[all]
```

### Generation of the configuration file for synapse-matrix

```
python3 -m synapse.app.homeserver --server-name YOUR_DOMAIN_NAME --config-path homeserver.yaml --generate-config --report-stats=no
deactivate
```

### Service configuration

```
vim ~/synapse/homeserver.yaml
```

Edit these parameters:

```
[..]
server_name: "YOUR_DOMAIN_NAME"
[..]
#IP listening parameters :
listeners:
- port: 8008
    tls: false
    bind_addresses: ['0.0.0.0']
    type: http
    x_forwarded: true
    resources:
       - names: [client, federation]
         compress: false
[..]
enable_registration: true
```

### Systemd configuration

```
addgroup synapse
sudo addgroup synapse
sudo adduser --system --home /home/pi/synapse/ --no-create-home --disabled-password --shell /bin/nologin --ingroup synapse synapse
```

### Create the systemd file

```
sudo vim /etc/systemd/system/matrix-synapse.service
```

Add this content

```
[Unit]  
Description=Matrix Synapse service  
After=network.target

[Service]  
Type=forking  
WorkingDirectory=/home/pi/synapse/  
ExecStart=/home/pi/synapse/env/bin/synctl start  
ExecStop=/home/pi/synapse/env/bin/synctl stop  
ExecReload=/home/pi/synapse/env/bin/synctl restart  
User=synapse  
Group=synapse  
Restart=always  
StandardOutput=syslog  
StandardError=syslog  
SyslogIdentifier=synapse

[Install]  
WantedBy=multi-user.target
```

### Service activation at server startup

```
sudo systemctl enable matrix-synapse
```

### Media storage directory and permissions

```
mkdir ~/synapse/media_store ~/synapse/uploads
chmod 770 ~/synapse/media_store ~/synapse/uploads
chmod 755 ~/synapse
sudo chown synapse:synapse ~/synapse ~/synapse/media_store ~/synapse/uploads
```

### Starting the service

```
sudo systemctl start matrix-synapse
```

### Creating the first account

```
sudo /home/pi/synapse/env/bin/register_new_matrix_user -c /home/pi/synapse/homeserver.yaml http://podcraft.ru:8008
```

The console asks you to enter the account name, password and answer “yes” to the question “Make admin”:

```
New user localpart[root]: admin  
Password: xxxxxxxxx  
Confirm password: xxxxxxx  
Make admin[no]: yes  
Sending registration request....  
Success
```

## Installation of Element clients (mobile and/or Desktop) and connection to the Matrix server

I refer you to the [Element website](https://element.io/) from where you can download the client adapted to your platform.

Start the Element client, by default, it offers a connection to the Matrix.org servers, click on “change”, indicate in the “url of the host server” box: “**http://YOUR_DOMAIN_NAME**“, or use the IP address of the server that hosts the Matrix application: “**http://[IP address]:[Matrix Port]**"
Ex: **http://192.168.1.10:8008** when Matrix listens on port 8008.

----
### Links:

- [1] [Matrix-Element : How to install Synapse-Matrix server - Raspberry PI - Rock64](https://www.mytinydc.com/en/blog/matrix-synapse-installationserveur-clientelement/)
- [2] [Client for Matrix - Element](https://element.io/)
