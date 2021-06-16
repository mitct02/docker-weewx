
### Basic usage:

The most basic usage of this image is to simply map a volume to a local weewx.conf like this:

`docker run -d --volume /Users/tom/weewx.conf:/home/weewx/weewx.conf mitct02/weewx:4.3.0`

The problem with this configuration is that it keeps your generated html files (./public_html) in your running Docker container.
This is OK if you have rsync or something else set in `weewx.conf` up to export the html content out of your container.
To be able to serve the html up via a web server, all you need to do is mount a local directory as public_html like this:

``docker run -d --volume /Users/tom/weewx.conf:/home/weewx/weewx.conf --volume /var/www/html/weewx/public_html/:/home/weewx/public_html/ mitct02/weewx:4.3.0``

You can then run a web server locally that uses the volume you mapped as its html root.

Here is a more comprehensive example:

docker run -it --rm --volume /Users/tom/extensions:/home/weewx/extensions --volume /Users/tom/weewx.conf:/home/weewx/weewx.conf --volume /tmp/public_html:/home/weewx/public_html --volume /Users/tom/bin/user:/home/weewx/bin/user --volume /Users/tom/archive:/home/weewx/archive mitct02/weewx:4.3.0

This one maps volumes for extensions, bin/user, weewx.conf itself, public_html, and archive/ where the sqllite database is stored
Note that docker is called with the `-it` and `--rm` flags, which ensure that the container is killed when exited.

### Building a Child Image:

The more complex way of using this image, which is useful if you want to run the image
 elsewhere, like in Kubernetes, is to use this image as a base image and extend it to install plugins, etc.

Create a Dockerfile with at least:

`FROM mitct02/weewx:4.3.0` (use the version you prefer)

Insert the correct version after the `:`.

Make a conf directory at the root of your working directory (where Dockerfile is).

Copy your configs into separate directories under ~/conf. For example, you may
have a config for dev, test, and prod. Or you may have a config for each location
where you have a weather station, or some other scheme. Each separate weewx instance is in its own config subdir.

Make a keys directory at the root of your working directory (where Dockerfile is). This is where you will store the public keys to be used for rsync. The content of this directory will be copied to /root/.ssh/ and will allow weewx to copy your web content to another box. You need this even if you do not plan to use rsync (but it can be an empty directory).

Then run docker build -t my-weewx

Since the web content is small and transient, it is ok
  to mount a host directory to avoid the complexity of using a data
  volume (though it is still fine to do it that way). You can also run a web server
  container alongside weewx using something like docker-compose or Kubernetes.

To mount a host volume in /tmp to hold the html and images:

docker run -d --name=weewx-webserver --restart=always -p 80:80 -v
/tmp/httpdroot:/usr/local/apache2/htdocs httpd

docker run -d --name=weewx-default --restart=always -e CONF=default -v /tmp/httpdroot:/home/weewx/public_html my-weewx

### Docker Compose Sample (with optional user defined driver)

Replace /mnt/docker/weewx with your storage location. If you are using the gw1000 driver from here: https://github.com/gjr80/weewx-gw1000 ,put the gw1000.py file, https://raw.githubusercontent.com/gjr80/weewx-gw1000/master/bin/user/gw1000.py, into your weewx volume on the host (see example below).

```
version: '2.3'
services:
  weewx:
    image: mitct02/weewx
    container_name: weewx
    volumes:
      - /mnt/docker/weewx/weewx.sbd:/home/weewx/weewx.sbd
      - /mnt/docker/weewx/public_html:/home/weewx/public_html 
      - /mnt/docker/weewx/weewx.conf:/home/weewx/weewx.conf
#      - /mnt/docker/weewx/gw1000.py:/home/weewx/bin/user/gw1000.py #Optional
    networks:
      - internal
    restart: unless-stopped
```
