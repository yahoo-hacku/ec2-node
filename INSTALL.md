# Setup and installation

First of all, make a clone or [fork of this repository](http://help.github.com/fork-a-repo/) and replace all occurrences of `hacku` with a name of your choice.

## Launch an EC2 instance

[Start a "micro" Amazon EC2 instance](https://console.aws.amazon.com/ec2/home) and use one of the following AMIs, depending on where you chose to launch it:

- US west: `ami-ad7e2ee8`
- US east: `ami-ccf405a5`
- EU west: `ami-fb9ca98f`
- Asia Pacific (Singapore): `ami-0c423c5e`

> These AMIs where taken from <http://uec-images.ubuntu.com/releases/10.10/release/>

Go with the defaults in the "wizard" presented. Chose to create a new key pair when asked and **be sure to make a secure backup of the private key** that you will download. A good place to put your private key is in `~/.ssh/hacku.pem` and then `chmod 0600 ~/.ssh/hacku.pem` so no one else can read it but you.

When the instance is green and "started", log in to the machine:

    ssh -i ~/.ssh/hacku.pem ubuntu@XXX.amazonaws.com

*Note: Replace `XXX.amazonaws.com` with the hostname or address of your instance*


## Install software

    sudo apt-get update
    sudo apt-get install nginx git-core daemon
    sudo chown -R www-data:www-data /var/www

Node.js:

    sudo apt-get install build-essential libssl-dev
    cd
    mkdir src
    git clone https://github.com/joyent/node.git src/node
    cd src/node
    # Optionally: git checkout v0.4.3
    ./configure
    JOBS=2 make
    sudo make install

NPM:

    sudo true && curl http://npmjs.org/install.sh | sudo sh
    sudo npm install connect connect_json move


## Checkout your source

    sudo mkdir /var/hacku
    sudo chown www-data:www-data /var/hacku

If your git repository is public (i.e. viewable by anyone):

    sudo -Hu www-data git clone https://github.com/you/hacku.git /var/hacku

If your git repository is private:

    sudo -Hu www-data ssh-keygen -t rsa  # chose "no passphrase"
    sudo cat /var/www/.ssh/id_rsa.pub
    # Add the key as a "deploy key" at https://github.com/you/hacku/admin
    sudo -Hu www-data git clone git@github.com:you/hacku.git /var/hacku


## Configure & start your services

Your Node.js web server:

    sudo ln -s /var/hacku/init.d/hacku-httpd /etc/init.d/
    sudo update-rc.d hacku-httpd defaults
    sudo invoke-rc.d hacku-httpd start
    
Optional `hacku-processor`:
    
    sudo ln -s /var/hacku/init.d/hacku-processor /etc/init.d/
    sudo update-rc.d hacku-processor defaults
    sudo invoke-rc.d hacku-processor start


## Configure Nginx

There are three different kinds of setups to chose from:

1. `hacku-http` -- HTTP only
2. `hacku-https` -- HTTPS with HTTP redirecting to HTTPS
3. `hacku-https-http` -- HTTPS and HTTP

If you are using HTTPS, make sure you have added your SSL certificate and key at `/var/hacku/ssl/ssl.crt` and `/var/hacku/ssl/ssl.key`.

Replace `hacku-https` below with the configuration of your choice:

    sudo ln -sf /var/hacku/etc/nginx/sites-available/hacku-https \
                /etc/nginx/sites-enabled/default
    sudo invoke-rc.d nginx restart


## Done

Your web app should now be operational.

If everything works, continue by reading [WORKFLOW.md](https://github.com/rsms/ec2-webapp/blob/master/WORKFLOW.md#readme)
