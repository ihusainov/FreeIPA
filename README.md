# FreeIPA-settings

**Install as docker container**

```bash
cd /opt
git clone https://github.com/freeipa/freeipa-container.git
cd freeipa-container
docker build -t freeipa-server -f Dockerfile .
mkdir /var/lib/ipa-data
```

**Next run configure ipa server, your need install bind dns server**

```bash
docker run --name freeipa-server -ti --sysctl net.ipv6.conf.all.disable_ipv6=0 -h ipa.mydomain.local --read-only -v /dev/urandom:/dev/random:ro -v /sys/fs/cgroup:/sys/fs/cgroup:ro -v /var/lib/ipa-data:/data freeipa-server ipa-server-install
```

**If all right you will see**
```bash
==============================================================================
Setup complete

Next steps:
        1. You must make sure these network ports are open:
                TCP Ports:
                  * 80, 443: HTTP/HTTPS
                  * 389, 636: LDAP/LDAPS
                  * 88, 464: kerberos
                  * 53: bind
                UDP Ports:
                  * 88, 464: kerberos
                  * 53: bind
                  * 123: ntp

        2. You can now obtain a kerberos ticket using the command: 'kinit admin'
           This ticket will allow you to use the IPA tools (e.g., ipa user-add)
           and the web user interface.
        3. Kerberos requires time synchronization between clients
           and servers for correct operation. You should consider enabling chronyd.

Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
The ipa-server-install command was successful

```

**Next run container and start work, where i have 2 ip in my server and i want to bind freeipa only to "192.168.0.15"**
 
```bash
docker run --name freeipa-server-domain --cap-add NET_ADMIN --sysctl net.ipv6.conf.all.disable_ipv6=0 -v /dev/urandom:/dev/random:ro -v /sys/fs/cgroup:/sys/fs/cgroup:ro -v /var/lib/ipa-data:/data -h ipa.mydomain.local -p 192.168.0.15:53:53/udp -p 192.168.0.15:53:53  -p 192.168.0.15:80:80 -p 192.168.0.15:443:443 -p 192.168.0.15:389:389 -p 192.168.0.15:636:636 -p 192.168.0.15:88:88 -p 192.168.0.15:464:464   -p 192.168.0.15:88:88/udp -p 192.168.0.15:464:464/udp -p 192.168.0.15:123:123/udp -p 192.168.0.15:7389:7389   -p 192.168.0.15:9443:9443 -p 192.168.0.15:9444:9444 -p 192.168.0.15:9445:9445  -e DEBUG_NO_EXIT=1 -d freeipa-server

docker ps
```

**Next go to web config, we need this url to access a FreeIPA**
```url 
https://ipa.mydomain.local/ipa/ui
```

**Congratulations!!!**

----
**Client install**
----

**Next we need connect client linux debian server to this Freeipa server, connect over ssh to the debian server**

```bash
apt install freeipa-client
```

**Next join to domain**
```bash
vim /etc/resolv.conf

domain mydomain.local
search mydomain.local
nameserver 192.168.0.15
```

***join to domain***

```bash
/usr/sbin/ipa-client-install


Discovery was successful!
Client hostname: vm-test.mydomain.local
Realm: MYDOMAIN.LOCAL
DNS Domain: mydomain.local
IPA Server: ipa.mydomain.local
BaseDN: dc=mydomain,dc=local

Continue to configure the system with these values? [no]: yes
Using default chrony configuration.
Attempting to sync time with chronyc.
Time synchronization was successful.

User authorized to enroll computers: admin
Password for admin@MYDOMAIN.LOCAL:

Successfully retrieved CA cert
    Subject:     CN=Certificate Authority,O=MYDOMAIN.LOCAL
    Issuer:      CN=Certificate Authority,O=MYDOMAIN.LOCAL
    Valid From:  2021-07-02 20:21:34
    Valid Until: 2041-07-02 20:21:34

Enrolled in IPA realm MYDOMAIN.LOCAL
Created /etc/ipa/default.conf
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
Configured /etc/krb5.conf for IPA realm MYDOMAIN.LOCAL
```

**Congratulations!!!**
