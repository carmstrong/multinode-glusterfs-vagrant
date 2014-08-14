# Multinode GlusterFS on Vagrant

This guide walks users through setting up a 3-node GlusterFS cluster, creating and starting a volume, and mounting it on a client.

It's fun to learn [GlusterFS](http://gluster.org), kids!

## Install prerequisites

Install [Vagrant](http://www.vagrantup.com/downloads.html) and a provider such as [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

We'll also need the [vagrant-cachier](https://github.com/fgrehm/vagrant-cachier) plugin so we don't pull all of these packages unnecessarily on four hosts.

```console
$ vagrant plugin install vagrant-cachier
```

## Start the VMs

This instructs Vagrant to start the VMs and install GlusterFS on them.

```console
$ vagrant up
```

## Probe for peers

Before we can create a volume spanning multiple machines, we need to tell Gluster to recognize the other hosts.

```console
$ vagrant ssh gluster-server-1 -c 'sudo gluster peer probe 172.21.12.12 ; sudo gluster peer probe 172.21.12.13'
```

## Create a volume

Now we can create and start our volume spanning multiple hosts.

```console
$ vagrant ssh gluster-server-1 -c 'sudo gluster volume create glustertest replica 3 transport tcp 172.21.12.11:/brick 172.21.12.12:/brick 172.21.12.13:/brick force'
```

```console
$ vagrant ssh gluster-server-1 -c 'sudo gluster volume start glustertest'
```

## Mount the volume

On our client, we can mount this volume and play around with it.

```console
$ vagrant ssh gluster-client -c 'sudo mkdir /mnt/glusterfs && sudo mount -t glusterfs 172.21.12.11:/glustertest /mnt/glusterfs'
```

Note here that we just need to specify one host to mount - this is because the gluster client will connect and get metadata about the
volume, and may never even talk to this host again! Neat!

## Play around

We can use this like a local filesystem:

```console
$ vagrant ssh gluster-client
gluster-client $ echo "hello" | sudo tee /mnt/glusterfs/f.txt
```

Or, we can write big chunks of data to see how it performs:

```console
gluster-client $ sudo dd if=/dev/urandom of=/mnt/glusterfs/junk bs=64M count=16
```

## Test the cluster

What happens when we take down a machine?

```console
$ vagrant halt gluster-server-1
```

```console
vagrant@gluster-client:/mnt/glusterfs$ ls
f.txt  junk  lolol.txt
```

Everything still works!

What happens if we take down a second machine?

```console
$ vagrant halt gluster-server-2
```

```console
vagrant@gluster-client:/mnt/glusterfs$ ls
f.txt  junk  lolol.txt
```

Everything still works!

## Inspecting cluster state

As you mess with things, two commands are helpful to determine what is happening:

```console
vagrant@gluster-server-x:/$ sudo gluster peer status
vagrant@gluster-server-x:/$ sudo gluster volume info
```

You can also tail the gluster logs:

```console
sudo tail -f /var/log/glusterfs/etc-glusterfs-glusterd.vol.log
```

As you bring down the other hosts, you'll see the healthy host report them as down in the logs:
```
[2014-08-14 23:00:10.415446] W [socket.c:522:__socket_rwv] 0-management: readv on 172.21.12.12:24007 failed (No data available)
[2014-08-14 23:00:12.686355] E [socket.c:2161:socket_connect_finish] 0-management: connection to 172.21.12.12:24007 failed (Connection refused)
[2014-08-14 23:01:02.611395] W [socket.c:522:__socket_rwv] 0-management: readv on 172.21.12.13:24007 failed (No data available)
[2014-08-14 23:01:03.726702] E [socket.c:2161:socket_connect_finish] 0-management: connection to 172.21.12.13:24007 failed (Connection refused)
```

Similarly, you'll see the host come back up:
```
[2014-08-14 23:02:34.288696] I [glusterd-handshake.c:563:__glusterd_mgmt_hndsk_versions_ack] 0-management: using the op-version 30501
[2014-08-14 23:02:34.293048] I [glusterd-handler.c:2050:__glusterd_handle_incoming_friend_req] 0-glusterd: Received probe from uuid: 1dc04e5c-958f-4eea-baab-7afb33aaee69
[2014-08-14 23:02:34.293415] I [glusterd-handler.c:3085:glusterd_xfer_friend_add_resp] 0-glusterd: Responded to 172.21.12.12 (0), ret: 0
[2014-08-14 23:02:34.294823] I [glusterd-sm.c:495:glusterd_ac_send_friend_update] 0-: Added uuid: 1dc04e5c-958f-4eea-baab-7afb33aaee69, host: 172.21.12.12
[2014-08-14 23:02:34.295016] I [glusterd-sm.c:495:glusterd_ac_send_friend_update] 0-: Added uuid: 929cfe49-337b-4b41-a1e6-bc1636d5c757, host: 172.21.12.13
[2014-08-14 23:02:34.296738] I [glusterd-handler.c:2212:__glusterd_handle_friend_update] 0-glusterd: Received friend update from uuid: 1dc04e5c-958f-4eea-baab-7afb33aaee69
[2014-08-14 23:02:34.296904] I [glusterd-handler.c:2257:__glusterd_handle_friend_update] 0-: Received uuid: 9c9f8f9b-2c12-45dc-b95d-6b806236c0bd, hostname:172.21.12.11
[2014-08-14 23:02:34.297068] I [glusterd-handler.c:2266:__glusterd_handle_friend_update] 0-: Received my uuid as Friend
[2014-08-14 23:02:34.297270] I [glusterd-handler.c:2257:__glusterd_handle_friend_update] 0-: Received uuid: 929cfe49-337b-4b41-a1e6-bc1636d5c757, hostname:172.21.12.13
[2014-08-14 23:02:34.297469] I [glusterd-rpc-ops.c:356:__glusterd_friend_add_cbk] 0-glusterd: Received ACC from uuid: 1dc04e5c-958f-4eea-baab-7afb33aaee69, host: 172.21.12.12, port: 0
[2014-08-14 23:02:34.313528] I [glusterd-rpc-ops.c:553:__glusterd_friend_update_cbk] 0-management: Received ACC from uuid: 1dc04e5c-958f-4eea-baab-7afb33aaee69
```

## Cleanup

When you're all done, tell Vagrant to destroy the VMs.

```console
$ vagrant destroy -f
```
