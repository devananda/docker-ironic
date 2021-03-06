===================================
Docker image for Ironic development
===================================

This reposotiry contains the dockerfile(s) and related scripts, configs, etc,
for building a docker image usable for easy local development of Ironic.

Quick start
-----------

Run the ``build.sh`` script to build or refresh your docker image.  It's a single
"docker build" command that creates an image named "ironic:devel".

Run the following command to start the container, mount in your working dir,
and make the ports available locally::

  docker run -it -p 6385:6385 -p 35357:35357 -v LOCAL_IRONIC_SOURCE_DIR:/opt/source/ironic ironic:devel bash

Then, in the container, run this to launch and manage the services::

  supervisord && supervisorctl

On your host, source the included ``openrc`` file to set your ENV vars and
connect with your openstack client, or do development on python-ironicclient.
You should be able to edit any files in your ironic source dir and simply
restart services from within supervisorctl to pick up and test any changes.
