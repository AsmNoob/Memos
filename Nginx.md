Intro
=====

Nginx is a lightweight, high performance web server designed to deliver large amounts of static content quickly with efficient use of system resources. Nginx’s strong point is its ability to efficiently serve static content, like plain HTML and media files. Some consider it a less than ideal server for dynamic content.

All Nginx configuration files are located in the **/etc/nginx/** directory. The primary configuration file is **/etc/nginx/nginx.conf.**

Prerequisites
-------------

- **Install the Nginx server**

	sudo apt-get install nginx

- **Run the server**

	sudo service nginx start

- **Testing it**

	Point your browser to your IP address[YourIPAddress](http://whatismyipaddress.com/), it should confirm that nginx was successfully installed

- **Check your VPS's IP address**
	
	ifconfig eth0 | grep inet | awk '{ print $2 }'

- **Open up the default virtual host file with this command**

	sudo nano /etc/nginx/sites-available/default

Preserve a Working Configuration
--------------------------------
For a really good restoration option, it's recommended making regular backups of the Nginx configuration, how ... ?

	Store the entire /etc/nginx/ directory in a Git repository

Understanding Nginx Functioning
-------------------------------

Initial Setup
-------------

Nginx Location
--------------

Configuring a static server (lvl 1)
-----------------------------------

Operation on Nginx configuration
--------------------------------

Configuration file's structure
------------------------------

Configuring static server (lvl 2)
---------------------------------

Setting up a Proxy Server
-------------------------

Setting up FastCGI Proxying
---------------------------
 [FastCGI Proxing]()

