[![Docker Build](https://img.shields.io/docker/build/inwinstack/tgt-rbd-docker.svg)](https://hub.docker.com/r/inwinstack/tgt-rbd-docker/)

# tgt-rbd-docker
Using tgt daemon in container

## Setup

- Pull docker image

```shell
docker pull inwinstack/tgt-rbd-docker
```

- Check ceph.conf and ceph.client.admin.keyring in /etc/ceph

```
[root@mon-1 ~]# ls -l /etc/ceph/
total 12
-rw-------. 1 root root  63 Jan 15 05:55 ceph.client.admin.keyring
-rw-r--r--. 1 root root 268 Jan 15 08:04 ceph.conf
-rw-r--r--. 1 root root  92 Nov 30 16:14 rbdmap
-rw-------. 1 root root   0 Jan 15 05:55 tmp6OEnwT
```

- Check RBD Pool is exists and rbd image has been created

```
[root@mon-1 ~]# ceph osd pool ls
iscsi
[root@mon-1 ~]# rbd -p iscsi ls
iscsi-rbd
```

- Create directory tgt and edit config targets.conf

```conf
<target iqn.2016.03.ceph:iscsi>
    driver iscsi
    bs-type rbd
    backing-store iscsi/iscsi-rbd  # Format is <iscsi-pool>/<iscsi-rbd-image>
</target>
```

- Start tgt container

```shell
docker run -d --net=host --name tgt -v /home/vagrant/tgt-docker/tgt/:/etc/tgt/ \ 
-v /etc/ceph:/etc/ceph -p 3260:3260 inwinstack/tgt-rbd-docker 
```

- Add restart argument if you need to auto restart this container

```shell
docker run -d --net=host --restart always --name tgt -v /home/vagrant/tgt-docker/tgt/:/etc/tgt/ \ 
-v /etc/ceph:/etc/ceph -p 3260:3260 inwinstack/tgt-rbd-docker 
```

## SELinux

- if SELinux is enabled, tgt config may not be mapped in container

```
[root@ceph-node ceph-conf]# getenforce 1
Enforcing
[root@ceph-node ceph-conf]# docker run -d --net=host --name tgt -v /root/ceph-conf/tgt/:/etc/tgt/ -v /etc/ceph:/etc/ceph -p 3260:3260 inwinstack/tgt-rbd-docker
988fe869346b1743f6245dbd5d89053d6943235f3505895881c91c4423c0c43a
[root@ceph-node ceph-conf]# docker logs tgt
tgtadm: failed to send request hdr to tgt daemon, Transport endpoint is not connected
Config file /etc/tgt/targets.conf not found. Exiting...
tgtd: iser_ib_init(3434) Failed to initialize RDMA; load kernel modules?
can't adjust oom-killer's pardon /proc/self/oom_score_adj, Permission denied
tgtd: work_timer_start(150) use signal based scheduler
tgtd: bs_init(387) use signalfd notification
```
