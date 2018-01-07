# Notes on Docker Images available for Oracle

## Overview

Hi, Welcome. This is just an area where I plan to jot down notes I've learned using Docker specifically with some of the available images out there - specifically Oracle Database images.

There are a few ways you may be building your image:

1. You own install scripts
2. From container-registry.oracle.com
3. From the sample Docker files hosted on GitHub

In all cases, my purposes are solely for development and/or testing. 

## Available options

### Your own installation scripts

The first method, your own install scripts - I mainly use this for Oracle XE, because, well the install process is rather straight forward. This is a good as it gives a very light weight image (as such, there are a lot of exclusions from this product that are available in SE2,EE). It may be that your server uses this version of the database, well because it doesn't involve any licensing fee's for a production installation which can be a huge bonus for someone starting out.

I use an unsupported Linux version - Ubuntu - for this and find the core requirements are:

* unzip
* bc
* libaio1
* rpm

In my particular working example, I have some additional requirements because I built it to allow downloading the image, but that's an experimental/unsupported program and subject to stop working depending on the Oracle SSO process.

For the full database versions, of course you could develop your Dockerfiles with the install process, but since they are made available it makes it a quick way to get going, especially since the full install process for SE2, EE can take some time, debugging could be time consuming.

### Oracle container registry

Oracle has an authenticated service that you can pull images from. They include versions for SE2 and EE. For the EE, they provide 2 different versions - one a slim version. So the raw image size varies from about 2gb to 4gb. 

### Sample docker files

Additionally, Oracle provides some example Dockerfiles in their GitHub repository over at: https://github.com/Oracle/docker-images. They provide all versions - XE, SE2, EE. You, as the user just need to download the software for the specific version you wish to install before hand. On my system, I currently have the SE2 image, and all installed, it comes to about 20gb - so make sure you have some vacant space available before starting.

## X forwarding

If you are on Linux, you can start an image in X forwarding mode. Not only for Oracle, this can be a good approach for containerizing your software. In your docker run command, you need to pass firstly your display socker as a mounted volume in the container, and also set an environment as to which display to send it to. So, for example I have an image on my system tagged as tschf/scottdb, I would run the following:

```
docker run -it -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=unix$DISPLAY tschf/scottdb
```
Then for any GUI programs, when you launch them they will get sent to your main systems display. What particular program would we want forwarded? Well, one is owm (Oracle wallet manager). In order to use this program, these volume and environments would need to be set, then it's just a matter of running `owm` from the container. 

At this point, I should point out I did face an odd issue. If I tried to run a program from the base image `oraclelinux:7-slim` this process worked without a hitch. But the image I have above which descends from `oracle/database:12.2.0.1-se2` (from the sample dockerfiles on GitHub), when I attempted to run a GUI program I would receive the following error:

```
[oracle@c783b6387686 ~]$ xclock
No protocol specified
Error: Can't open display: unix:0
```
Since they both come from the same core image, I'm not sure what has been changed in the installation process to cause this. I came across this issue: https://github.com/jessfraz/dockerfiles/issues/6, and basically it looks as though the image is not running as my OS user, so we have to give permission to root to access the X system. So issue the following before running the container:

```
xhost local:root
```

And all will work as expected. Note: this only applies for the current desktop session.

## Quick links

Resources to topics discussed that may be helpful:

- https://container-registry.oracle.com
- https://github.com/Oracle/docker-files

