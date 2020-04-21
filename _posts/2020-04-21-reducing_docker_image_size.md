---
layout: post
title: Reducing the size of my Docker images
categories: arm docker build-system
date: 2020-04-21 14:09 -0500
---
I recently started using Docker images to hold specific toolchains in a controlled environment. With the end goal to set up a portable and reproducible build system. In a recent [post]({% post_url 2020-04-13-building_with_docker %}), I described my base image with an ARM Cortex-M toolchain. After creating my ARM toolchain image, I started to expand to other toolchains and compilers. In this post, I will apply some best practices I came across to improve my images and reduce their size in the process.

<!--more-->
---

After creating my ARM toolchain image, I started to add images for other compilers to experiment with. At some point, I noticed the size of each container and became interested in their composition. My examples for this post:

```shell
PS> docker image list
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
renemoll/builder_arm_gcc   latest              033f62f87699        35 hours ago        971MB
renemoll/builder_clang     latest              c0597e09b2bf        29 minutes ago      950MB
```

# Decomposing the image

Let's look at what is inside the ARM toolchain image.

```shell
PS> docker history renemoll/builder_arm_gcc
117a48b37556        48 minutes ago      /bin/sh -c #(nop) ADD dir:2c03d348c707eb8cc6…   26.4kB
5b3d59a4d317        48 minutes ago      /bin/sh -c #(nop) WORKDIR /work                 0B
2b66eb9161ab        48 minutes ago      /bin/sh -c #(nop)  ENV PATH=/opt/gcc-arm-non…   0B
2f1c8883c156        49 minutes ago      /bin/sh -c wget -qO - https://developer.arm.…   505MB
52b90c935987        51 minutes ago      /bin/sh -c #(nop) WORKDIR /opt                  0B
e9b05e16c66c        51 minutes ago      /bin/sh -c apt-get update &&  apt-get upgrad…   392MB
d5d96de639a4        2 hours ago         /bin/sh -c #(nop)  LABEL version=1.0            0B
8d60aeb11f6e        2 hours ago         /bin/sh -c #(nop)  LABEL description=Builder…   0B
e9ccb229a23d        3 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>           3 weeks ago         /bin/sh -c mkdir -p /run/systemd && echo 'do…   7B
<missing>           3 weeks ago         /bin/sh -c set -xe   && echo '#!/bin/sh' > /…   811B
<missing>           3 weeks ago         /bin/sh -c [ -z "$(apt-get indextargets)" ]     985kB
<missing>           3 weeks ago         /bin/sh -c #(nop) ADD file:537f9883fb90d1938…   72MB
```

What we see here is a list of layers that together compose the image. Each layer represents a command in the Dockerfile. Next to the executed command, the size it adds to the image is listed. For example, the base image contributes 72MB, and adding a readme file adds 26kB. 

Two layers stand out, the `wget` and the `apt-get update` operation. The first downloads and installs the ARM toolchain. As expected, this is a significant contribution to the total image size. The second operation, the update command, updates the local apt repository and installs the various tools I consider necessary.

Looking at the other image (builder_clang), the package installation step is the main contributor to the final image size, adding 877MB. 

How can I reduce these sizes?

# Applying best practices

There are several blog posts with tips and best practices on how to write and improve Dockerfiles. I found [FROM:latest](https://www.fromlatest.io/#/) to be of great value as it lints your Dockerfile on the fly. Interactively parsing and analyzing your Dockerfile against common best practices.

Based on the linter's feedback, I removed the upgrade step, added the "no recommended packages" option and properly removed the apt cache. The result:

```shell
PS> docker image list
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
renemoll/builder_arm_gcc   latest              a2ff316c15f9        9 minutes ago       905MB
renemoll/builder_clang     latest              7da7f6a424f3        44 seconds ago      647MB
```

For the ARM image, this change does not do much. The Clang based machine sheds a little over a third of its size, definitely an improvement. 

I believe this is due to the omitting of the recommended packages. For the ARM image, I manually pull and install the toolchain, so there is little to gain with this improvement. Whereas the Clang toolchain is pulled in via apt. This time without the recommended packages, reducing the total installation size. This is supported by going through the [package's contents](https://packages.ubuntu.com/eoan/clang-9). It shows that various development libraries will not be selected for installation, adding up to hundreds of megabytes.

# Replacing packages

Next, I had a look at the packages I install for each image. For example, I install the build-essential package, but do I truly need this? Taking a look at the package [details](https://packages.ubuntu.com/eoan/build-essential), it contains some development tools, gcc, g++ and make. For my builders, I only need the latter. So let us see what happens when I replace build-essential with make.

```shell
PS> docker image list
REPOSITORY                 TAG                 IMAGE ID            CREATED              SIZE
renemoll/builder_arm_gcc   latest              a73640419c6f        19 minutes ago       754MB
renemoll/builder_clang     latest              1f2ae254e668        About a minute ago   524MB
```

After that I had a look at the installed packages, from inside the container. `dpkg-query` provides a way to list installed packages, and with the following command it shows their installed size (in kilobytes) as well:
```bash
dpkg-query -Wf '${Installed-Size}\t${Package}\n' | sort -n | tail -n 10
```

This did not provide me with further improvement options. Any significant size contributors are required packages. None for which there are smaller alternatives available.

# Results

I noted the total size of the images after each step in the following table:

| Scenario | ARM GCC | Clang |
| ----| --------| --------|
| Orignal | 971MB | 950MB |
| Without apt upgrade and recommendated packages  | 905MB | 647MB |
| Replaced packages  | 754MB | 524MB |

I am quite happy with these results. Sure, the ARM image did not reduce much sizewise. But knowing I stripped the images from unnecessary actions feels good. Furthermore, I have seen that toolchains pulled in from the OS' repository can be reduced significantly. I shrank the size of my Clang image by 44%!

A popular choice to reduce image sizes further is to base the image on Alpine Linux as it is only 5MB. This is partly achieved by using a different C runtime, Alpine Linux ships with [musl](http://www.musl-libc.org/). Most software will function but may need to be recompiled. Not only is this time consuming, but results may also [disappoint](https://pythonspeed.com/articles/alpine-docker-python/).

Personally I do not see a big advantage in changing OS. As it stands, it is only a small portion of the total size and the main contributors remain the required packages.

# References

1. [FROM:latest](https://www.fromlatest.io/#/)
2. [Clang-9 package details](https://packages.ubuntu.com/eoan/clang-9)
3. [build-essentials package details](https://packages.ubuntu.com/eoan/build-essential)
4. [musl](http://www.musl-libc.org/)
5. [Using Alpine can make Python Docker builds 50× slower](https://pythonspeed.com/articles/alpine-docker-python/)
