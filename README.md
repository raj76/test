

Upgrade Type and Resource Requirements
==============================

|Upgrade Type     |EXAScaler 3.x to 4.X non-embedded|
|Plan template    |Exascaler-3.x\_to\_4.x-Non-Embedded.md|
|Case             |108462                                |
|Customer         |University of Georgia                 |
|Location         |GA                               |
|Timezone         |EDT|
|Customer contact | Saravanaraj Ayyampalayam / (706) 542-0188 / raj76@uga.edu            |
|Entitlement      | BSOS                               |
|Upgrade planner           | Khoa Pham          |
|Upgrade team involved     | Khoa Pham          |
|Date For Upgrade          | 2018	|
|Upgrade Time Required     | 10.5 hours      |
|Support Required          | - |
|Remote or On-site Support | - |

Plan Assumptions
====================

## Exascaler 
- This upgrade plan requires an outage
- System must be completely healthy before upgrade
- The EXAScaler upgrade needs the direct involvement of DDN support during upgrade execution
- These instructions only work upgrading from EXAScaler 3.x to 4.x
  + Your current EXAScaler version must be at 3.0 or above
- The DDN upgrade engineer verifies Lustre source files are correct for any manual Lustre upgrades
- Lustre clients may need to be upgraded

## SFA
- Recommended to rebuild any storage pools in no redundancy state before proceeding with upgrade
- During upgrades there is a chance of a drive failure or drives will go missing for a period of time requiring pool rebuilds. 
- If verifies were disable for 3 or more months, 2 complete verify cycles must be completed before attempting an upgrade.
- There are no embedded VMs
- SFA public SSH keys will be changed during/after the upgrade. Please ensure the existing password is known.


Summary of work / Order / Timing
====================

| time | description |
|:----|----|
| 0.5 hour | Before EXAScaler upgrade work |
| 2.0 hour | Prep for EXAScaler Upgrade |
| 0.5 hour | EXAScaler Health Check |
| 4.0 hour | Upgrade EXAScaler Servers (0.5 hr per server)|
| 0.5 hour | Updating exascaler.conf |
| 0.5 hour | Multipath, SRP configuration & IPOIB datagram mode|
| 0.5 hour | ES Install steps |
| 0.5 hour | HA Configuration |
| 0.5 hour | HA startup & EXAScaler upgrade post work
| 0.5 hour | Optional: Lustre rpm upgrade |
| ? hour | Optional EF enclosure firmware upgrade & enable vdisk scrub |
| ? hour | Optional: Upgrade Mellanox/OmniPath Firmware, IPoIB configuration (0.25 hr per node) |
| 0.5 hour | Upgrade SFA post work (enable cache, enable bg verify, set sparing back to original setting, health checks) |
| **Total upgrade time** | 10.5  hours |


 System Details
====================

| **SFA** | | 
|:----|----|
|Subsystem Name|osctl |
|Controller Type|SFA14KX |
|Uptime for Controllers| Controller 0 72 Days 21 Hours 56 Minutes, Controller 1 72 Days 21 Hours 56 Minutes|
|Enclosure Type|6- SS14K(head +  5-SS8462/12) with 0 missing enclosures |
|Number of Mux Drives|0|
|Internal Disk Mirroring|mirrored|
|BBU Mfg Date| ups 0 - Fri Aug 30 18:04:57 2013; ups 1 - NOT AVAILABLE|
|Background Verify|Enabled|
|Write Caching|Enabled|

| **Current EXAScaler** |    |
|:----                  |---:|
|EXAScaler Version|3.3.0-r4         |
|Lustre version|2.7.21.3            |
|Lustre build|256.ddn20.g10dd357    |


|**Current Infiniband / OmniPath** |  |
|:----|----:|
|EXAScaler Server OFED/OPA version|MOFED 4.3.1.0.1| 
|Server HCA Firmware Server|12.18.1000   |

| **Current Ethernet** |              |
|:----|----:|
| EXAscaler server MTU size|-         |
| EXAscaler server Ethernet bonding?|No|
| Tagged VLAN?|N/A                      |


Upgrade Component List and Links
====================

## EXAScaler Upgrade Items Summary
|Description| Current Level | New Level  | Comments| Internel URL|
|:-----------|---------------|------------|---------|-------------|
|ExaScaler Release| - | 4.0.0 |ISO Image (3 GB)|[ES 4.0.0 ISO](https://sync.ddn.com:443/home/link/browser?ll=AAAAAAAAAJkAAEdGhVqQcG5jchGpFV5F75RCWgC3Fno%3D)
|Lustre Version| - | 2.10.4-ddn4 | upgrade | ZIP file | https://sync.ddn.com:443/home/link/browser?ll=AAAAAAAAAJkAAEdFxxASByPrKglhAb--YM9UWL6gHis%3D

| Lustre Client Version:| - | 

### Mellanox HCA firmware updater tool: mlxup v4.9.0
mlxup tool is located in `/root/EXAScaler-4.0.0/` 

   Alternatively download from [mellanox.com](http://www.mellanox.com/downloads/firmware/mlxup/4.9.0/SFX/linux_x64/mlxup)


Detailed Process
===================

## SFA pre-upgrade work

1. Disable verifies 
```
   $ set subsystem verify_policy false
```

2. Disable caching
```
   $ set pool * write_back_caching false
```

3. View existing jobs
```
   $ show job (view existing jobs)
```
   **NOTE:** Wait for all remaining jobs to complete before continuing

4. Document current "sparing policy" and "disk time-out" value
```
   $ show pool *
```

5. Set sparing policy to manual for all pools 
```
   $ set pool * sparing MANUAL
```

6. Set drive time out to 4 houra
```
   $ set pool * disk_timeout 240
```

7. Capture diags from each controller

    Password=user
```{.bash}
   # ssh diag@IP_ADDRESS_OF_CONTROLLER_0 tgz > SR[#####]_[customer-name]_C0_Diag.tgz
   # ssh diag@IP_ADDRESS_OF_CONTROLLER_1 tgz > SR[#####]_[customer-name]_C1_Diag.tgz
```

## SFA External Block Pre-upgrade

- Document server IPMI credentials and network configuration
- Download all required files with supplied links (eg. ISO image file, HCA firmware)
- Ensure that at least 3GB of free space are available in server root partition

## EXAScaler pre-upgrade work

- Review EXAscaler release notes for limitations or known issues for upgrades
- Review Upgrade section of EXAscaler Administration Guide 
- Ensure "Open Issues/Actions" items as completed before upgrade complete
- Ensure the system is healthy before starting upgrade

- Back up all EXAScaler configuration files for each HA group

    eg) /etc/multipath.conf, /etc/corosync/corosync.conf, /etc/corosync/authkey,  /etc/ddn/exascaler.conf, /etc/ddn/device\_alias.conf, /etc/ddn/ddn-ibsrp.conf, /etc/srp_daemon.conf, /etc/ddn/srp\_daemon.conf, /etc/ddn/tune\_devices.conf, /etc/clustershell/, /etc/syslog-ng/, /scratch/log/

- Backup OS configuration files 

    eg) /etc/sysconfig/network-scripts/ifcfg-\*, /etc/hosts, /etc/resolv.conf, /etc/sysconfig/network, /etc/ntp.conf, /etc/passwd, /etc/group, /etc/modprobe.conf, /etc/modprobe.d/, /etc/sysctl.conf, /etc/sysctl.d/, /root/.ssh/, /etc/ssh/, nsswitch, ldap/sssd, etc.

- Disable all external yum repos (ie. /etc/yum.repos.d) and custom parameters in the yum configuration files ie) /etc/yum.conf


## EXAScaler Health Check

Note: An outage is required for this upgrade. This is not an online procedure

### Client node Health checks

NOTE: Any discovered issues should be resolved before proceeding with
the upgrade.  Contact DDN support for assistance if necessary.

- From a client node, check lustre is mounted:

```{.bash}
   # mount -t lustre
     10.10.160.31@tcp1:10.10.160.32@tcp1:/pfs on /lustre/pfs/client type lustre (rw)
   # findmnt -t lustre
```

- Check client can access lustre servers

    Example output

```{.bash}
   # lfs check servers
   pfs-MDT0000-mdc-ffff88003d223400: active
   pfs-OST0000-osc-ffff88003d223400: active
   pfs-OST0001-osc-ffff88003d223400: active
   pfs-OST0002-osc-ffff88003d223400: active
   pfs-OST0003-osc-ffff88003d223400: active
   pfs-OST0004-osc-ffff88003d223400: active
```

- check client can access all targets

    Example output

```{.bash}
   # lctl dl
   0 UP mgc MGC10.10.160.31@tcp1 9cba63c0-f7ce-c14e-1d95-3312a141b982 5
   1 UP lov pfs-clilov-ffff88003d223400 581969a1-7a8a-d371-3683-2fe3c44e4677 4
   2 UP lmv pfs-clilmv-ffff88003d223400 581969a1-7a8a-d371-3683-2fe3c44e4677 4
   3 UP mdc pfs-MDT0000-mdc-ffff88003d223400 581969a1-7a8a-d371-3683-2fe3c44e4677 5
   4 UP osc pfs-OST0000-osc-ffff88003d223400 581969a1-7a8a-d371-3683-2fe3c44e4677 5
   5 UP osc pfs-OST0001-osc-ffff88003d223400 581969a1-7a8a-d371-3683-2fe3c44e4677 5
   6 UP osc pfs-OST0002-osc-ffff88003d223400 581969a1-7a8a-d371-3683-2fe3c44e4677 5
   7 UP osc pfs-OST0003-osc-ffff88003d223400 581969a1-7a8a-d371-3683-2fe3c44e4677 5
   8 UP osc pfs-OST0004-osc-ffff88003d223400 581969a1-7a8a-d371-3683-2fe3c44e4677 5
```

- df listing all targets:

    Example good output

```{.bash}
   # lfs df -h
   UUID                       bytes        Used   Available Use% Mounted on
   pfs-MDT0000_UUID           10.8G        4.0G        6.0G  40% /lustre/pfs/client[MDT:0]
   pfs-OST0000_UUID         1021.9M       52.9M      958.8M   5% /lustre/pfs/client[OST:0]
   pfs-OST0001_UUID         1021.9M       52.9M      958.8M   5% /lustre/pfs/client[OST:1]
   pfs-OST0002_UUID         1021.9M       52.9M      958.8M   5% /lustre/pfs/client[OST:2]
   pfs-OST0003_UUID         1021.9M       52.9M      958.8M   5% /lustre/pfs/client[OST:3]
   pfs-OST0004_UUID         1021.9M       52.9M      958.8M   5% /lustre/pfs/client[OST:4]
   
   filesystem summary:        12.0G      634.6M       11.2G   5% /lustre/pfs/client
```

    Example bad output

```{.bash}
   # lfs df -h
   UUID                       bytes        Used   Available Use% Mounted on
   pfs-MDT0000_UUID           10.8G        4.0G        6.0G  40% /lustre/pfs/client[MDT:0]
   pfs-OST0000_UUID         1021.9M       52.9M      958.8M   5% /lustre/pfs/client[OST:0]
   pfs-OST0001_UUID         : Resource temporarily unavailable 
   pfs-OST0002_UUID         1021.9M       52.9M      958.8M   5% /lustre/pfs/client[OST:2]
```

### Server Health Checks

- Check EXAscaler servers are online:

    From one EXAScaler server:

```{.bash}
    # esctl pingall
    # corosync-quorumtool
    # hastatus
    # clush -b -a "uptime"
```

- Ensure all servers are currently at same software levels

```{.bash}
   # clush -b -a "cat /etc/es_install_version"
   # clush -b -a "cat /proc/fs/lustre/version"
```     

- Verify all lustre devices are mounted on the servers

```{.bash}
   # clush -a "mount -t lustre"

   ----------------
   mds0
   ----------------
   /dev/mapper/vg_pfs-mgs on /lustre/mgs type lustre (rw)
   /dev/mapper/vg_pfs-mdt on /lustre/pfs/mdt type lustre (rw)
   ----------------
   oss0
   ----------------
   /dev/mapper/ost_pfs_0 on /lustre/pfs/ost_0 type lustre (rw)
   /dev/mapper/ost_pfs_1 on /lustre/pfs/ost_1 type lustre (rw)
```

- Verify lustre healthy on servers

    NOTE: All servers should report 'healthy'.  There is problem if another value is reported.

```{.bash}
   # clush -b -a "cat /proc/fs/lustre/health_check"
```

- Pull es\_showall logs before making any changes

```{.bash}
   # esctl showall -s <DDN_CASE_NO>
```


## Prep for ExaScaler Upgrade 

### Unmount all compute clients and unload lustre modules

For each client execute the following set of commands 

- Find lustre mount point

```{.bash}
   # mount -t lustre
     10.10.160.31@tcp1:10.10.160.32@tcp1:/pfs on /lustre/pfs/client type lustre (rw)
   # findmnt -t lustre
```

- Unmount lustre

```{.bash}
   # umount -a -t lustre
   # modprobe -rv lustre osc mgc
```

- Stop lustre network   

```{.bash}
   # service lnet stop
```

- Remove Lustre kernel modules

```{.bash}
   # lustre_rmmod
```


### Server Halt Services
From EXAScaler server

- Stop all Server cluster resources

```{.bash}
   # cluster_resources --action stop
```     

- Check status of HA shutdown for each HA group

    Log into any node in each HA group

```{.bash}
   # clush -g ha_heads "hastatus"
```

   NOTE: If HA '(corosync|pacemaker)' is not running, the following error will be generated:

   Example bad output

```
   Could not establish cib_ro connection: Connection refused (111)
   Connection to cluster failed: Transport endpoint is not connected
   Could not establish cib_ro connection: Connection refused (111)
   Connection to cluster failed: Transport endpoint is not connected
```     

- Verify that all Lustre devices are unmounted

```{.bash}
   # clush -a "mount -t lustre"
```

- Shutdown HA resources

    NOTE: Perform his action for each HA group. 

    Start from HA group with OST resources and last HA group with MDT resources.

```{.bash}
   # clush -g ha_heads "crm configure property stop-all-resources=true"
```

   Unmount remaining mounted Lustre devices (can take a few minutes)

- Log into any MDS node

```{.bash}
   # clush -a "es_mount --umount"
```

- Verify that all Lustre devices are unmounted

```{.bash}
   # clush -b -a "mount -t lustre" 
```

- Unload lustre modules

```{.bash}
   # clush -a "service lnet stop"
   # clush -a lustre_rmmod
```

- Stop the HA stack services (Pacemaker and Corosync) on alls node

```{.bash}
   # clush -a service pacemaker stop
   # clush -a service corosync stop
```

## Upgrade ExaScaler Servers

- Copy the EXAScaler iso file to all the nodes and mount as a loop device

    From EXAScaler server

```{.bash}
   # scp -p <ES_UPGRADE.ISO> root@<ES_Server>:/scratch/
   # sync-file /scratch/<iso_image_filename>
   # clush -a "mkdir -p /mnt/iso"
   # clush -a "mount -v -t iso9660 -o loop,ro /scratch/<ES_UPGRADE.ISO> /mnt/iso"
```

- Run the upgrade script in dry-run mode to check for errors

```{.bash}
   # clush -a "python /mnt/iso/files/es_upgrade.py -v --dry-run"
```

   Check each dry run for errors
	
- Run the upgrade script on each host to apply the upgrade

    NOTE: You can run multiple upgrades in parallel

```{.bash}
   # clush -a "python /mnt/iso/files/es_upgrade.py -y -v"
```

- Copy files from ISO image to EXAScaler server

```{.bash}
   # mkdir -v /root/EXAScaler-4.0.0/
   # cp -vp /mnt/iso/files/* /root/EXAScaler-4.0.0/
   # cp -vp /mnt/iso/Packages/kernel-abi-whitelists-*.noarch.rpm /root/EXAScaler-4.0.0/
   # cp -vp /mnt/iso/Packages/kernel-devel-*.x86_64.rpm /root/EXAScaler-4.0.0/
   # cp -vp /mnt/iso/Packages/kernel-lustre-devel-*.x86_64.rpm /root/EXAScaler-4.0.0/
   # cp -vp /mnt/iso/Packages/kernel-lustre-debug-devel-*.x86_64.rpm /root/EXAScaler-4.0.0/

   # sync-file /root/EXAScaler-4.0.0/
```

- Unmount ISO image

```{.bash}
   # clush -a "umount -v /mnt/iso"
```

- Reboot all EXAScaler nodes

```{.bash}
   # clush -a reboot
```

- Verify that new version is installed on all server nodes

```{.bash}
   # clush -b -a "cat /etc/es_install_version" 
```	

## Update exascaler.conf

    Make a backup copy of EXAScaler configuration file */etc/ddn/exascaler.conf* before modifying

    NOTE: Examples of valid configuration can be found in the folder */opt/ddn/es/example/

1. **SFA zoning**

For each SFA storage used by the cluster, create a SFA zone named [sfa \<sfa\_name\>] and add the following options to these sections:

|   controllers - IP addresses of both SFA controllers 	
|   user — SFA access account. The value is “user” by default.
|   password – password for SFA access account. The value is “user” by default.

   Example

|   [sfa sfa12k0]
|   controllers: 10.52.16.44 10.52.16.45
|   user: user
|   password: user

- In the [global] section, replace the controller IP-addresses with Zone names from sfa\_list parameter
  
   Example

|   [global]
|   sfa_list: sfa12k0

- In each [host \<hostname\>] section append host\_sfa\_list entry with appropriate SFA zone

   Example

|   [host mds0]
|   host_sfa_list: sfa12k0


2. **NTP**

In the [global] section, add IP address or hostname of NTP server to entry ntp\_list

   Example

|   [global]
|   ntp_list: 10.0.0.5


3. **Corosync rings**

In the [HA] section of */etc/ddn/exascaler.conf*, replace network interface names in parameter corosync\_nics  with corosync rings.

   NOTE: ring0 and ring1 are only allowed.

   Example

|   [HA]
|   corosync_nics: ring0 ring1

- For each Corosync ring, specify the network interface 

   Either in the [host defaults] section or specific hosts section [host \<node\>].

   Example

|   [host oss-1-srv]
|   ring0: ens3
|   ring1: ib0


4. **nic\_list** & **lnets**

Check network interfaces are listed in nic\_list parameters in the [host defaults] section or specific hosts named like *[host \<node\>]*.

   NOTE: entries in [host ] section override [host\_defaults] parameters

   Example

|   [host oss-1-srv]
|   nic_list: ens3 ib0


Check entries for lnets parameters in the [host defaults] section or specific hosts named like [host \<node\>] 

   Example

|   [host oss-1-srv]
|   lnets: tcp0(ens3) o2ib0(ib0)


5. **backend FS**

Set backend FS type to ldiskfs. See file */opt/ddn/es/examples/exascaler.conf* for details.

   In each [fs ] section append backfs entry:

|   [fs testfs]
|   backfs: ldiskfs


6. **lustre mount options**

Prevent lustre.mount command from changing DDN block device tuning parameters; option *max\_sectors_kb=0* will prevent lustre from changing current value. 

   In each [fs ] section append mount_opts entries:

|    [fs testfs]
|    mdt_mount_opts: max_sectors_kb=0
|    ost_mount_opts: max_sectors_kb=0


7. **lustre o2iblnd options**

Add or modify modprobe settings for lustre LND driver o2iblnd.
Option "conns\_per\_peer" sets number of QPs for each lustre rdma connection. See *modinfo ko2iblnd* for details.

Defaults: *conns_per_peer=1* for Mellanox and *conns_per_peer=4* for OmniPath. For backward compatibility we set *conn_per_peer=1*.

   NOTE: entries in [host ] section override [host\_defaults] parameters

   Example

|   [host_defaults]
|   modprobe_cfg:
|      options ko2iblnd conns_per_peer=1


## External Block SRP configuration

NOTE: Further information in "Known Issues" section of EXAScaler Release Notes

- confirm ib_srp devices are visible

```{.bash}
   # lsscsi -lH | grep -A2 ib_srp
```


## External Block Multipath configuration

NOTE: Multipath alias name format described in appendix of *Exascaler Install & Admin Guide*

- confirm ddn-ibsrp service is running (ie. opensm + srpd)

```{.bash}
   # service ddn-ibsrp start
```

- confirm SFA/EF multipath devices are visible

```{.bash}
   # lsscsi -l
```

- backup existing multipath.conf

```{.bash}
   # cp -vp /etc/multipath.conf /etc/multipath.conf.ESpre-upgrade
```

- Regenerate SFA device wwids and alias names with command "create\_table" for each SFA storage system:

    NOTE: connected block devices wwids will be listed in /dev/disk/by-id/

- From server connected to SFA storage:

```{.bash}
   # create-table --new-style --type multipath --sfa <sfa_ip_address> >> /etc/multipath-aliases.sfa
```

- check multipath aliases match new EXAScaler naming format

   Example 

   + old name format: *alias ost\_scratchfs\_1*
   + new name format: *alias scratchfs\_ost0001*

- Append multipath aliases to multipath.conf

```{.bash}
   # cp -v /etc/multipath.conf.ddn /etc/multipath.conf
   # cat /etc/multipath-aliases.sfa >> /etc/multipath.conf
```

- For EF enclosures restore EF multipath aliases from backup multipath.conf file
NOTE: EF enclosures usually connected to MDS servers. EF multipath devices may not be visible from OSS server nodes

- restart multipath service

```{.bash}
   # multipath -F
   # udevadm trigger
   # systemctl enable multipathd 
   # systemctl restart multipathd
   # multipath -l
```

- check device aliases listed in */dev/mapper/*

- copy file multipath.conf to other server nodes

   NOTE: ensure not to overwrite multipath.conf containing EF aliases

- flush & restart multipath service on other server nodes


## Infiniband IPoIB Configuration

Mellanox ConnectX-4 Infiniband adapters enable [IPoIB Connected Mode](http://www.rdmamojo.com/2015/02/16/ip-infiniband-ipoib-architecture/) by default. We have have found scaling issues with Connected Mode (CM)
with larger Infiniband networks ie >50 clients. [IP over Infiniband kernel documentation](https://www.kernel.org/doc/Documentation/infiniband/ipoib.txt)

We recommend disabling CM on EXAScaler servers and clients.

### Set IPoIB datagram mode

Configuration file */etc/infiniband/openib.conf* is used by Mellanox OFED.

- Set configuration parameter *SET_IPOIB_CM=no*

    Example
```{.bash}
   cp -vp /etc/infiniband/openib.conf /etc/infiniband/openib.conf.ESpre-upgrade
   sed --in-place -e 's/SET_IPOIB_CM=.*/SET_IPOIB_CM=no/' /etc/infiniband/openib.conf
```


## ES Install steps		
- Run script about\_this\_host from each node and validate no issues reported

|  # about_this_host --check

   Example
```   
   # about_this_host --check
   Host name: es51
   EXAScaler version: 3.2.0
   EXAScaler flavour: HPC
   Peers of this host are: es52
   Network interface: eth0 (192.168.3.51)
   Network interface: eth1 (172.16.2.51)
   Lustre is exported on interfaces: tcp(eth0)
   Stonith type: none
   Details for filesystem testfs30
   mdt runs on this host: /dev/vg_mdt0000_testfs30/mdt0000 /lustre/testfs30/mdt0000
   /dev/mapper/testfs30_mdt0000_s0 (prio: 1)
   mgs runs on this host: /dev/vg_mgs/mgs /lustre/mgs
   /dev/mapper/mgs (prio: 1)
   ost 0 runs on this host: /dev/mapper/testfs30_ost0000 /lustre/testfs30/ost0000
   ost 1 can failover to this host: /dev/mapper/testfs30_ost0001 /lustre/testfs30/ost0001
   FSCK logs are saved to /scratch/log/testfs30
```	

- Configure exascaler with the following command:

    Details about es\_install steps are documented in EXAScaler Install & Adminstration guide. Only a few es\_install steps are used during upgrades

    NOTE: The "es\_install --steps" recipe may change case-by-case 

    ie) depending on whether customer is OK with overwriting network config files, modprobe filea, IPMI, etc.

    NOTE: "es\_install --steps lvm" will overwrite /etc/lvm/lvm.conf. On some servers (eg. HP Proliant) the LVM config changes can cause the server to hang on next reboot.

    NOTE: "es\_install --steps ha" might complain about file exists /etc/corosync/corosync.conf

    IMPORTANT: skip step: nics, ipmi, restart\_network, lvm, & lustre


-  On each Exascaler node run the es\_install tool

    Example output:

```
   # es_install --debug --yes --steps os,kdump,ntp,modprobe,logging,mount_points
   Operation system configuration ...
   Operation system was successfully configured.
   Kdump configuration ...
   Kdump was successfully configured.
   Modprobe configuration ...
   Modprobe was successfully configured.
   Logging configuration ...
   Logging was successfully configured.
   Mount points configuration ...
   Mount points were successfully configured.
```

- Reboot server and check configuration

```
   # about_this_host --check
```

## Post upgrade mount and checks
Test and mount lustre targets on each node individually

- Verify that Lustre volumes can be mounted manually on each server node.

```
   # es_mount --dry-run
```	

- Mount Lustre targets

    Lustre targets are mounted in the following order:  1) MGS volume 2) MDT volumes 3) OST volumes
```
   # es_mount
```

- Check mounts

```
   # clush -a "mount -t lustre"
   # clush -a "lustre_recovery_status.sh"   
```	

- Verify that Lustre is healthy on all server nodes

```
   # clush -a "cat /proc/fs/lustre/health_check"	
```

- test mounting lustre on a client

```
   # mount_lustre_client --dry-run
```

- validate client lustre access

```
   # lfs check servers
   # lfs df -h
   # ls -l /path/to/lustre/
```

- after validation umount lustre client and lustre targets

   lustre targets are umounted in following order: 1) lustre clients 2) OST volumes 3) MDT volumes 4) MGS volume

```
   # mount_lustre_client --umount
   # es_mount --umount
   # clush -a "mount -t lustre"
```  

## Additional Upgrade Tasks

Complete Optional Upgrade tasks

		
## HA Configuration

   NOTE: If multiple HA groups are being used, use "clush -g ha\_heads" to run command on head node each HA group

   NOTE: HA group entries are stored in */etc/clustershell/groups.d/local.cfg*

- Erase current HA config

    This step will stop pacemaker & corosync services on all EXAScaler nodes

```
   # clush -a "service pacemaker start"
   # esctl cluster --action destroy
```

- Re-generate corosync configuration

```
   # clush -a "config-corosync --regen-config"
```

- Start corosync service

```
   # clush -a "systemctl enable corosync"
   # clush -a "systemctl start corosync"
```

- validate corosync service running

```
   # clush -g ha_heads "corosync-quorumtool"
```

### pacemaker HA configuration

   NOTE: corosync service must be running

   NOTE: umount lustre targets from EXAScaler nodes before starting

- Start pacemaker service

```
   # clush -a "systemctl enable pacemaker"
   # clush -a "systemctl start pacemaker"
```

- validate pacemaker service

```
   # clush -g ha_heads "hastatus"
```

   NOTE: If HA '(corosync|pacemaker)' is not running, the following error will be generated:

   Example bad output

```
   Could not establish cib_ro connection: Connection refused (111)
   Connection to cluster failed: Transport endpoint is not connected
   Could not establish cib_ro connection: Connection refused (111)
   Connection to cluster failed: Transport endpoint is not connected
```

- Re-generate HA configuration

    Run commands for each HA group

```
   # clush -g ha_heads "config-pacemaker --dry-run > /root/config_pacemaker.`date +%F`"
   # clush -g ha_heads "config-pacemaker"
```

- validate HA configuration

```
   # clush -g ha_heads "hastatus"
```


## HA startup
- Start all HA resources & lustre targets:

```
   # cluster_resources --action start
```

   NOTE: this command will umount & re-mount lustre targets

- check HA startup

```
   # clush -g ha_heads "hastatus"
```

- validate lustre targets mounted

```
   # clush -a "mount -t lustre"
   # clush -a "lustre_recovery_status.sh"   
```
	
- Verify that Lustre health on all server nodes

```
   # clush -b -a "cat /proc/fs/lustre/health_check"	
```


## EXAScaler post work
NOTE: Ask customer to run client side checks

- capture showall logs

```
   # esctl showall -s <ddn_support_case_number>
```

- upload ddn\_showall file to ftp.ddntsr.com

```
   # curl --ftp-ssl-ccc --keepalive-time 10 -T ./<path_to_ddn_showall_file> ftp://ftp.ddntsr.com/upload/ --user anonymous:<my_email_address>
```

## SFA post work

Enable cache, enable bg verify, set sparing back to original setting, health checks

1. Enable caching
```
   $ set pool * write_back_caching true
```

2. Resume any jobs that may have been previously paused
```
   $ show job
   $ resume job X #where X is the job id
```

3. Enable background verify
```
   $ set subsystem verify_policy true
```

4. For each spare pool, set sparing policy to what it was before the upgrade for all pools
```
   $ set pool * sparing <AUTO/SWAP/etc>
```

5. For each spare pool, set drive time-out to what it was before the upgrade for all pools
```
   $ set pool * disk_timeout <PREVIOUS VALUE>
```

6. Capture diags from each controller

    Password=user
```{.bash}
   # ssh diag@IP_ADDRESS_OF_CONTROLLER_0 tgz > SR[#####]_[customer-name]_C0_Diag.tgz
   # ssh diag@IP_ADDRESS_OF_CONTROLLER_1 tgz > SR[#####]_[customer-name]_C1_Diag.tgz
```
        
7. If required initialize Battery Life Remaining Feature
    a. Log in to the controller

    b. Issue the command: `show ups * all`

    c. If a battery manufacturing date is displayed, do nothing more. However, if the battery manufacturing date and life remaining are not available as shown below, proceed to Step 4. 

|     Battery Mfg. Date: NOT AVAILABLE 
|     Battery Life Remaining: NOT AVAILABLE

    d. Issue the command `SET UPS <encl-idx> <UPS-idx> BATTERY_MANUFACTURE_DATE=<yyyy>:<mm>:<dd>`

       Choose a date close to when the system was installed. 

    e. Verify it was set - issue the command `show ups * all` and you should see output similar to the following:

|     Battery Mfg. Date: Thu Sep 8 4:10:30 2012 
|     Battery Life Remaining: 730 days 

8. If required set Internal Disk Mirror if A or B Drives are NOT MIR State. 

```
   $  show internal_disk
   $  assign INTERNAL_DISK <enclosure-id> <object-id> to_system_disk
```


Appendix
===========

## IME product support matrix
From IME release notes:

- IME 1.0.0: EXAScaler 2.1.2
- IME 1.1.1: EXAScaler 2.4.0-r10 (tested with IB FDR)
- IME 1.1.2: Updated EXAScaler 2.4.0-r10 to EXAScaler 3.2.0

## Insight product support matrix
- Insight 1.0.0: EXAScaler 3.2.0
- Insight 1.0.1: EXAScaler 3.2.0 or higher


TODO/WIP
====================
Additional instructions

1. LVM volgroup activation

    NOTE: this step assumes the LVM volgroups are same on all EXAscaler nodes

    NOTE: no wildcards nor pattern matching is allowed

    - list LVM volume groups used by the OS; volgroups are usually *VolGroup00* and sometimes *VolGroup01*

    Example:
```
   # vgs
   VG                  #PV #LV #SN Attr   VSize   VFree  
   VolGroup00            1   4   0 wz--n-  23.47g      0 
   vg_mdt0000_testfs20   1   1   0 wz--n- 512.00g 508.00m
   vg_mgs                1   1   0 wz--n- 508.00m 252.00m
```

    - In exascaler.conf [global] section add the OS LVM volgroup names to entry *vg\_activiation\_list*

    Example:

|    [global]
|    vg_activation_list: VolGroup00


   NOTE: do NOT add any volumes managed by HA to *vg\_activation\_list*

This parameter controls entry *auto\_activation\_volume\_list* in */etc/lvm/lvm.conf*

The "auto\_activation\_volume\_list" is exclusive. When enabling this LVM parameter, if a volgroup is NOT listed, it won't be activated on boot. 

Hint: make sure you list your /root volgroup.

   On each Exascaler node run the es\_install tool

    Example:
```
   # es_install --debug --yes --steps lvm
```

2. email relay
   
   In exascaler.conf [global] section add email information

|   # site email domainname
|   email_domain: example.com
|   # external smtp server
|   email_relay: mail.example.com
|   # List of emails address to receive logging notification (comma delimited)
|   email_list: Test User <test@example.com>


   On each Exascaler node run the es\_install tool

    Example:
```
   # es_install --debug --yes --steps email
```


3. Project Quotas

   Support project quotas "space accounting" in ldiskfs backend

   # edit exascaler.conf

|   [conf_param_tunings]
|   # List of Lustre parameters which can be set using lctl conf_param command
|   # enable quotas user/group/project
|   testfs.quota.mdt: ugp
|   testfs.quota.ost: ugp


   Add "-O project" to mkfs command.

   edit exascaler.conf, in each [fs ] section append mke2fs\_opts entries:

|    [fs testfs]
|   # mke2fs options for the OSTs
|   ost_mke2fs_opts: -m1 -i 131072 -O project


   On each Exascaler node run es\_tunefs & es\_tune_lustre tools

    Example:
```
   # es_tunefs --dry-run
   # es_tune_lustre --dry-run
```

3. Enable Lustre jobstats

   Enable Lustre jobstats feature
   
   # edit exascaler.conf

   Replace \<fsname\> with name of lustre filesystem

|   [conf_param_tunings]
|   \<fsname\>.sys.jobid_var: procname_uid

   On each Exascaler node run es\_tune_lustre tool

    Example:
```
   # es_tune_lustre --dry-run
```

4. Lustre Progressive File Layout

   Set filesystem default lustre striping.

   manpage lfs-setstripe:

   "If the default file layout is set on the filesystem root directory, it will be used as the filesystem-wide default layout for all files that do not explicitly specify a layout and do not have a default layout on the parent directory.  The default layout set on a directory will be copied to any new subdirectories created within that directory at the time they are created."

   PFL command syntax:

|    lfs setstripe <--component-end|-E end1> [STRIPE_OPTIONS] [<--component-end|-E end2> [STRIPE_OPTIONS] ...] \<filename\>

   # example lustre striping

   Run *lfs setstripe* command on lustre client with lustre filesystem mounted

```{.bash}
    $ lfs setstripe \
    -E 16M --stripe-count=1 --stripe-size=16M \
    -E 64M  --stripe-count=4 --stripe-size=16M \
    -E 256M --stripe-count=8 --stripe-size=32M \
    -E -1 --stripe-count=16 --stripe-size=32M /lustre/testfs/client
```

Optional Tasks 
==============
[Optional] EF Enclosure firmware upgrade
====================

EF firmware can be upgraded via WebUI. Alternatively upgrade via console/CLI using these instrucions. CLI upgrade requires external host with ftp client to upload flash file.

   NOTE: EF firmware updates can 40 min when PFU is enabled

   NOTE: WebUI & CLI default credentials are manage/!manage

   NOTE: ftp default credentials are ftp/!ftp

- Current EF fw versions:
    + EF3015 Firmware TS252P006 (2017 Sept)
    + EF4024 Firmware GL222R050 (2017 Sept)

- Firmware Links (druva login required)
    + [{DDN-SWREPO}EF Series sync.ddn.com](https://sync.ddn.com/home/share/#op=viewcontent&shareid=831) 


## Avis
- Do not cycle power or restart devices during a firmware update. If the update is interrupted or there is a power failure, the module could become inoperative. If this occurs, contact DDN support.

- For single controller/single domain systems, I/O must be halted (ie. outage required) for upgrade.

- In dual-module enclosures, both controllers or both I/O modules must have same firmware version.

- Set "Partner Firmware Update option" so that, in dual-controller systems, both controllers are updated. When the Partner Firmware Update option is enabled, after the installation process completes and restarts the first controller, the system automatically installs the firmware and restarts the second controller.

- For dual controller systems, because the online firmware upgrade is performed while host I/Os are being processed, I/O performance is impacted during the upgrade process.

## WebGui Instructions

### WebUI Health Check

- In the Configuration View panel, select the System tab and then select 

|  *View* > *Overview*

|       The System Overview table shows:

|       The system’s health:

|       *OK*

|       *Degraded*

|       *Fault*

|       *Unknown*


### WebUI firmware update

NOTE: Confirm that all I/O to this storage system has been halted before starting

1. Restart the Management Controller component within both system controller modules: 

   from the *System* tab in the GUI, select button *Action* > *Restart System*

   The Controller Restart and Shut Down panel then opens:

   - Select the Restart operation
   - Select the controller type to restart: Management
   - Select both controller modules (A + B)

     + Click OK. A confirmation panel appears

     + Click Yes to continue, a message will describe the restart activity

2. Once controllers have been fully restarted, navigate to the GUI System tab and select *Action* > *Update Firmware*

3. Click *Browse* and select the firmware file to upload 

    NOTE: If Controller cannot be updated, the update operation is cancelled. Verify that you specified the correct firmware file and repeat the update. 

4. Click *OK*, a pop-up panel will appear to show upload progress

   NOTE: Do not perform a power cycle or controller restart during a firmware update. If the update is interrupted or there is a power failure, the module might become inoperative. 

5. When the update is complete, clear the history from your local web browser, then sign into the GUI. 

  - When Partner Firmware Upgrade(PFU) feature is enabled, a panel will display progress and the GUI will prevent other tasks until update of partner controller is complete. 
  - Allow up to 20 minutes for the PFU cycle to complete. 

6. Once the PFU tasks have completed, confirm system status in the GUI, collect system logs and submit to technical support. 


## CLI Instructions

### CLI Health Check
- Check DotHill enclosure health via ssh 

  default operator credentials manage/!manage
```
# echo -e "show system" | ssh -T manage@IP_ADDRESS_OF_DOTHILL_ENCLOSURE
  Password: 
  DDN EF3000 EF3015
  System Name: Eng-EF3015
  Version: TS251R004-05
.
  # show system
  System Information
  ------------------
.
  Health: OK
  Health Reason: 
```

### CLI Firmware update

- Download & extract firmware binary file
    + EF3015 name has format: *TSxxxRyyy-zz.bin*
    + EF4024 name has format: *GLxxxRyyy-zz.bin*

- Determine network-port IP addresses of the system controllers. Login to controller via ssh
```
 # ssh manage@IP_ADDRESS_OF_EF_ENCLOSURE
```
  
- Determine current FW version (EF3015 firmware name format: TSxxxRyyy-zz)
```
 # show version
 Controller A Versions
 ---------------------
 Bundle Version: TS251R004-05
. 
 Controller B Versions
 ---------------------
 Bundle Version: TS251R004-05
```

- Verify FTP service is enabled
```
 # show protocols  
 Service and Security Protocols
 ------------------------------
. 
 File Transfer Protocol (FTP): Enabled
```

- Verify that user "ftp" has permission to FTP service and has manage access rights.
```
 # show user ftp
 Username             Roles                            User Type    User Locale           WBI   CLI   FTP   SMI-S SNMP
 Authentication Type Privacy Type Password                         Privacy Password                 Trap Host Address
 ------------------------------------------------------------------------------------------------------------------
 ftp                  manage,monitor                   Standard     English                            x           
                                    ********                         ********                 
```

- (Dual controller enclosure) enable partner firmware update:
```
 # set job-parameters partner-firmware-upgrade enabled
 Info: Parameter 'partner-firmware-upgrade' was set to 'enabled'. (2017-02-17 14:31:28)
.
 Success: Command completed successfully. - The settings were changed successfully. (2017-02-17 14:31:28)
```

- from external Linux host, upload firmware to EF controller as filename "flash". 

   Log in with user ftp (user = ftp, password = !ftp).
```
 $ ftp <EF_IP_ADDRESS>
 $ bin
 $ put <TSxxxRyyy-zz.bin> flash 
```

- After file upload completes, firmware update will start. 

   Wait for the installation to complete. During installation, each updated module automatically restarts.

   Example EF3015 firmware update:
```
 $ ftp> put ./TS252P005.bin flash
 local: ./TS252P005.bin remote: flash
 227 Entering Passive Mode (192,168,40,231,56,94)
 150 Accepted data connection
 226-File Transfer Complete. Starting Operation:
 Checking component list
 mc bundle component check passed.
 Checking bundle integrity...
 Initial mc file integrity checks passed.
 Checking system health.
 System health check complete. Health state: OK
 Stopping Management Controller applications.
 Starting message server
 Initial connection to SC successful.
...
 STATUS: Updating Storage Controller firmware.
 Controller current bundle: TS251R004-05,loading bundle TS252P005
 Instructing SC to shut down and reboot when finished updating
 Waiting 5 seconds for SC to shutdown.
 Shutdown of SC successful.
 Sending new firmware to SC.
 Waiting for Storage Controller to complete programming.
 Please wait...
 Storage Controller has completed programming.
 Updating SC Image:Remaining size 0
 Waiting for SC reboot.
...
 Storage Controller has rebooted.
 Storage Controller has been updated, proceeding to next step.
 STATUS: Updating Management Controller firmware
...
 Finished updating Management Controller firmware
 Updating system configuration files
 System configuration complete
.
 ==========================================
 Software Component Load Summary:
.
 MC Software: SUCCESSFUL
 SC Software: SUCCESSFUL
 EC Software: NOT ATTEMPTED
 Expansion Software: NOT ATTEMPTED
 ==========================================
.
 Code load completed successfully.
 Restarting Management Controller...
 Rebooting...
```

- If Partner Firmware Update is disabled, after updating firmware on one controller, you must manually update the second EF controller.

- Review EF event logs for firmware update event codes [269] & [237]
```
 # show events
 2017-02-17 15:59:32 [269] #A1435: EF3015 Array SN#00C0FF142609 Controller A INFORMATIONAL Partner Firmware Update progress: PFU completed on local controller, SUCCESS (info: p1: 5, p2: 0, p3: 0, p4: 0)   
 2017-02-17 15:59:32 [269] #A1434: EF3015 Array SN#00C0FF142609 Controller A INFORMATIONAL Partner Firmware Update progress: PFU send package done, SUCCESS (info: p1: 17, p2: 0, p3: 0, p4: 0)   
 2017-02-17 15:59:32 [237] #B1386: EF3015 Array SN#00C0FF142609 Controller B INFORMATIONAL Firmware update progress: The SC app was updated. Saved in primary location in flash. The firmware is different so it was flashed; flashed successfully.   
 2017-02-17 15:58:45 [237] #B1385: EF3015 Array SN#00C0FF142609 Controller B INFORMATIONAL Firmware update progress: The firmware was verified. (from MC: no, for MC: no)   
 2017-02-17 15:58:34 [269] #A1433: EF3015 Array SN#00C0FF142609 Controller A INFORMATIONAL Partner Firmware Update progress: PFU sending package to partner SC,  (info: p1: 16, p2: 0, p3: 0, p4: 0)   
```

## EF Enclosure Background vdisk Scrub
This step is required to protect your data.

   NOTE: required minimum firmware version EF3015 TS252P005 or EF4024 GL222R050 

- Log in to the EF enclosure CLI as administrator (user ID manage, default password !manage) and enter the line command:
```
 # set job-parameters background-scrub enabled
 Info: Parameter 'background-scrub' was set to 'enabled'. (2017-02-17 14:29:22)
.
 Success: Command completed successfully. - The settings were changed successfully. (2017-02-17 14:29:22)
```

- Check the status of the vdisk background scrub with the command:
```
 # show job-parameters
 Job Parameters
 --------------
 Vdisk Background Scrub: Enabled
```

- Review EF enclosure event logs after completion of scrub to determine whether data integrity issues were found. 

   Log in to the CLI as administrator, then enter the line command:
```
 # show events
```    

[Optional] Upgrade HPE Proliant Servers Firmware & Drivers
====================

HPE Proliant servers require updated device drivers to support Redhat 7.4 or newer
 
   **Important**: EXAScaler 3.3(based on CentOS 7.4) will not function unless Proliant firmware & drivers are updated.

Download and install [HPE Supplimental Supplement Service Pack for ProLiant 2018.03.0](http://h17007.www1.hpe.com/us/en/enterprise/servers/products/service_pack/spp/index.aspx?version=2018.03.0) after re-imaging EXAScaler software.

  - [SPP 2018.03.0 Release Notes](https://downloads.hpe.com/pub/softlib2/software1/publishable-catalog/p291731480/v138520/SPP2018.03.0ReleaseNotes.pdf)


## ProLiant Gen6 & Gen7 Models RHEL7 Unsupported
   HPE Proliant Gen6 & Gen7 servers are not supported with EXAScaler 3.x. HPE has not provided RHEL7 compatible device drivers for older generation of SmartArray storage controllers based on cciss device driver (eg. P400i).

From HPE whitepaper:

| *HPE Smart Array Controller models no longer supported in Red Hat Enterprise Linux 7 include: P400, P400i, P800, E200, E200i, P700m, 6400, 641, 642, and 6i*

   - HPE [whitepaper for Proliant RHEL7 support](https://h20195.www2.hpe.com/V2/GetPDF.aspx/4AA3-1875ENW.pdf)
   - Redhat KB article for [older generation HPE SmartArray controllers](https://access.redhat.com/articles/118133)


## HPE Support for Redhat Linux 7

Proliant Redhat Linux 7 support appears defined by Linux processor support. HPE ProLiant Gen10 (Skylake) supports Redhat 7.3 or newer. HPE ProLiant Gen9 (Broadwell) supports Redhat Linux 7.2 or newer.

- [HPE Redhat Linux Product Support Matrix](http://h17007.www1.hpe.com/us/en/enterprise/servers/supportmatrix/exceptions/rhel_exceptions.aspx)


Proliant servers with P22x, P41x or P42x "smart array" storage controllers require firmware 8.00 or newer for RHEL7.
https://support.hpe.com/hpsc/swd/public/detail?sp4ts.oid=5295169&swItemId=MTX_42b6aa58956a438aa85bd73d0f&swEnvOid=4184
[Optional] Mellanox HCA firmware update
====================

   NOTE: use *mlxup v4.4.0* to apply HCA firmware version listed in EXAscaler 3.2.0 release notes

- Determine current HCA firmware version
```
# clush -a 'ibstat | egrep -i "CA |firmware"'
```

   Compare the above Mellanox version to the release notes for the EXAScaler version

   If server HCA firmware is below recommended level, proceed with upgrade

- Update HCA firmware

   Below we assume mlxup tool located in */root/EXAScaler-3.2.0/mlxup*; you may need to download mlxup tool from mellanox.com (see URL above)
```
# chmod +x /root/EXAScaler-3.2.0/mlxup
# /root/EXAScaler-3.2.0/mlxup --query
# /root/EXAScaler-3.2.0/mlxup --update
``` 

   After firmware completes, reboot server

- repeat firmware update steps for other EXAScaler server nodes


## Upgrade Mellanox Firmware with flint

   NOTE: This Mellanox fw applies to server HCA connecte to the SFA storage, *NOT* HCA connected to clients

1. Determine if an upgrade to the Mellanox drivers is necessary.
```
# clush -a "ibstat"
```

   Compare the above Mellanox driver versions to release notes for the current SFAOS version. Continue with if the versions on the servers nodes do not match the recommended HCA firmware version
  
2. Getting firmware from Mellanox support downloader:

   a. find PSID from ibv_devinfo.txt file in es_showall logs. The board_id is the PSID
```
# grep board_id ibv_devinfo.txt
  board_id:                       MT_1090120019
  board_id:                       MT_1090120019
```

   b. Download HCA firmware matching PSID from [http://www.mellanox.com/supportdownloader/](http://www.mellanox.com/supportdownloader/)

   To download older firmware:

  - under "Select a Family" column select "Adapter Cards"
  - Under "Select a Line" column select the card name found from inputting the PSID
  - Under "Select an OPN" column select the OPN from the firmware file name given for from inputting the PSID
  - Under "Product Support Information" column click "Check for older versions"

    Example:

    *fw-ConnectX3-rel-2_36_5000-MCX354A-FCB_A2-A5-FlexBoot-3.4.718.bin.zip*

    - the OPN is "MCX354A-FCB"
    - Under "Select a PSID (Rev)" column select the PSID that matches the PSID from ibv_devinfo.txt  
    - Under "Product Support Information" column click "Check for older versions"

   c. Download the appropriate firmware version and release notes


3. Flash the MLX adapters:

    a. copy .bin firmware file to server

    b. On first ES Server run mst start on server
```
   # mst start
```

    c. run mst status to get device names ex: /dev/mst/mt4099_pciconf0
```
   # mst status
```

    d. use the flint burn command with the –d flag and device path and the –i flag and firmware file name to burn the new fw to the card (upgrade the fw)
```
   # flint –d <DEVICE_NAME> –i <MELLANOX_FIRMWARE.BIN> burn
```

    e. reboot server

    f. after reboot, run ibstat to confirm new fw version
```
   # ibstat
```  

    g. run lsscsi -l to ensure access to the SFA devices
```
   # lsscsi -l
```


## Mellanox Firmware Matrix

- [Link to MLNX_OFED Firmware Driver Compatibility Matrix](http://www.mellanox.com/page/mlnx_ofed_matrix?mtag=linux_sw_drivers)

[Optional] Upgrade OmniPath HFI firmware
====================

Update the firmware on the Omni-Path Host Fabric Interface (HFI)

   NOTE: HFI firmware can be updated on embedded systems

There are three files available for Option ROM EPROM partitions. These default files
are packaged with Intel Fabric Suite (IFS) and Basic releases. See the Intel Omni-
Path Fabric Software Release Notes for the version provided in the release. The files
are:

- HFI1 UEFI Option ROM: HfiPcieGen3_x.x.x.x.x.efi
- UEFI UNDI Loader: HfiPcieGen3Loader_x.x.x.x.x.rom
- HFI1 platform file: hfi1_platform.dat

The HFI UEFI firmware is packaged in RPM hfi1-uefi.x86_64 and bundled
with Omni-Path Fabric Software (OFS) "basic" package.

NOTE: The included hfi1_platform.dat file is for Intel HFI
adapters. If your HFI adapter is from another manufacturer, you may
require different hfi1_platform.dat file. Contact your manufacturer's
support team to confirm.

Your HFI may also have Thermal Monitoring Module (TMM) firmware, an
optional micro-controller for thermal monitoring on vendor-specific
HFI adapters using the SMBus.

To upgrade the HFI firmware, perform the following steps:

- review fabric software release notes for any special instructions 

- confirm HFI UEFI rpm installed
```
   rpm -qa | grep -i hfi1-uefi
```

- confirm firmware files in */usr/share/opa/bios_images*
```
   # ls -l /usr/share/opa/bios_images
   -rw-r--r-- 1 root root 489168 Oct 4 08:44 HfiPcieGen3_1.6.0.0.0.efi 
   -rw-r--r-- 1 root root 65024 Oct 4 08:44 HfiPcieGen3Loader_1.6.0.0.0.rom 
   -rw-r--r-- 1 root root 19530 Oct 4 08:44 License_UEFI_Option_ROM 
   -rw-r--r-- 1 root root 252298 Oct 4 08:44 License_UEFI_Option_ROM.pdf 
```

- confirm platform file in */lib/firmware/updates/*
```
   # ls -l /lib/firmware/updates/hfi1_platform.dat
```

- determine the device path to be used in command 
```
   # hfi1_eprom -v 
   Using default device: /sys/bus/pci/devices/0000:04:00.0/resource0 
```

- check existing driver version
```
   # hfi1_control -i
   Driver Version: 0.9-294
   Driver SrcVersion: A08826F35C95E0E8A4D949D
   Opa Version: 10.3.0.0.81
   0: BoardId: Intel Omni-Path Host Fabric Interface Adapter 100 Series
   0: Version: ChipABI 3.0, ChipRev 7.17, SW Compat 3
   0: ChipSerial: 0x00790311
   0,1: Status: 5: LinkUp 4: ACTIVE
   0,1: LID=0x1 GUID=0011:7501:0179:0311
```

```
   # hfi1_eprom -V -b 
   Using device: /sys/bus/pci/devices/0000:04:00.0/resource0 
   driver file version: 1.4.2.0.0 
```

```
   # hfi1_eprom -V -o 
   Using device: /sys/bus/pci/devices/0000:04:00.0/resource0 
   loader file version: 1.4.2.0.0 
```

```
   # hfi1_eprom -V -c
```

- run upgrade command as instructed in OmniPath release notes 
```
   # cd /usr/share/opa/bios_images
   # hfi1_eprom -w -o HfiPcieGen3Loader_1.6.0.0.0.rom -b HfiPcieGen3_1.6.0.0.0.efi -c /lib/firmware/updates/hfi1_platform.dat

   Using device: /sys/bus/pci/devices/0000:04:00.0/resource0
   Erasing loader file... done
   Writing loader file... done
   Erasing driver file... done
   Writing driver file... done
```

- validate TMM firmware file *hfi1_smbus.fw*
```
   # opatmmtool -f /lib/firmware/updates/hfi1_smbus.fw fileversion
```

- check current TMM version
```
   # opatmmtool -fwversion 
```

```
   # opahfirev
   rdma-qe-15  - HFI 0000:04:00.0
   HFI:   hfi1_0
   Board: ChipABI 3.0, Board ID 0x1, ChipRev 7.17, SW Compat 3
   SN:    0x00790311
   Location:Discrete Socket:1 PCISlot:00 NUMANode:1 HFI0
   Bus:   Speed 8GT/s, Width x16
   GUID:  0011:7501:0179:0311
   SiRev: B1 (11)
   TMM:   10.0.0.0.696
```

- update TMM if required
```
   # opatmmtool -f /lib/firmware/updates/hfi1_smbus.fw update

   opatmmtool: Opened the driver interface
   File Firmware Version=10.2.1.0.3
   opatmmtool: Firmware length=51468
   opatmmtool: Waiting for device to erase flash...
   opatmmtool: Transmitting firmware
   opatmmtool: Successfully transmitted firmware to device
   opatmmtool: Firmware transmitted, wait for device ready
   Current Firmware Version=10.2.1.0.3
   Firmware Update Completed
```

- restart TMM micro-controller
```
   # opatmmtool reboot
```

- Reboot host & verify firmware version
```
   # hfi1_eprom -V -b 
   # hfi1_eprom -V -o
   # hfi1_eprom -V -c
   # opatmmtool -fwversion
```

## OmniPath Firmware Matrix

For reference Omnipath HFI firmware versions documented in Intel OPA  release notes:

|OPA version | Date      | HFI UEFI fw | HFI TMM fw    | source document                            |
|:-----------|-----------|------------:|--------------:|:-------------------------------------------|
|10.3.1      | 2017 Feb  | 1.3.2.0.0   | 10.2.1.0.3    | Intel_OP_Software_RN_J52019_v1_0.pdf       |
|10.3.2      | 2017 Sept | 1.3.2.0.0   | 10.2.1.0.3    | Intel_OP_Software_10_3_2_RN_J64261_v2_0.pdf|
|10.4.1      | 2017 May  | 1.4.0.0.0   | 10.4.0.0.146  | Intel_OP_Software_10_4_1_RN_J64255_v1_0.pdf|
|10.4.2      | 2017 June | 1.4.2.0.0   | 10.4.0.0.146  | Intel_OP_Software_10_4_2_RN_J66909_v1_0.pdf|
|10.5        | 2017 Sept | 1.5.2.0.0   | 10.4.0.0.146  | Intel_OP_Software_10_5_RN_J75208_v3_0.pdf  |
|10.6        | 2017 Oct  | 1.6.0.0.0   | 10.4.0.0.146  | Intel_OP_Software_10_6_RN_J82662_v1_0.pdf  |

[Optional] Lustre rpm update
====================

NOTE: Ensure the rpm packages have been approved by DDN engineering

   NOTE: Recommended to use pre-built rpm packages available from DDN engineering.
   The latest ES 3.2.0 rpm packages can be obtained from [EXAScaler jenkins](https://es-build.datadirectnet.com/view/Core/job/ES-3.2-core-rpms-installer/) or [ES Jenkins downstream build](https://es-build.datadirectnet.com/job/ES-3.2-core-rpms-staging-lustre/) (DDN VPN access required) 

   ES 3.2.0 rpm upgrade list: 
   kmod-lustre-common, kmod-lustre-el7.3, kmod-lustre-el7.3-ldiskfs, kmod-lustre-el7.3-mlnx3.4-o2ib-mlnx, kmod-lustre-el7.3-osd-ldiskfs, lustre, lustre-devel, lustre-iokit, lustre-osd-ldiskfs-mount, lustre-server, lustre-source, lustre-tests

   eg) upgrade ES 3.2.0 (lustre-2.7.21.3-18.ddn8) to lustre-2.7.21.3-90.ddn11 

   Example
```
# yum upgrade  kmod-lustre-common.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm \
kmod-lustre-el7.3.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm \
kmod-lustre-el7.3-ldiskfs.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm \
kmod-lustre-el7.3-mlnx3.4-o2ib-mlnx.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm \
kmod-lustre-el7.3-osd-ldiskfs.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm \
lustre.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm \
lustre-devel.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm \
lustre-iokit.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm \
lustre-osd-ldiskfs-mount.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm \
lustre-server.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm \
lustre-source.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm \                   
lustre-tests.x86_64 0:2.7.21.3-90.ddn11.g83d7061.el7.rpm
```

- ES 3.2.0 kmod-lustre & lustre upgrade list: 

  kmod-lustre-common, kmod-lustre-el7.3, kmod-lustre-el7.3-ldiskfs, kmod-lustre-el7.3-mlnx3.4-o2ib-mlnx, kmod-lustre-el7.3-osd-ldiskfs, lustre, lustre-devel, lustre-iokit, lustre-osd-ldiskfs-mount, lustre-server, lustre-source, lustre-tests



   Example update to ddn-lustre 2.7.21.3.ddn25
```
# yum kmod-lustre-* lustre-*

...
Resolving Dependencies
--> Running transaction check
---> Package kmod-lustre-common.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package kmod-lustre-common.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
---> Package kmod-lustre-el7.4.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package kmod-lustre-el7.4.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
---> Package kmod-lustre-el7.4-ldiskfs.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package kmod-lustre-el7.4-ldiskfs.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
---> Package kmod-lustre-el7.4-mlnx4.3-o2ib-mlnx.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package kmod-lustre-el7.4-mlnx4.3-o2ib-mlnx.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
---> Package kmod-lustre-el7.4-osd-ldiskfs.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package kmod-lustre-el7.4-osd-ldiskfs.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
---> Package lustre.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package lustre.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
---> Package lustre-devel.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package lustre-devel.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
---> Package lustre-iokit.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package lustre-iokit.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
---> Package lustre-osd-ldiskfs-mount.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package lustre-osd-ldiskfs-mount.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
---> Package lustre-server.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package lustre-server.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
---> Package lustre-source.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package lustre-source.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
---> Package lustre-tests.x86_64 0:2.7.21.3-256.ddn20.g10dd357.el7 will be updated
---> Package lustre-tests.x86_64 0:2.7.21.3-272.ddn25.g9b5a642.el7 will be an update
--> Finished Dependency Resolution
```
