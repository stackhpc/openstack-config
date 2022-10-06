=============================================
OpenStack Configuration
=============================================

This project contains Ansible playbooks and configuration of infrastructure on
an existing OpenStack cloud for the OpenStack
system.

Preparation
===========

Ensure that Ansible is installed, either via the system package manager or pip.
If required, use a virtualenv to avoid interference with the system python
packages. For example:

.. code-block::

   $ virtualenv venv
   $ source venv/bin/activate
   $ pip install -U pip
   $ pip install -r requirements.txt

Install Ansible role and collection dependencies from Ansible Galaxy:

.. code-block::

   $ ansible-galaxy role install \
        -p ansible/roles \
        -r requirements.yml

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
