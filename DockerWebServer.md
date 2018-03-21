# Running a web server on your Mac with docker

There are several options to run a webserver on your Mac. One is [MAMP](https://www.mamp.info)

In this document I'm recording the steps needed to run Apache with Docker

Most of the info in this document comes from the excellent series of youtube videos by [Peter Fisher].
(https://www.youtube.com/playlist?list=PLZdsdjcJ44WU_cY2Y1LFLnmsSjFD5BZLZ)

## Install Docker

Installing docker on your mac is [documented here](https://docs.docker.com/docker-for-mac/install)

## Decide what version of Linux you want to use

There are several Linux images you could use.

For example [Alpine](https://hub.docker.com/_/alpine) is a very lightweight image that only takes 5MB.

I'm more familiar with Debian based distributions so I will use [Ubuntu](https://hub.docker.com/_/ubuntu) here.

If you are more familair with RPM based distributions, [CentOS](https://hub.docker.com/_/centos) 
might be a good choice for you 

## Download the unbutu image, and test it

`docker pull ubuntu`

You can get a list of images installed on your machine with `docker images`. You should see your ubuntu image.
Note that you can have multiple ubuntu images. So while it is very easy to refer to an image by its name,
it is probably a better idea to refer to them by their unique ID.

Notice the difference in size between the **ubuntu** and the **alpine** images.

```
mistral: docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

ubuntu              latest              f975c5035748        2 weeks ago         112MB
ubuntu              <none>              0458a4468cbc        7 weeks ago         112MB
alpine              latest              3fd9065eaf02        2 months ago        4.15MB
ubuntu              14.04               c69811d4e993        7 months ago        188MB
ncareol/soloii      latest              baa1ee4c3541        9 months ago        570MB
```

See if you can run the image with the following command. If everything is fine, you should get a prompt that
looks similar to this: **root@34a7a9bcfbf2:/#**

`docker run -it ubuntu /bin/bash`

If things worked as expected, you are now root in a ubuntu container running on your Mac

## Images vs. Containers

I like to think about images and containers in term of object oriented terminology:
An image is like a class, and a container is an instance of this class.

You can get a list of **running** containers with `docker ps`
Try it from a different terminal (not the one running your ubuntu container)

```
mistral: docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED                  STATUS              PORTS               NAMES
34a7a9bcfbf2        ubuntu              "/bin/bash"         Less than a second ago   Up 13 minutes                           sharp_bose
```

Now exit from your ubuntu root shell (if you haven't already done so) and run `docker ps` again. 
The resulting list should be empty.

The ubuntu container isn't running anymore, but it still exists on your machine.
To get a list of containers, running or not, use `docker ps -a`

```
mistral: docker ps -a

CONTAINER ID        IMAGE               COMMAND             CREATED                  STATUS                     PORTS               NAMES
34a7a9bcfbf2        ubuntu              "/bin/bash"         Less than a second ago   Exited (0) 9 seconds ago                       sharp_bose
```
You could restart that container with `docker start 34a7a9bcfbf2`, and stop it again with `docker stop 34a7a9bcfbf2`
Finally, you can remove the container with `docker rm 34a7a9bcfbf2`

You can remove an image with `docker rmi <image ID>`. For example `docker rmi f975c5035748` for my ubuntu image listed above.

## Time to install Apache (or nginx, or ...)

First, start the container you previously exited and get a prompt back. Let's assume its ID is 34a7a9bcfbf2

```
docker start 34a7a9bcfbf2
docker exec -it 34a7a9bcfbf2 /bin/bash
```

That should get you back to your root prompt. Now we can install apache2.

```
apt_get update
apt-get install apache2
```
The configuration for apache is under /etc/apache2. 
By default it has a virtual host listening to port 80 with a document root at /var/www/html

## Create a new image with the changes we have made so far

Time to exit your container and commit the changes we have made to a new image. Make sure you do exit as saving a running dontainer can create problems.

Get the container ID with `docker ps -a`. We can now commit the changes to a new image

`docker commit 34a7a9bcfbf2 ubuntu_apache2`

Get a list of images, you should now see the newly created ubuntu_apache2 (or whateveryou decide to call it)

docker images

```
mistral: docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu_apache2      latest              4b5eefbe9c6c        2 hours ago         251MB
ubuntu              latest              f975c5035748        2 weeks ago         112MB
```
## Decide where you want to edit your web content

You could edit your web content in the container, but chances are you have better tools on your Mac. So let's decide where your document root will be on your machine, and map that to /var/www/html in the container.

For this example, I will use ~/web_stuff

```
cd ~
mkdir web_stuff
cd web_stuff
```
Now you can use your favorite editor to create index.html. (No, /bin/cat isn't my favorite editor :-) )

```
cat > index.html
<!DOCTYPE html>
<html>
  <body>
    <h1>This is a page from inside my container</h1>

    <p>This is a paragraph.</p>
    <p>This is another paragraph.</p>

  </body>
</html>
```

## Start the new image, and map ports, volumes, ...

We are done with setup. All we need to do now is to run a container of the new image, with apache running

**Warning** The following command is one line.

`docker run -p 8080:80 --name web1 -h powder.local -v ~/web_stuff:/var/www/html  ubuntu_apache2 /usr/sbin/apache2ctl -D FOREGROUND`

A few parameters of interest:
* -p8080:80

This maps port 8080 on your mac to port 80 in your container.

* -name web1

This is the name of the container. Not necessary, but easier to deal with than a random number.

* -h powder.local

This is your hostname. I haven't investigated what it means to name your hosname yet, but that prevents a warning from apache.

* -v ~/web_stuff:/var/www/html

This "mounts" your ~/web_stuff directory on /var/www/html in your container. /var/www/html is configured as the document root for your apache server. So when we browse to localhost:8080 we should get out index.html page.

* /usr/sbin/apache2ctl -D FOREGROUND

This starts the apache2 service in the container. We won't get a prompt, so if you need to stop the containier do `docker stop web1`

* no -ti

Note that we didn't use the -t and -i options here. We don't want an interactive container that will exit as soon as the apache2ctl command is done.

## Try it out

Open your browser and go to [localhost:8080](localhost:8080). You should see the content of the index.html we created before.

You should be able to edit the index.html file, then reload the page and see the changes.


## Stop it

Since we didn't run an interactive container, we can't stop it by exiting, or CTRL_C. 
To stop the container we need to do this instead

`docker stop web1`

## restart it

We unfortunately cannot restart the container with `docker restart` since the web server won't be started that way.
So the simplest way to restart is to remove the old one `docker rm web1` (docker won't let you start a new container with the same name as an existing one) and use the `docker run` command again.


