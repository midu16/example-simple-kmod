# DriverToolKit

## Purpose 

In this section we are going to show-case :

- How to build an out-of-tree kernel driver, for this example we have used `simple-kmod`.
- How to apply the out-of-tree kernel driver to your OCP cluster in the following stages:
    - as day2 operation;
    - as day1 operation.

## How to build the container-base image:

- Create the `namespace`:

```bash
$ oc create -f 01_create_namespace.yaml
```
! Note: Before proceeding with the `container base image` using the DriverToolKit we will need to configure [Image Registry Operator][imageregistryoperator]

[imageregistryoperator]: https://docs.openshift.com/container-platform/4.12/registry/configuring_registry_storage/configuring-registry-storage-baremetal.html#registry-removed_configuring-registry-storage-baremetal

- Configuring the `Image Registry Operator`:

```bash
$ oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'
config.imageregistry.operator.openshift.io/cluster patched
```

- Validate the state of the `Image Registry Operator`:

```bash
$ oc get clusteroperator image-registry
NAME             VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
image-registry   4.12.11   False       False         True       58s     Available: Error: storage backend not configured...
```
! Note: At this point we need to configure the storage backend.

- Configure the backend storage:

! Note: In this section we are going to configure the [block storage back-end storage][blockregistrystorage]. Please note that this is not recomanded for production wise clusters.

[blockregistrystorage]: https://docs.openshift.com/container-platform/4.12/registry/configuring_registry_storage/configuring-registry-storage-baremetal.html#installation-registry-storage-block-recreate-rollout-bare-metal_configuring-registry-storage-baremetal

```bash
oc patch config.imageregistry.operator.openshift.io/cluster --type=merge -p '{"spec":{"rolloutStrategy":"Recreate","replicas":1}}'
``` 

- Validate that the `Image Registry Operator` has been configured:

```bash
$ oc get clusteroperator image-registry
NAME             VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE   MESSAGE
image-registry   4.12.11   True        False         False      3m31s
```

- Building the container base image for each OCP release:

! Note: Make sure that the following parameters are according to your OCP release.

```bash
$ oc create -f 0000-buildconfig.yaml
```

- Validate that the image has been build succesfully:

```bash
$ oc get pods
NAME                               READY   STATUS      RESTARTS   AGE
simple-kmod-driver-build-1-build   0/1     Completed   0          2m7s
$ oc get all
NAME                                   READY   STATUS      RESTARTS   AGE
pod/simple-kmod-driver-build-1-build   0/1     Completed   0          103s

NAME                                                      TYPE     FROM         LATEST
buildconfig.build.openshift.io/simple-kmod-driver-build   Docker   Dockerfile   1

NAME                                                  TYPE     FROM         STATUS     STARTED              DURATION
build.build.openshift.io/simple-kmod-driver-build-1   Docker   Dockerfile   Complete   About a minute ago   58s

NAME                                                          IMAGE REPOSITORY                                                                                 TAGS   UPDATED
imagestream.image.openshift.io/simple-kmod-driver-container   image-registry.openshift-image-registry.svc:5000/simple-kmod-demo/simple-kmod-driver-container   demo   49 seconds ago
```

- Applying the `out-of-tree` kernel driver using DTK:

    - Switch to the `simple-kmod` namespace:
    ```bash
    $ oc project simple-kmod
    ```

    ! Note: Before applying the below step, you will need to enable the `serviceAccount` role capability to be able to push the new image in the internal registry: `oc policy add-role-to-user system:image-builder system:serviceaccount:pushed:pusher`

    - Apply the `out-of-tree` kernel driver:
    ```bash
    $ oc create -f 1000-drivercontainer.yaml
    ```
! Note: the `container-base-image` used in this demo was made publically available in the following [repo][simple-kmod-driver-container-image].

[simple-kmod-driver-container-image]: quay.io/midu/simple-kmod-driver-container:4.12.11

- Validating that the `out-of-tree` kernel driver:

```bash
$ oc get all
NAME                                     READY   STATUS              RESTARTS   AGE
pod/simple-kmod-driver-build-1-build     0/1     Completed           0          14m
pod/simple-kmod-driver-container-vkjqb   0/1     ContainerCreating   0          2s

NAME                                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                     AGE
daemonset.apps/simple-kmod-driver-container   1         1         0       1            0           node-role.kubernetes.io/worker=   2s

NAME                                                      TYPE     FROM         LATEST
buildconfig.build.openshift.io/simple-kmod-driver-build   Docker   Dockerfile   1

NAME                                                  TYPE     FROM         STATUS     STARTED          DURATION
build.build.openshift.io/simple-kmod-driver-build-1   Docker   Dockerfile   Complete   14 minutes ago   58s

NAME                                                          IMAGE REPOSITORY                                                                                                     TAGS   UPDATED
imagestream.image.openshift.io/simple-kmod-driver-container   default-route-openshift-image-registry.apps.someone-test.test412.com/simple-kmod-demo/simple-kmod-driver-container   demo   14 minutes ago
$ oc get all
NAME                                     READY   STATUS      RESTARTS   AGE
pod/simple-kmod-driver-build-1-build     0/1     Completed   0          14m
pod/simple-kmod-driver-container-vkjqb   1/1     Running     0          4s

NAME                                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                     AGE
daemonset.apps/simple-kmod-driver-container   1         1         1       1            1           node-role.kubernetes.io/worker=   4s

NAME                                                      TYPE     FROM         LATEST
buildconfig.build.openshift.io/simple-kmod-driver-build   Docker   Dockerfile   1

NAME                                                  TYPE     FROM         STATUS     STARTED          DURATION
build.build.openshift.io/simple-kmod-driver-build-1   Docker   Dockerfile   Complete   14 minutes ago   58s

NAME                                                          IMAGE REPOSITORY                                                                                                     TAGS   UPDATED
imagestream.image.openshift.io/simple-kmod-driver-container   default-route-openshift-image-registry.apps.someone-test.test412.com/simple-kmod-demo/simple-kmod-driver-container   demo   14 minutes ago
```
- Verify that the kernel module is loaded successfully on the host machines:

    - by accessing the container created by DTK:
    ```bash
    $ oc exec -it pod/simple-kmod-driver-container-vkjqb -- lsmod | grep simple
    simple_procfs_kmod     16384  0
    simple_kmod            16384  0
    ```
    - by accessing the host machine:
    ```bash
    $ ssh core@192.168.122.200
    Red Hat Enterprise Linux CoreOS 412.86.202303312131-0
    Part of OpenShift 4.12, RHCOS is a Kubernetes native operating system
    managed by the Machine Config Operator (`clusteroperator/machine-config`).

    WARNING: Direct SSH access to machines is not recommended; instead,
    make configuration changes via `machineconfig` objects:
    https://docs.openshift.com/container-platform/4.12/architecture/architecture-rhcos.html

    ---
    Last login: Sun Apr 23 04:21:00 2023 from 192.168.122.1
    [core@someone-test-sno ~]$ sudo lsmod | grep simple
    simple_procfs_kmod     16384  0
    simple_kmod            16384  0
    ```

## CPU/Memory usage of resources created by DTK

- Memory usage by the `namespace` resource created by DTK:

The following metric has been used in Prometheus: `namespace:container_memory_usage_bytes:sum{namespace="simple-kmod-demo"}`

![namespace-memory-usage](https://github.com/midu16/rhcos-layering/blob/d475e9f977c8d45b3046718734583475b2a47e1e/simple-kmod/dtk-simple-kmod/screen/Screenshot%202023-04-23%20at%2007-02-14%20Metrics%20%C2%B7%20Red%20Hat%20OpenShift.png)

- CPU usage by the `nammespace` resource created by DTK:

The following metric has been used in Prometheus: `namespace:container_cpu_usage:sum{namespace="simple-kmod-demo"}`

![namespace-cpu-usage](https://github.com/midu16/rhcos-layering/blob/d475e9f977c8d45b3046718734583475b2a47e1e/simple-kmod/dtk-simple-kmod/screen/Screenshot%202023-04-23%20at%2007-01-26%20Metrics%20%C2%B7%20Red%20Hat%20OpenShift.png)


## Resources

1. [How to fix the x509: certificate signed by unknown authority on login OpenShift internal registry][certificatesignedbyunknownauthority]

[certificatesignedbyunknownauthority]: https://access.redhat.com/solutions/6088891