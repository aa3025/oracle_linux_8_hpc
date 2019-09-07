Oracle Linux 8 HPC

This project is collection of bash scripts and config files to automatically deploy OL8-based HPC with PXE-install and kickstart files. Supports both BIOS-based and EFI PXE scenarios. Project's webpage is here: https://centoshpc.wordpress.com/

Deployment steps:

1. install the master node (a.k.a. server, head node) using default CenOS Live or Minimal ISO, boot it, make sure the network is up,  proceed further from its terminal. Your installed master node must have 2 network adapters: 
  - the external network adapter should be configured, up and running, e.g. with NetworkManager prior to deployment of HPC. 
  - the 2nd network adapter must be connected to the internal network of HPC (i.e. via switch or hub), where all the "compute" nodes will be booting up from. All compute nodes must be connected to the same hub with their (not necessarily) 1st network adapter and configured to boot from LAN (PXE boot).
  
Do not add any users during initial installation of the head node (only root). You will add other users after deployment of the HPC (using ./scripts/newuser).

2. get the updated version of this repository

  2.1 From https://github.com/aa3025/oracle_linux_8_hpc/

  2.2 Uncompress the archive or clone our project's git tree "git clone https://github.com/aa3025/oracle_linux_8_hpc.git"

  2.3 If you failed to do above steps do not proceed further.

Please do not e-mail asking for support. These scripts are not guaranteed to work and are provided for your self-development. However if you are interested, you can join this project and contribute to its development on GitHub.
3. Next, download Oracle Linux 8 install DVD from Oracle website.


3. cd OL8hpc

 and run "./install.sh Oraclexxxx.iso" from that folder. The install script will install quite a few necessary rpms for all the servers etc. Then you will be prompted to enter internal and external LAN interface names. This is the only input required form user.

4. Once install.sh finishes, go and power up all your compute nodes (its better to do it one-by-one in an orderly fasion, their hostnames will be based on their DHCP addresses (node1, node2...), so if you want any kind of "system" in their naming make sure they boot with some interval, so that previous one already obtained IP before the next one boots). They must be BIOS-configured to boot from network (PXE boot).

The nodes will install form kickstar file which they get from http folder of the master, post-configure themselves, and each will modify the master's dhcpd.conf, /etc/hosts file, /etc/pdsh/machines file and add their grub.cfg-xxx to /tftpboot on master, so that on the next boot they don't go into PXE install again, but boot form their local drives.

Once PXE-install finishes, the nodes will reboot themselves and will mount /home and /share from master via NFS. If you want to share pre-existing /home folder (when user files were already inside before the HPC deployment), its better to rename /home to some different name before this installation, and when everyithing finishes, stop nfs-server (service nfs-server stop), rename it back to /home and restart nfs-server service (service nfs-server start).

5. Check if HPC is deployed sucessfuly by doing e.g. "pdsh hostname", as a result of this the nodes must report back their hostnames.
6. Its a good idea to restart dhcpd on master for it to swallow updated /etc/dhcp/dhcpd.conf modified by the nodes. 
7. If you want to re-install the node deployed previosly, you just need to delete its /tftpboot/grub.cfg-01-xx-xx-xx-xx-xx-xx file /tftpboot/xx-xx-xx-xx-xx (its mac address file) and its records from /etc/pdsh/machines and /etc/dhcp/dhcpd.conf and reset the node without re-running ./install.sh ! 
8. Master server configuration is permanent, so it must be able to serve new deployments after reboot (the only thing you need to ensure is that install dvd iso file is mounted after reboot of the master node in /var/www/html/OL8).

(not tested yet with OL8) Then you can run optional "./postinstall_from_server.sh" script to add additional rpm's on the master and compute nodes and sync "/etc/hosts" file between the nodes".

New users can be deployed with the script (cd ./configs) './newuser username "real user name (comments)". Script will create a local user and group on the master and nodes, create ssh-keys for paswordless connection to the nodes.

You can use pdsh to install missing packages on the nodes in parallel or mass-copying files etc.


UEFI PXE boot port [from aa3025/centos7hpc repo] is not tested yet . September 2019.

Alex Pedcenko, September 2019, aa3025(at)live.co.uk , http://centoshpc.wordpress.com
