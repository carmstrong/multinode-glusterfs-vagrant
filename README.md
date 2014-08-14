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
$ vagrant ssh deis-server-1 -c 'gluster peer probe 172.21.12.12 ; gluster peer probe 172.21.12.13'
```

## Create a volume

Now we can create and start our volume spanning multiple hosts.

```console
$ vagrant ssh deis-server-1 -c 'gluster volume create glustertest replica 2 transport tcp 172.21.12.11:/brick 172.21.12.12:/brick 172.21.12.13:/brick'
```

```console
$ vagrant ssh deis-server-1 -c 'gluster start glustertest'
```

## Mount the volume

On our client, we can mount this volume and play around with it.

```console
$ vagrant ssh deis-client -c 'sudo mkdir /mnt/glusterfs && sudo mount -t glusterfs 172.21.12.11:/glustertest /mnt/glusterfs'
```

Note here that we just need to specify one host to mount - this is because the gluster client will connect and get metadata about the
volume, and may never even talk to this host again! Neat!

## Test!

What happens when we take down a machine?

```console
$ vagrant halt gluster-server-2
```

What happens if we instead take down the leader?

```console
$ vagrant halt gluster-server-1
```

## Cleanup

When you're all done, tell Vagrant to destroy the VMs.

```console
$ vagrant destroy -f
```
