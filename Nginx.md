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

- **Testing it** [YourIPAddress](http://whatismyipaddress.com/)

		Point your browser to your IP address, it should confirm that nginx was successfully installed

- **Check your VPS's IP address**
	
		ifconfig eth0 | grep inet | awk '{ print $2 }'

- **Open up the default virtual host file with this command**

		sudo nano /etc/nginx/sites-available/default

Preserve a Working Configuration
--------------------------------
For a really good restoration option, it's recommended making regular backups of the Nginx configuration, how ... ?

	Store the entire /etc/nginx/ directory in a Git repository

Introduction to Nginx Syntax
----------------------------

Taking an excerpts of a nginx.conf file, to analyse the different elements:
	
File excerpt: **/etc/nginx/nginx.conf**:

	user www-data;
	worker_processes 4;
	pid /run/nginx.pid;

	events {
	        worker_connections 768;
	        # multi_accept on;
	}

- **&** or **#** are comments balises
- **Settings syntax**: $variable $arguments (separated by spaces), don't forget the coma's.s
- Some settings have arguments that are themselves settings with arguments, we use the  **{ }** to deal with those.

Understanding Nginx Functioning(Nginx.config)
---------------------------------------------

###Core Directives of Nginx###

	user www-data;
	worker_processes 4;
	pid /run/nginx.pid;

	events {
	        worker_connections 768;
	        # multi_accept on;
	}

**user**:

Defines which Linux system user will own and run the Nginx server.

**worker_process**:

Defines how many threads, or simultaneous instances, of Nginx to run.

**pid**:

Defines where Nginx will write its master process ID, or PID. The PID is used by the operating system to keep track of and send signals to the Nginx process.


###HTTP 5Universal Config)###

####Handeling webtraffic####

File excerpt: **/etc/nginx/nginx.conf**

	http {

	    ##
	    # Basic Settings
	    ##

	    sendfile on;
	    tcp_nopush on;
	    tcp_nodelay on;
	    keepalive_timeout 65;
	    types_hash_max_size 2048;
	    # server_tokens off;

	    # server_names_hash_bucket_size 64;
	    # server_name_in_redirect off;

	    include /etc/nginx/mime.types;
	    default_type application/octet-stream;

	    ##
	    # Logging Settings
	    ##

	    access_log /var/log/nginx/access.log;
	    error_log /var/log/nginx/error.log;

	    ##
	    # Gzip Settings
	    ##

	    gzip on;
	    gzip_disable "msie6";


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

