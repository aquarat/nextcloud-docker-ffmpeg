# nextcloud-docker-ffmpeg
A personally optimised/customised Nextcloud Docker build script. It takes the normal Nextcloud Docker image and tacks on ffmpeg, raw previews, APCu, and REDIS linkage, among other things.

Build with :
docker build -t my-nextcloud:13 .

I use the resulting image like this :

# Start MariaDB                                                   
docker run --rm -v mariadbfolder:/var/lib/mysql -v some-extras:/extra --name nextcloud-mariadb --detach=false mariadb:10.3.3                                                                                                                       

# Start Nginx-proxy, then the Let's Encrypt Companion                            
docker run --rm -p 80:80 -p 443:443     --name nginx-proxy     -v nginx-stuff:/etc/nginx/conf.d/my_proxy.conf:ro -v some-certs:/etc/nginx/certs:ro     -v /etc/nginx/vhost.d     -v /usr/share/nginx/html     -v /var/run/docker.sock:/tmp/docker.sock:ro     --label com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy     jwilder/nginx-proxy                                    
docker run -v more-certs:/etc/nginx/certs:rw     -v /var/run/docker.sock:/var/run/docker.sock:ro     --volumes-from nginx-proxy     jrcs/letsencrypt-nginx-proxy-companion                                                                                              

# Start REDIS (this seems to really speed things up)                                                     
docker run --name nextcloud-redis -d redis                        

# Start Nextcloud                                                 
docker run -e "VIRTUAL_HOST=somedomain.com" -e "LETSENCRYPT_HOST=somedomain.com" -e "LETSENCRYPT_EMAIL=someaddress@somedomain.com" -v some-dir:/var/www/html --expose=80 --link nextcloud-mariadb:mysql --link nextcloud-redis:redis --name=nextcloud-container --detach=false --rm --sig-proxy=false my-nextcloud:13  
Not proxying signals prevents apache2 from dying when the terminal is resized [sigwinch], only useful if the container is attached.
Naturally running containers attached to a terminal is unnecessary and even abnormal.
--rm prevents the container's data persisting after the container is stopped (externally mounted volumes are unaffected).

Chunks of the Dockerfile were bastardised from jrottenberg's FFMpeg image : https://github.com/jrottenberg/ffmpeg
(Thank you jrottenberg!)
