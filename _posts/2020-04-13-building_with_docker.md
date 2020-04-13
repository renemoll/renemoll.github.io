---
layout: post
title: Compiling a bare metal application with a Docker image
categories: arm docker build-system
date: 2020-04-13 15:39 -0500
---
Recently I started to set up an automated build system for my code projects. One of the elements in there is utilizing Docker images for self-containing build environments and creating reproducible builds. Here, I focus on the creation of the image itself and how to use it to compile for bare metal ARM (Cortex-M in my case.)

<!--more-->
---

A post on Sticky Bits, [An Introduction to Docker for Embedded Developers](https://blog.feabhas.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/), inspired me to replace my virtual machines with Docker images. If you want to read up on the basics of using Docker, that post provides a good introduction. 

My goal is to create a set of images that allow me to compile my code with various toolchains and various targets. Each image build for a different toolchain.

# How to compile with containers

Once you installed Docker, you can execute the following in a terminal:
```shell
PS> docker run gcc gcc -v
```

What this does is, the Docker client creates a container based on the [GCC](https://hub.docker.com/_/gcc) image and retrieves the version of the GCC toolchain. Now the client automatically downloads any missing images from the [hub](https://hub.docker.com/search?q=&type=image), so you don’t have to worry about that.

The previous command looks very familiar to an ordinary call to _gcc_, the only addition is the prefix to execute it in a container.

## Hello world

Let's compile a simple program, a hello world:

```c
#include <stdio.h>
int main(void) 
{
  puts("Hello from Docker!");
  return 0;
}
```

Save the file as 'hello_docker.c' and execute the following command:
```shell
PS> docker run -v ${PWD}:/home -w /home gcc gcc -o hello hello_docker.c
```

> Note that I am using PowerShell, hence the ${PWD}. For bash, simply replace ${PWD} with $(pwd).

This command involves more than just running a command within a container. First of all, to compile a source file, that file needs be accessible inside the container. For that, I added the first argument "-v" to specify a mount-point that I use to mount my current folder onto the _/home_ folder inside the container. Next, I set the working folder to _/home_. This will be the working directory when the container starts. When we now execute the container, it enters the home directory, the exact location where I conveniently mounted my source file.

If everything went ok we can now execute the compiled application and see the output:
```shell
PS> docker run -v ${PWD}:/home -w /home gcc ./hello
Hello from Docker!
```

# Creating my custom images

To compile code for ARM Cortex-M, one needs a special toolchain. The [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm) is one example. This specific implementation comes pre-compiled and ready to go. The only thing I need to do is extract the tarball and integrating it into the system.

How would that translate into an image? One straight forward example is the following Dockerfile.

```
FROM ubuntu:19.10
LABEL description="Builder for bare metal ARM."

RUN apt-get update && \
	apt-get upgrade -y && \
	apt-get install -y \
		build-essential \
		bzip2 \
		wget && \
	apt-get clean

WORKDIR /opt
RUN wget -qO - https://developer.arm.com/-/media/Files/downloads/gnu-rm/9-2019q4/gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2 | tar -xj
ENV PATH "/opt/gcc-arm-none-eabi-9-2019-q4-major/bin:$PATH"

WORKDIR /work
ADD . /work
```

The example instructs Docker to create an image based on an existing Ubuntu image, install the latest tools required to retrieve and extract the toolchain, install that toolchain and, prepares the image for use. 

## Using it

Let us compile our example file again, but now using our shiny new ARM container:
```shell
PS> docker run -v ${PWD}:/work renemoll/builder_arm_gcc arm-none-eabi-gcc --specs=nosys.specs -o hello.elf hello_docker.c
```
> Bare metal targets tend not to support a C runtime, which means you have to provide it. In this case I link in the _nosys_ runtime, provided by the toolchain, for a minimal runtime.

We can do a quick verification with obj-dump:
```shell
PS> docker run -v ${PWD}:/work renemoll/builder_arm_gcc arm-none-eabi-objdump -a hello.elf
hello.elf:     file format elf32-littlearm
hello.elf
```
This shows the application is indeed compiled for ARM successfully.

## The final result

I extend the Dockerfile to include some additional packages. I added CMake and python3 for my build support scripting. You could also look at my image file here: [ARM Dockerfile](https://github.com/renemoll/builder_arm_gcc/blob/master/Dockerfile) and find the resulting image on Docker hub: [builder_arm_gcc](https://hub.docker.com/r/renemoll/builder_arm_gcc).

I will post more on my build system later on.

# References

1. [An Introduction to Docker for Embedded Developers](https://blog.feabhas.com/2017/09/introduction-docker-embedded-developers-part-1-getting-started/).
2. [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm).
3. My current [ARM Dockerfile](https://github.com/renemoll/builder_arm_gcc/blob/master/Dockerfile) and image: [builder_arm_gcc](https://hub.docker.com/r/renemoll/builder_arm_gcc).