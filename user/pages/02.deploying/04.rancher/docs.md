---
title: Rancher Deployment
taxonomy:
    category: docs
---

### Deploy Separate NeuVector Components with Rancher
The sample file will deploy one manager and 3 controllers. It will deploy an enforcer on every node. See the bottom section for specifying dedicated manager or controller nodes using node labels. Note: It is not recommended to deploy (scale) more than one manager behind a load balancer due to potential session state issues.

Note: Deployment on Rancher 2.x/Kubernetes should follow the Kubernetes reference section and/or Helm based deployment.

1. Deploy the catalog docker-compose-dist.yml, controllers will be deployed on the labelled nodes, enforcers will be deployed on the rest of nodes. (The sample file can be modified so that enforcers are only deployed to the specified nodes.)

2. Pick one of controllers for the manager to connect to. Modify the manager's catalog file docker-compose-manager.yml, set CTRL_SERVER_IP to the controller's IP, then deploy the manager catalog.

Here are the sample compose files. If you wish to only deploy one or two of the components just use that section of the file.

Rancher Manager/Controller/Enforcer Compose Sample File:
```
manager:
   scale: 1
   image: neuvector/manager
   restart: always
   environment:
     - CTRL_SERVER_IP=controller
   ports:
     - 8443:8443
controller:
   scale: 3
   image: neuvector/controller
   restart: always
   privileged: true
   environment:
     - CLUSTER_JOIN_ADDR=controller
   volumes:
     - /var/run/docker.sock:/var/run/docker.sock
     - /proc:/host/proc:ro
     - /sys/fs/cgroup:/host/cgroup:ro
     - /var/neuvector:/var/neuvector
enforcer:
   image: neuvector/enforcer
   pid: host
   restart: always
   privileged: true
   environment:
     - CLUSTER_JOIN_ADDR=controller
   volumes:
     - /lib/modules:/lib/modules
     - /var/run/docker.sock:/var/run/docker.sock
     - /proc:/host/proc:ro    
     - /sys/fs/cgroup/:/host/cgroup/:ro
   labels:
     io.rancher.scheduler.global: true
```

### Deploy Without Privileged Mode
On some systems, deployment without using privileged mode is supported. These systems must support the ability to add capabilities using the cap_add setting and to set the apparmor profile.

See the sections on deployment with Docker-Compose, Docker UCP/Datacenter for sample compose files.

Here is a sample Rancher compose file for deployment without privileged mode:
```
manager:
   scale: 1
   image: neuvector/manager
   restart: always
   environment:
     - CTRL_SERVER_IP=controller
   ports:
     - 8443:8443
controller:
   scale: 3
   image: neuvector/controller
   pid: host
   restart: always
   cap_add:
     - SYS_ADMIN
     - NET_ADMIN
     - SYS_PTRACE
   security_opt:
     - apparmor=unconfined
     - seccomp=unconfined
     - label=disable
   environment:
     - CLUSTER_JOIN_ADDR=controller
   volumes:
     - /var/run/docker.sock:/var/run/docker.sock
     - /proc:/host/proc:ro
     - /sys/fs/cgroup:/host/cgroup:ro
     - /var/neuvector:/var/neuvector
enforcer:
   image: neuvector/enforcer
   pid: host
   restart: always
   cap_add:
     - SYS_ADMIN
     - NET_ADMIN
     - SYS_PTRACE
     - IPC_LOCK
   security_opt:
     - apparmor=unconfined
     - seccomp=unconfined
     - label=disable
   environment:
     - CLUSTER_JOIN_ADDR=controller
   volumes:
     - /lib/modules:/lib/modules
     - /var/run/docker.sock:/var/run/docker.sock
     - /proc:/host/proc:ro    
     - /sys/fs/cgroup/:/host/cgroup/:ro
   labels:
     io.rancher.scheduler.global: true
```

### Using Node Labels for Manager and Controller Nodes
To control which nodes the Manager and Controller are deployed on, label each node. Pick the nodes where the controllers are to be deployed. Label them with "nvcontroller=true". (With the current sample file, no more than one controller can run on the same node.).

For the manager node, label it “nvmanager=true”.

Add labels in the yaml file. For example for the manager:
```
   labels:
     io.rancher.scheduler.global: true
     io.rancher.scheduler.affinity:host_label: "nvmanager=true"
```

For the controller:
```
   labels:
     io.rancher.scheduler.global: true
     io.rancher.scheduler.affinity:host_label: "nvcontroller=true"
```

For the enforcer, to prevent it from running on a controller node (if desired):
```
  labels:
     io.rancher.scheduler.global: true
     io.rancher.scheduler.affinity:host_label_ne: "nvcontroller=true"
```

