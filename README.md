subscribe-repos
===============

This playbook configures a RHEL server to receive its updates from a reposync server.
On your reposync server register the server against RHN and then run the following script as a cron job (or use the setup-reposerver role):

--8<-- snip -----

    #!/bin/bash

    SERVERIP=1.2.3.4 ## change me

    cd /var/www/html/repos

    #Full or Diff sync the Repos
    reposync -n -d -l --downloadcomps --download-metadata

    ls -l | grep ^d | awk '{print $9}' | while read dirs; do
      echo $dirs
      if [ -f ${dirs}/comps.xml ]; then
         createrepo -v ${dirs}/ -g comps.xml
      else
         createrepo -v ${dirs}/
      fi

      rf=/var/www/html/repofiles/${dirs}.repo
      echo "[$dirs]" > $rf
      echo "name=$dirs" >> $rf
      echo "baseurl=http://${SERVERIP}/repos/$dirs/" >> $rf
      echo "enabled=1" >> $rf
      echo "gpgcheck=0" >> $rf

    done
--8<-- snip -----

This role is one of the first to run after the base installation to set the repositories

Requirements
------------

To use this role you need a properly configured repo server as explained above, serving the repos

Role Variables
--------------

You can then set the following variables in the playbook:

Define the URL that points to the directory, where the repofiles exist in "reponame.repo", e.g. rhel-7-server-rpms.repo

    reposrvurl: http://$SERVERIP/repofiles/

Set this variable to true if you want to remove/disable all previously existing repositories. The default is false

     repo_reset: true

Use this to define the list of repositories you want to subscribe to

     repositories:
                  - rhel-7-server-rpms
                  - repo2
                  - repo3

Example Playbook
----------------

Here is an example playbook that adds two repositories on the clients (hosts in group `clients`) and disables all previously set repositories. The repository server that contain the repo files is set in `reposrvurl`

    - hosts: clients
      remote_user: root

      vars:
          reposrvurl: http://myserver.lan/repofiles/
          repo_reset: true
          repositories:
                  - rhel-7-server-rpms
                  - rhel-sap-hana-for-rhel-7-server-rpms


      roles:
         - { role: mk-ansible-roles.subscribe-repos }

License
-------

Apache License
Version 2.0, January 2004

Author Information
------------------

Markus Koch

Please leave comments in the github repo issue list
