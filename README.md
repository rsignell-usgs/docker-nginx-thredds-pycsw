#  Running THREDDS, pycsw, ERDDAP, ncWMS and TerriaMap via Docker

1. Setup a ubuntu or centos machine with 16GB+ RAM.  I used the NSF XSEDE Jetstream Atmosphere Interface to create an Ubuntu 16_04 m1.medium instance (CPU:6, Mem: 16GB, Disk: 60GB).

2. Get Rich Signell's Docker configuration for pycsw, thredds, erddap and nginx (with Let's Encrypt):
```
cd ~/github
git clone https://github.com/rsignell-usgs/docker-nginx-thredds-pycsw.git
mv docker-nginx-thredds-pycsw /opt/docker
```

3. Run Julien Chastang's nice script for installing Docker and Docker-compose, and also adds your username to the docker group.
Note: this docker install script logs you off so that changes can take effect, so you need to log back in.

```
cd /opt/docker 
chmod +x docker-install.sh
./docker-install.sh -u rsignell
```
4. Edit the let's encrypt script `do_get`, replacing the `CERTS` and `EMAIL` with your settings.
```
cd /opt/docker/nginx
vi do_get
```
5. Run the script to get your certificates.  
```
chmod +x do_get
./do_get
```
6. Stop the running docker container `CTRL-Z`, then remove the docker container `docker rm -f cert`

7. Grep for all the places to change the domain name
```
cd /opt/docker
grep 'js-169-194.jetstream-cloud.org' -r *
```
and edit those, specifying your endpoint (e.g. "js-169-102.jetstream-cloud.org") 

8. get your user id and the docker group id:
```
id
```
and edit the `thredds.env` file, setting `TOMCAT_USER_ID` to your user id (`uid`), and `TOMCAT_GROUP_ID` to the docker group id.

9. Edit docker-compose.yml and make sure it looks okay.
```
cd /opt/docker
vi docker-compose.yml
```

10. Create a data directory with a sample netcdf file
```
mkdir /data
cd /data
wget http://geoport.whoi.edu/thredds/fileServer/examples/bora_feb.nc
```
11. Fire up the containers. 
```
cd /opt/docker
docker-compose up -d
```
12. login to the pycsw container and run `pycsw_setup`:
```
docker exec -it pycsw bash
pycsw_setup
exit
```
13. Grep for all the tomcat-user passwords to change:
```

cd /opt/docker
grep 'changeme' -r *
```
and edit the `tomcat-users.xml` files to enter your SHA1 passwords (can use http://www.sha1-online.com/ to generate)

14. Bounce the Docker containers
```
cd /opt/docker
docker-compose down
docker-compose up -d
```

