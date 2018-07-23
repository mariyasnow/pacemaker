Pacemaker Basic
=================
Stripped down pacemaker/corosync basic setup to get started

* This will bring up 3 Vagrant boxes with a Host Only network and execute ansible role to install Corosync and Pacemaker

### Why?
Pacemaker is flexible and reliable for managing critical resources from router availibility to mysql instances. Legacy configs "just work" and can be rather tricky to desipher so this deploy strips off all the complications down to the basic functionality


## Setup
```
Mac or Linux

VirtualBox downloads - https://www.virtualbox.org/wiki/Downloads
Vagrant download - https://www.vagrantup.com/downloads.html
Virtualenv (optional but reccomended)
```

## Start 
```
mkvirtualenv pacemaker_testing 
pip install -r requirements.txt
vagrant up
```

## Test 

Start a dummy primitive resrouce 

```
vagrant@dangerzone1:~$ sudo crm configure primitive dummy0 ocf:pacemaker:Dummy op monitor interval=120s

NOTE: stonith should be handled more gracefully in prod
vagrant@dangerzone0:~$ sudo crm_resource --list
 dummy0	(ocf::pacemaker:Dummy):	Stopped 
vagrant@dangerzone0:~$ sudo crm_verify -L
Errors found during check: config not valid
  -V may provide more details
vagrant@dangerzone0:~$ sudo crm configure property stonith-enabled=false
vagrant@dangerzone0:~$ sudo crm_verify -L
vagrant@dangerzone0:~$ sudo crm_resource --list
 dummy0	(ocf::pacemaker:Dummy):	Started 


vagrant@dangerzone1:~$ sudo crm_mon -1
Last updated: Fri Jul 15 19:54:00 2016
Last change: Fri Jul 15 19:53:57 2016 via cibadmin on dangerzone1
Stack: corosync
Current DC: dangerzone0 (744751204) - partition with quorum
Version: 1.1.10-42f2063
3 Nodes configured
2 Resources configured


Online: [ dangerzone0 dangerzone1 dangerzone2 ]

 dummy0	(ocf::pacemaker:Dummy):	  Started dangerzone0 
```

Check the Nodes 

```
vagrant@dangerzone1:~$ sudo crm configure show
node $id="744751204" dangerzone0
node $id="744751205" dangerzone1
node $id="744751206" dangerzone2
primitive dummy0 ocf:pacemaker:Dummy \
	  op monitor interval="120s"
property $id="cib-bootstrap-options" \
	 dc-version="1.1.10-42f2063" \
	 cluster-infrastructure="corosync" \
	 no-quorum-policy="ignore" \
	 migration-threshold="1" \
	 stonith-enabled="false"
```
Stop the resource and see it move, Alternatively kill dangerzone0 and see the resource move

```
vagrant@dangerzone1:~$ sudo crm_resource --resource dummy0 --force-stop
Operation stop for dummy0 (ocf:pacemaker:Dummy) returned 0
 >  stderr: DEBUG: dummy0 stop : 0
vagrant@dangerzone1:~$ sudo crm_resource --resource dummy0 --locate
resource dummy0 is running on: dangerzone1 
```

## Stop

Don't forget to clean up interfaces 
```
vagrant destroy -f && VBoxManage hostonlyif remove vboxnet0 && VBoxManage hostonlyif remove vboxnet1
```

## Docs 

- https://github.com/ClusterLabs/pacemaker/tree/master/extra/resources resource options (basically shell scripts)
- https://www.virtualbox.org/manual/ch06.html#network_hostonly Vagrant networking

