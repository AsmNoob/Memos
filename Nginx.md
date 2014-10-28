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

- **Creating Symlinks if managing many sites**

		ln -s /etc/nginx/sites-available/dotcom /etc/nginx/sites-enabled/dotcom


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
- **Settings syntax**: Variable Arguments (separated by spaces).
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


###HTTP (5Universal Config)###

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

	    ##
	    # Many more configuration directives
	    ##

	}

**Include**:
The include statement at the beginning of this section includes the file mime.types located at /opt/nginx/conf/mime.types. What this means is that anything written in the file mime.types is interpreted as if it was written inside the http { } block. That way you can include a lot of information without cluttering the main configuration.

You can include all the files in a directory using this directive:

	include /etc/nginx/sites-enabled/*;

**Gzip**:
The gzip directive tells the server to use on-the-fly gzip compression to limit the amount of bandwidth used and speed up some transfers.

Operation on Nginx configuration
--------------------------------
###Starting, Stopping, and Reloading Configuration###

While nginx is running, it can be controlled by invoking this command:
	nginx -s signal

where the *signal* might be:

- **stop** — fast shutdown
- **quit** — graceful shutdown
- **reload** — reloading the configuration file
- **reopen** — reopening the log files 

Détail de chaque opération:
	- nginx -s quit

Used to stop nginx processes with waiting for the worker processes to finish serving current requests
	




Configuration file's structure
------------------------------

Configuring static server (lvl 2)
---------------------------------

Setting up a Proxy Server
-------------------------

Setting up FastCGI Proxying
---------------------------
 [FastCGI Proxing]()

