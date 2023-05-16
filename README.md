# Evaluation of out-of-tree driver management methods

- [Evaluation of out-of-tree driver management methods](#evaluation-of-out-of-tree-driver-management-methods)
  - [Purpose](#purpose)
  - [Prerequisites](#prerequisites)
  - [Environment used](#environment-used)
  - [Implementation](#implementation)
    - [RHCOS-Layering](#rhcos-layering)
    - [DriverToolKit](#drivertoolkit)
    - [KMM-Operator](#kmm-operator)
  - [Summary](#summary)
  - [Resources](#resources)

## Purpose 

In this section we are going to compare DriverToolKit, RHCOS-Layering and KernelModuleManagement-Operator from the following perspectives:

- Features
- Cluster resource consumptions
- Lifecycle management
- Cluster impact (i.e downtime, number of restarts, etc)
- Driver made available at cluster install time.

## Prerequisites 

In the implementation we are going to use:
- The [simple-kmod][simple-kmod-link] as a reference driver. This can be replaced with any other driver if the use-case is requiring.
- OCPv4.12.0+
- SNO (SingleNodeOpenShift)

[simple-kmod-link]: https://github.com/openshift-psap/simple-kmod.git

## Environment used

In the further tests a libvirt environment has been used, each node are using the following specifications:
- vCPU: 34
- Memory allocated: 40960MiB
- Root disk: 100GiB
- Application disks:
    - /dev/vdb: 100GiB
    - /dev/vdc: 100GiB
- OCP network type used: OVNKubernetes

## Implementation 

In this section we are going to discuss about the individual tools implementation.

### RHCOS-Layering

In this section we are going to discuss on how to [build][layering-simple-kmod] your `out-of-tree` kernel driver using RHCOS-Layering method.

The [Containerfile][layering-simple-kmod-containerfile] its reffering to two main container base images in your OCP base release, those will be used to produce your custom release which contains the `out-of-tree` kernel driver.

[layering-simple-kmod]: ./layering-simple-kmod/README.md
[layering-simple-kmod-containerfile]: ./layering-simple-kmod/Containerfile

Depending on what nodes of your cluster you have applied the layer image to, OpenShift Container Platform no longer automatically updates the node pool that uses the custom layered image. You become responsible to manually update your nodes as appropriate.
To update a node that uses a custom layered image, follow these general steps:
- The cluster automatically upgrades to version x.y.z+1, except for the nodes that use the custom layered image. It is not required to do a z-stream upgrade in the scenario that all your OpenShift Container Platform nodes are using the RHCOS layer. 
- You could then create a new Containerfile that references the updated OpenShift Container Platform image and the RPM that you had previously applied.
- Create a new machine config that points to the updated custom layered image.

### DriverToolKit

In this section we are going to discuss on how to [build][dkt-simple-kmod] your `out-of-tree` kernel driver using DriverToolKit method, what are the requirements in order to achieve this and what is the system resources consumption.


[dkt-simple-kmod]: ./dtk-simple-kmod/README.md
### KMM-Operator

In this section we are going to discuss on how to [build][kmm-simple-kmod] your `out-of-tree` kernel driver using KMM-Operator method, what are the requirements in order to achieve this and what is the system resources consumption.

[kmm-simple-kmod]: ./kmm-simple-kmod/README.md


## Summary 

| Out-Of-Tree method  | Additional resource consumption      | Day1  | Day2 | Upgrade rollout  | Memory consumption [B] |
|---------------------|--------------------------------------|-------|------|------------------| ---------------------- |
| RHCOS Layering      | -                                    | yes   | yes  |  no              |  #N/A                  |
| DriverToolKit       | yes                                  | yes   | yes  |  yes             |  800k - 1M             |
| KMM Operator        | yes                                  | yes   | yes  |  yes             |                        |

## Resources

1. [DriverToolKit][dtk]
2. [KMM][kmm]
3. 
4. [PauseMCPReboot][disable-reboot]

[dtk]: https://docs.openshift.com/container-platform/4.12/hardware_enablement/psap-driver-toolkit.html

[kmm]: https://docs.okd.io/4.12/hardware_enablement/kmm-kernel-module-management.html

[disable-reboot]: https://access.redhat.com/solutions/5477811
