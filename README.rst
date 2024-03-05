=============================================
OpenStack Configuration
=============================================

This project contains Ansible playbooks and configuration of infrastructure on
an existing OpenStack cloud for the OpenStack system. (Supported up to Yoga
release.)

Preparation
===========

Ensure that Ansible is installed, either via the system package manager or pip.
If required, use a virtualenv to avoid interference with the system python
packages. For example:

.. code-block::

   $ virtualenv venv
   $ source venv/bin/activate
   $ python -m pip install --upgrade pip
   $ pip install -r requirements.txt

Install Ansible role and collection dependencies from Ansible Galaxy:

.. code-block::
   $ ansible-galaxy collection install \
       -p ansible/collections \
       -r requirements.yml

Configuration
=============

Configuration should be added to ``etc/openstack-config/openstack-config.yml``.
Examples are provided in the ``examples`` directory.

Usage
=====

First, ensure that OpenStack authentication environment variables are set,
typically by sourcing an OpenStack environment file. If a Kayobe environment
was already configured, you can use the following command:

.. code-block::

   $ source ${KOLLA_CONFIG_PATH}/public-openrc.sh

If any Ansible variable is encrypted with Ansible Vault, make sure the
``ANSIBLE_VAULT_PASSWORD_FILE`` environment variable is set:

.. code-block::

   $ export ANSIBLE_VAULT_PASSWORD_FILE=<path-to-vault-password-file>

To configure OpenStack infrastructure:

.. code-block::

   $ tools/openstack-config

To run a specific playbook:

.. code-block::

   $ tools/openstack-config -p </path/to/playbook>

To specify additional arguments to ``ansible-playbook``, separate them with a
double hyphen (``--``):

.. code-block::

   $ tools/openstack-config -- <arguments>

For example, a vault secret stored as a file can be passed as an extra
configuration parameter:

.. code-block::

   $ tools/openstack-config -- --vault-password-file config-secret.vault


Magnum Cluster Templates
========================

To generate a new set of Magnum cluster templates and corresponding Glance image
definitions which utilise the latest stable upstream release tag, set the following
variables in `etc/openstack-config.yml`

.. code-block:: yaml

   magnum_default_flavor_name: # Chosen flavor on target cloud
   magnum_external_net_name: # External network
   magnum_loadbalancer_provider: # Octavia provider (e.g. 'ovn')

then run the provided playbook with

.. code-block:: bash

   $ tools/openstack-config -p ansible/generate-magnum-capi-templates.yml

This will create a ``generated-magnum-snippets`` directory in the repo root with
a timestamped sub-directory containing an ``images.yml`` file and a ``templates.yml``
file. The contents of these two files can then be added to any existing images and
cluster templates in ``etc/openstack-config.yml``. When deploying the updated config,
be sure to run the ``openstack-images.yml`` playbook *before* running the
``openstack-container-clusters.yml`` playbook, otherwise the Magnum API will return
an error referencing an invalid cluster type with image ``None``. This is handled
automatically if running the full ``openstack.yml`` playbook.
