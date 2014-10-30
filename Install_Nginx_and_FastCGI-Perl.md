Install Nginx and FastCGI-Perl
==============================

Set the Hostname
----------------

		echo "new_hostname" > /etc/hostname
		hostname -F /etc/hostname

If it exists, edit the file /etc/default/dhcpcd to comment out the SET_HOSTNAME directive:

**In File**: /etc/default/dhcpcd

   		#SET_HOSTNAME='yes'

Issue the following commands to make sure it is set properly:

		hostname
		hostname -f

Install Required Packages
-------------------------

		apt-get update
		apt-get upgrade
		apt-get install nginx fcgiwrap spawn-fcgi

Configure DNS
-------------

For this setup, we'll use 2 servers and the program bind9 to do so.

Install Bind:

		sudo apt-get install bind9

There are many conf files to modify, those 2 are the most important ones:

- named.conf.local: has the list of the domain names handled by the DNS server.
- db.mywebsite.com: has the definition of the domain name and the under-domains( meaning the IP adresses binded to our sites)

You can find them in the /etc/bind/ directory

		nano /etc/bind/named.conf.local

Go ahead and add this line:

		zone "mywebsite.com" {
		        type master;
		        allow-transfer {187.10.78.17;} ;
		        file "/etc/bind/db.mywebsite.com";
		};

**Type**: *master* indicates it's the main server.

**Allow-transfer**: indicates the IP addresses of the sub-servers having access to the main server's files.

**File**: Indicates the name of the file containing the detailed definition of the domain and sub-domains.

**NOTE**: For the sub-servers, you have to change the type to *slave* and the *allow-transfer {IP address}* to *masters{master's IP address}*.

Then open or create the file detailing the domain name:

		nano /etc/bind/db.mywebsite.com

This is a complex file, modify it with precaution:

		; TTL (Time To Live)
		$TTL        604800

		; General Information
		@        IN        SOA        mywebsite.com. root.mywebsite.com. (
		                              2                ; Serial
		                         604800                ; Refresh
		                          86400                ; Retry
		                        2419200                ; Expire
		                         604800 )              ; Negative Cache TTL
		;

		; domai, sub-domain registering and associated IP's adresses
		@       10800 IN      A         214.21.17.22
		@       10800 IN      MX 10     aspmx.l.google.com. 
		@       10800 IN      MX 20     aspmx.2.google.com. 
		uploads 10800 IN      A         92.143.23.113 
		dev     10800 IN      A         92.13.88.102
		www     10800 IN      A         92.13.88.105

There are 3 different sections to this file:

1. **TTL**: Indicates the time the other servers can keep in their cache the information coming from the main server (in seconds)

2. **General Information**:
	1. test

3. **Registering of the domain**

