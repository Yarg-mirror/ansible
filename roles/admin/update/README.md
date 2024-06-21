Update
=========

Update and Upgrade systems and manage reboot if needed

Requirements
------------

none

Role Variables
--------------

none

Dependencies
------------

none

Example Playbook
----------------

    - hosts: servers
      become: yes
      roles:
         - role: update

License
-------

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
