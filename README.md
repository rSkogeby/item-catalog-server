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

Create a virtual environment using 'venv' to create a directory venv:

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

Checkout the 'prod' branch:

```bash
cd item-catalog
git checkout prod
```

Install requirements
_Note: in your environment you always use 'pip', and not 'pip3'._

```bash
pip install -r requirements.txt
```

### Set up the Flask app

```bash
pip install wheel
```


## Configuration


## Firewall
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw enable
sudo ufw status
