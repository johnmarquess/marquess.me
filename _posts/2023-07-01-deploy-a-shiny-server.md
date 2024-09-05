---
layout: single
title:  "Deploy a Shiny server on digital ocean Ubuntu droplet"
date:   2023-08-01 18:45:00 +1000
tags: shiny ubuntu  
seo_tags: shiny ubuntu server
seo_title: "Deploy a Shiny server on digital ocean Ubuntu droplet"
seo_description: "How to set up a shiny server on a Digitalocean Ubuntu droplet."
excerpt: "This post will walk you through how to set up a shiny server on a Digitalocean Ubuntu droplet. "
---

![Shiny logo]({{ site.url }}{{ site.baseurl }}/assets/images/shiny_logo.jpg)

Shiny is a powerful web application framework for python and R, enabling the creation of interactive data visualizations deployed as web apps. To run these applications on a server, you need to install Shiny Server. In this blog post, we will walk you through the process of installing Shiny Server on Ubuntu.

## Prerequisites:
Before we begin, make sure you have the following :

1. An Ubuntu server with root access.
2. Basic knowledge of working with the Linux command line.

If you don't have a Ubuntu droplet already, follow this [link](https://m.do.co/c/74f204b4fdd4) to set one up. You will need to set up an account to create the droplet. I recommend the a low cost option. You can always upgrade later. I'm using Ubuntu 22.04 LTS (x64) for this example. You can see the instructions for setting up a droplet [here](https://docs.digitalocean.com/products/droplets/how-to/create/). There are [clear instructions here](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-22-04) for the inital server setup using a non-root user with elevated privileges.


### Update everything on the server
Now that you have logged on to your droplet, let's ensure that all system packages are up to date by running the following commands. I like to install build-essential. This is a meta-package that installs all the packages required for compiling software on Ubuntu. This will be useful later when we install R packages.


``` bash
sudo apt update
sudo apt upgrade
sudo apt install build-essential
```

I also like to install a few


### Install R and the packages
To run Shiny Server, we need to have R installed on our system. This can be a pain to set up securely. Luckily there is a helpful guide on on [CRAN](https://cran.rstudio.com/bin/linux/ubuntu/) to atke the pain out of it. I have included the steps for ceonvenience.

``` bash
# update indices
sudo apt update -qq

# install two helper packages we need
sudo apt install --no-install-recommends software-properties-common dirmngr

# add the signing key (by Michael Rutter) for these repos
# To verify key, run gpg --show-keys /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc 
# Fingerprint: E298A3A825C0D65DFD57CBB651716619E084DAB9
wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc

# add the R 4.0 repo from CRAN -- adjust 'focal' to 'groovy' or 'bionic' as needed
sudo add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"
```

```
sudo apt install --no-install-recommends r-base
```

### Install Required R Packages
Now that we have R installed, let's install some necessary packages. There are many ways to do this. I like to make things easy on myself and  get them from a repo. This is a good way to ensure that you have the latest versions and it's easy to update them later.

``` bash
# First add the repo key ID.
sudo add-apt-repository ppa:c2d4u.team/c2d4u4.0+

# Then install the packages (after updating the package index)
apt install --no-install-recommends r-cran-tidyverse r-cran-shiny r-cran-rmarkdown

# You can keeping adding to the list as needed if you intend to use R for other things.
```

### Download and Install Shiny Server
You can locate the latest version of the Shiny Sever from the [Posit website](https://posit.co/download/shiny-server/). There will be a link to download and install the most recent version. Here is what I used.

``` bash
# Get the app to install deb files
sudo apt-get install gdebi-core
# Download the latest version of Shiny Server
wget https://download3.rstudio.org/ubuntu-18.04/x86_64/shiny-server-1.5.20.1002-amd64.deb
# Install the package
sudo gdebi shiny-server-1.5.20.1002-amd64.deb
```

### Configure Shiny Server

You can start, although it will likely already be running, and verify Shiny Server using the following command:

``` 
sudo systemctl start shiny-server
```

To verify if Shiny Server is running correctly, open a web browser and enter your server's IP address or domain name followed by `:3838`. You should see the default Shiny Server landing page.

If you don't know you IP address, you can find it by running the following command:

``` bash
curl -4 icanhazip.com
```

You should see this page:

![Shiny landing page]({{ site.url }}{{ site.baseurl }}/assets/images/shiny_landing.png)

Success! You have successfully installed Shiny Server on your Ubuntu machine. Now you can begin developing and deploying interactive web applications using R's powerful visualization capabilities. 


### Configure Shiny Server and users/directories
By default, Shiny Server looks for applications in the `/srv/shiny-server/` directory. If you want to host your own applications, place them in this directory. Additionally, you can customise the configuration file located at `/etc/shiny-server/shiny-server.conf` to modify various settings like port number, log location, etc.

To host applications in a different directory, you can add a new location in the configuration file. For example, if you want to host applications in the `/home/user/shiny-apps/` directory, you can add the following lines to the configuration file:

``` bash
# Define a new server block
server {
  # Define the port on which the server listens
  listen 3838;
  # Define the location of the application directory
  location /shiny/ {
    app_dir /home/user/shiny-apps/;
  }
}
```

### Replace the port number 

By default, Shiny Server is access through port - `http://ipaddress:3838`. There are two reasons this is problematic: (a) it looks crappy, and (b) if you have set up SSL certificates to secure you domain with a reverse proxy then you wil not be able to embed your shiny apps in your website. To recity this I used the helpful advice from [this post](https://www.marinedatascience.co/blog/2019/04/28/run-shiny-server-on-your-own-digitalocean-droplet-part-2/) to resolve the port number to a folder name.

First, eplace the port numbers with the right location in you nginx config file. Navigate to the `/etc/nginx/sites-enabled/` and open the file default. As before, you can do it directly in the console using nano. 

``` bash
# using nano
sudo nano /etc/nginx/sites-enabled/default
```
Add the following lines to the beginning of file above `server {`. 

``` bash
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}
```

Then, add this as a new location block within the server block. 

``` bash
location /shiny/ {
  proxy_pass http://127.0.0.1:3838/;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade"; 
  rewrite ^(/shiny/[^/]+)$ $1/ permanent;
}
```

**Important**
The IP address 127.0.0.1 is NOT the droplet’s IP address and should be therefore not replaced with your own IP. Just copy the above lines with no further changes and paste them into your config file!

### Conclusion
In this post, we have walked you through the process of installing Shiny Server on Ubuntu. The first sections are enough to get you up and running to experiment with Shiny. The later sections will be useful when you run into the same frustrations I did when trying to embed shiny apps in on a secure domain.

