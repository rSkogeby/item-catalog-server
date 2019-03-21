# Item Catalog Server

i. The IP address and SSH port so your server can be accessed by the reviewer.
ii. The complete URL to your hosted web application.
iii. A summary of software you installed and configuration changes made.
iv. A list of any third-party resources you made use of to complete this project.

## Access

IP Address: 3.8.59.90
SSH Port: 2200

URL:
http://3.8.59.90
http://pintfindr.com


## Software

- Ubuntu LTS 18
- nginx
- uwsgi
- [Item Catalog](https://github.com/rSkogeby/item-catalog)
- python3-venv
- python3-pip
- python3-dev
- python3-setuptools
- build-essential
- libssl-dev
- libffi-dev


## Software installs

This guide assumes you start with a fresh Ubuntu LTS 18 install created with
Amazon Lightsail, and assumes you follow it through in a linear fashion.
It is almost entirely based on [Kathlin Juell's](https://www.digitalocean.com/community/users/katjuell) and [Justin Ellingwood's](https://www.digitalocean.com/community/users/jellingwood) article on
DigitalOcean: [How To Serve Flask Applications with uWSGI and Nginx on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uswgi-and-nginx-on-ubuntu-18-04).

Connect to your instance using an SSH client:

```bash
ssh ubuntu@<public-ip>
```

Update and upgrade:

```bash
sudo apt-get update & sudo apt-get upgrade
```

### Install nginx

Install nginx:

```bash
sudo apt-get update & sudo apt-get install nginx
```

Set firewall access for out- and ingoing on port 80/tcp:

```bash
sudo ufw allow 'Nginx HTTP'
```

### Install additional requisites from Ubuntu repository

```bash
sudo apt-get update
sudo apt-get install python3-pip python3-dev python3-setuptools build-essential libssl-dev libffi-dev
```

### Install and setup a Python virtual environment

Install venv for Python 3:

```bash
sudo apt-get install python3-venv
```

Create a virtual environment using `venv` to create a directory venv:

```bash
python3 -m venv venv
```

Activate your virtual environment:

```bash
source venv/bin/activate
```

### Clone Item Catalog
_With your virtual environment activated_

Clone repository:

```bash
git clone https://github.com/rSkogeby/item-catalog.git
```

Checkout the `prod` branch:

```bash
cd item-catalog
git checkout prod
```

Install requirements
_Note: in your environment you always use 'pip', and not 'pip3'._

```bash
pip install wheel
pip install -r requirements.txt
```


## uWSGI configuration

Walkthrough of how to configure the uWSGI application server container.

Deactivate the virtual environment:

```bash
deactivate
```

Copy the file `item-catalog.service` to `/etc/systemd/system`

```bash
sudo cp item-catalog.service /etc/systemd/system/
```

Configure the field `User` in `/etc/systemd/system/item-catalog.service` appropriately:

```bash
[Service]
User=ubuntu
Group=www-data
```

Also make sure the file structures agree with what you have set up. If you've followed
these notes to the point, you won't need to change those.

Start up your uWSGI server and enable it:

```bash
sudo systemctl start item-catalog
sudo systemctl enable item-catalog
```

Make sure `sudo systemctl status` doesn't generate any faults.

## Nginx configuration

Walkthrough to enable Nginx to pass `uwsgi` web requests to the uWSGI application
server container.

Copy `item-catalog.nginx` in `item-catalog` as per:

```bash
sudo cp item-catalog.nginx /etc/nginx/sites-available/item-catalog
```

_Note: the file should have no file-ending when in the sites-available directory._

Create a soft link to that file in the `sites-enabled` directory:

```bash
sudo ln -s /etc/nginx/sites-available/item-catalog /etc/nginx/sites-enabled
```

Restart nginx:

```bash
sudo systemctl restart nginx
```

_Note: anytime you'd like to implement a change to your sites-available config you will need to restart nginx._

At this stage you should be able to access Item Catalog on your <public_ip>.
If you are not able to do so, you might need to make appropriate configurations
on your host platform, or a typo or other error has been introduced at some stage
in the installation process. In which case you can use these commands to
troubleshoot nginx and uWSGI:

Check the Nginx error logs.
```bash
sudo less /var/log/nginx/error.log
```

Check the Nginx access logs.
```bash
sudo less /var/log/nginx/access.log
```

Checks the Nginx process logs.
```bash
sudo journalctl -u nginx
```

Check your Flask app's uWSGI logs.
```bash
sudo journalctl -u myproject
```


## Firewall
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw enable
sudo ufw status
