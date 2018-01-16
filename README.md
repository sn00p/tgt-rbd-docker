# tgt-rbd-docker
Put tgt daemon in container

- Pull docker image

```shell
docker pull minshenglin/tgt-rbd-docker
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

- Check RBD Pool is exists and rbd image has been crated

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
-v /etc/ceph:/etc/ceph -p 3260:3260 minshenglin/tgt-rbd-docker 
```
