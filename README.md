# OpenShift Automation Tools

[![License: Apache v2](https://img.shields.io/badge/license-Apache%20v2-brightgreen.svg)](https://www.apache.org/licenses/LICENSE-2.0)

Tools to automate OpenShift installation and maintenance. These scripts 
and playbooks allow fully automated connected or disconnected OpenShift 
installation and updates for those parts not automated by the standard 
OpenShift playbooks. Can be used to quickly setup a PoC environment or 
to experiment with OpenShift installation parameters. Local registry use 
will avoid fetching gigabytes of data from external network. Automation 
for platform level tasks like OS upgrades.

## Technical Overview

Official OpenShift documentation describes how to manually prepare nodes 
for installation, how to configure local registry for disconnected 
installations, and how to upgrade OpenShift cluster nodes' OS packages. 
However, currently (as of OCP 3.5 / 2017-06) no automated tools for 
these tasks are provided. Automation for additional tasks like complete 
uninstallation, setting up cluster wide Cockpit or configuring 
passwordless cross-node SSH access are also provided.

The following sections will describe the playbooks and scripts in 
detail. The provided Ansible inventory files provide a quick starting 
point with most commonly used parameters included. The playbooks and 
scripts have been tested both with RHEL and RHEL Atomic.

This repository contains branches for each tested OCP release, the first 
being OpenShift Container Platform 3.4. Before running any of the 
scripts or playbooks it is required to investigate their contents and 
adjust them for the local environment as needed. Playbooks are provided 
in standalone form to make it easier to copy and run them anywhere.

For introduction to OpenShift and official documentation, see

https://docs.openshift.com/container-platform/latest/welcome/index.html

For related supported and more feature-rich alternatives suitable for 
production environments, please consider
[Red Hat Satellite](https://www.redhat.com/en/technologies/management/satellite)
and
[Red Hat CloudForms](https://www.redhat.com/en/technologies/management/cloudforms).

## Prerequisites

* Environment suitable for OpenShift installation
  * https://docs.openshift.com/container-platform/latest/install_config/install/prerequisites.html
* Basic understanding of OpenShift Container Platform, Ansible,
  Kubernetes, and shell scripts
  * For OpenShift documentation see the [OpenShift documentation](https://docs.openshift.com/container-platform/latest/welcome/index.html)
  * For Kubernetes basics see http://kubernetesbyexample.com/

## Detailed Description

* Generic playbooks and scripts
  * [local-registry-setup-v2](bin/local-registry-setup-v2) - a simple script
    to setup a secured local Docker v2 registry. Creates and uses a self-
    signed cert (/etc/docker-distribution/registry/cert.crt).
  * [local-registry-sync](bin/local-registry-sync) - a script to sync all
    the needed container images needed for disconnected installations.
    This script will sync only the latest available versions to avoid
    downloading old and unnecessary images. After running the script the
    local registry contents (/etc/docker-distribution and /var/lib/registry)
    can be copied to other systems for example with a USB stick in case of
    fully disconnected environment. For a fully-featured and supported
    local registry, see:
    https://docs.openshift.com/container-platform/latest/install_config/install/stand_alone_registry.html
  * [ansible.cfg](conf/ansible.cfg) - Ansible example configuration
    with some optimizations to slightly reduce installation time and
    provide additional control and diagnostics.
  * [rhel-7-ocp.ks](conf/rhel-7-ocp.ks) - an example RHEL kickstart file
    to install OpenShift cluster hosts.
  * [ansible.hosts.compact](conf/ansible.hosts.compact) and [ansible.hosts.full](conf/ansible.hosts.full) -
    Ansible example inventory files for OpenShift Container Platform
    installation, compact and full variants.
  * [prep.yml](conf/prep.yml) - a playbook to automate host preparation
    described in https://docs.openshift.com/container-platform/latest/install_config/install/host_preparation.html.
    Docker storage configuration needs to be adjusted in the [template
    file](conf/docker-storage-setup.j2) for local environment. Supports
    either [Red Hat Subscription-Manager](https://access.redhat.com/solutions/253273)
    or .repo file based repository configuration (for .repo file based
    configuration the [repo/](repo/) directory must be populated locally
    with suitable .repo files). This playbook expects to find the local
    registry cert as conf/cert.crt if using a local registry.
  * [install-os-updates.yml](conf/install-os-updates.yml) - playbook to
    automatically and transparently apply available OS updates on nodes
    on live OpenShift clusters. Automates the steps described in
    https://docs.openshift.com/container-platform/latest/install_config/upgrading/os_upgrades.html.
  * [cross-node-cockpit.yml](conf/cross-node-cockpit.yml) - a playbook
    to configure [Cockpit](http://cockpit-project.org/) for all nodes
    in the OpenShift cluster with masters running the Cockpit Web UI.
    Note that this setup is unsupported and for PoC environments only.
    For a supported monitoring solution, use 
    [Red Hat CloudForms](https://www.redhat.com/en/technologies/management/cloudforms).
    Other alternatives for Kubernetes/OpenShift monitoring include
    https://blog.openshift.com/full-cluster-capacity-management-monitoring-openshift/
    and
    https://github.com/kubernetes/dashboard.
  * [uninst.yml](conf/uninst.yml) - a playbook to complete OpenShift
    uninstallation. See also https://github.com/openshift/openshift-ansible/issues/3082.
* Less useful playbooks and scripts
  * [auth.yml](conf/auth.yml) and [post.yml](conf/post.yml) - trivial
    playbooks to finalize OpenShift installation.
  * [cross-node-ssh.yml](conf/cross-node-ssh.yml) - a playbook to
    configure cross-node SSH access across all the cluster nodes.
  * [local-registry-setup-v1](bin/local-registry-setup-v1) - a simple script
    to setup a secured local Docker v1 registry (v1 registry is deprecated).
  * [speed-up.yml](conf/speed-up.yml) - a playbook to (maybe) speed up
    OpenShift installations by prefetching all the needed images on
    selected nodes prior installation.
  * [wipe-nfs.yml](conf/wipe-nfs.yml) and [wipe-docker.yml](conf/wipe-docker.yml) -
    playbooks to wipe out all NFS and Docker data. Use with caution!

There are also several helper scripts under [bin/](bin/) for installing 
OCP on KVM/libvirt. These scripts will require some customization prior 
using and thus are not ready to be used out of the box but they may be 
helpful in some cases. In a properly prepared environment, the 
[openshift-install](bin/openshift-install) or 
[openshift-install-atomic](bin/openshift-install-atomic) allow for a 
one-step installation of the entire OpenShift platform starting from 
RHEL/Atomic installation on VMs and then installing and finalizing 
OpenShift cluster. For more serious tools see the earlier mentioned 
supported products.

## License

Apache v2
