# OpenShift "gold image" Provisioner

To reduce the time it takes to build out an OpenShift environment, and facilitate automated testing we need to bake several packages, configurations and containers into a RHEL instance.

<<<<<<< HEAD
<<<<<<< f210d795a55efcf3266ba21b2d3f91f5088966b8
You may want to resize the VM image root disk:

qemu-img resize rhel-guest-image-7.2-20160302.0.x86_64.qcow2 +20G
virt-import ... 

The image provisioner will support AWS and OpenStack, and be automated with Ansible.  The following Ansible roles are run on an instance:

- provide cloud-init config
- RHEL OS setup (install latest packages, ssh keys)
- collectd-install (install and configure collectd)
- docker-config (setup storage)
- repo-install (setup custom repos)
- pbench-install (install and configure [pbench](https://github.com/distributed-system-analysis/pbench))
<<<<<<< de11837fdc93cca07ffac1c84a121ae7a3922b88
<<<<<<< d2dcb7d6b0db9d3b1303aac9a5f9d4652c5ea8f1
  - needs template for pbench-config and keys
=======
>>>>>>> initial commit of image provisioner for fast scale-out prototyping
=======
  - needs template for pbench-config and keys
>>>>>>> Update README
- aos-ansible (pull down supporting container images)
- openshift-rpm-install (install openshift RPMs, but do not configure)
=======
=======
>>>>>>> 30512e1f7fae0883d333b34f7f34cfa42b39d20e
This repo includes a set of playbooks that work in two phases, depending on what type of image you're generating.

1) generate an image (openstack/kvm/ec2)
2) customize the image (done via ansible, so requires only ssh and passwordless keys)

Instructions:

You need a system capable of running KVM guests, and *you have to be root* on that system to do some of the filesystem resizing operations.
sudo did not seem to work.

````
# cd $HOME
# git clone https://github.com/openshift/svt
# cd svt/image_provisioner
# customize the inventory file
# ansible-playbook -i inventory/test playbooks/setup-image.yaml
```

This takes about 2 minutes.

```
Import the qcow2 into local libvirt:
# export NAME=gold ; virt-install --import --name $NAME --ram 4096 --vcpus 4 --disk path=/tmp/rhel-guest-image-7.2-20160302.0.x86_64.qcow2_resized_2016-07-20.qcow2 --disk path=/tmp/cidata.iso,device=cdrom --network bridge=br0 --graphics vnc --check path_in_use=off --noautoconsole --os-variant rhel7
```

Login and get IP address
```
# virsh console gold 
```

Feed that IP into the next playbook which will run all the ansible roles in this repo.  Time depends on speed to pull images.

```
# ansible-playbook -i inventory/test playbooks/provision_gold_images.yaml 
```

Shutdown the guest and sync it to your target environment.  Compression steps are optional.
```
# virsh shutdown gold
# time lzop -1 -o /tmp/2016-07-23-ocp-3.3.0.9-gold.lzo /tmp/rhel-guest-image-7.2-20160302.0.x86_64.qcow2_resized_2016-07-21.qcow2
# scp -i /root/env_4.key -r /tmp/2016-07-23-ocp-3.3.0.9-gold.lzo root@10.2.1.218:/home/stack/
# time lzop -d 2016-07-23-ocp-3.3.0.9-gold.lzo
# time glance image-create --name 2016-07-23-ocp-3.3.0.9 --disk-format qcow2 --container-format bare --file 2016-07-23-ocp-3.3.0.9-gold --visibility public
```

EC2 instructions coming soon.
<<<<<<< HEAD
>>>>>>> update readme
=======
>>>>>>> 30512e1f7fae0883d333b34f7f34cfa42b39d20e