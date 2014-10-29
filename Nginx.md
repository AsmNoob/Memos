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

####Détail de chaque opération:####

- The operation used to stop nginx processes with waiting for the worker processes to finish serving current requests:
	
		nginx -s quit

- The operation reloading the nginx configuration to apply new changes:

		nginx -s reload

- The list of all nginx processes:

		ps -ax | grep nginx


Configuring static server
-------------------------

An important web server task is serving out files, files will be served from different local directories: /data/www (which may contain HTML files) and /data/images (containing images). This will require editing of the configuration file and setting up of a server block inside the http block with two location blocks. 

1. First, create the /data/www directory and put an index.html file with any text content into it and create the /data/images directory and place some images in it. 

2. Open the configuration file and add a server bloc:
	
	http {
    	server {
    	}
	}
The configuration file may have several server blocs depending on the different bloc they listen to.

3. Add the following location block to the server block: 

	location / {
    	root /data/www;
	}

This location block specifies the “/” prefix compared with the URI from the request. For matching requests, the URI will be added to the path specified in the root directive, that is, to /data/www, to form the path to the requested file on the local file system. If there are several matching location blocks nginx selects the one with the longest prefix. The location block above provides the shortest prefix, of length one, and so only if all other location blocks fail to provide a match, this block will be used.

4. Add the second location block: 

	location /images/ {
	    root /data;
	}

It will be a match for requests starting with /images/ (location / also matches such requests, but has shorter prefix). 

This is already a working configuration of a server that listens on the standard port 80 and is accessible on the local machine at http://localhost/.

5. To apply the new configuration, start nginx if it is not yet started or send the reload signal to the nginx’s master process, by executing: 

	nginx -s reload


**IMPORTANT**:

 	In case something does not work as expected, you may try to find out the reason in access.log and error.log files in the directory /usr/local/nginx/logs or /var/log/nginx. 

Setting up a Proxy Server
-------------------------

One of the frequent uses of nginx is setting it up as a proxy server, which means a server that receives requests, passes them to the proxied servers, retrieves responses from them, and sends them to the clients.

1. First, define the proxied server by adding one more server block to the nginx’s configuration file with the following contents:

	server {
	    listen 8080;
	    root /data/up1;

	    location / {
	    }
	}

This will be a simple server that listens on the port 8080 (previously, the listen directive has not been specified since the standard port 80 was used) and maps all requests to the /data/up1 directory on the local file system. Create this directory and put the index.html file into it. Note that the root directive is placed in the server context. Such root directive is used when the location block selected for serving a request does not include own root directive. 

2. use the server configuration from the previous section and modify it to make it a proxy server configuration. In the first location block, put the proxy_pass directive with the protocol, name and port of the proxied server specified in the parameter (in our case, it is http://localhost:8080): 

	server {
	    location / {
	        proxy_pass http://localhost:8080;
	    }

	    location /images/ {
	        root /data;
	    }
	}

3.  We will modify the second location block, which currently maps requests with the /images/ prefix to the files under the /data/images directory, to make it match the requests of images with typical file extensions. The modified location block looks like this: 

	location ~ \.(gif|jpg|png)$ {
	    root /data/images;
	}

The parameter is a regular expression matching all URIs ending with .gif, .jpg, or .png. A regular expression should be preceded with ~. The corresponding requests will be mapped to the /data/images directory. 

**IMPORTANT**
 When nginx selects a location block to serve a request it first checks location directives that specify prefixes, remembering location with the longest prefix, and then checks regular expressions. If there is a match with a regular expression, nginx picks this location or, otherwise, it picks the one remembered earlier. 

 4. The resulting configuration of a proxy server will look like this: 

	server {
	    location / {
	        proxy_pass http://localhost:8080/;
	    }

	    location ~ \.(gif|jpg|png)$ {
	        root /data/images;
	    }
	}

This server will filter requests ending with .gif, .jpg, or .png and map them to the /data/images directory (by adding URI to the root directive’s parameter) and pass all other requests to the proxied server configured above.

To apply new configuration, send the reload signal to nginx as described in the previous sections. 

Setting up FastCGI Proxying
---------------------------
 [FastCGI Proxing]()

nginx can be used to route requests to FastCGI servers which run applications built with various frameworks and programming languages such as PHP.

The most basic nginx configuration to work with a FastCGI server includes using the fastcgi_pass directive instead of the proxy_pass directive, and fastcgi_param directives to set parameters passed to a FastCGI server. Suppose the FastCGI server is accessible on localhost:9000. Taking the proxy configuration from the previous section as a basis, replace the proxy_pass directive with the fastcgi_pass directive and change the parameter to localhost:9000. In PHP, the SCRIPT_FILENAME parameter is used for determining the script name, and the QUERY_STRING parameter is used to pass request parameters. The resulting configuration would be: 


	server {
	    location / {
	        fastcgi_pass  localhost:9000;
	        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	        fastcgi_param QUERY_STRING    $query_string;
	    }

	    location ~ \.(gif|jpg|png)$ {
	        root /data/images;
	    }
	}

This will set up a server that will route all requests except for requests for static images to the proxied server operating on localhost:9000 through the FastCGI protocol. 

More information
----------------

###Server###
The HTTP block of the nginx.conf file contains the statement include /etc/nginx/sites-enabled/*;. This allows for server block configurations to be loaded in from separate files found in the sites-enabled sub-directory. Usually these are symlinks to files stored in /etc/nginx/sites-available/. By using symlinks you can quickly enable or disable a virtual server while preserving its configuration file. Nginx provides a single default virtual host file, which can be used as a template to create virtual host files for other domains:

	cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example.com

####Basic Server block####

	server {
	        listen 80 default_server;
	        listen [::]:80 default_server ipv6only=on;

	        root /usr/share/nginx/html;
	        index index.html index.htm;

	        # Make site accessible from http://localhost/
	        server_name localhost;

	        location / {
	                # First attempt to serve request as file, then
	                # as directory, then fall back to displaying a 404.
	                try_files $uri $uri/ /index.html;
	                # Uncomment to enable naxsi on this location
	                # include /etc/nginx/naxsi.rules
	}


####Ports####
####Name-based virtual hosting####
####Access logs####
####Location####
####Location Root and Index####
####Best practice####
