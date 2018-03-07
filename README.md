# docker-template
### template for running ERDDAP, THREDDS, NGINX docker containers

1. Setup a ubuntu or centos machine with plenty of RAM.  I used the NSF XSEDE Jetstream Atmosphere Interface to create an Ubuntu 16_04 m1.medium instance (CPU:6, Mem: 16GB, Disk: 60GB).

2. Ssh to the machine and run Julien's nice script for installing Docker and Docker-compose, and also adds your username to the docker group.
```
mkdir github
git clone https://github.com/Unidata/xsede-jetstream.git
cd xsede-jetstream
chmod +x docker-install.sh
./docker-install.sh -u rsignell
```
Note: the docker install script logs you off so that changes can take effect, so you need to log back in.

5. Get Rich Signell's Docker configuration for pycsw, thredds, erddap and nginx (with Let's Encrypt):
```
cd ~/github
git clone https://github.com/rsignell-usgs/docker-template.git
mv docker-template /opt/docker
```
6. Edit the Let's encrypt script, replacing the [CERTS line](https://github.com/rsignell-usgs/docker-template/blob/master/nginx/do_get#L2) with your endpoint
```
cd /opt/docker
cd nginx
chmod +x do_get
vi do_get
```
7. After modifying the CERTS and EMAIL lines in do_get, run it:
```
./do_get
```
8. Grep for all the places to change the domain name
```
cd /opt/docker
grep 'do.maltlab.com' -r *
```
and edit those, specifying your endpoint (e.g. "") 

9. Edit docker-compose.yml and make sure it looks okay.
```
cd /opt/docker
vi docker-compose.yml
```
10. Fire up the containers. 
```
cd /opt/docker
docker-compose up -d
```
11. login to the pycsw container and run `pycsw_setup`:
```
docker exec -it pycsw bash
pycsw_setup
exit
```
12. Grep for all the tomcat-user passwords to change:
```
cd /opt/docker
grep 'changeme' -r *
```
and edit the `tomcat-users.xml` files to enter your SHA1 passwords (can use http://www.sha1-online.com/ to generate)

13. Bounce the Docker containers
```
cd /opt/docker
docker-compose down
docker-compose up -d
```


