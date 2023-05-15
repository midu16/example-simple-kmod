# RHCOS-Layering

## Purpose 

In this section we are going to show-case :

- How to build an out-of-tree kernel driver, for this example we have used `simple-kmod`.
- How to apply the out-of-tree kernel driver to your OCP cluster in the following stages:
    - as day2 operation;
    - as day1 operation.


## How to build the container-base image:

- Building the container base image for each OCP release:

```bash
$ export OCP_VERSION="4.12.0"
$ RHEL_COREOS_NAME=$(curl -s https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$OCP_VERSION/release.txt | grep -m1 'rhel-coreos-8' | awk -F ' ' '{print $2}')
$ DRIVER_TOOLKIT_NAME=$(curl -s https://mirror.openshift.com/pub/openshift-v4/clients/ocp/$OCP_VERSION/release.txt | grep -m1 'driver-toolkit' | awk -F ' ' '{print $2}')
$ podman build -t simple-kmod:4.12.0 --no-cache --build-arg rhel_coreos_release=${RHEL_COREOS_NAME} --build-arg driver_toolkit_release=${DRIVER_TOOLKIT_NAME} .
[1/3] STEP 1/2: FROM quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:6db665511f305ef230a2c752d836fe073e80550dc21cede3c55cf44db01db365 AS kernel-query
[1/3] STEP 2/2: RUN rpm -qa kernel --queryformat '%{VERSION}-%{RELEASE}.%{ARCH}' > /kernel-version.txt
--> 4cfc9b8e6c5
[2/3] STEP 1/7: FROM quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:b53883ca2bac5925857148c4a1abc300ced96c222498e3bc134fe7ce3a1dd404 AS builder
Trying to pull quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:b53883ca2bac5925857148c4a1abc300ced96c222498e3bc134fe7ce3a1dd404...
Getting image source signatures
Copying blob 058a59a2efb7 done
Copying blob d8190195889e skipped: already exists
Copying blob f0f4937bc70f done
Copying blob 833de2b0ccff done
Copying blob 97da74cc6d8f skipped: already exists
Copying config 157d46e25a done
Writing manifest to image destination
Storing signatures
[2/3] STEP 2/7: COPY --from=kernel-query /kernel-version.txt /kernel-version.txt
--> e8d10db31b2
[2/3] STEP 3/7: RUN dnf install -y make gcc
Updating Subscription Management repositories.
Unable to read consumer identity
Subscription Manager is operating in container mode.

This system is not registered with an entitlement server. You can use subscription-manager to register.

Red Hat Universal Base Image 8 (RPMs) - BaseOS  939 kB/s | 837 kB     00:00
Red Hat Universal Base Image 8 (RPMs) - AppStre 961 kB/s | 3.2 MB     00:03
Red Hat Universal Base Image 8 (RPMs) - CodeRea 172 kB/s |  30 kB     00:00
Package make-1:4.2.1-11.el8.x86_64 is already installed.
Package gcc-8.5.0-10.1.el8_6.x86_64 is already installed.
Dependencies resolved.
================================================================================
 Package       Architecture Version                 Repository             Size
================================================================================
Upgrading:
 cpp           x86_64       8.5.0-16.el8_7          ubi-8-appstream        10 M
 gcc           x86_64       8.5.0-16.el8_7          ubi-8-appstream        23 M
 libgcc        x86_64       8.5.0-16.el8_7          ubi-8-baseos           81 k
 libgomp       x86_64       8.5.0-16.el8_7          ubi-8-baseos          207 k

Transaction Summary
================================================================================
Upgrade  4 Packages

Total download size: 34 M
Downloading Packages:
(1/4): libgcc-8.5.0-16.el8_7.x86_64.rpm         431 kB/s |  81 kB     00:00
(2/4): libgomp-8.5.0-16.el8_7.x86_64.rpm         94 kB/s | 207 kB     00:02
(3/4): cpp-8.5.0-16.el8_7.x86_64.rpm            118 kB/s |  10 MB     01:30
(4/4): gcc-8.5.0-16.el8_7.x86_64.rpm            256 kB/s |  23 MB     01:33
--------------------------------------------------------------------------------
Total                                           372 kB/s |  34 MB     01:34
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1
  Upgrading        : cpp-8.5.0-16.el8_7.x86_64                              1/8
  Running scriptlet: cpp-8.5.0-16.el8_7.x86_64                              1/8
  Upgrading        : libgomp-8.5.0-16.el8_7.x86_64                          2/8
  Running scriptlet: libgomp-8.5.0-16.el8_7.x86_64                          2/8
  Upgrading        : libgcc-8.5.0-16.el8_7.x86_64                           3/8
  Running scriptlet: libgcc-8.5.0-16.el8_7.x86_64                           3/8
  Upgrading        : gcc-8.5.0-16.el8_7.x86_64                              4/8
  Running scriptlet: gcc-8.5.0-16.el8_7.x86_64                              4/8
  Running scriptlet: gcc-8.5.0-10.1.el8_6.x86_64                            5/8
  Cleanup          : gcc-8.5.0-10.1.el8_6.x86_64                            5/8
  Running scriptlet: cpp-8.5.0-10.1.el8_6.x86_64                            6/8
  Cleanup          : cpp-8.5.0-10.1.el8_6.x86_64                            6/8
  Cleanup          : libgcc-8.5.0-10.1.el8_6.x86_64                         7/8
  Running scriptlet: libgcc-8.5.0-10.1.el8_6.x86_64                         7/8
  Running scriptlet: libgomp-8.5.0-10.1.el8_6.x86_64                        8/8
  Cleanup          : libgomp-8.5.0-10.1.el8_6.x86_64                        8/8
  Running scriptlet: libgomp-8.5.0-10.1.el8_6.x86_64                        8/8
  Verifying        : libgcc-8.5.0-16.el8_7.x86_64                           1/8
  Verifying        : libgcc-8.5.0-10.1.el8_6.x86_64                         2/8
  Verifying        : libgomp-8.5.0-16.el8_7.x86_64                          3/8
  Verifying        : libgomp-8.5.0-10.1.el8_6.x86_64                        4/8
  Verifying        : cpp-8.5.0-16.el8_7.x86_64                              5/8
  Verifying        : cpp-8.5.0-10.1.el8_6.x86_64                            6/8
  Verifying        : gcc-8.5.0-16.el8_7.x86_64                              7/8
  Verifying        : gcc-8.5.0-10.1.el8_6.x86_64                            8/8
Installed products updated.

Upgraded:
  cpp-8.5.0-16.el8_7.x86_64              gcc-8.5.0-16.el8_7.x86_64
  libgcc-8.5.0-16.el8_7.x86_64           libgomp-8.5.0-16.el8_7.x86_64

Complete!
--> 614ae6791ca
[2/3] STEP 4/7: WORKDIR /
--> b79a1a91f0c
[2/3] STEP 5/7: RUN git clone https://github.com/openshift-psap/simple-kmod.git
Cloning into 'simple-kmod'...
--> 666ddb7c73b
[2/3] STEP 6/7: WORKDIR /simple-kmod
--> cf3973424b3
[2/3] STEP 7/7: RUN make all KVER=$(cat /kernel-version.txt)
make -C /lib/modules/4.18.0-372.40.1.el8_6.x86_64/build M=/simple-kmod EXTRA_CFLAGS=-DKMODVER=\\\"e852852\\\" modules
make[1]: Entering directory '/usr/src/kernels/4.18.0-372.40.1.el8_6.x86_64'
  CC [M]  /simple-kmod/simple-kmod.o
  CC [M]  /simple-kmod/simple-procfs-kmod.o
  Building modules, stage 2.
  MODPOST 2 modules
  CC      /simple-kmod/simple-kmod.mod.o
  LD [M]  /simple-kmod/simple-kmod.ko
  CC      /simple-kmod/simple-procfs-kmod.mod.o
  LD [M]  /simple-kmod/simple-procfs-kmod.ko
make[1]: Leaving directory '/usr/src/kernels/4.18.0-372.40.1.el8_6.x86_64'
gcc -o spkut ./simple-procfs-kmod-userspace-tool.c
--> 8b18d3e519b
[3/3] STEP 1/4: FROM quay.io/openshift-release-dev/ocp-v4.0-art-dev@sha256:6db665511f305ef230a2c752d836fe073e80550dc21cede3c55cf44db01db365
[3/3] STEP 2/4: COPY --from=builder /simple-kmod /simple-kmod
--> e25d3945c77
[3/3] STEP 3/4: WORKDIR /simple-kmod
--> 83e840a670d
[3/3] STEP 4/4: RUN make install KVER=$(rpm -qa kernel --queryformat '%{VERSION}-%{RELEASE}.%{ARCH}') &&     rm -rf /simple-kmod &&     ostree container commit
install -v -m 755 spkut /bin/
'spkut' -> '/bin/spkut'
install -v -m 755 -d /lib/modules/4.18.0-372.40.1.el8_6.x86_64/
install -v -m 644 simple-kmod.ko        /lib/modules/4.18.0-372.40.1.el8_6.x86_64/simple-kmod.ko
'simple-kmod.ko' -> '/lib/modules/4.18.0-372.40.1.el8_6.x86_64/simple-kmod.ko'
install -v -m 644 simple-procfs-kmod.ko /lib/modules/4.18.0-372.40.1.el8_6.x86_64/simple-procfs-kmod.ko
'simple-procfs-kmod.ko' -> '/lib/modules/4.18.0-372.40.1.el8_6.x86_64/simple-procfs-kmod.ko'
depmod -F /lib/modules/4.18.0-372.40.1.el8_6.x86_64/System.map 4.18.0-372.40.1.el8_6.x86_64
[3/3] COMMIT simple-kmod:4.12.0
--> 12253cb0822
Successfully tagged localhost/simple-kmod:4.12.0
12253cb082269524cb7ddca5d5f11b7ae321f7821b0ae83c13f74197fbb396be
```

- Publish the container base image in `quay.io`:

```bash
$ podman tag localhost/simple-kmod:4.12.0 quay.io/midu/simple-kmod:4.12.0
$ podman push quay.io/midu/simple-kmod:4.12.0
```

## How to use the out-of-tree driver in a OCP cluster:

In this section we are going to detail the operations that are required to be applied for consuming the out-of-tree driver on a SNO or MNO cluster.

### How to apply the out-of-tree driver as `day2-operation`:

```bash
$ oc create -f 99-simple-kmod-master.yaml
```

```bash
$ oc create -f 99-simple-kmod-worker.yaml
```
### How to apply the out-of-tree driver as `day1-operation`:

