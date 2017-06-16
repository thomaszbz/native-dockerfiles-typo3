# Create Docker test containers for TYPO3 CMS

![License MIT](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)
[![Average time to resolve an issue](https://isitmaintained.com/badge/resolution/thomaszbz/native-dockerfiles-typo3.svg)](https://isitmaintained.com/project/thomaszbz/native-dockerfiles-typo3 "Average time to resolve an issue")
[![Percentage of issues still open](https://isitmaintained.com/badge/open/thomaszbz/native-dockerfiles-typo3.svg)](https://isitmaintained.com/project/thomaszbz/native-dockerfiles-typo3 "Percentage of issues still open")

This repository provides Dockerfiles to create basic test environments for TYPO3 CMS developers and integrators.

This manual describes how to create a Docker image and a running Docker container to test TYPO3 CMS
and TYPO3 extensions against a running instance of [TYPO3 CMS](https://typo3.org/).

This manual is written for an ubuntu 16.04 host system with Linux kernel version 4.4.
It should also apply to most Debian-based systems with a recent Linux kernel. Other distros might require
minor changes. Other operating systems than GNU/Linux are not covered by this manual (e. g. Windows, OSX).

In this manual, the Docker container is run natively by the docker daemon without using any further virtualization
technology (e. g. vmware, VirtualBox, KVM, etc.). Instead, isolation happens using the infrastructure
of the Linux kernel which renders a container with very fast file shares (which you can edit on your disk
in an IDE of your choice).

All ``docker`` commands of this manual require root privileges for execution.

Keep in mind that running Docker natively might not be secure enough for production environments.

The latest version of this repository can be found at [GitHub](https://github.com/thomaszbz/native-dockerfiles-typo3).
The content of this repository is [MIT-licensed](./LICENSE).

## Features

* Provisions a Docker image which can run a TYPO3 CMS instance (all tests of TYPO3's install tool go green)
* Provides very fast file shares which are mounted by the container
* Allows you to edit your project's source code in your IDE without having to sync files (via ssh or ftp)
* Allows to add one or multiple TYPO3 extensions via file share
* Provides a user with name ``webadmin`` for the UID/GID mapping to your host system user (your UID/GID)
* Minimalistic approach using Docker's basic provisioning (no Vagrant/Chef/Puppet/etc. required) and run by docker
  natively (without further virtualization technology like vmware, VirtualBox, KVM, etc.)
* Fully benefits of CoW file systems which are supported by Docker (like BTRFS). Adding a new container won't
  consume more than a few megabytes once the base image is created.

## Prerequisites

### Checkout sources with git

Clone source code repositories (for later pushes you might want to go for the ssh protocol instead)

    cd /path/to/workspace
    git clone https://github.com/thomaszbz/native-dockerfiles-typo3.git
    git clone https://github.com/mblaschke/TYPO3-metaseo.git metaseo
    git clone https://git.typo3.org/Packages/TYPO3.CMS.git

Check out a version you want to run your tests against

    cd TYPO3.CMS
    git checkout TYPO3_6-2-14
    cd ..

### Install Docker

[Install Docker](http://docs.docker.com/installation/) version 1.8.1 at least (Kernel 3.19 is known to work).
After becoming root with a `sudo su - root`, remove old Docker related repository entries in `/etc/apt/sources.list`
and `/etc/apt/sources.list.d` and install Docker 17 community edition on Ubuntu 16.04 with

    apt-get remove docker docker-engine
    apt-get install apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    apt-get update
    apt-get install docker-ce

Test Docker installation:

    docker run hello-world

Download base images:

    docker pull debian:jessie
    docker pull debian:stretch

List Docker images:

    docker images

## Provision Docker image

Cd to the directory which contains the ``Dockerfile``.

    cd /path/to/workspace
    cd native-dockerfiles-typo3/typo3-debian-jessie # For TYPO3 CMS 6.2 or 7.6
    cd native-dockerfiles-typo3/typo3-debian-stretch # For TYPO3 CMS 7.6 or 8.7 or 9.x

Open the ``Dockerfile`` and replace ``1000`` with your UID/GID (must correspond to the owner of the files
which shall be shared to the container).

Optional: feel free to replace the string ``{{password}}`` in the ``Dockerfile`` with a MySQL password of your choice.

Provision a Docker image for TYPO3 CMS with name ``2bis10/typo3``:

    docker build -t 2bis10/typo3 .

## Create new Docker container

Create new container with name ``typo3`` while

* ``$wsdir`` contains the path to your workspace
* ``metaseo`` is optional and adds a TYPO3 CMS extension. Can be replaced with any other extension.

The created container should also keep running in detached mode:

    export wsdir=`cd ..;cd ..;pwd`
    docker run -dit --name typo3 -v $wsdir/TYPO3.CMS:/var/webadmin/typo3_src -v $wsdir/metaseo:/var/www/typo3/typo3conf/ext/metaseo 2bis10/typo3

## Maintain TYPO3 CMS instance

Remove TYPO3 CMS composer dependencies:

    docker exec -it  --user="webadmin" typo3 /bin/bash -c 'cd /var/webadmin/typo3_src && rm -Rf vendor/* bin/* Packages/Libraries/* typo3/vendor/*'

Update composer

    docker exec -it typo3 /bin/bash -c 'curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer'

Install or update TYPO3 CMS composer dependencies to the pinned version in the repository:

    docker exec -it  --user="webadmin" typo3 /bin/bash -c 'cd /var/webadmin/typo3_src && composer install -o'

Start MySQL:

    docker exec -it typo3 service mysql start

Start Apache web server:

    docker exec -it typo3 service apache2 start

Enable TYPO3's [install tool](https://docs.typo3.org/typo3cms/SecurityGuide/GuidelinesIntegrators/InstallTool/Index.html):

    docker exec -it --user="www-data" typo3 touch /var/www/typo3/typo3conf/ENABLE_INSTALL_TOOL
    docker exec -it --user="www-data" typo3 touch /var/www/typo3/FIRST_INSTALL

Get containers's IP adress:

    docker exec -it typo3 ifconfig|grep "addr:172"

The TYPO3 CMS instance should now be accessible via http. Just enter the IP in a browser of your choice.

You can follow the instructions of the TYPO3 CMS installation procedures while the database will be accessible
with

    user: admin
    password: {{password}}

## Maintain Docker container

Bash into running container:

    docker exec -it typo3 bash

Start container with interactive shell:

    docker start -i typo3

Boot container:

    docker start typo3

Shut down container:

    docker stop typo3

List containers:

    docker ps -a

Destroy container:

    docker rm typo3


List images:

    docker images

Delete image:

    docker rmi 2bis10/typo3

## Limitations

The simple approach of this project might or might not be usable for everybody. If you want to use a more
sophisticated approach you might want to have a look at Markus Blaschke's
[TYPO3-docker-boilerplate](https://github.com/webdevops/TYPO3-docker-boilerplate) which provides tons of
additional features and runs with multiple operation systems including GNU/Linux, Windows and OSX.

## Donations

You can motivate us to continue work on this project by donating to a good cause of your choice. We suggest
donations to the [Red Cross refugee help](https://www.drk.de/en/).
Please tell us about your donation via
[List of donations](https://github.com/thomaszbz/native-dockerfiles-typo3/issues/26).

## Copyright, license and contribution

&copy; 2015-2017 Thomas Mayer. The content of this repository is [MIT-licensed](./LICENSE).
In case you want to make contributions I assume that the [MIT-license](./LICENSE) also applies for the code you provide.

[![PHP development brought to you by 2bis10 IT-Services UG (haftungsbeschränkt)](https://www.2bis10.de/fileadmin/templates/2bis10/img/favicon.svg)](https://www.2bis10.de "PHP development brought to you by 2bis10 IT-Services UG (haftungsbeschränkt)")
![Piwik](https://piwik.2bis10.de/piwik.php?idsite=5&rec=1&url=https://github.com/thomaszbz/native-dockerfiles-typo3)