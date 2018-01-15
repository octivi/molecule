*******
Install
*******

Requirements
============

* `libvirtd`_ daemon (local or remote) with configured network and storage pool
* `virsh`_ binary (in Debian libvirt-clients package)

  .. _`Libvirtd`: https://libvirt.org

  .. _`virsh`: https://libvirt.org/sources/virshcmdref/html/

  .. _`libvirt-clients`: http://packages.debian.org/libvirt-clients

Install
=======

Install required software on local node

.. code-block:: bash

  $ sudo apt-get install libvirt-clients

Install required software on remote node (e.g. remote-ci-server.local), create directory for
storage pool and copy base VM image(s)

.. code-block:: bash

  $ sudo mkdir /media/molecule
  $ sudo cp ~/debian-stretch-root-2GiB-20171019.qcow2 /media/molecule/debian-stretch

Configure libvirtd and virsh (and probably ssh), so you can connect from local node to remote
machine without providing password (e.g. via ssh authorized keys)

.. code-block:: bash

  $ virsh --connect qemu+ssh://remote-ci-server.local/system list

Prepare, define and start network dedicated for molecule instances.
libvirt documentation regarding networking is available on:
- https://wiki.libvirt.org/page/Networking
- https://libvirt.org/formatnetwork.html

Great libvirt Networking Handbook by Jamie Nguyen is available on https://jamielinux.com/docs/libvirt-networking-handbook/index.html

.. code-block:: bash

  cat <<EOF > net-molecule.xml
  <network>
    <name>molecule</name>
    <bridge name="molecule"/>
    <forward mode="route" dev="br0"/>
    <ip address="10.4.208.1" netmask="255.255.255.0">
      <dhcp>
        <range start="10.4.208.2" end="10.4.208.254"/>
      </dhcp>
    </ip>
  </network>
  EOF

  $ virsh --connect qemu+ssh://remote-ci-server.local/system net-define net-molecule.xml
  $ virsh --connect qemu+ssh://remote-ci-server.local/system net-autostart molecule
  $ virsh --connect qemu+ssh://remote-ci-server.local/system net-start molecule

Prepare, define and start storage pool dedicated for molecule instances.
libvirt documentation regarding storage is available on:
- https://libvirt.org/storage.html
- https://libvirt.org/formatstorage.html

.. code-block:: bash

  cat <<EOF > pool-molecule.xml
  <pool type="dir">
    <name>molecule</name>
    <target>
      <path>/media/molecule</path>
    </target>
  </pool>
  EOF

  $ virsh --connect qemu+ssh://remote-ci-server.local/system pool-define pool-molecule.xml
  $ virsh --connect qemu+ssh://remote-ci-server.local/system pool-start molecule
  $ virsh --connect qemu+ssh://remote-ci-server.local/system pool-autostart molecule
