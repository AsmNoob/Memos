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

		; domai, sub-domain recording and associated IP's adresses
		@       10800 IN      A         214.21.17.22
		@       10800 IN      MX 10     aspmx.l.google.com. 
		@       10800 IN      MX 20     aspmx.2.google.com. 
		uploads 10800 IN      A         92.143.23.113 
		dev     10800 IN      A         92.13.88.102
		www     10800 IN      A         92.13.88.105

There are 3 different sections to this file:

1. **TTL**: Indicates the time the other servers can keep in their cache the information coming from the main server (in seconds)

2. **General Information**:
	1. **Serial**: Everytime you modificate this file, you have to increment the variable to indicate the file has changed.
	2. **Refresh**: Indicates after how long the slave servers have to refresh their cache.
	3. **Retry**: Indicates after how long the slave servers will try to access the main server in case of failure.
	4. **Expire**: Indicates after how long the slave servers will stop trying to access the main server.
	5. **Negative Cache TTL**: Minimal living time of the recording cache of the other recording below this one.

3. **Recording of the domain**

		Domain TTL  IN      Type      Value

		@       10800 IN      A         214.21.17.22

*Every line defines a new recording*:

There are 5 informations needed for every recording:

1. **Domain**: indicates the domain or sub-domain concerned ("@" : base domain = mywebsite.com, "www" = www.mywebsite.com)
2. **TTL**: time to live for this recording, it means the other sub-domains will keep this recording in their cache for so long.
3. **Class**: For internet it's always "IN"
4. **Type of recording**:
	1. **A**: Base type IPv4, it's used to established the bond between a domain and an IP address.
	2. **AAAA**: Same as A type but for IPv6 addresses.
	3. **CNAME**: Allows to create aliases for domain names(Use the FQDN).
	4. **NS**: Delegation of the domain's management.
	5. **MX**: Mail servers, for people sending mails to name@youwebsite.com. Usually there are 2 types used at least "MX10" and "MX20" so mails with different priorities are classified correctly and in case of server failure the mail service will still work.
	6. **TXT**: Allows you to insert a String as an answer.
5. **The value**: It's the address defining the recording, usually an IP address.



After all that, restart it to load the new changes:

		sudo /etc/init.d/bind9 restart

