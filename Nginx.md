Introduction to Nginx
=====================

*Nginx is a lightweight, high performance web server designed to deliver large amounts of static content quickly with efficient use of system resources. Nginx’s strong point is its ability to efficiently serve static content, like plain HTML and media files. Some consider it a less than ideal server for dynamic content.*

**Note**:All Nginx configuration files are located in the **/etc/nginx/** directory. The primary configuration file is **/etc/nginx/nginx.conf.**

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

1. **First, create the /data/www directory and put an index.html file with any text content into it and create the /data/images directory and place some images in it.**

2. **Open the configuration file and add a server bloc:**
	
		http {
			server {
			}
		}

The configuration file may have several server blocs depending on the different bloc they listen to.

3. **Add the following location block to the server block:**

		location / {
			root /data/www;
		}

This location block specifies the “/” prefix compared with the URI from the request. For matching requests, the URI will be added to the path specified in the root directive, that is, to /data/www, to form the path to the requested file on the local file system. If there are several matching location blocks nginx selects the one with the longest prefix. The location block above provides the shortest prefix, of length one, and so only if all other location blocks fail to provide a match, this block will be used.

4. **Add the second location block:**

		location /images/ {
			root /data;
		}

It will be a match for requests starting with /images/ (location / also matches such requests, but has shorter prefix). 

This is already a working configuration of a server that listens on the standard port 80 and is accessible on the local machine at http://localhost/.

5. **To apply the new configuration, start nginx if it is not yet started or send the reload signal to the nginx’s master process, by executing:**

		nginx -s reload


**IMPORTANT**:

*In case something does not work as expected, you may try to find out the reason in access.log and error.log files in the directory /usr/local/nginx/logs or /var/log/nginx.*

Setting up a Proxy Server
-------------------------

One of the frequent uses of nginx is setting it up as a proxy server, which means a server that receives requests, passes them to the proxied servers, retrieves responses from them, and sends them to the clients.

1. **First, define the proxied server by adding one more server block to the nginx’s configuration file with the following contents:**

		server {
			listen 8080;
			root /data/up1;

			location / {
			}
		}

This will be a simple server that listens on the port 8080 (previously, the listen directive has not been specified since the standard port 80 was used) and maps all requests to the /data/up1 directory on the local file system. Create this directory and put the index.html file into it. Note that the root directive is placed in the server context. Such root directive is used when the location block selected for serving a request does not include own root directive. 

2. **Use the server configuration from the previous section and modify it to make it a proxy server configuration. In the first location block, put the proxy_pass directive with the protocol, name and port of the proxied server specified in the parameter (in our case, it is http://localhost:8080):**

		server {
			location / {
				proxy_pass http://localhost:8080;
			}

			location /images/ {
				root /data;
			}
		}

3.  **We will modify the second location block, which currently maps requests with the /images/ prefix to the files under the /data/images directory, to make it match the requests of images with typical file extensions. The modified location block looks like this:**

		location ~ \.(gif|jpg|png)$ {
			root /data/images;
		}

The parameter is a regular expression matching all URIs ending with .gif, .jpg, or .png. A regular expression should be preceded with ~. The corresponding requests will be mapped to the /data/images directory. 

**IMPORTANT**
*When nginx selects a location block to serve a request it first checks location directives that specify prefixes, remembering location with the longest prefix, and then checks regular expressions. If there is a match with a regular expression, nginx picks this location or, otherwise, it picks the one remembered earlier.*

 4. **The resulting configuration of a proxy server will look like this:** 

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

Nginx can be used to route requests to FastCGI servers which run applications built with various frameworks and programming languages such as PHP.

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
*The HTTP block of the nginx.conf file contains the statement include /etc/nginx/sites-enabled/*;. This allows for server block configurations to be loaded in from separate files found in the sites-enabled sub-directory. Usually these are symlinks to files stored in /etc/nginx/sites-available/. By using symlinks you can quickly enable or disable a virtual server while preserving its configuration file. Nginx provides a single default virtual host file, which can be used as a template to create virtual host files for other domains*:

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


The server block is where the typical Nginx user will make most of his or her changes to the default configuration. Generally, you’ll want to make a separate file with its own server block for each virtual domain on your server. More configuration options for the server block are shown in the following sections.

####Ports####

*The listen directive, which is located in the server block, tells Nginx the hostname/IP and the TCP port where it should listen for HTTP connections. By default, Nginx will listen for HTTP connections on port 80.The listen directive, which is located in the server block, tells Nginx the hostname/IP and the TCP port where it should listen for HTTP connections. By default, Nginx will listen for HTTP connections on port 80.*

#####Common examples for the listen directive:#####

1. **These are the default listen statements in the default virtual host file. The argument default_server means this virtual host will answer requests on port 80 that don’t specifically match another virtual host’s listen statement. The second statement listens over IPv6 and behaves in the same way.**

		listen 80 default_server;
		listen [::]:80 default_server ipv6only=on;


2. **These two examples direct Nginx to listen on 127.0.0.1; that is, the local loopback interface. localhost is conventionally set as the hostname for 127.0.0.1 in /etc/hosts.**
		
		listen     127.0.0.1:80;
		listen     localhost:80;

3. **The third pair of examples also listen on localhost, but they listen for responses on port 8080 instead of port 80.**

		listen     127.0.0.1:8080;
		listen     localhost:8080;

4. **The fourth pair of examples specify a server listening for requests on the IP address 192.168.3.105. The first listens on port 80 and the second on port 8080.**

		listen     192.168.3.105:80;
		listen     192.168.3.105:8080;

5. **The fifth set of examples tell Nginx to listen on all domains and IP addresses on a specific port. listen 80; is equivalent to listen *:80;, and listen 8080; is equivalent to listen *:8080;.**

		listen     80;
		listen     *:80;
		listen     8080;
		listen     *:8080;

6. **Finally, the last set of examples instruct the server to listen for requests on port 80 for the IP addresses 12.34.56.77, 12.34.56.78, and 12.34.56.79.**

		listen     12.34.56.77:80;
		listen     12.34.56.78:80;
		listen     12.34.56.79:80;       


####Name-based virtual hosting####

*The server_name directive, which is located in the server block, lets the administrator provide name-based virtual hosting. This allows multiple domains to be served from a single IP address. The server decides which domain to serve based on the request header it receives (for example, when someone requests a particular URL).*

**Note**:Typically, you will want to create one file per domain you want to host on your server. Each file should have its own server block, and the server_name directive is where you specify which domain this file affects.

#####Common examples for the server_name directive:#####

1. **This example directs Nginx to process requests for example.com. This is the most basic configuration.**

		server_name   example.com;

2. **The second example instructs the server to process requests for both example.com and www.example.com.**

		server_name   example.com www.example.com;

3. **These two examples are equivalent. *.example.com and .example.com both instruct the server to process requests for all subdomains of example.com, including www.example.com, foo.example.com, etc.**

		server_name   *.example.com;
		server_name   .example.com;

4. **The fourth example instructs the server to process requests for all domain names beginning with example., including example.com, example.org, example.net, example.foo.com, etc.**

		server_name   example.*;

5. **The fifth example instructs the server to process requests for three different domain names. Note that any combination of domain names can be listed in a single server_name directive.**

		server_name   example.com isatio.com icann.org;

6. **Finally, if you set server_name to the empty quote set (”“), Nginx will process all requests that either do not have a hostname, or that have an unspecified hostname, such as requests for the IP address itself.**

		server_name   "";


####Access logs####

*The access_log directive can be set in the http block in nginx.conf or in the server block for a specific virtual domain. It sets the location of the Nginx access log. By defining the access log to a different path in each server block, you can sort the output specific to each virtual domain into its own file. An access_log directive defined in the http block can be used to log all access to a single file, or as a catch-all for access to virtual hosts that don’t define their own log files.*

#####Common examples for the access_log directive:#####

1. **You can use a path relative to the current directory:** */etc/nginx/nginx.conf*

		access_log logs/example.access.log;

2. **You can use a full path:** */etc/nginx/sites-available/example.com*

		access_log /srv/www/example.com/logs/access.log;

3. **You can also disable the access log, although this is not recommended:** */etc/nginx/nginx.conf*

		access_log off;

####Location####

*The location setting lets you configure how Nginx will respond to requests for resources within the server. Just like the server_name directive tells Nginx how to process requests for the domain, such as http://example.com, the location directive covers requests for specific files and folders.*

#####Common examples for the location directive:#####

*These first five examples are literal string matches, which match any part of an HTTP request that comes after the host segment. So, for example, if someone requests:*

		location / { } 
		location /images/ { } 
		location /blog/ { } 
		location /planet/ { } 
		location /planet/blog/ { }

**Request**: http://example.com/

**Returns**: Assuming that there is a server_name entry for example.com, the location / directive will determine what happens with this request.

**NOTE**:Nginx always fulfills request using the most specific match. So, for example:

**Request**: http://example.com/planet/blog/ or http://example.com/planet/blog/about/

**Returns**: This is fulfilled by the location /planet/blog/ setting because it is more specific, even though location /planet/ also matches this request.

*When a location directive is followed by a tilde (\~), Nginx performs a regular expression match. These matches are always case-sensitive. So, IndexPage.php would match the first example above, but indexpage.php would not. In the second example, the regular expression ^/BlogPlanet(/|index\.php)$ will match requests for /BlogPlanet/ and /BlogPlanet/index.php, but not /BlogPlanet, /blogplanet/, or /blogplanet/index.php. Nginx uses* [*Perl Compatible Regular Expressions*](http://perldoc.perl.org/perlre.html) *

		location ~ IndexPage\.php$ { }
		location ~ ^/BlogPlanet(/|/index\.php)$ { }

*If you want matches to be case-insensitive, use a tilde with an asterisk (\~*). The examples above all specify how Nginx should process requests that end in a particular file extension. In the first example, any file ending in: .pl, .PL, .cgi, .CGI, .perl, .Perl, .prl, and .PrL (among others) will match the request.*

		location ~* \.(pl|cgi|perl|prl)$ { }
		location ~* \.(md|mdwn|txt|mkdn)$ { }

*Adding a caret and tilde (\^\~) to your location directives tells Nginx, if it makes a match for a particular string, to stop searching for more specific matches and use the directives here instead. Other than that, these directives work like the literal string matches in the first group. So, even if there’s a more specific match later, if a request matches one of these directives, the settings here will be used. See below for more information about the order and priority of location directive processing.*

		location ^~ /images/IndexPage/ { }
		location ^~ /blog/BlogPlanet/ { }

*Finally, if you add an equals sign (=) to the location setting, this forces an exact match with the path requested and then stops searching for more specific matches. For instance, the final example will match only http://example.com/, not http://example.com/index.html. Using exact matches can speed up request times slightly, which can be useful if you have some requests that are particularly popular.*

		location = / { }

Directives are processed in the following order:

1. Exact string matches are processed first. If a match is found, Nginx stops searching and fulfills the request.
2. Remaining literal string directives are processed next. If Nginx encounters a match where the \^\~ argument is used, it stops here and fulfills the request. Otherwise, Nginx continues to process location directives.
3. All location directives with regular expressions (\~ and \~*) are processed. If a regular expression matches the request, Nginx stops searching and fulfills the request.
4. If no regular expressions match, the most specific literal string match is used.

####Location Root and Index####
####Best practice####
