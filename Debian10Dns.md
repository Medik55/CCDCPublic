### CCDC DNS Files:

##### Global Setup: 
`cat named.conf.options`
```bash  
options {
    directory "/var/cache/bind";

    forwarders {
        8.8.8.8;  // Google DNS
        8.8.4.4;  // Google DNS
    };

    querylog yes;
    allow-query { 
	    localhost; 
	    172.20.240.0/24; // Internal Zone
	    172.20.241.0/24; // Public Zone
	    172.20.242.0/24; // User Zone
	};

    recursion yes; // allow queries from higher DNS servers
    dnssec-validation auto;
    auth-nxdomain no;
    listen-on { any; };
};

```
##### Local Setup and CCDC DNS Zones:
`cat /etc/bind/named.conf.local`
```bash  
zone "ccdcteam01.internal" {  
    type master;  
    file "/etc/bind/db.ccdcteam01.internal";  
};

zone "ccdcteam01.user" {  
    type master;  
    file "/etc/bind/db.ccdcteam01.user";  
};

zone "ccdcteam01.public" {  
    type master;  
    file "/etc/bind/db.ccdcteam01.public";  
};

// Reverse DNS Lookup Zones (best practice)
zone "0.240.20.172.in-addr.arpa" {
    type master;
    file "/etc/bind/db.172.20.240";
};

zone "0.241.20.172.in-addr.arpa" {
    type master;
    file "/etc/bind/db.172.20.241";
};

zone "0.242.20.172.in-addr.arpa" {
    type master;
    file "/etc/bind/db.172.20.242";
};

```

##### Internal `/etc/bind/db.ccdcteam01.internal`
Standard DNS
```bash
$TTL 86400
@   IN  SOA ns1.ccdcteam01.internal. admin.ccdcteam01.internal. (
            01        ; Serial
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            86400 )   ; Negative Cache TTL
;
@       IN  NS      ns1.ccdcteam01.internal.

ns1     IN  A       172.20.240.20    ; Current DNS server
docker  IN  A       172.20.240.10    ; Docker server
```
Reverse DNS `/etc/bind/db.172.20.240`
```bash
$TTL 86400
@   IN  SOA ns1.ccdcteam01.internal. admin.ccdcteam01.internal. (
            01        ; Serial
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            86400 )   ; Negative Cache TTL
;
@       IN  NS      ns1.ccdcteam01.internal.

10      IN  PTR     docker.ccdcteam01.internal.
20      IN  PTR     ns1.ccdcteam01.internal.
```
##### User `/etc/bind/db.ccdcteam01.user`
Standard DNS
```bash
$TTL 86400
@   IN  SOA ns1.ccdcteam01.user. admin.ccdcteam01.user. (
            01        ; Serial
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            86400 )   ; Negative Cache TTL
;
@       IN  NS      ns1.ccdcteam01.user.

ns1         IN  A   172.20.242.10    ; Ubuntu server acting as DNS
ad-server   IN  A   172.20.242.200   ; Active Directory server
workstation IN  A   172.20.242.210   ; User workstation
```
Reverse DNS `/etc/bind/db.172.20.242`
```
$TTL 86400
@   IN  SOA ns1.ccdcteam01.user. admin.ccdcteam01.user. (
            1         ; Serial
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            86400 )   ; Negative Cache TTL
;
@       IN  NS      ns1.ccdcteam01.user.

10      IN  PTR     ns1.ccdcteam01.user.
200     IN  PTR     ad-server.ccdcteam01.user.
210     IN  PTR     workstation.ccdcteam01.user.
```
##### Public `/etc/bind/db.ccdcteam01.public`
Standard DNS
```bash
$TTL 86400
@   IN  SOA ns1.ccdcteam01.public. admin.ccdcteam01.public. (
            1         ; Serial
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            86400 )   ; Negative Cache TTL
;
@       IN  NS      ns1.ccdcteam01.public.

ns1        IN  A   172.20.241.20     ; Splunk server
splunk     IN  A   172.20.241.20     ; Splunk server
ecomm      IN  A   172.20.241.30     ; E-Comm server
webmail    IN  A   172.20.241.40     ; WebMail server
```
Reverse DNS `/etc/bind/db.172.20.241`
```
$TTL 86400
@   IN  SOA ns1.ccdcteam01.public. admin.ccdcteam01.public. (
            1         ; Serial
            604800    ; Refresh
            86400     ; Retry
            2419200   ; Expire
            86400 )   ; Negative Cache TTL
;
@       IN  NS      ns1.ccdcteam01.public.

20      IN  PTR     splunk.ccdcteam01.public.
30      IN  PTR     ecomm.ccdcteam01.public.
40      IN  PTR     webmail.ccdcteam01.public.

```

#### Check and Apply
```bash
# Check Bind Configuration
sudo named-checkconf

# Check Zone Files
sudo named-checkzone ccdcteam01.internal /etc/bind/db.ccdcteam01.internal
sudo named-checkzone ccdcteam01.user /etc/bind/db.ccdcteam01.user
sudo named-checkzone ccdcteam01.public /etc/bind/db.ccdcteam01.public

sudo named-checkzone 240.20.172.in-addr.arpa /etc/bind/db.172.20.240
sudo named-checkzone 242.20.172.in-addr.arpa /etc/bind/db.172.20.242
sudo named-checkzone 241.20.172.in-addr.arpa /etc/bind/db.172.20.241

# restart Bind Service
sudo systemctl restart bind9

# Test DNS Forward Lookup
dig @localhost ns1.ccdcteam01.internal
dig @localhost ad-server.ccdcteam01.user
dig @localhost webmail.ccdcteam01.public

# Test Reverse DNS Lookup
dig @localhost -x 172.20.240.10
dig @localhost -x 172.20.242.200
dig @localhost -x 172.20.241.40

```