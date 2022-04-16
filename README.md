# Installing & Running podman on WSL2

### Podman works great on WSL2 but there are a few tweaks needed to install and run it cleanly:

### 1. Install & Start Ubuntu-20.04 WSL2 distro
We will assume you know how to do this and instructions for this are everywhere so we are not going to cover this here.  The rest of this assumes you are running as the default non-root user in your WSL2 Ubuntu-20.04

### 2. Update the apt repository
```
echo "deb https://download.opensuse.org/repositories/devel:/kubic:\
/libcontainers:/stable/xUbuntu_20.04/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list

curl -L "https://download.opensuse.org/repositories/devel:/kubic:\
/libcontainers:/stable/xUbuntu_20.04/Release.key" | sudo apt-key add -
```
### 2. Install podman
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install podman
```
### 3. Tweaks needed for  WSL2
At this point podman will actually appear to work fine.  You can download images and run containers just like you would expect.  But there is a hidden problem waiting.

If you start a container and then exit WSL and shut down that WSL instance and then try to get back in and do pretty much anything with podman, you'll notice that everything seems broken.  

```
$ podman ps -a
WARN[0000] "/" is not a shared mount, this could cause issues or missing mounts with rootless containers
ERRO[0000] error joining network namespace for container 88a8d5c7115598aeaa31fcd1cee8c084fee3ab2577b4f61dc317053d7da032f9: error retrieving network namespace at /tmp/podman-run-1000/netns/cni-f73b0b0b-155d-3c43-30b2-278280c003f1: unknown FS magic on "/tmp/podman-run-1000/netns/cni-f73b0b0b-155d-3c43-30b2-278280c003f1": ef53
Error: error joining network namespace of container 88a8d5c7115598aeaa31fcd1cee8c084fee3ab2577b4f61dc317053d7da032f9: error retrieving network namespace at /tmp/podman-run-1000/netns/cni-f73b0b0b-155d-3c43-30b2-278280c003f1: unknown FS magic on "/tmp/podman-run-1000/netns/cni-f73b0b0b-155d-3c43-30b2-278280c003f1": ef53
```
Looks like a disaster, right?  Fortunately this is easy to fix.  These are actually two separate issues:

To fix `WARN[0000] "/" is not a shared mount` :
```
sudo chmod 4755 /usr/bin/newgidmap
sudo chmod 4755 /usr/bin/newuidmap
```
To fix `ERRO[0000] error joining network namespace for container` :
```
sudo rm -rf /tmp/*
echo "none  /tmp  tmpfs  defaults  0 0" | sudo tee -a /etc/fstab
```
Shutdown the WSL instance and restart.  This makes /tmp a truly temporary mount that does not survive a restart and avoids podman getting locked up.
