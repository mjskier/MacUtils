# Running a web server on your Mac with docker

There are several options to run a webserver on your Mac. One is [MAMP](https://www.mamp.info)
In this document I'm recording the steps needed to run Apache with Docker

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

I like to think about images and containers in term of object oriented <.....>
An image is like a class, and a container is an instance of this class.

You can get a list of **running** containers with `docker ps`

```
mistral: docker ps

CONTAINER ID        IMAGE               COMMAND             CREATED                  STATUS              PORTS               NAMES
34a7a9bcfbf2        ubuntu              "/bin/bash"         Less than a second ago   Up 13 minutes                           sharp_bose
```

Now exit from your ubuntu root shell (if you haven't already done so) and run `docker ps` again. 
The resulting list should be empty.

The ubuntu container isn't running anymore, but it still exists on your machine.
To get a list of containers, runningor not, use `docker ps -a`

```
mistral: docker ps -a

CONTAINER ID        IMAGE               COMMAND             CREATED                  STATUS                     PORTS               NAMES
34a7a9bcfbf2        ubuntu              "/bin/bash"         Less than a second ago   Exited (0) 9 seconds ago                       sharp_bose
```
You could restart that container with `docker start 34a7a9bcfbf2`, and stop it again with `docker stop 34a7a9bcfbf2`
Finally, you can remove the container with `docker rm 34a7a9bcfbf2`

You can remove an image with `docker rmi <image ID>`. For example `docker rmi f975c5035748` for my ubuntu image listed above.


