# KernelModuleManagement-Operator

## Table of Contents

- [KernelModuleManagement-Operator](#kernelmodulemanagement-operator)
  - [Table of Contents](#table-of-contents)
  - [Purpose](#purpose)
  - [Prerequisites](#prerequisites)
  - [Environment used](#environment-used)
  - [How to install KMM Operator on a SNO node](#how-to-install-kmm-operator-on-a-sno-node)
  - [CPU/Memory usage of resources created by KMM-Operator](#cpumemory-usage-of-resources-created-by-kmm-operator)
    - [KMM-Operator Memory Usage namespace layer](#kmm-operator-memory-usage-namespace-layer)
    - [KMM-Operator CPU Usage namespace layer](#kmm-operator-cpu-usage-namespace-layer)
  - [Configuring the KMM-Operator:](#configuring-the-kmm-operator)
  - [How to build and sign a out-of-tree kernel driver with KMM:](#how-to-build-and-sign-a-out-of-tree-kernel-driver-with-kmm)
  - [Architecture](#architecture)

## Purpose 

In this section we are going to show-case :

- How to build an out-of-tree kernel driver, for this example we have used `simple-kmod`.
- How to apply the out-of-tree kernel driver to your OCP cluster in the following stages:
    - as day2 operation;
    - as day1 operation.

## Prerequisites 

In the implementation we are going to use:
- The [simple-kmod][simple-kmod-link] as a reference driver. This can be replaced with any other driver if the use-case is requiring.
- OCPv4.12.0+
- SNO (SingleNodeOpenShift)
- KMMv1.0.0 

## Environment used

In the further tests a libvirt environment has been used, each node are using the following specifications:
- vCPU: 34
- Memory allocated: 40960MiB
- Root disk: 100GiB
- Application disks:
    - /dev/vdb: 100GiB
    - /dev/vdc: 100GiB
- OCP network type used: OVNKubernetes


## How to install KMM Operator on a SNO node

- Create the `openshift-kmm` namespace:

```bash
$ oc create -f 01_create_kmm_namespace.yaml
```

- Create the `OperatorGroup`:

```bash
$ oc create -f 02_create_kmm_operatorgroup.yaml
```

- Create the `Subscription`:

```bash
$ oc create -f 03_create_kmm_subscription.yaml
```
- Validate that the `KMM-Operator` has been deployed successfully:

```bash
$ oc get pods -n openshift-kmm
NAME                                               READY   STATUS    RESTARTS   AGE
kmm-operator-controller-manager-86b89b5547-8fpv5   2/2     Running   0          57m
[admin@INBACRNRDL0102 kcli-plan-openshift-4120ec5]$ oc get all -n openshift-kmm
NAME                                                   READY   STATUS    RESTARTS   AGE
pod/kmm-operator-controller-manager-86b89b5547-8fpv5   2/2     Running   0          58m

NAME                                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kmm-operator-controller-manager-metrics-service   ClusterIP   172.30.141.15   <none>        8443/TCP   58m

NAME                                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/kmm-operator-controller-manager   1/1     1            1           58m

NAME                                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/kmm-operator-controller-manager-86b89b5547   1         1         1       58m
$ oc get csv -n openshift-kmm
NAME                              DISPLAY                    VERSION   REPLACES   PHASE
kernel-module-management.v1.0.0   Kernel Module Management   1.0.0                Succeeded
```
## CPU/Memory usage of resources created by KMM-Operator

- Memory usage by the `namespace` resource created by KMM-Operator:

The following metric has been used in Prometheus: `namespace:container_memory_usage_bytes:sum{namespace="openshift-kmm"}`

### KMM-Operator Memory Usage namespace layer
![MEMORY-USAGE](screen/02_memory_usage_bytes_openshift_kmm.png)

- CPU usage by the `nammespace` resource created by KMM-Operator:

The following metric has been used in Prometheus: `namespace:container_cpu_usage:sum{namespace="openshift-kmm"}`

### KMM-Operator CPU Usage namespace layer
![CPU-USAGE](screen/01_cpu_usage_openshift_kmm.png)

## Configuring the KMM-Operator:

```bash
$ oc create ns kmm-tests
namespace/kmm-tests created
```
> **Note**
> <mark>The `kmm-tests` namespace in which the Module will be created.</mark>


## How to build and sign a out-of-tree kernel driver with KMM:

- Adding the keys for secureboot:

```bash
$ openssl req -x509 -new -nodes -utf8 -sha256 -days 36500 -batch -config configuration_file.config -outform DER -out my_signing_key_pub.der -keyout my_signing_key.priv
```



## Architecture

In the following Diagram, we are going to presen the `Hub-Spoke` KMM-Operator architecture :

![KMM-HUB-Spoke-Architecture](screen/KMM.png)



[deploy_kmod]: https://openshift-kmm.netlify.app/documentation/deploy_kmod/

[kmm_operator]: https://docs.openshift.com/container-platform/4.12/hardware_enablement/kmm-kernel-module-management.html

