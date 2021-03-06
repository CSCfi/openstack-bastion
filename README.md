# Bastion hosts for OpenStack projects

Create one or more bastion hosts in an OpenStack project using Ansible and
Heat.

#### Table of contents

1. [Overview](#overview)
2. [Requirements](#requirements)
    * [Software](#software)
    * [Cloud environment](#cloud-environment)
3. [Usage](#usage)
4. [Extending](#extending)
    * [User management](#user-management)
5. [Contributors](#contributors)

## Overview

Bastion hosts are hosts that can be used as a jump point to an otherwise
inaccessible network. This is useful in the context of an OpenStack project as
you don't necessarily want to reserve a floating IP for all your instances. This
way you can isolate certain resources so that there is only a single point of
access to them and also conserve floating IP addresses so that you don't need as
big of a quota. Wikipedia has more information:

https://en.wikipedia.org/wiki/Bastion_host

The Ansible playbooks here are meant to create one or more such bastions hosts
in OpenStack tenants. Heat is used to describe the bastion host and the
OpenStack resources it depends on.

## Requirements

### Software

* Ansible >= 2.2
* shade >= 1.8.0

For instructions on how to install Ansible, see [the official
documentation](https://docs.ansible.com/).

Shade is a client library used by Ansible to communicate with OpenStack clouds.
The easiest way to install it is using pip:

```bash
$ pip install shade
```

### Cloud environment

You will also need access to an OpenStack cloud. The project you use for
deployment needs to have a network and a router in place. If floating IPs are
available for your project then this should already be the case.

The OpenStack installation needs to have at least Heat and Neutron in addition
to Nova. The version of the Heat template used here is for Liberty. If you want
to run against an older version of OpenStack, you will need to modify the
template version. It is not guaranteed that the template will work against
older versions of OpenStack. Contact your OpenStack operator's support if you
are unsure whether the cloud fulfills these requirements.

## Usage

Clone this repository to a directory on your machine. We will refer to your
local clone as `openstack-bastion` from from now on.

The functionality for deploying the Heat stack for the bastion host is
separated into its own role. This and other roles listed in requirements.yml
need to be installed first:

```bash
$ cd openstack-bastion
$ ansible-galaxy -f -r requirements.yml install
```

Note that the `-f` flag means force and will overwrite any version of the roles
installed under `openstack-bastion/roles` with new versions from the repository
URLs listed in requirements.yml.

You will need to get an openrc file from OpenStack so that Ansible can interact
with it. The easiest way to get it is to login to the web interface and go to
Compute -> Access & Security -> API Access -> Download OpenStack RC File. Once
you have the file, you will need to source it (the name of the file may be
different):

```bash
$ source openrc.sh
```

After that you can fill in the parameters for the Heat stack. First copy the
example Heat parameter file to the `openstack-bastion` directory under a new
name (the name given to the new file is important here):

```bash
$ cd openstack-bastion
$ cp roles/ansible-role-bastion-host/files/example-heat-vars.yml bastion-heat-vars.yml
```

Edit the file with your favorite editor and fill in all the variables. You can
find documentation about the variables in the Heat template under
`roles/ansible-role-bastion-host/files/bastion-heat-stack.yml`.

Once you have the variables filled in for your Heat stack and an openrc file
sourced in your environment, you can run Ansible to create the bastion host:

```bash
$ ansible-playbook site.yml
```

If you do not set a `heat_stack_name` variable, this will generate a bastion
Heat stack with a random human readable name. If you later want to update this
Heat stack, you can do so by specifying the Heat stack name for the command
above. If the randomly generated name was bastion-kops-chaste-thermo, you would
run:

```bash
$ ansible-playbook site.yml -e "heat_stack_name=bastion-kops-chaste-thermo"
```

## Extending

The playbooks in this repository only contain a basic installation for a bastion
host. If you want to use this for a real project, you probably need to add some
project specific customizations. You can either directly use the Ansible role
for creating the bastion host or you can fork this repository and extend it.

### User management

The `ansible-role-users` role is used to create users based on variables set in
group_vars. By default no users are created because the variables in
`group_vars/all/users.yml` are empty. If you want to create other users besides
the default user in the image, you need to add your users to the `adminusers`
and/or `admin_root_keys` dictionaries under `group_vars/all/users.yml`. You can
find documentation for how to fill in user accounts there in the repository for
ansible-role-users:

https://github.com/CSCfi/ansible-role-users

## Contributors

  * Risto Laurikainen - https://github.com/rlaurika
