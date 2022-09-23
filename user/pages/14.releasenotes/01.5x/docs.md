---
title: 5.x Release Notes
taxonomy:
    category: docs
---

### Release Notes for 5.x (Open Source Version)

#### 5.0.3 September, 2022
##### Enhancements
+ Do not display the EULA after successful restart from persistent volume.
+ Use the image filter in vulnerability profile setting to skip container scan results.
+ Support scanner in GitHub actions at https://github.com/neuvector/neuvector-image-scan-action.
+ Add Enforcer environment variables for disabling secrets scanning and running CIS benchmarks
```
    env:
      - name: ENF_NO_SECRET_SCANS  (available after v4.4.4)
        value: "1"
      - name: ENF_NO_AUTO_BENCHMARK (after v5.0.3)
        value: "1"
```

##### Bug Fixes
+ Enforcer unable to start occasionally.
+ Connection leak on multi-cluster federation environments.
+ Compliance page not loading some times in Security Risks -> Compliance 

#### 5.0.2 July 2022
##### Enhancements
+ Rancher hardened and SELinux clusters are supported.

##### Bug Fixes
+ Agent process high cpu usage on k3s systems.
+ AD LDAP groups not working properly after upgrade to 5.0.
+ Enforcer keeps restating due to error=too many open files (rke2/cilium).
+ Support log is unable to download successfully.

#### 5.0.1 June 2022
##### Enhancements
+ Support vulnerability scan of openSUSE Leap OS (in scanner image).
+ Scanner: implement wipe-out attributes during reconstructing image repo.
+ Verify NeuVector deployment and support for SELinux enabled hosts. See below for details on interim patching until helm chart is updated.
+ Distinguish between Feature Chart and Partner Charts in Rancher UI more prominently.+ Improve ingress annotation for nginx in Rancher helm chart. Add / update
ingress.kubernetes.io/protocol: https to nginx.ingress.kubernetes.io/backend-protocol: "HTTPS".
+ Current OpenShift Operator supports passthrough routes for api and federation services. Additional Helm Value parameters are added to support edge and re-encrypt route termination types. 

##### Bug Fixes
+ AKS cluster could add unexpected key in admission control webhook.
+ Enforcer is not becoming operational on k8s 1.24 cluster with 1.64 containerd runtime. Separately, enforcer sometimes fails to start.
+ Any admin-role user(local user or SSO) who promotes a cluster to fed master should be automatically promoted to fedAdmin role.
+ When sso using Rancher default admin into NeuVector on master cluster, the NeuVector login role is admin, not fedAdmin.
+ Fix several goroutine crashes.
+ Implicit violation from host IP not associated with node.
+ ComplianceProfile does not show PCI tag.
+ LDAP group mapping sometimes is not shown.
+ Risk Review and Improvement tool will result in error message "Failed to update system config: Request in wrong format".
+ OKD 3.11 - Clusterrole error shows even if it exists.

##### CVE Remediations
+ High CVE-2022-29458 cve found on ncurses package in all images.
+ High CVE-2022-27778 and CVE-2022-27782 found on curl package in Updater image.

##### Details on SELinux Support
NeuVector does not need any additional setting for SELinux enabled clusters to deploy and function. Tested deploying NeuVector on RHEL 8.5 based SELinux enabled RKE2 hardened cluster. Neuvector deployed successfully if PSP is enabled and patching Manager and Scanner deployment. The next chart release should fix the below issue.

Attached example for enabling psp from Rancher chart and given below the commands for patching Manager and Scanner deployment. The user ID in the patch command can be any number.

```
kubectl patch deploy -ncattle-neuvector-system neuvector-scanner-pod --patch '{"spec":{"template":{"spec":{"securityContext":{"runAsUser": 5400}}}}}'
kubectl patch deploy -ncattle-neuvector-system neuvector-manager-pod --patch '{"spec":{"template":{"spec":{"securityContext":{"runAsUser": 5400}}}}}'
```

Example for enabling PSP:

```
[neuvector@localhost nv]$ getenforce
Enforcing
[neuvector@localhost nv]$ sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33

[neuvector@localhost nv]$ kk get psp
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
NAME                      PRIV    CAPS                                      SELINUX    RUNASUSER          FSGROUP     SUPGROUP    READONLYROOTFS   VOLUMES
global-restricted-psp     false                                             RunAsAny   MustRunAsNonRoot   MustRunAs   MustRunAs   false            configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim
neuvector-binding-psp     true    SYS_ADMIN,NET_ADMIN,SYS_PTRACE,IPC_LOCK   RunAsAny   RunAsAny           RunAsAny    RunAsAny    false            *
system-unrestricted-psp   true    *                                         RunAsAny   RunAsAny           RunAsAny    RunAsAny    false            *
[neuvector@localhost nv]$ nvpo.sh
NAME                                        READY   STATUS    RESTARTS   AGE     IP           NODE                    NOMINATED NODE   READINESS GATES
neuvector-controller-pod-54f69f7f9c-6h822   1/1     Running   0          5m51s   10.42.0.29   localhost.localdomain   <none>           <none>
neuvector-enforcer-pod-jz77b                1/1     Running   0          5m51s   10.42.0.30   localhost.localdomain   <none>           <none>
neuvector-manager-pod-588488bb78-p6vf9      1/1     Running   0          111s    10.42.0.32   localhost.localdomain   <none>           <none>
neuvector-scanner-pod-87474dcff-s8vgt       1/1     Running   0          114s    10.42.0.31   localhost.localdomain   <none>           <none>
```


#### 5.0.0 General Availability (GA) Release May 2022
#####Enhancements
+ Automated Promotion of Group Modes. Promotes a Group’s protection Mode based on elapsed time and criteria. Does not apply to CRD created Groups. This features allows a new application to run in Discover for some time period, learning the behavior and NeuVector creating allow-list rules for Network and Process, then automatically moving to Monitor, then Protect mode. Discover to Monitor criterion: Elapsed time for learning all network and process activity of at least one live pod in the Group. Monitor to Protect criterion: There are no security events (network, process etc) for the timeframe set for the Group.
+ Support for Rancher 2.6.5 Apps and Marketplace chart. Deploys into cattle-neuvector-system namespace and enables SSO from Rancher to NeuVector. Note: Previous deployments from Rancher (e.g. Partner catalog charts, version 1.9.x and earlier), must be completely removed in order to update to the new chart.
+ Support scanning of SUSE Linux (SLE, SLES), and Microsoft Mariner
+ Zero-drift process and file protection. This is the new default mode for process and file protections. Zero-drift automatically allows only processes which originate from the parent process that is in the original container image, and does not allow file updates or new files to be installed. When in Discover or Monitor mode, zero-drift will alert on any suspicious process or file activity. In Protect mode, it will block such activity. Zero-drift does not require processes to be learned or added to an allow-list. Disabling zero-drift for a group will cause the process and file rules listed for the group to take effect instead.
+ Split policy mode protection for network, process/file. There is now a global setting available in Settings -> Configuration to separately set the network protection mode for enforcement of network rules. Enabling this (default is disabled), causes all network rules to be in the protection mode selected (Discover, Monitor, Protect), while process/file rules remain in the protection mode for that Group, as displayed in the Policy -> Groups screen. In this way, network rules can be set to Protect (blocking), while process/file policy can be set to Monitor, or vice versa.
+ WAF rule detection, enhanced DLP rules (header, URL, full packet). Used for ingress connections to web application pods as well as outbound connections to api-services to enforce api security.
+ CRD for WAF, DLP and admission controls. NOTE: required additional cluster role bindings/permissions. See Kubernetes and OpenShift deployment sections. CRD import/export and versioning for admission controls supported through CRD. 
+ Rancher SSO integration to launch NeuVector console through Rancher Manager. This feature is only available if the NeuVector containers are deployed through Rancher. This deployment pulls from the mirrored Rancher repository (e.g. rancher/mirrored-neuvector-controller:5.0.0) and deploys into the cattle-neuvector-system namespace. NOTE: Requires updated Rancher release 2.6.5 May 2022 or later, and only admin and cluster owner roles are supported at this time.
+ Supports deployment on RKE2.
+ Support for Federation of clusters (multi-cluster manager) through a proxy. Configure proxy in Settings -> Configuration, and enable proxy when configuring federation connections.
+ Monitor required rbac's clusterrole/bindings and alert in events and UI if any are missing.
+ Support criteria of resource limitations in admission control rules.
+ Support Microsoft Teams format for webhooks.
+ Support AD/LDAP nested groups under mapped role group.
+ Support clusterrolebindings or rolebindings with group info in IDP for Openshift.

#####Bug Fixes
+ Fix issue of worker federation role backup should restore into non-federated clusters.
+ Improve page loading times for large number of CVEs in Security Risks -> Vulnerabilities
+ Allow user to switch mode when they select all groups in Policy -> Groups menu. Warn if the Nodes group is also selected.
+ Collapse compliance check items of the same name and make expandable.
+ Enhance security of gRPC communications.
+ Fixed: unable to get correct workload privileged info in rke2 setup.
+ Fix issue with support of openSUSE Leap 15.3 (k8s/crio).


####Other Updates
+ Helm chart update appVersion to 5.0.0 and chart version to 2.2.0
+ Removed serverless scanning feature/menu.
+ Removed support for Jfrog Xray scan result integration (Artifactory registry scan is still supported).
+ Support for deployment on ECS is no longer provided. The allinone should still be able to be deployed on ECS, however, the documentation of the steps and settings is no longer supported.
+ Starting with 5.0.0, software license keys are no longer needed. NeuVector 5.0.0 is completely open source.


### Upgrading from NeuVector 4.x to 5.x

For Helm users, update to NeuVector Helm chart 2.0.0 or later. If updating an Operator or Helm install on OpenShift, see note below.

1. Delete old neuvector-binding-customresourcedefinition clusterrole
```
kubectl delete clusterrole neuvector-binding-customresourcedefinition
```

2. Apply new update verb for neuvector-binding-customresourcedefinition clusterrole
```
kubectl create clusterrole neuvector-binding-customresourcedefinition --verb=watch,create,get,update --resource=customresourcedefinitions
```

3. Delete old crd schema for Kubernetes 1.19+
```
kubectl delete -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/crd-k8s-1.19.yaml
```

4. Create new crd schema for Kubernetes 1.19+
```
kubectl apply -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.0.0/crd-k8s-1.19.yaml
kubectl apply -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.0.0/waf-crd-k8s-1.19.yaml
kubectl apply -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.0.0/dlp-crd-k8s-1.19.yaml
kubectl apply -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/5.0.0/admission-crd-k8s-1.19.yaml
```

5. Create a new Admission, DLP and WAF clusterrole and clusterrolebinding
```
kubectl create clusterrole neuvector-binding-nvwafsecurityrules --verb=list,delete --resource=nvwafsecurityrules
kubectl create clusterrolebinding neuvector-binding-nvwafsecurityrules --clusterrole=neuvector-binding-nvwafsecurityrules --serviceaccount=neuvector:default
kubectl create clusterrole neuvector-binding-nvadmissioncontrolsecurityrules --verb=list,delete --resource=nvadmissioncontrolsecurityrules
kubectl create clusterrolebinding neuvector-binding-nvadmissioncontrolsecurityrules --clusterrole=neuvector-binding-nvadmissioncontrolsecurityrules --serviceaccount=neuvector:default
kubectl create clusterrole neuvector-binding-nvdlpsecurityrules --verb=list,delete --resource=nvdlpsecurityrules
kubectl create clusterrolebinding neuvector-binding-nvdlpsecurityrules --clusterrole=neuvector-binding-nvdlpsecurityrules --serviceaccount=neuvector:default
```

6. Update image names and paths for pulling NeuVector images from Docker hub (docker.io), e.g.
+ neuvector/manager:5.0.0
+ neuvector/controller:5.0.0
+ neuvector/enforcer:5.0.0
+ neuvector/scanner:latest
+ neuvector/updater:latest

Optionally, remove any references to the NeuVector license and registry secret in Helm charts, deployment yaml, configmap, scripts etc, as these are no longer required to pull the images or to start using NeuVector.

**Note about SCC and Upgrading via Operator/Helm**

Privileged SCC is added to the Service Account specified in the deployment yaml by Operator version 1.3.4 and above in new deployments. In the case of upgrading the NeuVector Operator from a previous version to 1.3.4 or Helm to 2.0.0, please delete Privileged SCC before upgrading.
```
oc delete rolebinding -n neuvector system:openshift:scc:privileged
```

#### Beta 1 version released April 2022
+ Feature complete, including Automated Promotion of Group Modes. Promotes a Group’s protection Mode based on elapsed time and criteria. Does not apply to CRD created Groups. This features allows a new application to run in Discover for some time period, learning the behavior and NeuVector creating allow-list rules for Network and Process, then automatically moving to Monitor, then Protect mode. Discover to Monitor criterion: Elapsed time for learning all network and process activity of at least one live pod in the Group. Monitor to Protect criterion: There are no security events (network, process etc) for the timeframe set for the Group.
+ Support for Rancher 2.6.5 Apps and Marketplace chart. Deploys into cattle-neuvector-system namespace and enables SSO from Rancher to NeuVector. Note: Previous deployments from Rancher (e.g. Partner catalog charts, version 1.9.x and earlier), must be completely removed in order to update to the new chart.
+ Tags for Enforcer, Manager, Controller: 5.0.0-b1 (e.g. neuvector/controller:5.0.0-b1)


####Preview.3 version released March 2022
***Important***: To update previous preview deployments for new CRD WAF, DLP and Admission control features, please update the CRD yaml and add new rbac/role bindings:
```
kubectl apply -f https://raw.githubusercontent.com/neuvector/manifests/main/kubernetes/latest/crd-k8s-1.19.yaml
kubectl create clusterrole neuvector-binding-nvwafsecurityrules --verb=list,delete --resource=nvwafsecurityrules
kubectl create clusterrolebinding neuvector-binding-nvwafsecurityrules --clusterrole=neuvector-binding-nvwafsecurityrules --serviceaccount=neuvector:default
kubectl create clusterrole neuvector-binding-nvadmissioncontrolsecurityrules --verb=list,delete --resource=nvadmissioncontrolsecurityrules
kubectl create clusterrolebinding neuvector-binding-nvadmissioncontrolsecurityrules --clusterrole=neuvector-binding-nvadmissioncontrolsecurityrules --serviceaccount=neuvector:default
kubectl create clusterrole neuvector-binding-nvdlpsecurityrules --verb=list,delete --resource=nvdlpsecurityrules
kubectl create clusterrolebinding neuvector-binding-nvdlpsecurityrules --clusterrole=neuvector-binding-nvdlpsecurityrules --serviceaccount=neuvector:default
```
#####Enhancements
+ Support scanning of SUSE Linux (SLE, SLES), and Microsoft Mariner
+ Zero-drift process and file protection. This is the new default mode for process and file protections. Zero-drift automatically allows only processes which originate from the parent process that is in the original container image, and does not allow file updates or new files to be installed. When in Discover or Monitor mode, zero-drift will alert on any suspicious process or file activity. In Protect mode, it will block such activity. Zero-drift does not require processes to be learned or added to an allow-list. Disabling zero-drift for a group will cause the process and file rules listed for the group to take effect instead.
+ Split policy mode protection for network, process/file. There is now a global setting available in Settings -> Configuration to separately set the network protection mode for enforcement of network rules. Enabling this (default is disabled), causes all network rules to be in the protection mode selected (Discover, Monitor, Protect), while process/file rules remain in the protection mode for that Group, as displayed in the Policy -> Groups screen. In this way, network rules can be set to Protect (blocking), while process/file policy can be set to Monitor, or vice versa.
+ WAF rule detection, enhanced DLP rules (header, URL, full packet)
+ CRD for WAF, DLP and admission controls. NOTE: required additional cluster role bindings/permissions. See Kubernetes and OpenShift deployment sections. CRD import/export and versioning for admission controls supported through CRD. 
+ Rancher SSO integration to launch NeuVector console through Rancher Manager. This feature is only available if the NeuVector containers are deployed through Rancher. NOTE: Requires updated Rancher release (date/version TBD).
+ Supports deployment on RKE2.
+ Support for Federation of clusters (multi-cluster manager) through a proxy.
+ Monitor required rbac's clusterrole/bindings and alert in events and UI if any are missing.
+ Support criteria of resource limitations in admission control rules.


#####Bug Fixes
+ Fix issue of worker federation role backup should restore into non-federated clusters.

####Preview.2 version released Feb 2022
+ Minor file and license changes in source, no features added.

####Support for deployment on AWS ECS Deprecated
Support for deployment on ECS is no longer provided. The allinone should still be able to be deployed on ECS, however, the documentation of the steps and settings is no longer supported.

#### 5.0 'Tech Preview' January 2022
##### Enhancements
+ First release of an unsupported, 'tech-preview' version of NeuVector 5.0 open source version.
+ Add support for OWASP Top-10, WAF-like rules for detecting network attacks in headers or body. Includes support for CRD definitions of signatures and application to appropriate Groups.
+ Removes Serverless scanning features.

##### Bug Fixes
+ TBD

##### Other
+ Helm chart v1.8.9 is published for 5.0.0 deployments. If using this with the preview version of 5.0.0 the following changes should be made to values.yml:
  - Update the registry to docker.io
  - Update image names/tags to the preview version on Docker hub
  - Leave the imagePullSecrets empty





