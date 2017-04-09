# FAQ

## Network or DNS problems

### Symptomatic in guest:

Apt-get returns messages like:

    Could not resolve 'httpredir.debian.org'

Bash into your container or add a line to the Dockerfile ("RUN") to see if your network works:

    ping 8.8.8.8
    RUN ping 8.8.8.8

If your network works, see if your DNS is works:

    ping www.example.com
    RUN ping www.example.com 

If your DNS is not working, you can check your DNS settings with

    cat /etc/resolv.conf
    RUN cat /etc/resolv.conf

### Solution:

1. Check your host system's internet connection. 
2. Make sure that you use Docker version 1.8.1 or later and (in doubt) a package which is built by Docker. There have
   been [problems](https://github.com/docker/docker/issues/15681) using older packages and packages from other vendors.
3. [Check](https://github.com/docker/docker/issues/15681) if your guest has no network at all or if just the
   DNS resolution fails.

