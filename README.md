# Practice DNS sistema.sol #
## Introduction ##

First, we create two virtual machines, one which will be the master and one which will be the slave. We need to configure the static IPs of the servers.

The objective of this practice is that the two machines that we have created previously act as DNS servers with a domain called "sistema.sol". To do this, we will use Vagrant which will allow us to automate the configuration.

Then, we install bing9 on both machines.


## Configuration ##

The first thing would be to configure both servers (earth and venus) to be DNS by default. To do this we will configure the file `/etc/resolv.conf` in both servers.

* ##### /etc/resolv.conf #####

```conf
nameserver 192.168.57.103
nameserver 192.168.57.102
```

Now, we must establish trust networks, activate or deactivate recursion... To do this, we must configure the file `named.conf.options`.


* ##### named.conf.options #####


```conf
acl confiables {
        192.168.57.0/24;
        127.0.0.0/8;
};

options {
        directory "/var/lib/bind";


        forwarders {
            208.67.222.222;
        };

        allow-transfer { 192.168.57.103; };
        listen-on port 53 { 192.168.57.102; };

        recursion yes;
        allow-recursion { confiables; };


        dnssec-validation yes;

        // listen-on-v6 { any; };
};
```

Now, we will establish the role of each server and we must define the zones, where the files will be stored.  To do this, we will use the `named.conf.local` file.

* ##### named.conf.local #####

First, we configurate the master server

**Master**

```conf
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "sistema.sol" {
        type master;
        file "/var/lib/bind/db.sistema.sol";
};

zone "57.168.192.in-addr.arpa" {
        type master;
        file "/var/lib/bind/sistema.sol.rev";
};
```
Then, we configurate the slave server

**Slave**

```conf
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "sistema.sol" {
        type slave;
        file "/var/lib/bind/db.sistema.sol";
        masters {
                 192.168.57.102;
        };
};

zone "57.168.192.in-addr.arpa" {
        type slave;
        file "/var/lib/bind/sistema.sol.rev";
        masters {
                 192.168.57.102;
        };
};
```

Once this is configured, we should reboot to finish applying the changes.

### ZONES CONFIGURATION ###

#### FORWARD ZONE (/var/lib/bind/sistema.sol.dns) ####

The zone configuration file has been stored in `/var/lib/bind/sistema.sol.dns` as we have indicated in the file `named.conf.local`.

* (We will use the file "/etc/bind/db.empty" as template)

* ##### /var/lib/bind/sistema.sol.dns #####

```conf
;
; zone: sistema.sol.
;

$TTL	86400
@	IN	SOA	tierra.sistema.sol. admin.sistema.sol. (
			      1		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			  7200 )	; Negative Cache TTL
;

; RESOLVE RECORDS (RR)
; name servers (NS)
@			IN	NS	tierra.sistema.sol.
@			IN	NS	venus.sistema.sol.

; host addresses (A)
tierra.sistema.sol.	IN	A	192.168.57.103
mercurio.sistema.sol.	IN	A	192.168.57.101
venus.sistema.sol.	IN	A	192.168.57.102
marte.sistema.sol.	IN	A	192.168.57.104

; mail servers (MX)
sistema.sol.		IN	MX 10	marte.sistema.sol.

; aliases
ns1.sistema.sol.	IN	CNAME	tierra.sistema.sol.
ns2.sistema.sol.	IN	CNAME	venus.sistema.sol.
mail.sistema.sol.	IN	CNAME	marte.sistema.sol.
```

#### REVERSE ZONE (/var/lib/bind/sistema.sol.rev) ####

* ##### /var/lib/bind/sistema.sol.rev #####

```conf
;
; 57.168.192
$TTL    86400
@       IN      SOA     tierra.sistema.sol. admin.deaw.es. (
                                1       ;   Serial
                                3600    ;   Refresh
                                1800    ;   Retry
                                604800  ;   Expire
                                86400 )  ;   Negative Cache TTL
;
@       IN      NS      tierra.sistema.sol.
102     IN      PTR     tierra.sistema.sol.
103     IN      PTR     venus.sistema.sol.
104     IN      PTR     marte.sistema.sol.
```
