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

- Ubuntu LTS 18.04
- nginx
- [Item Catalog](https://github.com/rSkogeby/item-catalog) (and all its requirements)
- python3-venv
- python3-pip
- python3-dev
- python3-setuptools
- build-essential
- libssl-dev
- libffi-dev


## Software installs

This guide assumes you start with a fresh Ubuntu LTS 18.04 install created with
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

Firewall configs are set up later, see section Firewall.


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

Delete the default instance nginx puts in `sites-enabled`

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Restart nginx:

```bash
sudo systemctl restart nginx
```

_Note: anytime you'd like to implement a change to your sites-available config you will need to restart nginx._


### Errorshooting and trouble finding

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
sudo journalctl -u item-catalog
```


## Add a non-standard SSH port

We'll be changing the SSH port to 2200.

In the Lightsail management console, open a port in the firewall configuration:

```bash
Application = Custom
Protocol = TCP
Port = 2200
```

In your Ubuntu instance edit the port in `sshd_config` in `/etc/ssh/` directory to 2200:

```bash
sudo vim /etc/ssh/sshd_config
```

and do the following change:

```bash
- # Port 22
+ Port 2200
```

`:wq` to write and quit, and proceed to restart sshd:

```bash
sudo systemctl restart sshd
```

When everything has been set up we'll also configure the firewall to allow incoming
requests on port 2200.

## Create grader user

```bash
sudo adduser grader
```

Give `sudo` privileges:

```bash
usermod -aG sudo grader
```

### Create grader user - setup keypair

Create keypair on server and client.

On client:

```bash
ssh-keygen
```

Specify directory: `/Users/richard/.ssh/<filename>`

Add key to SSH authentication agent for implementing single sign-on with SSH:

```bash
ssh-add .ssh/<filename>
```

On host (your Ubuntu instance):

While logged in as your normal user, switch to the grader us:

```bash
su - grader
```

Create an `.ssh` directory and add an `authorized_keys` file to it:

```bash
mkdir .ssh
vim .ssh/authorized_keys
```

paste the content of your <filename>.pub created earlier during your SSH keygen
into the authorized_keys file on your host.

Set privileges:

```bash
chmod 700 ~/.ssh
chmod 644 ~/.ssh/authorized_keys
```

Now you should be able to login on the grader using:

```bash
ssh grader@<public_ip> -p 2200 -i ~/.ssh/<filename>
```

on your client.

## Disable password logins

Edit `/etc/ssh/sshd_config`:

```bash
/PasswordAuthentication
```

_Note: forward-slash in vim to search for line with PasswordAuthentication in it._

Append the line with `no`:

```bash
- PasswordAuthentication
+ PasswordAuthentication no
```

Restart `service ssh`:

```bash
sudo service ssh restart
```

## Disallow root login

Edit `sshd_config`:

```bash
sudo vim /etc/ssh/sshd_config
```

Type `/` to search for `PermitRootLogin`, uncomment the line and append with `no`:

```bash
- #PermitRootLogin yes
+ PermitRootLogin no
```

## Configure timezone

Configure timezone to UTC.

```bash
sudo dpkg-reconfigure tzdata
```

Select option 'None of the above' > UTC.

## Firewall

Configure the firewall as:


```bash
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw enable
sudo ufw status
```

Allow incoming requests on port 2200 for SSH access.
Set firewall access for out- and ingoing on port 80/tcp:

```bash
sudo ufw allow 'Nginx HTTP'
```

## Setting up PostgreSQL

```bash
sudo apt-get install postgresql
```

Start PostgreSQL on startup:

```bash
sudo update-rc.d postgresql enable
```

Start PostgreSQL:

```bash
sudo service postgresql start
```

With the user and database setup, switch user to continue the setup:

```bash
sudo su - postgres
```

Start a postgres interactive query session to make sure things are working:

```bash
psql
```

To exit the query session type `\q`, and to get back to your other user `exit`.

In your main user create an a database:

```bash
sudo -u postgres createdb item_catalog_db
```




# List of help

- []()
- []()
- []()
- [Setup for PostgreSQL](https://www.godaddy.com/garage/how-to-install-postgresql-on-ubuntu-14-04/)
- [Timezone config](https://help.ubuntu.com/community/UbuntuTime)
