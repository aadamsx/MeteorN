# MeteorN - A Simple Tool to Run Meteor App with an Nginx Reverse Proxy

MeteorN is meant to help you run your Meteor app with a reverse proxy of Nginx at the front end in docker containers.

A reverse proxy can serve many purpose such as firewall or load balancer.

But for me, the most important thing is that reverse proxy can help me add SSL support to my web app.

This project is composed of two parts.

Use the `Basic` version if you do not need SSL.

Otherwise, check the `SSLImplementd` one out.

## Which Branch to use:
- master for newest version
- pre1.4 for older version

## How to Use the Basic Version


###  1. Build Your Meteor App into a Plain Node Js Application

The Meteor tool has a command `meteor build` that creates a deployment bundle which contains a plain Node.js application.

Before using it, you'll have to install all the npm dependencis for your app in advance.

Also it's important to chose a correct target architecture for your app, you can specify it  with `--architecture`.

Issue these commands inside your Meteor app folder.

~~~shell
npm install --production
meteor build /tmp --architecture os.linux.x86_64
~~~

After finishing these two commands, you will find a bundle file in **/tmp**.(Named as projectName.tar.gz)

###  2. Clone MeteorN

~~~shell
cd /tmp
git clone https://github.com/lo-tp/MeteorN.git
cp projectName.tar.gz  MeteorN/Basic
cd MeteorN/Basic
~~~

###  3. Configurations

Change the node version in accordance with the rules described below:

- Node 4.4.7 for Meteor 1.4.x
- Node 0.10.43 for Meteor 1.3.x and earlier


This can be done by opening **dockerfiles/node.dockerfile** and edit this line:

~~~
FROM node:xxx
~~~

Then open **docker-compose.yml** and edit this line:

~~~
- ROOT_URL=http://127.0.0.1
~~~

Change `http://127.0.0.1` to your domain name.


### 4. Build And Run
~~~shell
docker-compose build
docker-compose up
~~~
You can also run `docker-compose up -d` to run your application in the background.

Now your app is happily running at your domain name.

## How to Use the SSLImplemented Version
It's largely similiar to the use of the `Basic` version.

All the relevant files are located in the `MeteorN/SSLImplemented` folder.

There is one more thing to be done before using this version.

You'll have to copy your SSL cirtificate file and SSL cirtificate key to the `MeteorN/SSLImplemented/ssl` folder respectively as cert.pem and privkey.pem.

Then run `docker-compose up` and your meteor app is ready to go with SSL support.

**One thing to mention: The http traffic is redirected to https by default.**

If you want to keep the http working, you can edit the Nginx configure file located at `conf` folder.

## How to Use the lsEncrypt Version

In addtion to the steps described in the [Basic Version](#how-to-use-the-basic-version).

Some more file modifications are necessary.

Find these lines in `docker-compose.yml`.
```yml
environment:
	- DOMAIN=www.test.xyz
	- MAIL=test@hotmail.com
```
Change the values of `MAIL` and `DOMAIN` to your domain name and email address.

Then open `conf/cert.conf`.
```
domains = www.test.xyz
email=test@hotmail.com
rsa-key-size = 4096
authenticator = webroot
webroot-path = /tmp
```
Change the first two lines to your domain name and email address.

Also change all the domain names from **www.test.xyz** to your own contained in `conf/nginx.conf`.

```
http{
	upstream app_servers {
		server meteor:8080;
	}
	server {
		listen 80 ;
		server_name www.test.xyz                				#This Line
		location '/.well-known/acme-challenge' {
			...
		}
		location / {
			return    301 https://$server_name$request_uri;
		}
	}
	server { 
		listen 443; 
		ssl on; 
		ssl_certificate /etc/letsencrypt/live/www.test.xyz/cert.pem;            #This Line
		ssl_certificate_key /etc/letsencrypt/live/www.test.xyz/privkey.pem;     #This Line
		...
		location / {
		...
		}
	}
}

events {
	worker_connections 1024;
}
```

Now run `docker-compose up` and wait for the **Nginx** container to automatically get the necessary SSL files from [Let's Encrypt][lpt] and serve your website in https mode.

**Notice: Don't play with this version too frequently, there is a [quota][qt] on how many certificates you can get per week**.

## Use the Nginx Reverse Proxy to Do Other Thing

You can edit the `conf/nginx.conf` to implement other functionalities with the reverse proxy.

[lpt]:https://letsencrypt.org/
[qt]:https://letsencrypt.org/docs/rate-limits/
