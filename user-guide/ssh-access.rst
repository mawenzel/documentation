.. _ssh-access:

==========
SSH Access
==========

Every team member can get access to any of the team's EC2 instances by using the :ref:`piu` command line tool:

.. code-block:: bash

    $ pip3 install --upgrade stups-piu
    $ # assumptions: region is Ireland, team name is "myteam", private EC2 instance has IP "172.31.146.1"
    $ piu 172.31.146.1 "Troubleshoot problem XY"
    # enter even URL (e.g. https://even.stups.example.org)
    # enter odd hostname "odd-eu-west-1.myteam.example.org"
    $ ssh -A odd-eu-west-1.myteam.example.org # agent-forwarding must be used!
    $ ssh 172.31.146.1 # jump from bastion to private instance

Piu will remember the URL of :ref:`even` and the hostname of :ref:`odd` in the local config file (``~/.config/piu/piu.yaml`` on Linux).
You can overwrite settings on the command line:

.. code-block:: bash

    $ piu -O odd-eu-west-1.myotherteam.example.org 172.31.1.1 test


