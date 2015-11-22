# CoreOS Vagrant Kubernetes

This repo provides a template Vagrantfile to create a CoreOS virtual machine with a single node Kubernetes cluster using the VirtualBox software hypervisor based off [coreos-vagrant](https://github.com/coreos/coreos-vagrant) and inspired by this [coreos blog post](https://coreos.com/blog/introducing-the-kubelet-in-coreos/) about `kubelet` being added to CoreOS.
After setup is complete you will have a single CoreOS virtual machine with Kubernetes running on your local machine. It will also download and install `kubectl` if it isn't already installed.

## Streamlined setup

1) Install dependencies

* [VirtualBox][virtualbox] 4.3.10 or greater.
* [Vagrant][vagrant] 1.6 or greater.

2) Clone this project and get it running!

```
git clone https://github.com/jtblin/coreos-vagrant-kubernetes.git
cd coreos-vagrant-kubernetes
```

3) Startup and SSH

There are two "providers" for Vagrant with slightly different instructions.
Follow one of the following two options:

**VirtualBox Provider**

The VirtualBox provider is the default Vagrant provider. Use this if you are unsure.

```
./configure
vagrant ssh
```

**VMware Provider**

The VMware provider is a commercial addon from Hashicorp that offers better stability and speed.
If you use this provider follow these instructions.

VMware Fusion:
```
PROVIDER=vmware_fusion
./configure
vagrant ssh
```

VMware Workstation:
```
PROVIDER=vmware_workstation
vagrant ssh
```

``./configure`` triggers vagrant to download the CoreOS image (if necessary) and (re)launch the instance. It will also download `kubectl` if needed and setup the kube-ui and kube-dns services.

``vagrant ssh`` connects you to the virtual machine.
Configuration is stored in the directory so you can always return to this machine by executing vagrant ssh from the directory where the Vagrantfile was located.

4) Get started [using CoreOS][using-coreos]

#### [kubernetes] cluster

By default, a cluster named `core` will be configured. You can change the name of the cluster
by setting up the `CLUSTER_NAME` environment variable before you run `vagrant up`:

```
CLUSTER_NAME=my-cluster
./configure
vagrant ssh
```

You can interact with your kubernetes cluster with the `kubectl` cli:

    kubectl --cluster=core get pods

This repo will also setup `kube-ui` and `kube-dns` addons.

Access [kubernetes ui](http://172.17.8.101:8080/) or [cadvisor ui](http://172.17.8.101:4194/containers/).


[virtualbox]: https://www.virtualbox.org/
[vagrant]: https://www.vagrantup.com/downloads.html
[using-coreos]: http://coreos.com/docs/using-coreos/
[kubernetes]: http://kubernetes.io/

#### Shared Folder Setup

There is optional shared folder setup.
You can try it out by adding a section to your Vagrantfile like this.

```
config.vm.network "private_network", ip: "172.17.8.150"
config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true,  :mount_options   => ['nolock,vers=3,udp']
```

After a 'vagrant reload' you will be prompted for your local machine password.

#### Provisioning with user-data

The Vagrantfile will provision your CoreOS VM(s) with [coreos-cloudinit][coreos-cloudinit] using the `user-data.sample.yaml` found in the project directory which setup the kubernetes cluster.
coreos-cloudinit simplifies the provisioning process through the use of a script or cloud-config document.

Check out the [coreos-cloudinit documentation][coreos-cloudinit] to learn about the available features.

[coreos-cloudinit]: https://github.com/coreos/coreos-cloudinit

#### Configuration

The Vagrantfile will parse the `config.rb` file containing a set of options used to configure your CoreOS cluster.

## New Box Versions

CoreOS is a rolling release distribution and versions that are out of date will automatically update.
If you want to start from the most up to date version you will need to make sure that you have the latest box file of CoreOS.
Simply remove the old box file and vagrant will download the latest one the next time you `vagrant up`.

```
vagrant box remove coreos --provider vmware_fusion
vagrant box remove coreos --provider vmware_workstation
vagrant box remove coreos --provider virtualbox
```

## Docker Forwarding

By setting the `$expose_docker_tcp` configuration value you can forward a local TCP port to docker on
each CoreOS machine that you launch. The first machine will be available on the port that you specify
and each additional machine will increment the port by 1.

Follow the [Enable Remote API instructions][coreos-enabling-port-forwarding] to get the CoreOS VM setup to work with port forwarding.

[coreos-enabling-port-forwarding]: https://coreos.com/docs/launching-containers/building/customizing-docker/#enable-the-remote-api-on-a-new-socket

Then you can then use the `docker` command from your local shell by setting `DOCKER_HOST`:

    export DOCKER_HOST=tcp://localhost:2375
