# Assignement 1: Manage multiple containers

docker container run --publish 80:80 --detach --name nginx nginx
docker container run --publish 8080:80 --detach --name httpd httpd
docker container run --publish 3306:3306 --detach --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql

## Commands to check

docker container ls
docker container ps --all 
docker container logs mysql --tail all
docker container rm {{ container_name }

Start up a NEW image and run the `bash` command shell :
docker container run --tty --interactive --name proxy nginx bash

Get into an EXISTING running image:
docker container exec -it mysql bash

Fix: use `nginx:alpine` instead of `nginx` in upcoming labs.


## Docker Networks: CLI Management

show networks: docker network ls
Inspect networks: docker network inspect
Create a network: docker network create --driver
Attach network to a container: docker network connect
Detach network from a container: docker network disconnect

Networks:
 * bridge - brindging through Docker proxy firewall
 * host - bridge directly to netowork host is on
 * none : Only on the host, never leaves

## Docker Networks: DNS

Docker has built-in "DNS Naming"

When container "new_nginx" and "my_nginx" are build on same network, then ping by hostname will resolve each properly.

### My hint for simpler output
docker network inspect --format '{{json  .Containers }}'  my_app_net | jq 

# Assignement 2: CLI App Testing

Get the `curl` version from `ubuntu:14.04` and `centos:7` using `-it`; then try `docker container --rm`.

CentOS: `docker container run -it --name centos_7 centos:7`
Result:

```
[root@c05bc7ff712e /]# curl --version
curl 7.29.0 (x86_64-redhat-linux-gnu) libcurl/7.29.0 NSS/3.44 zlib/1.2.7 libidn/1.28 libssh2/1.8.0
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smtp smtps telnet tftp 
Features: AsynchDNS GSS-Negotiate IDN IPv6 Largefile NTLM NTLM_WB SSL libz unix-sockets 
```

Ubuntu: `docker container run -it --name ubuntu_14 ubuntu:14.04`
Result:

```
root@ba7eb70ae8b0:/# curl --version
curl 7.35.0 (x86_64-pc-linux-gnu) libcurl/7.35.0 OpenSSL/1.0.1f zlib/1.2.8 libidn/1.28 librtmp/2.3
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtmp rtsp smtp smtps telnet tftp 
Features: AsynchDNS GSS-Negotiate IDN IPv6 Largefile NTLM NTLM_WB SSL libz TLS-SRP 
```

### My hint: Use Red Hat or other registries:

Use this to search the remote registry (non "docker.io"):
    `docker search registry.access.redhat.com/ubi`

And use this to pull down and use a specific one:
    `docker container run -it --name rhubi_8 registry.access.redhat.com/ubi8/ubi:latest`

Red Hat 7: `docker container run -it --name rhubi_7 registry.access.redhat.com/ubi7/ubi:latest`
Result:

```
[root@fe07b67853cc /]# curl --version
curl 7.29.0 (x86_64-redhat-linux-gnu) libcurl/7.29.0 NSS/3.90 zlib/1.2.7 libidn/1.28 libssh2/1.8.0
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smtp smtps telnet tftp 
Features: AsynchDNS GSS-Negotiate IDN IPv6 Largefile NTLM NTLM_WB SSL libz unix-sockets 
```

Red Hat 8: `docker container run -it --name rhubi_8 registry.access.redhat.com/ubi8/ubi:latest`
Result:

```
[root@edca9f242b31 /]# curl --version
curl 7.61.1 (x86_64-redhat-linux-gnu) libcurl/7.61.1 OpenSSL/1.1.1k zlib/1.2.11 brotli/1.0.6 libidn2/2.2.0 libpsl/0.20.2 (+libidn2/2.2.0) libssh/0.9.6/openssl/zlib nghttp2/1.33.0
Release-Date: 2018-09-05
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smb smbs smtp smtps telnet tftp 
Features: AsynchDNS IDN IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz brotli TLS-SRP HTTP2 UnixSockets HTTPS-proxy PSL 
```

Using `docker container run --rm -it registry.access.redhat.com/ubi8/ubi:latest`

# Assignement : DNS Round Robin Test

Fixup:
 * elasticsearch newer versions require environment variable changes to work correctly.
 * `centos` is no longer a supported image by Red Hat, so we can use `rockylinux:9` as a drop-in replacement for `centos`, or `alpine` for Linux easy access to utilities like nslookup and curl.

Research the `--network-alias search` for `docker run` - it gives them an additional DNS name to respond to.

 - Run `alpine nslookup search` with `--net` to see the two containers list for the same DNS anme
 - Run `centos curl -s search:9200` with `--net` multiple times until see both "name" fields show

docker network create search_net
docker network list
* Spin up four systems that will listen on :9200 and report their "name"
for X in 1 2 3 4 ; do
  docker container run -d --net search_net --network-alias search --name es_${X} elasticsearch:2
done
docker container ps -a

* Spin up another image to check the DNS in this network
docker container run -it --rm --net search_net registry.access.redhat.com/ubi8/ubi:latest bash
* Inside the "ubi" container
  * Install the jq tool
  dnf -y install jq
  * Run the "curl" command 25 times:
  for X in $(seq 1 25) ; do curl -s search:9200 | jq .name ; done

NOTE: The output of the for/curl loop above should show multiple "name" lines, roughly different.  These names are the ones that the `elasticsearch:2` auto generated.


Or you can run:

    dnf -y install jq procps
    watch -n 1 '(for X in $(seq 1 25) ; do curl -s search:9200 | jq .name ; done ) | sort'

Shows the 25 responses in a bit easier to see sorted layout.

# Container Images

