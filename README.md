# Docker Hadoop From Scratch

Docker container configuration to create a local network of 3 nodes in order to simulate distributed computation.

Each node is based on the Docker image openjdk:8-alpine (Alpine Linux Kernel, OpenJDK v8) and has been augmented to setup and start a SSH deamon. The SSH deamon configuration is based on [panubo / docker-sshd](https://github.com/panubo/docker-sshd)

License : MIT

## Start and shutdown

### Start

From the root directory containing the _docker-compose.yml_ file:

1. Configure SSH keys to be used to log on the nodes

Two keys are required : 
- 1 to connect from the host to any container (as root but using your local user's key)
- 1 to connect from a container to another container (as root) 

Create a private-public pair of a RSA key (security matters less as this key is only used on the virtual network within the host):

```shell
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/my-login/.ssh/id_rsa): id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in id_rsa.
Your public key has been saved in id_rsa.pub.
The key fingerprint is: ...
The key's randomart image is: ...
```

Create a file named _public-keys_ that contains the 2 public keys. 

For example, if you have an ECDSA public key in your home, create a symbolic link:

```shell
$ cat ~/.ssh/id_ecdsa.pub id_rsa.pub > public-keys
```

2. launch 

On first launch will download the image and build the containers

```shell
$ docker-compose up
```

Will start the three nodes and show the log

### Shutdown 

From another shell (do not abort the running docker-compose process):

```shell
$ docker-compose down
```

## Usage

The three nodes SSH port (22) is remapped on localhost ports : 2201, 2202, 2203. The SSH login of _root_ user is enabled. Credentials are based on your localhost public key : ~/ssh/id_ecdsa.pub. If your public key is another format

Example 1, open an SSH terminal on node 1: 

```shell
$ ssh -p 2201 root@localhost
```

Example 2, copy a file to node 2:
```shell
$ scp -P 2202 README.md root@localhost:
```

SSH logins between two node 1 and node 3 of the network, use container name (see [docker-compose.yml](docker-compose.yml)):
```ssh
da512e42bb3f:~# ssh hadoop-from-scratch-worker-3
```

## Caveats

As the docker containers are reinitialized everytime the container is started, you may have some issues when login with SSH :

```shell
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:Kks0Wp+sG79L+tT7ZAARwI+JlAOSiGjkqD7rWOsMBUo.
Please contact your system administrator.
Add correct host key in /Users/my-login/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /Users/my-login/.ssh/known_hosts:25
ECDSA host key for [localhost]:2203 has changed and you have requested strict checking.
Host key verification failed.
```

To fix this, either:
- Cleanup the /Users/my-login/.ssh/known_hosts file
- Or, add the following to /Users/my-login/.ssh/config

```shell
Host localhost
   StrictHostKeyChecking no
   UserKnownHostsFile=/dev/null
```

(Be careful to indent lines 2 and 3)