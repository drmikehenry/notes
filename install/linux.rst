###############################
Linux Cross-distro Installation
###############################

.. contents::

***************
Reference notes
***************

Vim shortcut for pasting
========================

For convenient pasting from Vim, use ``~/bin/pastewin`` against a convenient
KDE Konsole (e.g., ``konsole-3``)::

  nmap <f8> VY:silent !pastewin konsole-3<CR>
  xmap <f8>  Y:silent !pastewin konsole-3<CR>

Then press <F8> on a line to paste it, or visually select to paste the
entire selection.

To remove maps::

  nunmap <f8>|xunmap <f8>

pastewin
--------

#!/bin/bash


if [ $# -ne 1 ]; then
    cat <<'EOF'
Usage: pastewin WINDOWPATTERN

Uses ``xdotool`` to paste into another window using
the ``Shift+Insert`` shortcut.  This works for Konsole
(the primary use case) and might work for other apps
as well.

WINDOWPATTERN is matched against the window name, class,
and classname.  Exactly one window must match; if more
or fewer match, nothing is done.
EOF
    exit 1
fi


die()
{
    echo "$@" 1>&2
    exit 2
}


pattern="$1"

oldwin=$(xdotool getwindowfocus) ||
    die "Could not determine old window"

newwin=$(xdotool search --onlyvisible --name --class --classname "$pattern") ||
    die "failed to find matching window"

# If replacing newlines with spaces changes things, found 2 or more windows.
newwin2=${newwin//$'\n'/ }
[ "$newwin" == "$newwin2" ] ||
    die "matched multiple windows: $newwin2"

xdotool windowactivate --sync "$newwin" &&
xdotool key 'shift+Insert' &&
xdotool windowactivate --sync "$oldwin"


Package management
==================

Ubuntu
------

- References:

  - https://help.ubuntu.com/community/Repositories/CommandLine
  - https://help.ubuntu.com/community/MetaPackages

- Update list of available packages::

    apt-get update

- Install a package::

    apt-get install packagename
    # Can also remove using "install" command:
    apt-get install PACKAGE_TO_REMOVE-

- Reinstallation::

    apt-get --reinstall install PACKAGE

- Remove a package::

    apt-get remove packagename
    # Can also install using "remove" command:
    apt-get remove PACKAGE_TO_INSTALL+

  Full removal, including config files::

    apt-get --purge remove PACKAGE

  Full removal after partial removal::

    apt-get purge PACKAGE

- Upgrade one more more packages, without adding or removing any packages::

    apt-get upgrade

  Show list of packages to be upgraded, without doing it::

    apt-get -u upgrade
    apt-get --dry-run upgrade

- Upgrade one more more packages, allowing additions and removals::

    apt-get dist-upgrade

  Show list of packages to be upgraded, without doing it::

    apt-get -u dist-upgrade
    apt-get --dry-run dist-upgrade

- List versions of installed package::

    apt-show-versions PACKAGE

- List packages that are upgradeable::

    apt-show-versions -u

- Search for packages::

    apt-cache search KEYWORD_OR_PACKAGE

- See details of particular package to install::

    apt-cache show PACKAGE

- More general information (fewer details)::

    apt-cache showpkg PACKAGE

- Just show dependency information::

    apt-cache depends PACKAGE

- Just show reverse dependency information::

    apt-cache rdepends PACKAGE

- Map filenames back to packages::

    apt-file search FILENAME

  (similar to ``dpkg -S``, but works on uninstalled packages too)

- List contents of a package::

    apt-file list PACKAGE

- Update database of package files::

    apt-file update


- Installing source packages::

    apt-get source PACKAGE

- Install and auto-build source package::

    apt-get -b source PACKAGE

- Build source package later, after downloading::

    dpkg-buildpackage -rfakeroot -uc -b

- Install a local PACKAGE.deb file::

    dpkg -i PACKAGE.deb

  Auto-fix missing dependencies after ``dpkg -i`` failure::

    apt-get install -f

- Install packages needed to build source PACKAGE::

    apt-get build-dep PACKAGE

  Note: must have enabled the corresponding source repository.

- Just show the packages needed to build source PACKAGE::

    apt-cache showsrc PACKAGE

- Query package owning a file::

    dpkg --search filename-pattern
    dpkg -S filename-pattern
    dpkg -S /path/to/file

- Query installed packages::

    dpkg --list
    dpkg -l

    dpkg --list package-name-pattern
    dpkg -l package-name-pattern

- Query installed package contents::

    dpkg --listfiles packageName
    dpkg -L packageName

- Query installed package info::

    dpkg --status packageName
    dpkg -s packageName

- Query package file contents::

    dpkg --contents packageFile.deb

- Query package file info::

    dpkg --info packageFile.deb

- Display updates for existing packages::

    apt-get dist-upgrade --dry-run

- Configuration of repositories via command-line:
  https://help.ubuntu.com/community/Repositories/CommandLine

- Adding a Personal Package Archive (ppa) can be done as follows::

    add-apt-repository ppa:someppa/ppa

  This will create a file in ``/etc/apt/sources.list.d/``, e.g. this command::

    add-apt-repository ppa:disper-dev/ppa

  Results in this file::

    /etc/apt/sources.list.d/disper-dev-ubuntu-ppa-wily.list

- Removing is similar::

    add-apt-repository --remove ppa:someppa/ppa

- Manual temporary removal may be done by renaming the ``.list`` file to
  ``.list.save``, e.g.::

    mv /etc/apt/sources.list.d/some-file.list{,.save}
    apt-get update

- Using non-ppa repositories is a bit more verbose, e.g.::

    add-apt-repository "deb http://download.virtualbox.org/virtualbox/debian $(lsb_release -cs) contrib"

  Note that this adds lines directly to ``/etc/apt/sources.list``, rather than
  to a separate file in ``/etc/apt/sources.list.d``.

  In addition, the repository key must be added, e.g.::

    curl http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc |
      apt-key add -

- To purge everything from a given repository, first install the ppa-purge
  tool::

    agi ppa-purge

  Then purge as follows::

    ppa-purge ppa:someppa/ppa

[Fedora]
--------

- In Fedora 22, ``yum`` became obsolete.  It is replaced with ``dnf`` (Dandified
  Yum).  Mostly, they have compatible syntax.

- See ``man yum2dnf`` for details on changes.

- To examine groups in detail::

    dnf group list -v

  The Group Name comes first (e.g., ``KDE Plasma Workspaces``), and the
  groupid (GID) comes next (e.g., ``kde-desktop-environment``).

[CentOS]
--------

- CentOS still uses ``yum``.

- Can use ``yumdb`` to view yum database information.  E.g.::

    yumdb info cabextract

  yielding the following (note the ``from_repo`` field)::

    Loaded plugins: fastestmirror, langpacks
    cabextract-1.5-1.el7.x86_64
         checksum_data = 844c5274f2c7fae5a35f62f566f4030c3bdec3fd7544d8edcdbaff111ea5b794
         checksum_type = sha256
         command_line = -y install curl cabextract xorg-x11-font-utils fontconfig
         from_repo = epel
         from_repo_revision = 1509287263
         from_repo_timestamp = 1509289057
         installed_by = 0
         origin_url = http://mirror.cs.pitt.edu/epel/7/x86_64/Packages/c/cabextract-1.5-1.el7.x86_64.rpm
         reason = user
         releasever = 7
         var_infra = stock
         var_uuid = 5e782d19-b783-4bb3-b78b-1322a030107b

Old releases
============

- [Ubuntu] When running out-of-date distro:

  https://askubuntu.com/questions/91815/how-to-install-software-or-upgrade-from-an-old-unsupported-release

  Basically, :%s/us.archive.ubuntu.com/old-releases.ubuntu.com/g
  in /etc/apt/sources.list

Firewall rules
==============

Ubuntu
------

- Uncomplicated Firewall (``ufw``) is installed by default.

- See helpful Wiki page for examples:
  https://help.ubuntu.com/community/UFW

- Allow or deny a port::

    ufw allow 5355/udp
    ufw deny 5355/udp

- Setup to "deny" by default, allow ssh, then enable firewall::

    ufw default deny
    ufw allow ssh
    ufw enable

- To enable logging::

    ufw logging on

  Then watch log::

    tail --follow=name /var/log/kern.log

  Disable logging via::

    ufw logging off

Fedora, CentOS
--------------

- **WARNING** You have to invoke ``firewall-cmd`` **TWICE** to make changes;
  the first invocation will update the running firewall, and the second will use
  ``--permanent`` to make the same change in the permanent files.

- Get the status of firewalld::

    firewall-cmd --state

- Reload without losing state::

    firewall-cmd --reload

- Complete reload, losing state (only in case of severe firewall problems)::

    firewall-cmd --complete-reload

- Get list of zones::

    firewall-cmd --get-zones

- Get list of services::

    firewall-cmd --get-services

- List all zones with details::

    firewall-cmd --list-all-zones

- Add a service::

    firewall-cmd --add-service ssh
    firewall-cmd --permanent --add-service ssh

- Remove a service::

    firewall-cmd --remove-service ssh
    firewall-cmd -permanent --remove-service ssh

- Query a service (exit code is one if enabled, zero otherwise; no output)::

    firewall-cmd --query-service ssh

- Add/remove/query a port number (instead of a service name)::

    firewall-cmd --add-port 22
    firewall-cmd --permanent --add-port 22
    firewall-cmd --remove-port 22
    firewall-cmd --permanent --remove-port 22
    firewall-cmd --query-port 22
    firewall-cmd --permanent --query-port 22

Systemd boot targets
====================

- With systemd, it's not longer ``/etc/inittab`` that determines the "run level"
  for booting; instead, the default target determines that.

- Determining default boot target::

    systemctl get-default

- Setup for text-only login at next boot::

    systemctl set-default multi-user.target

- Setup for graphical login at next boot::

    systemctl set-default graphical.target

- Make a target active right now::

    systemctl isolate graphical.target

Partitioning
============

gdisk type codes
----------------

0700 Microsoft basic data  0c01 Microsoft reserved    2700 Windows RE
4100 PowerPC PReP boot     4200 Windows LDM data      4201 Windows LDM metadata
7501 IBM GPFS              7f00 ChromeOS kernel       7f01 ChromeOS root
7f02 ChromeOS reserved     8200 Linux swap            8300 Linux filesystem
8301 Linux reserved        8302 Linux /home           8400 Intel Rapid Start
8e00 Linux LVM             a500 FreeBSD disklabel     a501 FreeBSD boot
a502 FreeBSD swap          a503 FreeBSD UFS           a504 FreeBSD ZFS
a505 FreeBSD Vinum/RAID    a580 Midnight BSD data     a581 Midnight BSD boot
a582 Midnight BSD swap     a583 Midnight BSD UFS      a584 Midnight BSD ZFS
a585 Midnight BSD Vinum    a800 Apple UFS             a901 NetBSD swap
a902 NetBSD FFS            a903 NetBSD LFS            a904 NetBSD concatenated
a905 NetBSD encrypted      a906 NetBSD RAID           ab00 Apple boot
af00 Apple HFS/HFS+        af01 Apple RAID            af02 Apple RAID offline
af03 Apple label           af04 AppleTV recovery      af05 Apple Core Storage
be00 Solaris boot          bf00 Solaris root          bf01 Solaris /usr & Mac Z
bf02 Solaris swap          bf03 Solaris backup        bf04 Solaris /var
bf05 Solaris /home         bf06 Solaris alternate se  bf07 Solaris Reserved 1
bf08 Solaris Reserved 2    bf09 Solaris Reserved 3    bf0a Solaris Reserved 4
bf0b Solaris Reserved 5    c001 HP-UX data            c002 HP-UX service
ea00 Freedesktop $BOOT     eb00 Haiku BFS             ed00 Sony system partitio
ef00 EFI System            ef01 MBR partition scheme  ef02 BIOS boot partition
fb00 VMWare VMFS           fb01 VMWare reserved       fc00 VMWare kcore crash p
fd00 Linux RAID

General partitioning information
--------------------------------

- Arch notes:

  - https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface

  - https://wiki.archlinux.org/index.php/GRUB

- Use ``gdisk`` to manipulate partitions on a GPT disk interactively::

    gdisk /dev/sda

  Use ``sgdisk`` for scripted operations.

- Make sure every partition starts on a 1 MB boundary (typically 2048 sectors).
  By default, ``gdisk`` will start the first partition at sector 2048 to stay
  aligned, but it's worth verifying this during partitioning.

- For BIOS-based systems, a BIOS boot partition is required as the first part of
  any GPT drive (at the start of the drive).  Use 1 MB as the size.  Use hex
  code ``ef02`` for BIOS boot partition.  This partition type is needed for
  BIOS-based motherboards that must boot a GPT-partitioned drive.  This is
  because grub needs space to expand beyond the few bytes available in the MBR.
  It must be at minimum 30 KB or so, but 1 MB provides some room to grow.
  Though a BIOS boot partition isn't required on EFI-based systems, it's cheap
  so it might as well be there.

- For EFI-based systems, an EFI System Partition is required.  It must be 512
  MB at minimum, but 1 GB is comfortable for expansion.  Use type ``ef00`` for
  EFI System Partition.  It should be FAT32-formatted, which can be
  accomplished via::

    mkfs.fat -F32 /dev/sda2

- When using Linux RAID or btrfs, a separate ``/boot`` partition is required.
  Use type ``8300`` ("Linux"), with 1 GB of space (500 MB at bare minimum).

- Create the "big" partition to take up almost all remaining space (typically
  for RAID, but applies to whichever partition should get the "rest" of the
  drive).  This is easiest to do by creating all other partitions first, then
  creating the big partition last.  Alternatively, another way is to create a
  temporary dummy partition with size equal to the desired extra space, create a
  temporary "big" partition to determine the size, and measure the size with
  ``gdisk`` ``i`` command.  Round this number of sectors down to a multiple of 1
  MB by dividing by 2048, then re-multiplying the integer portion again by 2048.
  Finally, delete both and re-create the RAID volume with the calculated number
  of sectors.  Use type ``fd00`` for Linux software RAID.

- To clone partition tables across drives, use ``sgdisk`` as follows::

    for i in b c d e f; do
      sgdisk -R /dev/sd$i /dev/sda
      sgdisk -G /dev/sd$i
    done

- To print partition information for a drive::

    sgdisk -p /dev/sda

- To wipe partition information clean, "zap" the drive::

    gdisk /dev/sda

    x  (to use expert's menu)
    z  (to zap and exit)

  Using sgdisk::

    sgdisk --zap-all /dev/sda

RAID commands
-------------

- To create a RAID device:

  - RAID10 far layout (recommended for RAID10)::

      mdadm --create /dev/md/0 --level raid10 --layout f2 -n 4 /dev/sd[abcd]1

  - RAID10 near layout::

      mdadm --create /dev/md/1 --level raid10 -n 4 /dev/sd[abcd]1

  - RAID0 (striped volume)::

      mdadm --create /dev/md/0 --level raid0 -n 2 /dev/sd[ab]1

- To force synchronization of a RAID volume::

    echo check > /sys/block/md/0/md/sync_action

- To mount udev system on /mnt/alt/root/dev::

    mount -o bind /dev /mnt/alt/root/dev

- To assemble a previously existing RAID array from its pieces:

  - Examine the raw partitions::

      mdadm --examine /dev/sda1

    Note UUID field, e.g.::

      UUID : 40dd77f4:3c31be30:2e1d0dea:7b7155b9

    There will be two or more raw partitions that share this UUID.

  - Assemble the array, specifying the UUID and the desired MD device::

      mdadm --assemble --uuid 40dd77f4:3c31be30:2e1d0dea:7b7155b9 /dev/md/0

  - Re-generate ``/etc/mdadm.conf`` via a scan::

      mdadm --examine --scan > /etc/mdadm.conf

  - Assemble array using configuration file information::

      mdadm --assemble /dev/md2

- To remove/destroy an existing RAID array:

  - Remove any LVM volume groups::

      vgremove name-of-group

  - Stop the RAID array::

      mdadm --stop /dev/md/0

  - Then zero the superblocks for all associated partitions::

      for i in a b c d e f; do mdadm --zero-superblock /dev/sd${i}1; done

journalctl replaces syslog
==========================

New idioms:

- ``cat /var/log/messages`` becomes ``journalctl``.

- ``tail -f /var/log/messages`` becomes ``journalctl -f``.

- ``grep foobar /var/log/messages`` becomes ``journalctl | grep foobar``.

Memory testing
==============

- Boot Linux distro, execute memtest86+ for a couple of days.

Drive testing
=============

- Hard drive burn-in:

  - https://forums.freenas.org/index.php?threads/how-to-hard-drive-burn-in-testing.21451/

  - https://www.thomas-krenn.com/en/wiki/SMART_tests_with_smartctl

- Test all drives at the same time (but only one test running on each drive at
  one time).  Between tests, look at status output via::

    for i in a b c d e f g h; do smartctl -a /dev/sd$i; done | less

  Look for ``SMART Self-test log structure`` section showing status of completed
  tests, and look for ``Self-test execution status`` to see if a test is
  currently ongoing.

  Tests::

    for i in a b c d e f g h; do smartctl -t short /dev/sd$i; done
    for i in a b c d e f g h; do smartctl -t conveyance /dev/sd$i; done
    for i in a b c d e f g h; do smartctl -t long /dev/sd$i; done

- Destructive read-write tests via badblocks:

  - Execute these in separate login prompts::

      badblocks -p100 -svw /dev/sda
      badblocks -p100 -svw /dev/sdb
      badblocks -p100 -svw /dev/sdc
      badblocks -p100 -svw /dev/sdd
      badblocks -p100 -svw /dev/sde
      badblocks -p100 -svw /dev/sdf

Disk-related
============

- Use ``blkid`` to see the block device UUIDs, e.g.::

    $ blkid
    /dev/sda1: UUID="Z6L1pb-..." TYPE="LVM2_member"
    /dev/mapper/vg_t61-lv_swap: UUID="c1b48db8-..." TYPE="swap"
    /dev/mapper/vg_t61-lv_ubuntu10.10: UUID="25cb5f65-..." TYPE="ext4"

- Check partition information::

    cat /proc/partitions

  Sample output::

    major minor  #blocks  name

       8        0  312571224 sda
       8        1  311564578 sda1
    [...]

- Check size of a block device::

    # In 512-byte sectors:
    blockdev --getsize /dev/sda

    # In bytes:
    blockdev --getsize64 /dev/sda

    # Show report of all block devices or single device::
    blockdev --report
    blockdev --report /dev/sda

    # Report on LV as well::
    blockdev --report /dev/mapper/vg_SERVER-lv_test

- Can also view UUID via::

    ls -l /dev/disk/by-uuid/

- Rescan a partition via::

    echo 1 > /sys/block/sdc/device/rescan

  Seems likely to work for PCI devices, since this is a symlink::

    readlink /sys/block/sdc
    ../devices/pci0000:00/..../block/sdc

- Show which partitions the kernel thinks it has::

    cat /proc/partitions

    major minor  #blocks  name

       8        0  125034840 sda
       8        1  124022784 sda1
       8        2     488448 sda2
       8       16  125034840 sdb
       8       17  124022784 sdb1
       8       18     512000 sdb2
       9        0  248043520 md0
     253        0   11718656 dm-0
     253        1   25600000 dm-1
     253        2  146481152 dm-2
     253        3   39059456 dm-3

- Hard drive burn-in:

  - https://forums.freenas.org/index.php?threads/how-to-hard-drive-burn-in-testing.21451/

  - https://www.thomas-krenn.com/en/wiki/SMART_tests_with_smartctl

- Test all drives at the same time (but only one test running on each drive at
  one time).  Between tests, look at status output via::

    for i in a b c d e f g h; do smartctl -a /dev/sd$i; done | less

  Look for ``SMART Self-test log structure`` section showing status of completed
  tests, and look for ``Self-test execution status`` to see if a test is
  currently ongoing.

  Tests::

    for i in a b c d e f g h; do smartctl -t short /dev/sd$i; done
    for i in a b c d e f g h; do smartctl -t conveyance /dev/sd$i; done
    for i in a b c d e f g h; do smartctl -t long /dev/sd$i; done

- Destructive read-write tests via badblocks:

  - Execute these in separate login prompts::

      badblocks -p100 -svw /dev/sda
      badblocks -p100 -svw /dev/sdb
      badblocks -p100 -svw /dev/sdc
      badblocks -p100 -svw /dev/sdd
      badblocks -p100 -svw /dev/sde
      badblocks -p100 -svw /dev/sdf

- Mounting a drive image (``dd`` image) using loopback device:

  Given file ``x.dd`` as a raw image of a drive having two partitions with NTFS
  volumes on them::

    DEV=$(sudo losetup --partscan --find --read-only --show x.dd)
    mkdir partition_1 partition_2
    sudo mount -t ntfs -o ro "${DEV}p1" partition_1
    sudo mount -t ntfs -o ro "${DEV}p2" partition_2

GNU Parted
==========

- Launch against a particular drive::

    parted /dev/sda

- Print out partitions::

    print

- Create a new "disk label" (partition table) for GUID Partition Table (GPT)::

    mklabel gpt

- Create a new primary partition for RAID::

    mkpart primary start end

  E.g., using sector units to match::

    mkpart primary 34s 3902343784s

- Set RAID flag on partition #1::

    set 1 raid on

Info on UUID
============

- Explanation of a good bit of UUID on-disk layout:
  http://ubuntuforums.org/showthread.php?t=1286774

Hardware-related
================

- Use ``lspci`` to scan PCI devices.

- Use ``lsusb`` to scan USB devices.

- Use ``lspcmcia`` to scan PC-CARD/PCMCIA devices.

- Use ``lshal`` to show HAL information.

- Use ``lshw`` to list hardware information.

- Use ``lshw-gui`` (Fedora) or ``lshw-gtk`` (Ubuntu) for GUI version.

- Use ``dmidecode`` to display system DMI table (SMBIOS information).

- Use ``cat /proc/scsi/scsi`` to see hard disk devices.

Keyboard repeat rate/delay
==========================

- Query current rate/delay settings::

    xset q | grep 'repeat delay'

  With result::

    auto repeat delay:  250    repeat rate:  32

- Change via::

    xset r rate 250 32

Benchmarking
============

- Time disk read speed::

    hdparm -t /dev/sda

Using ``/etc/alternatives``
===========================

- See the ``update-alternatives`` tool to manipulate the alternatives
  symlinks.

- To install locally compiled Gvim::

    for i in gvim gview gvimdiff; do
      update-alternatives --install /usr/bin/$i $i /usr/local/bin/$i 500
    done

Unix groups
===========

- In Ubuntu, the group definitions are found in:
  /usr/share/doc/base-passwd/users-and-groups.html

- Selected group meanings:

  - adm - for monitoring tasks.
  - dialout - direct access to serial ports.
  - fax - can use fax software.
  - cdrom - direct access to CD-ROM.
  - floppy - direct access to floppy drive.
  - tape - direct access to tape drive.
  - video - direct access to a video device.
  - plugdev - direct access to certain removable devices w/o fstab help.
  - lpadmin - add, modify, remove printers.

  Normal user can get rid of tape, floppy to save space on the 16-group
  limit for NFS3.

Querying configuration
======================

- The ``getconf`` tool allows for querying of various variables.

  - To see which variables are available::

      getconf -a

  - To query a particular variable::

      getconf VARIABLE_NAME

  - To query the maximum command-line length (e.g., as used by xargs)::

      getconf ARG_MAX

    (Currently on Fedora 17, ARG_MAX is 2 MB.)

wget
====

- Mirroring via wget::

    wget -m --no-parent http://somesite.com/some/path/

  If robots.txt is stopping things, then either add the following to
  ``~/.wgetrc``::

    robots = off

  Or else use ``-e`` to pass this along::

    wget -e 'robots=off' -m --no-parent http://somesite.com/some/path/

[Fedora] [CentOS] SELinux management
====================================

- SELinux references:

  - http://wiki.centos.org/HowTos/SELinux

  - http://www.centos.org/docs/5/html/Deployment_Guide-en-US/sec-sel-building-policy-module.html

  - http://www.centos.org/docs/5/html/Deployment_Guide-en-US/sec-sel-policy-customizing.html

  - http://www.gentoo.org/proj/en/hardened/selinux/selinux-handbook.xml?part=2&chap=5

  - http://www.slideshare.net/biertie/how-to-live-with-selinux

- Determining SELinux failures::

    journalctl | grep --after 5 SELinux

  Typical output::

    Jan 05 04:18:53 SERVER.DOMAIN.com setroubleshoot[31961]: SELinux is preventing /usr/bin/python2.7 from getattr access on the file /usr/bin/rpm. For complete SELinux messages. run sealert -l 8998b74b-4d0d-4e58-abd5-46e1a7a03d1f

  To see the details from the above output::

    sealert -l 8998b74b-4d0d-4e58-abd5-46e1a7a03d1f

- Audit to see why an access has failed an access::

    sealert -l 8998b74b-4d0d-4e58-abd5-46e1a7a03d1f | audit2why

    type=AVC msg=audit(1388913532.542:2824): avc:  denied  { getattr } for  pid=16011 comm="pyzor" path="/usr/bin/rpm" dev="dm-1" ino=1443104 scontext=system_u:system_r:spamc_t:s0 tcontext=system_u:object_r:rpm_exec_t:s0 tclass=file

            Was caused by:
                    Missing type enforcement (TE) allow rule.

                    You can use audit2allow to generate a loadable module to allow this access.

- Audit to see allow the failed access::

    sealert -l 8998b74b-4d0d-4e58-abd5-46e1a7a03d1f | audit2allow

    #============= spamc_t ==============
    allow spamc_t rpm_exec_t:file getattr;

- Create module file to hold the result; write resulting "Type Enforcement" (TE)
  policy to stdout::

    sealert -l 8998b74b-4d0d-4e58-abd5-46e1a7a03d1f | audit2allow -m local_mail

    module local_mail 1.0;

    require {
            type spamc_t;
            type rpm_exec_t;
            class file getattr;
    }

    #============= spamc_t ==============
    allow spamc_t rpm_exec_t:file getattr;

  Typically this is stored in a "Type Enforcement" file ending in ``.te``.

- Compile the Type Enforcement file into a module::

    checkmodule -M -m -o local_mail.mod local_mail.te

- Package this module::

    semodule_package -o local_mail.pp -m local_mail.mod

- Load the package and make it active; this is persistent across reboots::

    semodule -i local_mail.pp

- Remove a module; no longer persists::

    semodule -r local_mail

- Unpackage a module to see what's inside::

    semodule_unpackage local_mail.pp

[Fedora] [CentOS] SELinux port management
=========================================

To enable ``name_bind`` for a port (e.g., for allowing SSHD to bind to a
non-standard port), use ``semanage``.

- To list permitted ports for ssh::

    semanage port -l | grep ssh

  Expect to see::

    ssh_port_t                     tcp      22

- To add a new "name_bind"-capable port (this may take a while; it's unclear
  why it can be so slow)::

    semanage port -a -t ssh_port_t -p tcp 12345

  Now expect to see the following ports for ssh::

    ssh_port_t                     tcp      12345, 22

ZFS
===

ZFS references
--------------

- https://wiki.ubuntu.com/Kernel/Reference/ZFS

- https://forums.freenas.org/index.php?threads/slideshow-explaining-vdev-zpool-zil-and-l2arc-for-noobs.7775/

- https://constantin.glez.de/2010/01/23/home-server-raid-greed-and-why-mirroring-still-best/

- http://jrs-s.net/2015/02/06/zfs-you-should-use-mirror-vdevs-not-raidz/

- https://pthree.org/2013/01/03/zfs-administration-part-xvii-best-practices-and-caveats/

- https://github.com/zfsonlinux/zfs-auto-snapshot

- http://blog.programster.org/sharing-zfs-datasets-via-nfs

- http://open-zfs.org/wiki/Performance_tuning

- https://github.com/zfsnap/zfsnap

- https://wiki.archlinux.org/index.php/ZFS

- https://wiki.archlinux.org/index.php/ZFS/Virtual_disks

- https://github.com/zfsonlinux/zfs/wiki/Ubuntu-16.04-Root-on-ZFS

- https://janweitz.de/article/creating-a-zfs-zroot-raid-10-on-ubuntu-16.04/

- Check health of zfs volume, including scrubbing (for FreeBSD, Ubuntu):
  https://calomel.org/zfs_health_check_script.html

Create a pool
-------------

- **Note** Should use ``/dev/disk/by-id`` names instead of ``/dev/sd?`` names.

- Data should ideally be power-of-two number of disks.  Six disks is nice for
  raidz2 (4 data + 2 parity).

- Assuming partition 1 is for ZFS, create volume named after host::

    zpool create -o ashift=12 \
      -O relatime=on -O canmount=off -O compression=lz4 \
      -O mountpoint=/ -R /mnt \
      SERVER raidz2 /dev/disk/by-id/ata-WDC_WD40EFRX-*-part1

Show pool statistics (size used, size free)
-------------------------------------------

- Show for all pools::

    zpool list

Replace a drive in a volume before failure with a spare drive
-------------------------------------------------------------

- Replace drive sda1 with sdb1, triggering a resilvering operation::

    zpool replace SERVER /dev/sda1 /dev/sdb1
    zpool status SERVER
      pool: SERVER
     state: ONLINE
      scan: resilvered 276K in 0h0m with 0 errors on Mon Jan  9 00:30:01 2017
    config:

            NAME        STATE     READ WRITE CKSUM
            SERVER       ONLINE       0     0     0
              sdb1      ONLINE       0     0     0

    errors: No known data errors

Replace in a raidz volume
-------------------------

- Replace drive sda1 with sdd1, triggering a resilvering operation::

    zpool replace SERVER /dev/sda1 /dev/sdd1
    zpool status SERVER
      pool: SERVER
     state: ONLINE
      scan: resilvered 188K in 0h0m with 0 errors on Mon Jan  9 00:32:53 2017
    config:

            NAME             STATE     READ WRITE CKSUM
            SERVER            ONLINE       0     0     0
              raidz1-0       ONLINE       0     0     0
                replacing-0  ONLINE       0     0     0
                  sda1       ONLINE       0     0     0
                  sdd1       ONLINE       0     0     0
                sdb1         ONLINE       0     0     0
                sdc1         ONLINE       0     0     0

  Note how the above shows a temporary mirror named ``replacing-0`` that binds
  the old sda1 and the new sdd1 together while resilvering.

  After completion::

    zpool status SERVER
      pool: SERVER
     state: ONLINE
      scan: resilvered 188K in 0h0m with 0 errors on Mon Jan  9 00:32:53 2017
    config:

            NAME        STATE     READ WRITE CKSUM
            SERVER       ONLINE       0     0     0
              raidz1-0  ONLINE       0     0     0
                sdd1    ONLINE       0     0     0
                sdb1    ONLINE       0     0     0
                sdc1    ONLINE       0     0     0

Take a drive offline and bring back online
==========================================

- Create a volume for testing and a filesystem mounted at /mnt::

    zpool create SERVER raidz /dev/sd[abc]1
    zfs create -o mountpoint=/mnt SERVER/test

- Take a drive offline::

    zpool offline SERVER /dev/sda1
    zpool status

      pool: SERVER
     state: DEGRADED
    status: One or more devices has been taken offline by the administrator.
            Sufficient replicas exist for the pool to continue functioning in a
            degraded state.
    action: Online the device using 'zpool online' or replace the device with
            'zpool replace'.
      scan: none requested
    config:

            NAME        STATE     READ WRITE CKSUM
            SERVER       DEGRADED     0     0     0
              raidz1-0  DEGRADED     0     0     0
                sda1    OFFLINE      0     0     0
                sdb1    ONLINE       0     0     0
                sdc1    ONLINE       0     0     0

- Generate a big file::

    dd bs=1024k count=4000 if=/dev/urandom of=/mnt/bigfile

- Bring the drive back online and check status::

    zpool online SERVER /dev/sda1
    zpool status

  With a large-sized file, you can see the resilvering taking place when the
  drive is brought back online.

Initiate a scrub
================

- To scrub a given volume::

    zpool scrub VOLUME

- To check progress of scrub operation::

    zpool status VOLUME

  Getting back, for example::

    # Output:
      pool: SERVER
     state: ONLINE
      scan: scrub repaired 0 in 1h27m with 0 errors on Sun Mar 11 01:51:02 2018
    config:

            NAME                   STATE     READ WRITE CKSUM
            SERVER                  ONLINE       0     0     0
              mirror-0             ONLINE       0     0     0
                ata-WDC_WD40[...]  ONLINE       0     0     0
                ata-WDC_WD40[...]  ONLINE       0     0     0
                ata-WDC_WD40[...]  ONLINE       0     0     0
              mirror-1             ONLINE       0     0     0
                ata-WDC_WD40[...]  ONLINE       0     0     0
                ata-WDC_WD40[...]  ONLINE       0     0     0
                ata-WDC_WD40[...]  ONLINE       0     0     0

    errors: No known data errors

****************
Pre-installation
****************

Machine hardware configuration
==============================

- Ensure SATA is configured in BIOS as AHCI.

- Setup BIOS to boot as UEFI (if desired).

- Setup boot order to boot from USB first (if desired).

- [VM] Make sure network is configured for bridging (if desired).

- [VM] Set Processors to 2 in VirtualBox System | Processor.

Boot media creation
===================

- Can ``dd`` a .iso file directly to a USB flash drive and boot from that.

- [Ubuntu] Download *both* desktop and server iso files.

  - Need desktop iso for "Try Ubuntu" mode, installing supporting utilities,
    etc.

  - Need server iso for installing onto RAID volumes.

  - Ubuntu 18.04:

    - Desktop: ubuntu-18.04.1-desktop-amd64.iso

    - Server: ubuntu-18.04.1-server-amd64.iso

- [Fedora] Download the live DVD iso file from:
  http://fedoraproject.org/en/get-fedora-all

  - Fedora 20: Fedora-Live-Desktop-x86_64-20-1.iso

- [CentOS] Download the DVD iso file from a mirror at:
  http://isoredirect.centos.org/centos/7/isos/x86_64/

  - CentOS 7.4: CentOS-7-x86_64-DVD-1708.iso

Ubuntu desktop "Try Ubuntu" for pre-installation
================================================

- Boot Ubuntu desktop installer.

  **NOTE** server installation media does **not** allow using apt.

- "Try Ubuntu without installing".

- Update the apt cache::

    apt-get update

- Install ssh server::

    apt-get install -y openssh-server

- Set password for ``ubuntu`` user to ``ubuntu``::

    passwd ubuntu

- Get IP address of machine::

    ip addr

- On another machine::

    ssh ubuntu@ip_address_of_install_machine

SSD secure erase
================

- Use Ubuntu desktop "Try Ubuntu", as described above.

  (Note: Do not use the server installer, since it doesn't seem to like putting
  the machine to sleep using ``echo -n mem > /sys/power/state``, and the sleep
  trick is often necessary to unfreeze the SSDs.)

- Devices must not be frozen; if they *are* frozen, can try:

  - Use external USB-connected SATA adapter and hotplug the drives after booting
    to avoid BIOS freezing as explained here:
    https://ata.wiki.kernel.org/index.php/ATA_Secure_Erase

  - After booting, suspend to RAM and resume (which hopefully unfreezes the
    drives).

  - Check for "not frozen"::

      hdparm -I /dev/sda | grep frozen

  - If frozen, try to sleep the machine (suspend to RAM) by pressing sleep
    button or by trying::

      echo -n mem > /sys/power/state

    Wake machine back up by pressing power button.

    (Works for LAPTOP, WORKSTATION)

- Set a user password::

    hdparm --user-master u --security-set-pass password /dev/sda

  With output::

    security_password="password"

    /dev/sdd:
    Issuing SECURITY_SET_PASS command, password="password", user=user, mode=high

  Verify it succeeded::

    hdparm -I /dev/sda

  Should see similar to below in ``Security`` section (with ``enabled`` on line
  by itself)::

    Security:
       Master password revision code = 65534
               supported
               enabled
       not     locked
       not     frozen
       not     expired: security count
               supported: enhanced erase
       Security level high
       2min for SECURITY ERASE UNIT. 2min for ENHANCED SECURITY ERASE UNIT.

- Issue erase command::

    time hdparm --user-master u --security-erase password /dev/sda

  With output similar to::

    security_password="password"

    /dev/sdd:
      Issuing SECURITY_ERASE command, password="password", user=user

    real  0m17.148s
    user  0m0.002s
    sys   0m0.003s

- Can also discard all blocks on the device (which may help in lieu of secure
  erase)::

    blkdiscard /dev/sda

GPT partition setup
===================

- Use Ubuntu desktop "Try Ubuntu", as described above.

  (Note: Do not use the server installer, since it lacks ``sgdisk`` and the
  ability to install via ``apt``.)

Standard disk partitioning
--------------------------

- Example system has two drives ``sda`` and ``sdb``.

- If any volumes, pool, RAID devices, etc., have been automatically
  mounted, unmount them before proceeding.

- Wipe out/zeroize anything on the disks::

    # Two drives:
    for i in /dev/sd[ab]; do sgdisk --zap-all $i; done

    # Four drives:
    for i in /dev/sd[abcd]; do sgdisk --zap-all $i; done

- Setup each drive as this table shows, using commands that follow:

    ====  ====== ====   ============================================
    Part  Size   Type   Purpose
    ====  ====== ====   ============================================
     1    (big)  --->   FD00(RAID); 8E00(LVM); BF00(ZFS)
     2    1MB    EF02   BIOS Boot partition
     3    1GiB   EF00   EFI System Partition (ESP)
     4    1GiB   8300   Possible /boot volume
     9    (???)  BF07   End-of-disk padding ("Solaris Reserved 1")
    ====  ====== ====   ============================================

  The BIOS Boot partition must come at the start of the disk, followed by the
  ESP and the possible /boot volume.  The end-of-disk padding comes at the end
  of the disk, and the main data area takes up the largest chunk in the middle.

  **Padding is required for ZFS.  It may also be desirable for overprovisioning
  of SSDs.**

- Allocate the BIOS Boot partition using sector counts for legacy BIOS booting.
  Use ``-a1`` to set the alignment defaults to 1 sector and map sectors 34
  through 2047, taking it to the 1MiB boundary::

    sgdisk /dev/sda -a1 -n 2:34:2047 -t 2:EF02

- Allocate ESP and /boot volumes.  Starting values of ``0`` means the start of
  the biggest unused area::

    sgdisk /dev/sda     -n 3:0:+1GiB -t 3:EF00
    sgdisk /dev/sda     -n 4:0:+1GiB -t 4:8300

- Allocate padding at the end of the disk.  Use approximate values using ``MB``
  instead of ``MiB`` and ``GB`` instead of ``GiB`` to allow ``sgdisk`` to round
  to good boundaries::

    # For overprovisioning padding, choose size (perhaps 10% of physical size):
    sgdisk /dev/sda     -n 9:-28GB:0 -t 9:BF07

    # For ZFS-sized padding:
    sgdisk /dev/sda     -n 9:-100MB:0 -t 9:BF07

- Allocate the remainder of the disk space for the "big" partition::

    # For RAID (type FD00).
    sgdisk /dev/sda     -n 1:0:0 -t 1:FD00

    # For ZFS (type A504).
    sgdisk /dev/sda     -n 1:0:0 -t 1:A504

- Display final partitioning::

    sgdisk /dev/sda -p

  With output::

    Disk /dev/sda: 500118192 sectors, 238.5 GiB
    Logical sector size: 512 bytes
    Disk identifier (GUID): 4AC7E8B4-5F8E-4EF0-B354-8C348494E621
    Partition table holds up to 128 entries
    First usable sector is 34, last usable sector is 500118158
    Partitions will be aligned on 2-sector boundaries
    Total free space is 0 sectors (0 bytes)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1         4196352       441397901   208.5 GiB   FD00
       2              34            2047   1007.0 KiB  EF02
       3            2048         2099199   1024.0 MiB  EF00
       4         2099200         4196351   1024.0 MiB  8300
       9       441397902       500118158   28.0 GiB    BF07

- Clone partition information onto other disks::

    # Two drives [sda, sdb]:
    for i in b; do
      sgdisk -R /dev/sd$i /dev/sda
      sgdisk -G /dev/sd$i
    done

    # Four drives [sda, sdd]:
    for i in b c d; do
      sgdisk -R /dev/sd$i /dev/sda
      sgdisk -G /dev/sd$i
    done

    # Six drives [sdc, sdf]:
    for i in d e f g h; do
      sgdisk -R /dev/sd$i /dev/sdc
      sgdisk -G /dev/sd$i
    done

- Create filesystems on all EFI System Partitions (ESP)::

    # Two drives:
    for i in /dev/sd[ab]3; do mkfs.vfat -F32 $i; done

    # Four drives:
    for i in /dev/sd[abcd]3; do mkfs.vfat -F32 $i; done

- Partitioning is now complete.

RAID setup
==========

- Use Ubuntu desktop "Try Ubuntu", as described above.

  (Note: Do not use the server installer, since it lacks the ability to install
  via ``apt``.  Though it has ``mdadm`` on the media, you have to manually
  install it.)

- Install ``mdadm``::

    apt-get install -y mdadm

- To create a RAID device:

  - RAID10 far layout (recommended for RAID10)::

      mdadm --create /dev/md/0 --level raid10 --layout f2 -n 4 /dev/sd[abcd]1

  - RAID10 near layout::

      mdadm --create /dev/md/1 --level raid10 -n 4 /dev/sd[abcd]1

  - RAID0 (striped volume)::

      # Two drives:
      mdadm --create /dev/md/0 --level raid0 -n 2 /dev/sd[ab]1

      # Four drives:
      mdadm --create /dev/md/0 --level raid0 -n 4 /dev/sd[abcd]1

  - RAID1 (mirrored volume)::

      # Two drives:
      mdadm --create /dev/md/0 --level raid1 -n 2 /dev/sd[ab]1

    Do not worry about this warning::

      mdadm: Note: this array has metadata at the start and
          may not be suitable as a boot device.  If you plan to
          store '/boot' on this device please ensure that
          your boot-loader understands md/v1.x metadata, or use
          --metadata=0.90

LVM setup
=========

- Use Ubuntu desktop installer, "try Ubuntu", as described above.

**NOTE** Look at per-machine instructions for specifics.

- Create an LVM physical volume::

    # For a RAID system:
    pvcreate /dev/md/0

- Create a volume group named after the machine::

    # For host "HOSTNAME":
    vgcreate HOSTNAME /dev/md/0

- Create logical volumes.  Name the root volume after the distro (e.g.,
  "ubuntu18_04").  **Choose machine-specific sizes**::

    lvcreate HOSTNAME -L 50G -n ubuntu18_04
    lvcreate HOSTNAME -L 366G -n home

*****************
Base Installation
*****************

[Ubuntu] Base Install
=====================

- Create partitions and volumes using desktop iso as shown above.

- Boot from Ubuntu Server Install DVD (use ``UEFI`` boot).

- Choose "Install Ubuntu Server".

- Language: English

- Country: United States

- Detect keyboard layout: NO

- Country of origin for keyboard: English (US)

- Keyboard layout: English (US)

- Choose network interface if asked.

- Hostname (single word, not FQDN, no underscore); e.g.,
  ``ubuntu18-04`` for VMs.

- Setup Power User account (will have UID=1000):

  - Full name: Power User
  - Username: poweruser
  - Password:

- Timezone: America/New_York or Eastern

- If asked about mounted volumes, verify what's mounted and unmount it.
  Installer may pre-mount an EFI partition under ``/media``.

- Partition disks: Manual

- Choose mount points from previously created partitions and volumes:

  - Allow reformatting of volumes.

  - Choose ``ext4`` for filesystem type.

  - Typical volumes to mount:  ``/``, ``/boot``, ``/home``
    (see partitioning steps for device names)

- Finish partitioning and write changes to disk.

- Wait for ``Installing the system`` to complete.

- Package manager proxy: None (leave blank)

- Choose ``No automatic updates``.

- Install ``OpenSSH server``; leave the rest for later.

- Wait for ``Installation complete``.

- Reboot into new system.

[Fedora] Base Install
=====================

- Boot from Live CD image ("Live-based") or Network Install image ("Net-based").

- [Live-based] At "Start Fedora Live" option, press Tab key to edit kernel
  options.

- [Net-based] At "Install or upgrade Fedora" option, press Tab key to edit
  kernel options.

- Append any required kernel options for boot (e.g., ``nox2apic``).

- Custom partitioning:

  - [Net-based] Press Alt-F2 and login as root to get a shell.

  - [Live-based] Just run a terminal in the GUI.

  Then follow custom partitioning instructions using ``mdadm`` et al.

- [Live-based] When prompted, choose "Install to Hard Drive".

- Choose English (United States).

- Choose "System | Installation Destination" to choose which devices will
  participate in the installation.  This allows exclusion of drives from even
  being mounted or used as swap, so that the fstab will not even mention them.

- Under "Local Standard Disks" tab, choose drives to use in any capacity (for
  installation or mounting).

- Choose "I will configure partitioning".

- Choose "Done".  "Manual partitioning" will appear.

- **NOTE** The installer for Fedora 22 seems to have a bug wherein something
  crashes and stops the ``md`` driver, losing the RAID volume.  If that
  happens, re-assemble the RAID volume using something like the following::

    # Using scan:
    mdadm --assemble --scan

    # Or being explicit:
    mdadm --assemble /dev/md/0 /dev/sd[ab]4

  Then rescan the volume groups::

    lvm vgscan

  Then re-load the disks on the manual partitioning screen using the
  recycle-like arrow so the LVM volumes are re-detected and restart the
  partitioning process.

  See these bugs for Fedora 22:

  - https://bugzilla.redhat.com/show_bug.cgi?id=1225671

- Keep "Partition Scheme" as "LVM".

- Choose "Custom partitioning".

- At Manual Partitioning screen:

  - To create a mount point:

    - Choose "+" (Add a new mountpoint):

      - Enter mount point (e.g., ``/boot``).

      - Enter size (e.g. ``500 MB``); leave blank to take up the maximum space.

      - Choose "Add mount point".

  - For boot volumes, use a standard partition.

  - For other volumes, use an LVM volume with naming as follows:

    - Volume group named after host (e.g., ``LAPTOP``).

    - Logical volume named after function (e.g., ``fedora20``, ``home``, etc.).

  - When creating a logical volume based on LVM, choose "Modify" on volume
    group to setup group name, volume name, and RAID level for volume group.

  - Use wrench-like icon (next to "+", "-") to configure details of a volume.
    LVM volumes don't appear to be configurable, but standard partitions can be
    forced to reside on a particular drive.

  - **Create partitions in order**, such that standard partitions are created
    before LVM volumes.  This way, the standard partitions will reside in
    ``sda1``, ``sdb1``, etc.

  - Partitions to create (typical; example for LAPTOP):

    - ``/boot``, ``500 MB``, standard partition (sdb1).

    - ``/mnt/alt/boot``, ``500 MB``, standard partition (sdb1).

    - ``/``, ``25 GB``, LVM volume ``fedora20``.

    - ``/mnt/alt/root``, ``25 GB``, LVM volume ``altroot``.

    - ``/home``, ``177 GB`` (remaining space), LVM volume ``home``.

- Choose "Network & Host Name":

  - Choose Hostname name with domain (e.g., ``SERVER.DOMAIN.com``).
    **LOOK INTO THIS MORE**.  Seems like /etc/hostname ought to end up
    with *just* the host's name and not a FQDN:
    http://serverfault.com/questions/331936/setting-the-hostname-fqdn-or-short-name

    Consider adding this to ``/etc/hosts`` like Debian/Ubuntu do::

      127.0.1.1       SERVER.DOMAIN.com   SERVER

    But this means that the IP address will be local when on SERVER proper,
    which might be a change for some software.

- Choose Localization:

  - Choose nearest city (Americas/New York).

- Choose Begin Installation.

- Choose root password.

- Create user:

  - If /home/poweruser already exists, allow the installer to setup that
    directory.  **However**, it may cause problems using an old home
    directory.  If so, move the old /home/poweruser out of the way and create
    a new one as follows (it may be necessary to login via a text terminal)::

      mv /home/poweruser{,.orig}
      cp -a /etc/skel /home/poweruser
      chown -hR poweruser: /home/poweruser
      restorecon -R -vv /home/poweruser

  - Choose "Power User" with username "poweruser".

  - Check "Make this user administrator".

- [Net-based] Choose desired network interface (e.g., ``eth0`` or
  ``wlan0``).  For wireless connections, setup wireless settings.

- [Net-based] Choose installation type as "Software Development".

- [Net-based] Leave "Customize later" selected and choose "Next".

- Wait while installation image is copied to hard drive.

- Reboot.

- At grub prompt, edit kernel command line options if necessary.

[CentOS] Base Install
=====================

- Boot from DVD image.

- If need custom kernel options, press ``e`` on ``Install CentOS 7`` and append
  them (e.g., ``nox2apic``).

- Select "Install CentOS 7".

- [First time through] Press Ctrl-Alt-F1 for text console; perform custom
  partitioning; reboot.

- Choose language as "English" and "English (United States)".

- Choose SYSTEM | INSTALLATION DESTINATION:

  - Choose all physical disks for installation.

  - Choose "I will configure partitioning".

  - Choose "Done"; brings up MANUAL PARTITIONING screen.

  - Setup RAID, LVM, mount points, etc., as desired; press Done.

- NETWORK & HOST NAME:

  - Wired Ethernet:

    - Choose "ON".

    - Configure:

      - Check "Automatically connect to this network when it is available".

  - Wireless Ethernet: Choose "ON", Configure:

    - Choose "ON":

      - Network name: toyland

    - Configure:

      - Check "Automatically connect to this network when it is available".

  - Host name:

    - Choose FQDN (e.g., ``LAPTOP.DOMAIN.com``).

- Begin installation.

- ROOT PASSWORD:

  - Set root password.

- Defer USER CREATION until after login.

- Reboot into CentOS.

**************************
Users and Home directories
**************************

First-time login
================

- Login to text console (as ``root`` if possible, else ``poweruser``).

- [if using ``poweruser``]: become root::

    sudo -i

- [Ubuntu] Grant root a password to allow logging in as root directly from the
  console::

    passwd

Remote login for setup
======================

For convenience, may remotely login to continue setup.

[Ubuntu]
--------

- Remotely login as poweruser, then become root::

    ssh poweruser@ubuntuhost
    sudo -i

  (Until ssh configuration is done, can't login via ssh as root.)

- At first, must use ``vi`` instead of ``vim`` (until full support is
  installed).

[Fedora]
--------

- Start the ssh service::

    systemctl start sshd
    systemctl enable sshd

- Remotely login as root directly::

    ssh root@fedorahost

[CentOS]
--------

- Already setup for remote login as root.

Firewall setup
==============

Ubuntu
------

- Uncomplicated Firewall (``ufw``) is installed by default, but disabled.

- Setup to "deny" by default, allow ssh, then enable firewall::

    ufw default deny
    ufw allow ssh
    ufw enable

    # Press 'y' to allow firewall to enable.

Hostname/FQDN
=============

- Verify that ``/etc/hosts`` contains lines like the below::

    127.0.0.1       localhost.localdomain localhost
    127.0.1.1       LAPTOP.DOMAIN.com LAPTOP
    1.2.3.251 SERVER.DOMAIN.com SERVER

  The line for ``127.0.1.1`` is needed for ``hostname -f`` to give the proper
  FQDN back.

  **NOTE** The line for SERVER is important for working around new
  systemd-resolved behavior.  Not sure yet how to work around this otherwise.

Simple aliases
==============

- Setup root aliases for installation::

    vi ~/.bashrc

    # Ubuntu aliases
    alias agi='apt-get install -y'
    alias agu='apt-get update'
    alias agdu='apt-get dist-upgrade'
    alias acs='apt-cache search'

    # CentOS/Fedora aliases
    alias yi='yum -y install'
    alias ys='yum search'
    alias ygi='yum -y groupinstall'
    alias yu='yum -y upgrade'

    # Fedora-only aliases
    alias dnfi='dnf -y install'
    alias dnfs='dnf search'
    alias dnfgi='dnf -y groupinstall'
    alias dnfu='dnf -y upgrade'

  Now, source these new definitions::

    . ~/.bashrc

VM Guest Additions
==================

- [Ubuntu] Install prerequisites::

    agi build-essential linux-headers-generic dkms

- [Fedora] [CentOS] Install prerequisites::

    ygi 'Development Tools'
    yi install dkms kernel-devel

- Reboot if needed to ensure running kernel matches latest kernel.

- Install Guest Additions using menu Devices | Insert Guest Additions CD image.

- Insert Guest Additions CD, mount, install::

    mkdir /mnt/tmp
    mount /dev/sr0 /mnt/tmp
    /mnt/tmp/VBoxLinuxAdditions.run

  Generally, errors indicate that the kernel source is not installed, or that
  the source version doesn't match the running kernel.

- Setup shared clipboard:

  - Devices | Shared Clipboard | Bidirectional

- Reboot::

    reboot

wget/curl
=========

- Install::

    agi wget curl

    yi wget curl

Default editor
==============

- [Ubuntu] Set default editor to vim (probably already the default)::

    sudo update-alternatives --config editor

  (In general, note that ``/etc/alternatives/`` are symlinks to the
  alternatives.)

  Choose ``/usr/bin/vim.basic`` if not already selected.

- [CentOS] Already defaults to vim.

echod
=====

- Create simple ``echod`` utility::

    vi /usr/bin/echod

  Do ``:set sw=4``, paste this contents, outdent all::

    #!/usr/bin/python

    import textwrap, sys
    args = sys.argv[1:]
    out = sys.stdout
    while args:
        arg = args.pop(0)
        if arg == "-o":
            out = open(args.pop(0), "w")
        elif arg == "-a":
            out = open(args.pop(0), "a")
        else:
            out.write(textwrap.dedent(arg).strip() + "\n")

- Make script executable::

    chmod +x /usr/bin/echod

sudo
====

- Edit ``/etc/sudoers`` configuration file::

    visudo

  - [Fedora] [CentOS] Adjust ``secure_path`` to include important directories
    like ``/usr/local/sbin`` and ``/usr/local/bin``::

      #Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin
      Defaults    secure_path = /usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

  - [Fedora] [CentOS]:

    - Can stick with default::

        ## Allows people in group wheel to run all commands
        %wheel  ALL=(ALL)       ALL

    - Or can opt for passwordless operation by commenting out the above and
      using this::

        ## Same thing without a password
        %wheel        ALL=(ALL)       NOPASSWD: ALL

Initial Vim
===========

- Install full Vim with plugin dependencies::

    agi vim vim-gtk exuberant-ctags ruby ruby-dev

    yi vim vim-X11 ctags ruby ruby-devel perl-ExtUtils-Embed

IPv6
====

- Generally leave IPv6 enabled now.  Can verify if it's disabled by::

    cat /proc/sys/net/ipv6/conf/all/disable_ipv6

  A ``1`` means it's disabled.

Repository tools
================

[Ubuntu] Additional apt tools
-----------------------------

- Install apt-related tools::

    agi apt-{doc,dpkg-ref,file,show-source,show-versions,utils}

- Setup apt-file::

    sudo apt-file update

  Query using apt-file, e.g.::

    apt-file search maelstrom

[Ubuntu] Package Manager Alternatives
-------------------------------------

- ``wajig`` is a wrapper around a pile of tools::

    agi wajig

- ``aptitude`` is a unified tool to replace a pile of tools (pre-installed)::

    agi aptitude aptitude-doc-en

  - May not have equivalent to ``apt-get source``.

Additional repositories
=======================

[Ubuntu] Partner repositories
-----------------------------

- Uncomment "partner" repositories in ``/etc/apt/sources.list``::

    vim /etc/apt/sources.list

    deb http://archive.canonical.com/ubuntu xenial partner

  Update repositories::

    agu

[Ubuntu] GetDeb repository
--------------------------

**Note** Seems to be down a lot; skipping this for now.

GetDeb contains newer releases for certain packages:
http://www.getdeb.net/welcome/

Installation instructions:
http://www.getdeb.net/updates/Ubuntu/16.04#how_to_install

It's hard to find a direct link to the packages, but this appears to be
the top-level for packages:
http://archive.getdeb.net/ubuntu/rpool/

Setup GetDeb manually:

- Install repository, setup key::

    add-apt-repository "deb http://archive.getdeb.net/ubuntu $(lsb_release -cs)-getdeb apps"
    curl http://archive.getdeb.net/getdeb-archive.key | apt-key add -
    apt-get update

- To remove::

    add-apt-repository -r "deb http://archive.getdeb.net/ubuntu $(lsb_release -cs)-getdeb apps"

[Ubuntu] Source repositories
----------------------------

- Edit ``/etc/apt/sources.list`` and uncomment all desired ``deb-src`` lines,
  e.g.::

    vim /etc/apt/sources.list

    deb-src http://us.archive.ubuntu.com/ubuntu/ xenial-updates main restricted

- Update apt cache::

    agu

[Fedora] rpmfusion repositories
-------------------------------

- Configuration information from:
  http://rpmfusion.org/Configuration/

- Configure repositories for rpmfusion::

    yi --nogpgcheck http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

[Fedora] ATrpms repositories
----------------------------

- Instructions from:
  http://www.mjmwired.net/resources/mjm-fedora-f17.html

- This has just a couple of packages that aren't in rpmfusion, but it can
  conflict with rpmfusion.  Configure this repository, but don't keep it
  enabled.

- Create configuration file for ATrpms repo::

    echod -o /etc/yum.repos.d/atrpms.repo '
      [atrpms]
      name=Fedora Core $releasever - $basearch - ATrpms
      baseurl=http://dl.atrpms.net/f$releasever-$basearch/atrpms/stable
      gpgkey=http://ATrpms.net/RPM-GPG-KEY.atrpms
      enabled=0
      gpgcheck=1
    '

- Import the GPG key::

    rpm --import http://packages.atrpms.net/RPM-GPG-KEY.atrpms

[CentOS] Additional repositories
--------------------------------

Non-http mirrors of repositories:

- CentOS itself:

  - https://www.centos.org/download/mirrors/

  - ftp://ftp.gtlib.gatech.edu/pub/centos/

  - Comment out ``mirrorlist``; edit ``baseurl``.  E.g.::

      #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra

      baseurl=ftp://ftp.gtlib.gatech.edu/pub/centos/$releasever/os/$basearch/

- EPEL:

  - https://mirrors.fedoraproject.org/mirrorlist?repo=epel-7&arch=x86_64

  - Choose: ftp://ftp.utexas.edu/pub/epel/7/

  - Comment out ``metalink``; edit ``baseurl``.  E.g.::

      #metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch

      baseurl=ftp://ftp.utexas.edu/pub/epel/7/$basearch

- Nux Dextop:

  - Available via rsync for mirroring::

      rsync -avH li.nux.ro::li.nux.ro/nux/dextop/ /path/to/destination/

  - No apparent source of ftp-based mirroring.

  - Disabling for now.

Normal repositories:

- List of additional repositories along with recommendations:
  https://wiki.centos.org/AdditionalResources/Repositories

- EPEL (Extra Packages for Enterprise Linux):
  https://fedoraproject.org/wiki/EPEL

  - Setup EPEL::

      rpm -Uvh http://download.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-10.noarch.rpm

  - **Subscribe to EPEL announce list** (see wiki page for details).

- Nux Dextop:

  - Setup Nux Dextop (**Note** Requires EPEL to be installed first)::

      rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
      rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm

- Psychotic Ninja (enabled only temporarily):

  - http://wiki.psychotic.ninja/index.php?title=Main_Page
  - http://wiki.psychotic.ninja/index.php?title=Usage

  - Setup repo key::

      rpm --import http://wiki.psychotic.ninja/RPM-GPG-KEY-psychotic

  - Setup repo (**note** even though this is ``i386`` and ``el6``, it's a a
    unified package that works across all releases and architectures, so the
    below is correct)::

      rpm -ivh http://packages.psychotic.ninja/6/base/i386/RPMS/psychotic-release-1.0.0-1.el6.psychotic.noarch.rpm

[Ubuntu] Setup fastest mirror
-----------------------------

- **Skipping this in general**; it can cause issues later when the mirror
  is down.

- The tool ``apt-select`` can determine the fastest mirror:

  - URL: https://github.com/jblakeman/apt-select

  - Install::

      pip install apt-select

  - Generate a temporary ``sources.list`` file in current directory, e.g.::

      apt-select -c -t 3 -m one-week-behind

  - Compare against original ``sources.list`` file::

      meld /etc/apt/sources.list sources.list

  - Use new ``sources.list`` file::

      sudo cp /etc/apt/sources.list{,.$(date '+%Y_%m_%d')}

      sudo cp sources.list /etc/apt/sources.list

- To manually setup different mirrors, edit ``/etc/apt/sources.list``.

  Original contents::

    deb cdrom:[Ubuntu 18.04.1 LTS _Xenial Xerus_ - Releas amd64 (20160719)]/
    xenial main restricted
    deb http://archive.ubunbu.com/ubuntu/ xenial main restricted
    deb http://security.ubunbu.com/ubuntu/ xenial-security main restricted
    deb http://archive.ubunbu.com/ubuntu/ xenial-updates main restricted

  Change the base URL to use something like ``https://mirror.umd.edu/ubuntu``,
  and comment out the cdrom::

    # deb cdrom:[Ubuntu 18.04.1 LTS _Xenial Xerus_ - Releas amd64 (20160719)]/
    deb https://mirror.umd.edu/ubuntu/ xenial main restricted
    deb https://mirror.umd.edu/ubuntu/ xenial-security main restricted
    deb https://mirror.umd.edu/ubuntu/ xenial-updates main restricted

[Fedora] [CentOS] SELinux
=========================

- [optional] Set SELinux to be permissive::

    vim /etc/selinux/config

    SELINUX=permissive

  This prevents SELinux from denying or breaking anything, and gives a chance to
  see what's going wrong and fix it.

  - A reboot is required for this to take effect.

- [Skip; trying to keep SELinux enabled] Disable SELinux::

    vim /etc/selinux/config

    SELINUX=disabled

  - Alternatively, disable SELinux using GUI::

      system-config-selinux

  - A reboot is required for this to take effect.

- Install policy tools::

    yi checkpolicy policycoreutils-devel

Setup additional filesystems
============================

- Install partitioning tools::

    agi gparted gdisk

Setup ZFS
=========

- Install ZFS utilities::

    agi zfsutils-linux

- Wipe out anything on ZFS disks::

    for i in /dev/sd[cdefgh]; do sgdisk --zap-all $i; done

- Allocate the BIOS Boot partition using sector counts for legacy BIOS booting.
  Use ``-a1`` to set the alignment defaults to 1 sector and map sectors 34
  through 2047, taking it to the 1MiB boundary::

    sgdisk /dev/sdc -a1 -n 2:34:2047 -t 2:EF02

- Allocate ESP and /boot volumes.  Starting values of ``0`` means the start of
  the biggest unused area::

    sgdisk /dev/sdc     -n 3:0:+1GiB -t 3:EF00
    sgdisk /dev/sdc     -n 4:0:+1GiB -t 4:8300

- Allocate padding at the end of the disk.  Use approximate value of ``100MB``
  instead of exact ``100MiB`` to allow ``sgdisk`` to round to good bounaries::

    sgdisk /dev/sdc     -n 9:-100MB:0 -t 9:BF07

- Allocate the remainder of the disk space for zfs::

    sgdisk /dev/sdc     -n 1:0:0 -t 1:BF00

- Display final partitioning::

    sgdisk /dev/sdc -p

    Disk /dev/sdc: 7814037168 sectors, 3.6 TiB
    Logical sector size: 512 bytes
    Disk identifier (GUID): E5C31FA1-2D2E-4DAF-B29A-A4C4E2585CAA
    Partition table holds up to 128 entries
    First usable sector is 34, last usable sector is 7814037134
    Partitions will be aligned on 8-sector boundaries
    Total free space is 0 sectors (0 bytes)

    Number  Start (sector)    End (sector)  Size       Code  Name
       1         4196352      7813832327   3.6 TiB     BF00
       2              34            2047   1007.0 KiB  EF02
       3            2048         2099199   1024.0 MiB  EF00
       4         2099200         4196351   1024.0 MiB  8300
       9      7813832328      7814037134   100.0 MiB   BF07

- Clone partition information onto other disks::

    for i in d e f g h; do
      sgdisk -R /dev/sd$i /dev/sdc
      sgdisk -G /dev/sd$i
    done

- [optional] Insert any playing around here, creating and destroying pools,
  etc., making sure to destroy all pools before continuing::

    # If created any pools for testing, destroy them now:
    zpool destroy SERVER

- Create a root pool named after the machine.  Use short names for brevity, then
  later use ``/dev/disk/by-id`` names as shown below::

    zpool create \
      -o ashift=12 \
      -O relatime=on \
      -O canmount=off \
      -O compression=lz4 \
      -m none \
      SERVER \
      mirror sdc1 sdd1 sde1 \
      mirror sdf1 sdg1 sdh1

- View pool status::

    zpool status SERVER

- If pool was created with short disk names like ``/dev/sda1``, then you can
  export and re-import as below to use the more permanent ``by-id`` names::

    zpool export SERVER
    zpool import -d /dev/disk/by-id/ SERVER

  *Note* You may alternatively supply ``-R /mnt`` if you want to use the
  ``altroot`` value of ``/mnt``; otherwise, the volumes will mount starting at
  root::

    zpool export SERVER
    zpool import -R /mnt -d /dev/disk/by-id/ SERVER

- Create a filesystem:

  - At filesystem creation, must choose a mount point.

  - Create new filesystem below the overall pool name (``SERVER`` in this case).

- [SERVER] Create filesystems:

  - **NOTE** Before mounting ZFS-based /home, move any home directories
    out of the way, then move them back after creating the filesystems::

      mv /home/poweruser /
      zfs create -o mountpoint=/m SERVER/m
      zfs create -o mountpoint=/home SERVER/home
      mv /poweruser /home

- Automatic zfs scrub:

  - Save ``/m/sys/bin/zfs_health.sh`` from
    https://calomel.org/zfs_health_check_script.html.

  - Edit script to use Ubuntu date instead of FreeBSD::

      ### Ubuntu with GNU supported date format
      scrubRawDate=$(/sbin/zpool status $volume | grep scrub | awk '{print $11" "$12" " $13" " $14" "$15}')
      scrubDate=$(date -d "$scrubRawDate" +%s)

      ### FreeBSD with *nix supported date format
      # scrubRawDate=$(/sbin/zpool status $volume | grep scrub | awk '{print $15 $12 $13}')
      # scrubDate=$(date -j -f '%Y%b%e-%H%M%S' $scrubRawDate'-000000' +%s)

  - Setup cron jobs to kick of scrub and check health::

      vim /etc/crontab

      # zfs scrub:
        30  23        *           *  fri  root  zpool scrub SERVER

      # zfs health check:
        30  05        *           *   *   root  /m/sys/bin/zfs_health.sh

Setup NFS client support
========================

- Install NFS utilities::

    agi nfs-common

    yi nfs-utils

Setup autofs
============

- Install::

    agi autofs

    yi autofs

- [ubuntu] Enable /net::

    mkdir /net

  Uncomment this line in ``/etc/auto.master``::

    vim /etc/auto.master

    /net -hosts

- Enable autofs service (restarting because Ubuntu starts the daemon running
  before ``/etc/auto.master`` can be edited)::

    systemctl enable autofs
    systemctl restart autofs

Setup automount for ``/m/``
===========================

- [non-SERVER] Create symlink for ``/m``::

    ln -s /net/SERVER.DOMAIN.com/m /m

  **NOTE** SERVER.DOMAIN.com might resolve to public IP.

Create users
============

- If there are pre-existing home directories, generally it's best to rename
  them out of the way before creating the new users.  That way, the old
  configuration files are left intact and won't cause problems with newer
  versions of the software.  When ready, the old data files can be moved into
  place.

- Create all of the following users so UIDs from ``/m/`` are displayed
  properly.

  *Note* The ``-m`` (create home directory) and ``-s /bin/bash`` (setup shell)
  switches are needed only on Ubuntu, but they work properly on other distros as
  well::

    useradd -u 2001 -m -s /bin/bash -c 'Michael Henry' mike

- [CentOS] For proper ssh permissions, home directories should not be group
  writable.  Centos (at least) creates home directories as group-writable; to
  fix::

    chmod g-w /home/*

- Set password for login accounts (**one at a time**)::

    passwd mike

- Modify groups to make ``mike`` an ``administrator``:

  - [Ubuntu]  (Note: This entails more groups than ``poweruser`` has)::

      for i in adm cdrom sudo plugdev lxd lpadmin sambashare video; do
          usermod -aG $i mike
      done

    Note: Default groups for poweruser::

      adm cdrom sudo dip plugdev lxd lpadmin sambashare

    Note that ``floppy``, ``tape``, and ``dip`` used to be in this list, but it
    made the list too large for the 16-group NFS limit when other needed groups
    are added later.

  - [Fedora] [CentOS]::

      usermod -aG wheel,dialout mike

- Create data group::

    groupadd -g 1999 data

- Add all users to data group::

    for i in mike ; do usermod -aG data $i; done

- [Fedora] [CentOS] If the home directories are pre-existing from another
  distro, ensure all home directories have correct SELinux contexts::

    restorecon -R /home/*/

- [optional] Remove ``poweruser`` account now that standard accounts are
  available::

    userdel -r poweruser

[Fedora] [CentOS] Home directories not under /home
==================================================

- Setup context of /m to include home_root_t::

    semanage fcontext -a -t home_root_t /m
    restorecon -R /m

- Check permissions with ``-Z`` flag, e.g.::

    ls -Zd /home/USER

  Home directory itself should have this context::

    unconfined_u:object_r:user_home_dir_t:s0

  Files within home directory should have this context::

    unconfined_u:object_r:user_home_t:s0

- Change context to match another file like this::

    chcon --reference RefFile FileToChange

SSH
===

Note: Before locking down the ssh server, configure the ssh keys for
passwordless login.

- Setup ``mike`` ssh:

  - Setup ``~mike/.ssh/id_rsa*`` keys:

    - If can ssh from the outside, then from another box do (as ``mike``)::

        # For generic machine named "thishost":
        ssh-copy-id -i ~/.ssh/id_rsa thishost
        scp ~/.ssh/id_rsa* thishost:.ssh/

        # E.g., for LAPTOP-wired:
        ssh-copy-id -i ~/.ssh/id_rsa LAPTOP-wired
        scp ~/.ssh/id_rsa* LAPTOP-wired:.ssh/

        # E.g., for WORKSTATION:
        ssh-copy-id -i ~/.ssh/id_rsa WORKSTATION
        scp ~/.ssh/id_rsa* WORKSTATION:.ssh/

    - Otherwise, ``ssh mike@localhost`` to pre-create the ``~mike/.ssh``
      directory, then bring over ``~mike/.ssh/id_rsa*`` and setup
      authorized_keys manually::

        ssh mike@localhost
        cd ~/.ssh
        scp 'remotehost:.ssh/id_rsa*' .
        cp id_rsa.pub authorized_keys
        exit

- As root, setup ``root`` ssh.  Note that the ``ssh-copy-id`` trick won't work
  here if the server has already been configured for passwordless login
  via ``PermitRootLogin without-password``::

    ssh localhost   # Then CTRL-C without typing password
    cp ~mike/.ssh/id_rsa* /root/.ssh/
    cp ~mike/.ssh/id_rsa.pub /root/.ssh/authorized_keys

- Configure SSH clients.  Note that host-specific settings should come first,
  since the first-found setting that matches wins::

    vim /etc/ssh/ssh_config

    # Add these lines at the top of the file:
    Host *
            ForwardX11 yes
            ServerAliveInterval 300
            SendEnv COLORFGBG

  Already the default::

    ForwardX11Trusted yes

- Configure SSH server::

    vim /etc/ssh/sshd_config

    # Generally these lines can be put anywhere, as long as they aren't in
    # conflict with the same setings elsewhere in the file.  But by default,
    # the file is generally just a bunch of comments.

    AllowUsers root mike
    AcceptEnv COLORFGBG

    # Allow client to forward any ports from this server.
    GatewayPorts yes

    # Already the default; these are synonyms:
    # PermitRootLogin without-password
    # PermitRootLogin prohibit-password

- [Ubuntu] Configure ssh firewall::

    # ssh already allowed by default on port 22, so this is redundant:
    ufw allow ssh

    # [SERVER]
    ufw allow 12345

- [Fedora] [CentOS] Configure firewall::

    # On CentOS, ``ssh`` is already added.
    firewall-cmd --add-service ssh
    firewall-cmd --permanent --add-service ssh

    # [SERVER]
    firewall-cmd --add-port 12345/tcp
    firewall-cmd --permanent --add-port 12345/tcp

  - [SERVER] Tell SELinux to permit the additional port::

      semanage port -a -t ssh_port_t -p tcp 12345

- Enable and (re)start the ssh service::

    # Ubuntu:
    systemctl enable ssh.service
    systemctl restart ssh.service

    # Non-Ubuntu:
    systemctl enable sshd.service
    systemctl restart sshd.service

[optional] Reusing connections
------------------------------

- Can reuse connections to the same host.  See:
  http://protempore.net/~calvins/howto/ssh-connection-sharing/

- Could add per-user SSH configuration::

    vim ~/.ssh/config

    Host SERVER SERVER.DOMAIN.com
        ControlPath ~/.ssh/master-%l-%r@%h:%p
        ControlMaster auto

  Downsides include the fact that the first connection will become the
  master, and it cannot exit until all slaves have exited.  To use this
  effectively, it should probably not be the default.  Instead, a master
  should be explicitly started and slaves explicitly joined to that
  master.

- [Fedora] May need to restore SELinux context on ``/root/.ssh``::

    restorecon -R -vv /root/.ssh

Git
===

- Install::

    agi git-core git-doc git-svn git-gui qgit

    yi git git-gui qgit

Mr/MyRepos
==========

- Now known as "myrepos" tool: https://myrepos.branchable.com/

- Install::

    agi myrepos

    yi myrepos

- Will acquire ``~/.mrconfig`` from clone of ``home`` (this comes later).

Bulk installation
=================

Optionally, can begin a bulk installation (e.g., KDE and/or system updates) that
can be working in the background as the next steps are taken.

Clone standard repositories
===========================

- Clone home (may want to do this for root as well as mike)::

    cd ~
    git clone ssh://SERVER/srv/git/home.git

  (Pause for password entry.)::

    tar -C ~/home -c . | tar -C ~ -x
    rm -rf ~/home

- Restart shell to acquire Bash aliases, PATH (including ``~/bin``), etc.

- Now that ``~/.gitconfig`` is in place, the ``SERVER`` alias is present, so::

    perl -pi -e 's@ssh://SERVER/srv/git/@SERVER:@' ~/.git/config

- Update (backslash prevents alias expansion for ``mr``, since don't have
  ``pytee`` installed yet)::

    \mr up

  This will clone ``~/.vim`` among other things.

- Perform additional Vim setup::

    cd ~/.vim
    ./setup.py

  If still using cpsm::

    cd bundle/cpsm
    ./install.sh

- Pull down projects::

    mkdir ~/projects
    cd ~/projects
    git-clone-SERVER-projects

- For root's vim::

    # Can launch before mike-user clone above finishes.
    cd /root
    git clone ~mike/.vim
    cd .vim
    ./setup.py

- For root's home directory::

    cd /root
    git clone ~mike/ home
    tar -C ~/home -c . | tar -C ~ -x
    rm -rf ~/home

Bash
====

- **Retain original .bash_history**:

  - For ``root`` and ``mike``, prepend ``~/.bash_history`` from previous
    installation to current history file using text editor.

- Use ``~/.bashrc`` settings from source control in general.

Update grub
===========

- Update grub configuration to display boot-time messages and append any
  hardware-specific options.

- [Ubuntu]: Change grub configuration lines::

    vim /etc/default/grub

    #GRUB_TIMEOUT_STYLE=hidden

  Then update grub to regenerate ``/boot/grub/grub.cfg`` and the initramfs::

    update-grub

- [Fedora] [CentOS]: Update grub kernel command line options::

    vim /etc/default/grub

  Remove ``rhgb`` and ``quiet`` and append any hardware-specific options.

  Then update the grub configuration file and regenerate the initramfs::

    # [BIOS systems]
    grub2-mkconfig -o /boot/grub2/grub.cfg
    # [EFI systems]
    grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg

    dracut --force

  **NOTE** Do not point at ``/etc/grub2.cfg``, as that symlink will be
   clobbered because of backup behavior in ``grub2-mkconfig``.

***
X11
***

KDE Installation
================

- Optional for systems that will run X11.

- [Ubuntu] Install kubuntu::

    agi kubuntu-desktop

- [Fedora] Install KDE::

    ygi 'KDE Software Compilation'

- [CentOS] Install KDE::

    ygi 'KDE Plasma Workspaces'

**NOTE** Do not login to KDE yet; wait until home directory has been setup.

- Verify will boot graphically::

    systemctl get-default

  If ``multi-user.target``, switch to graphical::

    systemctl set-default graphical.target

- Launch display manager now::

    systemctl isolate graphical.target

nVidia proprietary driver
=========================

TODO: Try out nVidia driver.

[Ubuntu]
--------

- System | Hardware Drivers | choose nVidia driver (if appropriate).

  - **Driver Problem**

    - Had to install latest nVidia driver to fix freezing problem when
      windows were maximized.  270.41.06-0ubuntu1 had the problem. Followed
      directions from nVidia's site.

    - Needed to disable nouveau driver before installation would work.

    - Involved several files:

      - Change grub defaults::

          vim /etc/default/grub

        Change the following::

          GRUB_HIDDEN_TIMEOUT=3
          GRUB_HIDDEN_TIMEOUT_QUIET=false
          GRUB_CMDLINE_LINUX_DEFAULT=""
          GRUB_TERMINAL=console

      - ``/etc/grub.d/*`` (these control construction of config file;
        shouldn't have to edit them).

      - ``/boot/grub/grub.cfg`` (generated by ``update-grub``).

      - ``/etc/modprobe.d/*.conf`` (these files cause blacklisting of various
        modules).  Create ``/etc/modprobe.d/blacklist-nouveau.conf``::

          blacklist vga16fb
          blacklist nouveau
          blacklist rivafb
          blacklist nvidiafb
          blacklist rivatv

        After updating the blacklist, update the initramfs::

          update-initramfs -u

[Fedora]
--------

Following these instructions:
http://www.if-not-true-then-false.com/2012/fedora-17-nvidia-guide/

- **Install prerequisites**; otherwise, Fedora 20 can hang at boot::

    yi kernel-devel acpid

- Install drivers from rpmfusion::

    yi akmod-nvidia xorg-x11-drv-nvidia-libs

  This includes a blacklist file for nouveau
  (``/etc/modprobe.d/blacklist-nouveau.conf``).

- Setup VDPAU/VAAPI support for video acceleration::

    yi vdpauinfo libva-vdpau-driver libva-utils

For some reason, the below recommended solution doesn't work to control the
brightness:

- Enable the laptop brightness control:

  - Configure X11::

      vim /etc/X11/xorg.conf

    Insert this line inside the ``Section "Device"`` section::

      Option "RegistryDwords" "EnableBrightnessControl=1"

disper utility for nVidia proprietary driver
--------------------------------------------

- Install::

    agi disper

    yi disper

- List connected devices::

    disper -l

- Extend displays (multi-head)::

    disper -e
    disper -d auto -e

Multi-head setup via xrandr
===========================

- Useful with open-source drivers and standard XRandR model.

- Use ``xrandr`` to configure side-by-side displays.

  - Outputs:

    - LVDS1 - Low Voltage Differential Signaling (Internal LCD).
    - DVI1 - External DVI port.
    - VGA1 - External VGA port.

  - Show current configuration::

      xrandr -q

  - Configure ``LVDS1`` (laptop screen) and ``DVI1`` (external DVI #1)
    to be clones::

      xrandr --output LVDS1 --auto --output --DVI1 --auto --same-as LVDS1

  - Turn off ``LVDS1`` output::

      xrandr --output LVDS1 --off

  - Configure ``DVI1`` (external DVI #1) and ``VGA1`` (external VGA #1)
    to be extended desktop::

      xrandr --output DVI1 --auto --output VGA1 --auto --right-of DVI1

************************
System update and reboot
************************

- [Ubuntu] From a root prompt, install updates::

    apt-get -y dist-upgrade

- [Fedora] [CentOS] From a root prompt, install updates::

    yum -y upgrade

- **Note**: a restart may be needed after updates::

    reboot

*********************
Mission-critical apps
*********************

Login as ``mike``
=================

- May now login as ``mike`` user (into KDE if installed).

xdg setup of directories
========================

- freedesktop.org is site for X-related standardization.

- xdg-user-dirs is package for using standardized directories for
  users (e.g., Downloads, Documents, ...):
  http://freedesktop.org/wiki/Software/xdg-user-dirs

- Defaults on Fedora and Ubuntu (taken from ``~/.config/user-dirs.dirs``)::

    XDG_DESKTOP_DIR="$HOME/Desktop"
    XDG_DOWNLOAD_DIR="$HOME/Downloads"
    XDG_TEMPLATES_DIR="$HOME/Templates"
    XDG_PUBLICSHARE_DIR="$HOME/Public"
    XDG_DOCUMENTS_DIR="$HOME/Documents"
    XDG_MUSIC_DIR="$HOME/Music"
    XDG_PICTURES_DIR="$HOME/Pictures"
    XDG_VIDEOS_DIR="$HOME/Videos"

- Can query variables, e.g.::

    xdg-user-dir DOCUMENTS

  Produces (using default configuration)::

    /home/mike/Documents

- Change default locations using the configuration tool (this will update
  ``~/.config/user-dirs.dirs``)::

    xdg-user-dirs-update --set DOCUMENTS  ~/x
    xdg-user-dirs-update --set DOWNLOAD   ~/download
    xdg-user-dirs-update --set MUSIC      ~/music
    xdg-user-dirs-update --set PICTURES   ~/pictures
    xdg-user-dirs-update --set VIDEOS     ~/videos

  Note: Leave the following directories at their original names with leading
  capital letters, since they are potentially still useful but will stay out of
  the way of tab-completion for more common directories:

  - ``Desktop``

  - ``Public``

  - ``Templates``

- Remove default directories, setup symlinks::

    cd ~
    rmdir Documents Downloads Music Pictures Videos
    rm -f examples.desktop

    ln -s /m/shared/pictures ~/pictures
    ln -s /m/shared/videos ~/videos

    # [non-SERVER]
    ln -s /home/m/mike/x ~/x
    ln -s /home/m/shared/download ~/download
    ln -s /home/m/shared/music/Music ~/music

    # [SERVER]
    ln -s /m/mike/x ~/x
    ln -s /m/shared/download ~/download
    ln -s /m/shared/music/Music ~/music

KDE Wallet/kwallet
==================

http://homepages.inf.ed.ac.uk/da/id/gpg-howto.shtml

- https://wiki.archlinux.org/index.php/KDE_Wallet

- [Ubuntu] kdewallet is already setup.

- [CentOS] Wallet isn't setup properly out-of-the-box:

  - Start ``kdewalletmanager``; runs in the task bar.

  - Open the wallet manager.

  - Create the default wallet; must be named ``kdewallet``.

  - Choose blowfish encryption instead of GPG (too hard to figure out GPG).

Keychain
========

Gentoo keychain is a small script that sets up ssh-agent at first login.
The agent thereafter will maintain the keys in memory for passwordless usage of
ssh.

- Install::

    agi keychain

    # Fedora
    yi keychain

    # CentOS
    yi --enablerepo=psychotic keychain

- Append the following configuration lines::

    vim ~/.bash_profile

    # Require interactive and stdin/stdout are ttys.
    if [ -n "$PS1" ] && tty -s && tty -s 0<&1; then
        keychain --nogui --quiet ~/.ssh/id_rsa && . ~/.keychain/$HOSTNAME-sh
    fi

- For ssh-add during KDE startup:

  - First, setup kwallet with a key (run ``kwalletmanager``).

  - For plasma5::

      echod -o ~/.config/autostart-scripts/ssh-add.sh '
        #!/bin/sh
        ssh-add < /dev/null
      '
      chmod +x ~/.config/autostart-scripts/ssh-add.sh

  - For plasma4::

      echod -o ~/.kde/Autostart/ssh-add.sh '
        #!/bin/sh
        ssh-add < /dev/null
      '
      chmod +x ~/.kde/Autostart/ssh-add.sh

    Might also need ``~/.kde4`` on some systems.

- Make sure kwallet is setup, then logout/login.

cron
====

- Adjust root's crontab as follows::

    vim /etc/crontab

    SHELL=/bin/bash

  Also change PATH if on SERVER::

    PATH=/m/sys/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

  Ensure mail goes to root::

    MAILTO=root

anacron
=======

- anacron controls the launch of daily and weekly jobs in /etc/cron.daily and
  /etc/cron.weekly.

- Adjust time of daily launch of anacron::

    vim /etc/cron.d/anacron

    30 2    * * *   root    test -x /etc/init.d/anacron && ...

Unison
======

- Note: If the replica format changes, the replicas will have to be
  regenerated.  The replica format is currently 22, which works
  with Unison 2.27 through 2.40 (at least).

- **WORSE**, there are version incompatibilities in the OCaml libraries.
  Fedora 20 (SERVER) and Ubuntu 15.10 (WORKSTATION) use ocaml v4.01, so they are
  compatible.  But Fedora 22 (LAPTOP) uses ocaml v4.02, which uses a different
  serialization format:

  - https://lists.fedoraproject.org/pipermail/devel/2015-May/210443.html
  - https://bugzilla.redhat.com/show_bug.cgi?id=1219362

- The simplest fix is to build multiple versions of unison on the server and use
  ``-addversionno`` on the client side.  See later instructions for building
  OCaml and unison appropriately.

- Install pre-packaged version::

    # Ubuntu 18.04 uses 2.48.4 by default.
    agi unison unison-gtk

- Alternatively, build unison from source:

  - Install proper version of ocaml: https://ocaml.org/docs/install.html

  - Use same ocaml version across all systems.

  - Currently using 4.02.3: https://ocaml.org/releases/4.02.html

  - Unpack, configure, build ocaml (following directions in INSTALL file)::

      tar -xf ocaml-4.02.3.tar.gz
      cd ocaml-4.02.3
      ./configure
      make world.opt

  - Install ocaml::

      sudo make install

  - Get source for unison 2.48.4:

    - http://www.seas.upenn.edu/~bcpierce/unison/download/releases/unison-2.48.4/

    - http://www.seas.upenn.edu/~bcpierce/unison/download/releases/unison-2.48.4/unison-2.48.4-manual.html

    - Unpack and build::

        mkdir unison-2.48.12
        cd unison-2.48.12
        tar -xf unison-2.48.12.tar.gz
        cd src
        make UISTYLE=text

    - Install under version-specific name (``unison-major.minor``)::

        sudo cp unison /usr/local/bin/unison-2.48
        sudo chown -h root:root /usr/local/bin/unison-2.48
        sudo chmod 775 /usr/local/bin/unison-2.48

Configure unison
----------------

- [non-SERVER] As root, create ``/home/m`` directory::

    sudo mkdir /home/m
    sudo chown -h mike:data /home/m
    sudo chmod 775 /home/m

After distro upgrade
^^^^^^^^^^^^^^^^^^^^

- After distro upgrade, can restore unison files from original directories,
  e.g.::

    rsync -ax /path/to/old/home_m/ /home/m/
    rm -rf /root/.unison
    ln -s /home/m/sys/unison/WORKSTATION-root ~/.unison

From scratch
------------

- Create location for configuration::

    mkdir -p /home/m/sys/unison/LAPTOP-root

- Symlink ``/root/.unison`` to ``/home/m/sys/unison/LAPTOP-root``::

    rm -rf /root/.unison
    ln -s /home/m/sys/unison/LAPTOP-root /root/.unison

- Steal and modify ``default.prf`` from ``/m/sys/unison/xxx``,
  placing into ``/root/.unison/default.prf``, e.g.::

    cp /m/sys/unison/LAPTOP-root/default.prf /root/.unison/default.prf
    vim /root/.unison/default.prf

- Have to pre-create sub-paths from ``default.prf`` above.

  **Verify the paths before doing this**.  Check for ``path = xxx`` in
  ``default.prf`` and pre-create these paths::

    cd /home/m
    mkdir -p mike shared/{download,music} sys/{bin,unison}

- **Examine and remove any old Unison archive files** for previous
  synchronization attempts (**on both sides**, if using saved ``/home/m``).
  Archive files start with ``ar``, and they have an associated fingerprint file
  starting with ``fp``.  Delete both after finding out which file matches::

    ssh root@SERVER
    cd /root/.unison
    less ar8e1f839be1ad42103cbd6c4f18e0d12d
    rm ar8e1f839be1ad42103cbd6c4f18e0d12d
    rm fp8e1f839be1ad42103cbd6c4f18e0d12d

Running unison
--------------

- Synchronize with server (as root)::

    unison -addversionno -ui text

  **Note** The above ``-addversionno`` switch ensures that the proper version
  of unison is run on the server side.  It may be left off when the version
  is known to be the same.

  The ``~/bin/unison.sh`` script forces ``-addversionno`` to ensure the right
  version is always used.

- Setup nightly unison sync:

  - Add the following to ``/etc/crontab``::

      # min hour dom  mon dow  user  what
      # Main machine (WORKSTATION):
        30  2     *    *   *   root  /root/bin/unison.sh -batch -terse
      # Laptop (LAPTOP):
        30  4     *    *   *   root  /root/bin/unison.sh -batch -terse

Build custom Vim
================

- Install vim build dependencies::

    # Note: must have setup source repositories first.
    apt-get -y build-dep vim-gnome
    agi python python3 python-dev python3-dev

    ygi 'Development Tools'
    yi ncurses-devel

    yum-builddep -y vim-X11
    dnf builddep -y vim-X11

- Follow remaining instructions in ``~/.vim/doc/notes.txt``, using
  invocation::

    mkdir -p ~/build
    cd ~/build
    ~/.vim/buildtool /m/shared/download/programming/vim/8.0

- Install new vim::

    sudo tar -C / -zxf vim-8.0/vim-8.0.1148.x86_64.tar.gz

Neovim
======

- References:

  - https://github.com/neovim/neovim/wiki/Installing-Neovim

- Add PPA::

    add-apt-repository ppa:neovim-ppa/stable
    agu

- Install::

    agi neovim

[Ubuntu] Extras
===============

- Install extras::

    agi kubuntu-restricted-extras

  Provides codecs, flash player, Microsoft fonts, ...

Firefox setup
=============

TODO Verify instructions for firefox.

- Install::

    agi firefox

- Add DOMAIN-ca.crt certificate authority:

  - Browse to:
    http://SERVER.DOMAIN.com/certs/

  - Click on ``DOMAIN-ca.crt``.

  - Check to trust for all purposes:

    - To identify websites.
    - To identify email users.
    - To identify software developers.

  - Verify can access:
    https://SERVER.DOMAIN.com/

- To have Firefox make itself the default browser, exit Firefox and run::

    firefox -silent -setDefaultBrowser

  See http://kb.mozillazine.org/Default_browser for details.

- Create a new profile::

    firefox -ProfileManager

  - Create a new profile "mike"; choose "mike" as folder name.

  - Start Firefox.

- Edit | Preferences

  - General | Startup |

    - Check "Restore previous session".

    - Check "Always check if Firefox is your default browser".

  - General | Files and Applications | Downloads |
    "Always ask you where to save files"

  - General | Digital Rights Management (DRM) Content |
    Check "Play DRM-controlled content"

  - Home | New Windows and Tabs:

    - Homepage and new windows: Custom URLs, https://encrypted.google.com/

    - New tabs: Blank Page

  - Search | Search Bar | "Add search bar in toolbar"

  - Search | Default Search Engine:

    - Choose default search engine: Google

    - Uncheck "Show search suggestions in address bar results"

  - Search | One-Click Search Engines, remove junk

  - Privacy & Security | Forms & Passwords | Use a master password (then Set a
    Master Password).

  - Privacy & Security | Cookie and Site Data |

     Accept third-party cookies and site data: Never

  - Privacy & Security | Tracking Protection:

    - Use Tracking Protection to block known trackers: Always

    - Send websites a "Do Not Track" signal: Always

  - Privacy & Security | "Firefox Data Collection and Use":

    - Uncheck "Allow Firefox to send technical and interaction data to
      Mozilla".

    - Uncheck "Allow Firefox to send backlogged crash reports on your behalf.

- Visit URL about:config

  - Turn off the full-screen nofification by setting
    ``full-screen-api.warning.timeout`` to ``0``.

  - Change ``browser.tabs.closeWindowWithLastTab`` to ``false``.

  - [already default] Change ``accessibility.typeaheadfind.casesensitive`` to
    ``0``.

  - Change ``geo.enabled`` to ``false`` to disable "share my location".

  - Search for ``webno``, disable both of these to avoid push notifications::

      dom.webnotifications.enabled = false
      dom.webnotifications.serviceworker.enabled = false

- Close Firefox.

- [Ubuntu] Setup ``x-www-browser`` alternative to be Firefox::

    sudo update-alternatives --config x-www-browser

- Migration from old installation:

  - Migration notes:
    ``http://kb.mozillazine.org/Transferring_data_to_a_new_profile_-_Firefox``

  - Copy over ``~/.mozilla/firefox/default/places.sqlite*``
    for bookmarks.

  - Copy over ``~/.mozilla/firefox/default/{key3.db,key4.db,logins.json}``
    for saved passwords.

Bookmark synchronization
------------------------

- Use ``~/bin/bookmarks`` to push or pull bookmarks.

- Setup cron jobs for push/pull via ``crontab -e``.

- On main machine, add this cron job::

    #min hr   dom mon  dow     what

     00  04    *   *    *      ~/bin/bookmarks push

- On slave machines, add this cron job::

    #min hr   dom mon  dow     what

     03  04    *   *    *      ~/bin/bookmarks pull

Firefox plugins
===============

Firefox developer tools
-----------------------

- Already built-in.

- Invoke via Tools | Web Developer | ....

vimium-ff
---------

- Install URL:
  https://addons.mozilla.org/en-US/firefox/addon/vimium-ff/

- Preferences:

  - Excluded URLs and keys:

    - Add the following URLs::

        https://feedly.com/*

  - Custom key mappings::

      map b scrollFullPageUp

  - Advanced Options:

    - Check "Don't let pages steal the focus on load".

textern
-------

**Need to install two parts.**

- Install URL:
  https://addons.mozilla.org/en-US/firefox/addon/textern/

- References:
  https://github.com/jlebon/textern

- Bring down native app::

    cd ~/build
    git clone --recurse-submodules https://github.com/jlebon/textern
    cd textern

- Install native app::

    make native-install USER=1

- In Firefox preferences for textern:

  - Set "External Editor" to ``["gvim", "--nofork"]``.

  - Set Shortcut to ``Ctrl+Shift+E``.

- Press Ctrl-Shift-E to bring up editor.

*********************
Desktop Customization
*********************

Keyboard shortcuts with ``activate`` script
===========================================

- Install ``wmctrl`` so ``activate`` command works::

    agi wmctrl

    yi wmctrl

xdotool
=======

- Install::

    agi xdotool

- Use::

    # Simulate pressing of middle button:
    xdotool click 2

Setup KDE System Settings Configuration
=======================================

**NOTE** Don't try this via remote X11; some portions attach to the X server
side, which can affect settings on the client machine.

Invoke System Settings via K | Applications | System | System Settings menu
or via ``systemsettings``.


For Plasma5:

- K | Applications | Settings | System Settings

  - Appearance section

    - Fonts

      - [Optional] Increase all fonts by two points.

      - [Optional] "Adjust All Fonts" to "DejaVu Sans" family (except for
        Fixed width).

      - [Optional] Force fonts DPI: set to 96 DPI if mis-detected in
        ``journalctl | grep DPI``.  Have seen mis-detection as 129 DPI, 147 DPI,
        and 145 DPI.  Fixes problem with too-large fonts.

  - Workspace | Desktop Behavior

    - Desktop Effects

      - Enable Wobbly Windows, Set Wobbliness to halfway mark.

    - Screen Edges

      - Set "Active Screen Edge Actions" to "No Action" for all corners
        (by default, only top-left corner is configured; defaults to
        "Present Windows - All Desktops").

      - Uncheck "Maximize windows by dragging them to the top of the screen".

      - Uncheck "Tile windows by dragging them to the side of the screen".

    - Screen Locking

      - Lock screen automatically after: 30 minutes

      - Require password after locking: 60 seconds

    - Virtual Desktops

      - Number of desktops: Set to 6 (leaving Ctrl-F7 to Ctrl-F12 free).

      - Number of rows: 2

  - Workspace | Window Management | Window Behavior

    - Focus | Slide "Policy" to "Focus Follows Mouse".

    - Focus | Delay focus by: 0 ms

    - Focus | Uncheck "Click raises active windows".

    - Window Actions | Inactive Inner Window | Left button "Activate".

    - Window Actions | Inactive Inner Window | Middle button "Activate".

    - Window Actions | Inactive Inner Window | Right button "Activate".

    - Moving | Windows | Check Display window geometry when moving or
      resizing.

  - Workspace | Shortcuts:

    - Global Keyboard Shortcuts

      - Set KDE Component to System Settings:

        - Lower Window to Alt+2.

        - Maximize Window Horizontally to Alt+4.

        - Maximize Window Vertically to Alt+3.

        - Pack Grow Window Horizontally to Meta+Shift+Right.

        - Pack Grow Window Vertically to Meta+Shift+Up.

        - Pack Shrink Window Horizontally to Meta+Shift+Left.

        - Pack Shrink Window Vertically to Meta+Shift+Down.

        - Pack Window Down to Meta+Ctrl+Down.

        - Pack Window to the Left to Meta+Ctrl+Left.

        - Pack Window to the Right to Meta+Ctrl+Right.

        - Pack Window Up to Meta+Ctrl+Up.

        - Raise Window to Alt+1.

        - Window Operations Menu to Alt+Space.
          (Alt+Space defaults to krunner "Run Command", redundant with Alt+F2.)

    - Custom Shortcuts

      - Create new group named ``Local`` using Edit | New Group.

      - Create new items in ``Local`` group using Edit | New | Global
        Shortcut | Command/URL:

        **Note**: ``~/bin/konsolex`` appears to be obsolete now; it doesn't
        work with ``konsolex``, and does work with regular ``konsole``.

        - "amarok"

          - Trigger: Meta+Ctrl+K
          - Action: activate amarok amarok

        - "copysel"

          - Trigger: Meta+Ctrl+C
          - Action: copysel

        - "Chrome":

          - Action: activate chrome chrome
          - Trigger: Ctrl+Alt+C

        - "Firefox":

          - Trigger: Ctrl+Alt+F
          - Action: activate firefox navigator.firefox

        - "Gvim0":

          - Trigger: Ctrl+Alt+0
          - Action: activate "gvim --servername GVIM0 --name GVIM0" "GVIM0.Gvim"

        - "Gvim)":

          - Trigger: Ctrl+Alt+)
          - Action: activate "gvim --servername GVIM) --name GVIM)" "GVIM).Gvim"

        - "Konsole":

          - Trigger: Ctrl+Alt+4
          - Action: konsole

        - "Konsole1":

          - Trigger: Ctrl+Alt+1
          - Action: activate "konsole --name konsole-1" "konsole-1.Konsole"

        - "Konsole2":

          - Trigger: Ctrl+Alt+2
          - Action: activate "konsole --name konsole-2" "konsole-2.Konsole"

        - "Konsole3":

          - Trigger: Ctrl+Alt+3
          - Action: activate "konsole --name konsole-3" "konsole-3.Konsole"

        - "Konsole8":

          - Trigger: Ctrl+Alt+8
          - Action: activate "konsole --name konsole-8" "konsole-8.Konsole"

        - "Konsole9":

          - Trigger: Ctrl+Alt+9
          - Action: activate "konsole --name konsole-9" "konsole-9.Konsole"

        - "Speedcrunch":

          - Trigger: Ctrl+Alt+=
          - Action: activate speedcrunch speedcrunch

        - "Thunderbird":

          - Trigger: Ctrl+Alt+T
          - Action: activate thunderbird Mail.Thunderbird

  - Personalization | Applications

    - Default Applications

      - Email client to ``/usr/bin/thunderbird``.

      - Web Browser to ``/usr/bin/firefox``.

    - Launch Feedback

      - Set "Busy Cursor" Startup indication timeout to 1 second.

      - Set "Taskbar Notification" Startup indication timeout to 1 second.

  - Hardware

    - Input Devices | Keyboard

      - Delay: 250 ms

      - Rate: 32 repeats/sec

    - Display and Monitor

      - Compositor:

        - **Hopefully obsolete in Ubuntu 18.04**

        - Rendering Backend: XRender

          **Note** The backend used to be OpenGL 2.0, but after some update in
          Ubuntu 16.04, that backend no longer works.  The transparency feature
          got broken (see URL below).  The symptoms are the lack of hover text
          support in Firefox, the fact that buttons on Firefox for uBlock,
          uMatrix, etc., don't seem to show any drop-down menus, etc.  Changing
          to XRender on-the-fly seems to work (no logout required).
          https://askubuntu.com/questions/1053428/last-update-broke-transparencies

    - Power Management | Energy Saving:

      - AC Power:

        - Uncheck "Dim screen".

        - Check "Screen Energy Saving"; switch off after 30 minutes.

        - Button events handling:

          - When laptop lid closed: Lock screen

      - On Battery:

        - [default] Check "Dim screen"; after 2 minutes.

        - [default] Check "Screen Energy Saving"; switch off after 5 minutes.

        - Button events handling:

          - [default] When laptop lid closed: Suspend

      - On Low Battery:

        - [default] Check "Dim screen"; after 1 minute.

        - [default] Check "Screen Energy Saving"; switch off after 2 minutes.

        - Button events handling:

          - [default] When laptop lid closed: Suspend


For Plasma4:

- K | Applications | Settings | System Settings:

  - Application Appearance section:

    - Font:

      - [Optional] Increase all fonts by two points.

      - [Optional] "Adjust All Fonts" to "DejaVu Sans" family (except for
        Fixed width).

      - Force fonts DPI: set to 96 DPI if mis-detected in ``journalctl | grep
        DPI``.  Have seen mis-detection as 129 DPI, 147 DPI, and 145 DPI.
        Fixes problem with too-large fonts.

  - Manage Notifications:

    - Launch Feedback:

      - Set "Busy Cursor" Startup indication timeout to 1 second.

      - Set "Taskbar Notification" Startup indication timeout to 1 second.

  - Shortcuts and Gestures:

    - Custom Shortcuts:

      - Create new group named ``Local`` using Edit | New Group.

      - Create new items in ``Local`` group using Edit | New | Global
        Shortcut | Command/URL:

        - "amarok"

          - Trigger: Meta+Ctrl+K
          - Action: activate amarok amarok

        - "copysel"

          - Trigger: Meta+Ctrl+C
          - Action: copysel

        - "Chrome":

          - Action: activate chrome chrome
          - Trigger: Ctrl+Alt+C

        - "Firefox":

          - Trigger: Ctrl+Alt+F
          - Action: activate firefox navigator.firefox

        - "Gvim0":

          - Trigger: Ctrl+Alt+0
          - Action: activate "gvim --servername GVIM0 --name GVIM0" "GVIM0.Gvim"

        - "Gvim)":

          - Trigger: Ctrl+Alt+)
          - Action: activate "gvim --servername GVIM) --name GVIM)" "GVIM).Gvim"

        - "Konsole":

          - Trigger: Ctrl+Alt+4
          - Action: konsole

        - "Konsole1":

          - Trigger: Ctrl+Alt+1
          - Action: activate "konsolex --name konsole-1" "konsole-1.Konsole"

        - "Konsole2":

          - Trigger: Ctrl+Alt+2
          - Action: activate "konsolex --name konsole-2" "konsole-2.Konsole"

        - "Konsole3":

          - Trigger: Ctrl+Alt+3
          - Action: activate "konsolex --name konsole-3" "konsole-3.Konsole"

        - "Konsole8":

          - Trigger: Ctrl+Alt+8
          - Action: activate "konsolex --name konsole-8" "konsole-8.Konsole"

        - "Konsole9":

          - Trigger: Ctrl+Alt+9
          - Action: activate "konsolex --name konsole-9" "konsole-9.Konsole"

        - "Speedcrunch":

          - Trigger: Ctrl+Alt+=
          - Action: activate speedcrunch speedcrunch

        - "Thunderbird":

          - Trigger: Ctrl+Alt+T
          - Action: activate thunderbird Mail.Thunderbird

    - Global Keyboard Shortcuts:

      - Set KDE Component to KWin:

        - Lower Window to Alt+2.

        - Maximize Window Horizontally to Alt+4.

        - Maximize Window Vertically to Alt+3.

        - Pack Grow Window Horizontally to Meta+Shift+Right.

        - Pack Grow Window Vertically to Meta+Shift+Up.

        - Pack Shrink Window Horizontally to Meta+Shift+Left.

        - Pack Shrink Window Vertically to Meta+Shift+Down.

        - Pack Window Down to Meta+Ctrl+Down.

        - Pack Window to the Left to Meta+Ctrl+Left.

        - Pack Window to the Right to Meta+Ctrl+Right.

        - Pack Window Up to Meta+Ctrl+Up.

        - Raise Window to Alt+1.

        - Window Operations Menu to Alt+Space.
          (Alt+Space defaults to krunner "Run Command", redundant with Alt+F2.)

  - Workspace Appearance and Behavior | Desktop Effects:

    - Enable Wobbly Windows, Set Wobbliness to halfway mark.

  - Workspace Appearance and Behavior | Window Decorations:

    - Window Decorations | Configure Decoration | Fine Tuning:

      - Check "Outline active window title".

  - Workspace Appearance and Behavior | Default Applications:

    - Email client to ``/usr/bin/thunderbird``.

    - Web Browser to ``/usr/bin/firefox``.

  - Workspace Appearance and Behavior | Window Behavior:

    - Focus | Slide "Policy" to "Focus Follows Mouse".

    - Focus | Delay focus by: 0 ms

    - Focus | Uncheck "Click raises active windows".

    - Window Actions | Inactive Inner Window | Left button "Activate".

    - Window Actions | Inactive Inner Window | Middle button "Activate".

    - Window Actions | Inactive Inner Window | Right button "Activate".

    - Moving | Windows | Check Display window geometry when moving or
      resizing.

  - Workspace Appearance and Behavior | Workspace Behavior:

    - Virtual Desktops

      - Number of desktops: Set to 6 (leaving Ctrl-F7 to Ctrl-F12 free).

      - Number of rows: 2

      - Change all names to "Desktop n" (n in [1, 6]).

    - Screen Edges

      - Set "Active Screen Edge Actions" to "No Action" for all corners
        (by default, only top-left corner is configured).

      - Uncheck "Maximize windows by dragging them to the top of the screen".

      - Uncheck "Tile windows by dragging them to the side of the screen".

  - Hardware:

    - Display and Monitor | Screen Locker:

      - Start automatically after: 30 minutes

      - Require password after: 60 seconds

    - Input Devices | Keyboard

      - Delay: 250 ms

      - Rate: 32 repeats/sec

    - Power Management | Energy Saving

      - AC Power:

        - Uncheck "Dim screen".

        - Uncheck "Screen Energy Saving".

        - Button events handling:

          - When laptop lid closed: Lock screen

      - On Battery:

        - Check "Dim screen"; after 2 minutes.

        - Check "Screen Energy Saving"; switch off after 5 minutes.

        - Button events handling:

          - When laptop lid closed: Sleep

      - On Low Battery:

        - Check "Dim screen"; after 1 minute.

        - Check "Screen Energy Saving"; switch off after 2 minutes.

        - Button events handling:

          - When laptop lid closed: Sleep


More KDE setup:

- Right-click on the task bar, choose ``Task Manager Settings``:

  - General:

    - Set ``Sorting`` to ``Manually``.

    - Set ``Grouping`` to ``Do Not Group``.

    - Check ``Show only tasks from the current desktop``.

- Right-click on clock, choose ``Digital Clock Settings``:

  - Appearance

    - Check "Show date"

    - Date format: Short date

- Click on taskbar "hamburger" on right side.  Drag "height" until date and time
  are comfortable to read.

Fonts
=====

- Reference articles:
  - https://gist.github.com/Earnestly/7024056
  - http://www.rastertragedy.com/

- Font setup may end up with this error::

    Fontconfig warning: "/etc/fonts/conf.d/50-user.conf", line 14:
    reading configurations from ~/.fonts.conf is deprecated.

  This occurs because the old location ``~/.fonts.conf`` is now deprecated.
  Apparently this gets created by installing a font via KDE's GUI.  So after
  that file is created, it can be moved to the new location as follow::

    mkdir -p ~/.config/fontconfig
    mv ~/.fonts.conf ~/.config/fontconfig/fonts.conf

  This should solve the error message.  The fonts themselves will still live
  in ``~/.fonts`` without problem.

- Top font from this article:
  http://hivelogic.com/articles/top-10-programming-fonts/

  - Inconsolata

  - Consolas (non-free Microsoft font)

  - Deja Vu Sans Mono

- I prefer "Deja Vu Sans Mono" to "Inconsolata".

- Also trying out "Hack" font, which supports Powerline fonts.

- Install fixed-width fonts::

    agi fonts-inconsolata ttf-bitstream-vera fonts-dejavu fonts-hack-ttf

    # Centos: some of these require nux-dextop repository.
    yi google-droid-sans-fonts google-droid-sans-mono-fonts \
      levien-inconsolata-fonts bitstream-vera-sans-mono-fonts

- Install an individual font file via:

  - Launch ``systemsettings`` | Font | Font Management:

    - If the is already installed, delete the old version.

    - Choose "Add" button.

    - Add fonts as "Personal" fonts (to ensure they are properly probed by Vim
      in ``~/.fonts/``).

- Install Hack:

  - [Ubuntu] May need to remove older version first:

    - First determine current version::

        dpkg -l 'fonts-hack-*'

    - If necessary, remove older fonts::

        apt-get remove 'fonts-hack-*'

  - Download from: https://github.com/source-foundry/Hack/releases

  - Save in ~/download/fonts/

  - Unzip Hack-v3.000-ttf.zip

  - Install via ``systemsettings`` as above.

- Install Pragmata Pro:

  - As 'mike' user::

      mkdir -p ~/tmp/ppro
      cd ~/tmp/ppro
      unzip /m/shared/purchased/fonts/pragmatapro-0.817.zip

  - Install all four fonts via ``systemsettings`` as above.

- Consider "Anonymous Pro" font someday.

Microsoft core fonts
====================

- Ref: http://mscorefonts2.sourceforge.net/

- Provides the following twelve fonts:

  Andale, Arial, Arial Bold, Comic Sans, Courier New, Georgia, Impact,
  Times New Roman, Trebuchet, Verdana, Tahoma, Wingdings

[Ubuntu]
--------

- Install::

    agi ttf-mscorefonts-installer

  Accept license agreement.

[Fedora] [CentOS]
-----------------

- Install prerequisites::

    yi curl cabextract xorg-x11-font-utils fontconfig

- Install fonts::

    rpm -i https://downloads.sourceforge.net/project/mscorefonts2/rpms/msttcore-fonts-installer-2.6-1.noarch.rpm

- Also have cached copy of the fonts here::

    ~/download/fonts/corefonts/

Font detection
==============

Bookmarklets to detect the font on a web page:

- http://code.stephenmorley.org/articles/whats-that-font-bookmarklet/

- This one doesn't work, except on its own site (?):
  http://chengyinliu.com/whatfont.html

Upload an image to detect the font:

- http://www.myfonts.com/WhatTheFont/

Font identifier tools:

- http://www.vandelaydesign.com/font-identifier-tools/

Other font setup
================

References:

- http://seasonofcode.com/posts/how-to-set-default-fonts-and-font-aliases-on-linux.html

- Ubuntu 16.04 defaults::

    for family in serif sans-serif monospace Arial Helvetica Verdana "Times New Roman" "Courier New"; do \
      echo -n "$family: "; \
      fc-match "$family";  \
    done

  With output::

    serif: DejaVuSerif.ttf: "DejaVu Serif" "Book"
    sans-serif: DejaVuSans.ttf: "DejaVu Sans" "Book"
    monospace: DejaVuSansMono.ttf: "DejaVu Sans Mono" "Book"
    Arial: Arial.ttf: "Arial" "Regular"
    Helvetica: n019003l.pfb: "Nimbus Sans L" "Regular"
    Verdana: Verdana.ttf: "Verdana" "Regular"
    Times New Roman: Times_New_Roman.ttf: "Times New Roman" "Regular"
    Courier New: Courier_New.ttf: "Courier New" "Regular"


Konsole
=======

- [CentOS] Install extra terminfo entries::

    yi ncurses-term

  Provides many extra terminfo entries, e.g. ``screen-256color-bce``.

- Run konsole and configure:

  **Warning** Must apply configuration changes in one tab before switching to
  another tab.

  - Settings | Edit Current Profile

    - General | Environment Edit |

      - [Fedora19+] Skip; 256-color is now handled system-wide:
        http://fedoraproject.org/wiki/Features/256_Color_Terminals

      - [Already default in Ubuntu] ``TERM=xterm-256color`` (default was
        ``TERM=xterm``)

    - Ubuntu 18.04 default font is "Hack 9-pt"; keeping this in general:

      - [obsolete] Appearance | Select Font, choose Pragmata Pro 11-pt regular.

    - Scrolling

      - Set "Fixed size scrollback" to 100000 lines

  - Settings | Configure Shortcuts:

    - Set "What's This?" shortcut to None (default: Shift+F1).

- [only for KDE5] Make current tab more distinguishable:

  - Create file ``~/.config/konsole.css`` with contents::

      QTabBar::tab::selected {
          background: lightblue;
          color: black;
          font: bold;
      }

  - In konsole, choose Settings | Configure Konsole | TabBar:

    - Check "Use user-defined stylesheet".

    - Browse for ``~/.config/konsole.css``.

Klipper
=======

- To work around a bug in GVim (https://github.com/vim/vim/issues/1023):

  - Right-click on Klipper

  - Check "Ignore selection"

  Fixes crash with GVim on KDE when selecting large amounts of text.

dolphin
=======

- Launch ``dolphin``, then use menu Control | Configure Dolphin | General, check
  "Use common view properties for all folders".  This eliminates littering
  ``.directory`` files everywhere.

Synaptics Touchpad
==================

- System Settings | Hardware | Input Devices | Touchpad:

  - Scrolling:

    - For touchpads *with* two-finger scrolling:

      - Enable Vertical Two-Fingers Scrolling.

      - Enable Horizontal Two-Fingers Scrolling.

      - Disable Vertical Edge Scrolling.

      - Disable Horizontal Edge Scrolling.

    - For touchpads *without* two-finger scrolling:

      - Enable Vertical Edge Scrolling.

      - Enable Horizontal Edge Scrolling.

      - Disable Vertical Two-Fingers Scrolling.

      - Disable Horizontal Two-Fingers Scrolling.

  - Tapping:

    - Disable Tapping (on machines that are too sensitive to accidental taps).

    - Tapping | One Finger means Left Button

To test whether tapping will work::

  synclient TapButton1=1

View all Synaptics settings::

  synclient -l

For more information::

  man synaptics
  https://fedoraproject.org/wiki/Input_device_configuration

Sound setup
===========

[Fedora] On Fedora 20, the headphone jack is no longer tracked correctly.  When
switching between laptop-only mode with Speakers to either headphones or docked
sound, the mixer must manually be switched.  Right-click on the sound icon and
choose "Audio setup", then choose "Audio Hardware Setup" tab.  Set "GF108 High
Definition Audio Controller" profile to "Off", then choose "Sound Card" to be
"Built-in Audio" and choose "Connector" to be "Speaker" to use built-in
speakers; otherwise, choose "Headphones".

Microphone testing:

https://bbs.archlinux.org/viewtopic.php?id=196525

View audio input devices::

  arecord -l

Capture microphone with verbose output (shows amplitude)::

  arecord -vvv -f dat /dev/null

e.g.::

  Max peak (12000 samples): 0x000001d5 #                    1%
  Max peak (12000 samples): 0x00000108 #                    0%
  Max peak (12000 samples): 0x00007896 ###################  94%

Record five seconds to a file, then play it back::

  arecord -d 5 test-mic.wav
  aplay test-mic.wav

GPG support
===========

Most parts are already installed for use with ``kleopatra`` or ``kgpg``.

- Install::

    yi gnupg2-smime

seahorse
========

- A GNOME-based app for encryption key management.

- Install::

    agi seahorse

kgpg
====

- Install::

    agi kgpg

    # CentOS, Fedora: already installed.

kleopatra GPG support
=====================

- [Fedora] [CentOS] Already installed with KDE.

- Install::

    agi kleopatra

    # [Fedora] [CentOS] Already installed with KDE.

- Launch ``kleopatra``.

- If already have a personal OpenGPG key pair, import it (probably from
  KeepassX thumb drive ``DOMAIN.asc`` file); then make sure to mark it
  as "my certificate" so the trust chain works right (still figuring this out).

- Create a new personal OpenGPG key pair:

  - Name: Michael Richard Henry

  - Email: USER@DOMAIN.com

  - Use standard GPG passphrase.

- Choose "Make a Backup of Your Key Pair" using ASCII armor, save to
  ``USER.asc``.  Copy to KeypassX thumb drives.

Konqueror
=========

- Install::

    agi konqueror

    yi konqueror

********
Printing
********

Printer Setup
=============

Using hplip with new HP M553n.

- Install hplip::

    agi hplip hplip-gui

    yi hplip hplip-gui

- [CentOS] Package ``hpijs`` contains some PPD files, e.g.::

    rpm -ql hpijs | grep m553

  Yielding::

    /usr/share/ppd/HP/hp-color_laserjet_m553-ps.ppd.gz

- Setup using command-line (preferred):

  - Reference: http://linuxgyd.blogspot.com/2015/04/12-cups-lpadmin-command-examples-to.html

  - List available printer models::

      lpinfo -m | grep _m553

    [Ubuntu] with result::

      postscript-hp:0/ppd/hplip/HP/hp-color_laserjet_m553-ps.ppd HP Color LaserJet M553 Postscript (recommended)

    [CentOS] with result::

      lsb/usr/HP/hp-color_laserjet_m553-ps.ppd.gz HP Color LaserJet M553 Postscript

  - Create printers::

      # [Ubuntu]
      for p in {crayon,pencil}; do
        lpadmin -p $p -v socket://1.2.3.128
        lpadmin -p $p -m postscript-hp:0/ppd/hplip/HP/hp-color_laserjet_m553-ps.ppd
        lpadmin -p $p -L 'Computer room'
      done

      # [CentOS]
      for p in {crayon,pencil}; do
        lpadmin -p $p -v socket://1.2.3.128
        lpadmin -p $p -m lsb/usr/HP/hp-color_laserjet_m553-ps.ppd.gz
        lpadmin -p $p -L 'Computer room'
      done

  - Configure printers::

      lpadmin -p pencil  -D 'Black & White printer'
      lpadmin -p crayon  -D 'Color printer'

      lpadmin -p pencil  -o 'HPColorAsGray=True'

      for p in {crayon,pencil}; do
        lpadmin -p $p -E
      done

      # Set the default printer.
      lpadmin -d crayon

View printer options::

  lpoptions -p pencil -l

- [Alternative] Setup using GUI:

  - Run ``hp-setup``, answer questions:

    - Network/Ethernet/Wireless network
    - Printer name: crayon
    - Description: HP Color LaserJet M553n
    - Location: Computer room
    - PPD file (default):
      postscript-hp:0/ppd/hplip/HP/hp-color_laserjet_m553-ps.ppd

  - More configuration via ``systemsettings`` | Printers:

    - Check "Default printer".

    - For black & white printers:

        - Printer Options | Color | Print Color as Gray: On

- Work-around hplib "system tray" bug:

  Probably obsolete in Ubuntu 18.04.

  - Ref: https://bugs.launchpad.net/hplip/+bug/1453303

  - If hplip's system tray application won't start with the message "No system
    tray detected on this system", the above link shows a work-around that will
    slow down starting of the tray utility until the system tray is up:

    In ``/etc/xdg/autostart/hplip-systray.desktop``, change this::

      Exec=hp-systray -x

    to this::

      Exec=sleep 15 && hp-systray -x

    If the "HP" icon isn't in the system tray, this may be the problem.

Printing text from the command line
===================================

- Recommended on LUG mailing list::

    enscript -P myprinter -fTimes-Roman12 mydoc.txt

  "Since enscript can't handle UTF-8 I use"::

    paps | lpr

  "with my own wrapper since I don't like the paps defaults and it takes an
  awful lot of options to get what I like."



*****
Admin
*****

Network
=======

- [SERVER] Disable NetworkManager, setup static IP address:

  - Disable NetworkManager::

      vim /etc/NetworkManager/NetworkManager.conf

    Comment out ``dns=dnsmasq``, change ``managed`` to ``true``::

      [main]
      plugins=ifupdown,keyfile
      #dns=dnsmasq

      [ifupdown]
      managed=true

  - Setup static IP::

      vim /etc/network/interfaces

    Change from ``dhcp`` to full settings::

      auto eno1
      # iface eno1 inet dhcp
      iface eno1 inet static
      # Use 1.2.3.251 for final SERVER configuration.
      address 1.2.3.252
      gateway 1.2.3.250
      netmask 255.255.255.0
      dns-search DOMAIN.com
      dns-nameservers 1.2.3.250

- [desktop, laptop] Configure wired and wireless settings via NetworkManager.

- [laptop] Setup wired Ethernet for "hotplug" to avoid delay at boot:

  - Still a problem in Ubuntu 18.04.

  - References:

    - https://ubuntuforums.org/showthread.php?t=2323253&page=2

    - https://askubuntu.com/questions/773973/ubuntu-16-04-system-boot-waits-saying-raise-network-interfaces

  - When installing via the server ISO, the primary network interface will be
    configured for ``auto``; if that interface is not available at boot, the
    system will wait up to 5 minutes for it to become available.

  - Change ``auto`` to ``allow-hotplug`` as shown for example below::

      vim /etc/network/interfaces

      # The primary network interface
      # auto enp0s25
      allow-hotplug enp0s25
      iface enp0s25 inet dhcp

- Shrink the 300-second timeout for ``auto`` interfaces to something saner::

    vim /etc/dhcp/dhclient.conf

    # timeout 300;
    timeout 5;

- [LAPTOPle] Setup search domain for DHCP on "public" networks:

  - Enumerate interfaces via ``ip link``.

  - For each non-loopback interface (e.g., ``enp0s25``, ``wlp3s0``, etc.),
    create a per-interface configuration file for dhclient with name
    ``/etc/dhcp/dhclient-enp0s25.conf`` (for example)::

      echod -o /etc/dhcp/dhclient-enp0s25.conf '
        supersede domain-name "DOMAIN.com";
      '

      echod -o /etc/dhcp/dhclient-wlp3s0.conf '
        supersede domain-name "DOMAIN.com";
      '

[Ubuntu] cron schedule
======================

- Ubuntu default times for daily, weekly, and monthly jobs is during the 6 a.m.
  hour.  Correct this to 2 a.m.::

    vim /etc/crontab

    25 2    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
    47 2    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
    52 2    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )

logwatch
========

- Install::

    agi logwatch

    yi logwatch

- Optionally configure (but defaults are good)::

    vim /usr/share/logwatch/default.conf/logwatch.conf

journal limits
==============

- Set limits for systemd journal in , and make
  the storage persistent::

    vim /etc/systemd/journald.conf

    [Journal]
    Storage=persistent
    SystemMaxUse=200M
    SystemMaxFileSize=32M

Snapshot
========

- Install rsnapshot::

    agi rsnapshot

    yi rsnapshot

- Create snapshot root mount point; make it immutable::

    mkdir /snapshot
    chattr +i /snapshot

- [SERVER] Using zfs: create separate volume for snapshots::

    zfs create -o mountpoint=/snapshot SERVER/snapshot

- Using LVM: create separate volume for snapshots, format, mount:

  **Note**: LVM volumes starting with "snapshot" are reserved :-(

  - [SERVER]::

      lvcreate SERVER -L 459G -n lv_snapshot
      mke2fs -t ext4 /dev/SERVER/lv_snapshot
      mount /dev/SERVER/lv_snapshot /snapshot

  - [WORKSTATION]::

      lvcreate WORKSTATION_spinner -L 500G -n lv_snapshot
      mke2fs -t ext4 /dev/WORKSTATION_spinner/lv_snapshot
      mount /dev/WORKSTATION_spinner/lv_snapshot /snapshot

  - Edit ``/etc/fstab`` for snapshots::

      vim /etc/fstab

      /dev/mapper/SERVER-lv_snapshot /snapshot ext4  defaults  0 2

    or::

      /dev/mapper/WORKSTATION_spinner-lv_snapshot /snapshot ext4  defaults  0 2

- Adjust configuration file as follows, **using tab characters** to separate
  fields::

    cp -a /etc/rsnapshot.conf{,.orig}
    vim /etc/rsnapshot.conf

    ## NOTE: Ensure paths in .conf file are correct; particularly, use
    ## /bin/cp and /bin/rm (they are not in /usr/bin).

    snapshot_root  /snapshot/
    no_create_root 1

    retain       hourly  10
    retain       daily   7
    retain       weekly  4
    retain       monthly 12

    exclude             /**/.swp
    exclude             /**/.*.swp
    exclude             /**/.dropbox.cache
    exclude             /home/**/*-tmp/
    exclude             /home/*/.*/
    exclude             /home/*/.xsession-errors
    exclude             /home/*/vms/
    exclude             /home/*/mail/.imap/

    # Just for SERVER.
    exclude             /m/**/*-tmp/
    exclude             /m/torrent/
    exclude             /m/tmp/
    exclude             /m/srv/nfs/
    exclude             /m/**/iTunes/**/*[cC]ache*
    exclude             /m/**/thunderbird/**/*[cC]ache*

    # SERVER backups:
    backup  /m/             SERVER/
    backup  /etc/           SERVER/
    backup  /home/          SERVER/
    backup  /usr/local/     SERVER/

    # WORKSTATION backups:
    backup  /etc/           WORKSTATION/
    backup  /home/          WORKSTATION/
    backup  /usr/local/     WORKSTATION/

- Setup cron for proper intervals::

    vim /etc/crontab

    # min hour     dom         mon dow  user  what
      00  23        1           *   *   root  rsnapshot monthly
      10  23        1,8,15,22   *   *   root  rsnapshot weekly
      20  23        *           *   *   root  rsnapshot daily
      30  7-23      *           *   *   root  rsnapshot hourly

Simple mail setup
=================

postfix for satellite nodes
---------------------------

- Install postfix::

    agi postfix

- [Ubuntu]:

  - When installing, answer the configuration questions as:

    - Choose ``satellite`` email system if not installing for
      SERVER.DOMAIN.com.

    - Choose FQDN for System mail name (e.g., ``LAPTOP.DOMAIN.com``).

    - Choose ``mailman.DOMAIN.com`` for SMTP relay host.

    - Leave remaining questions (if any) at their defaults.

  - Can also do ``dpkg-reconfigure postfix`` to answer these questions again.

- To allow ``/etc/hosts`` to override ``mailman.DOMAIN.com``, add this line
  to the postfix configuration and reload::

    echod -a /etc/postfix/main.cf '
      # Use "native" DNS (allowing /etc/hosts to work).
      smtp_host_lookup = native
    '

    systemctl restart postfix

mail aliases
------------

- Ensure ``root`` and ``mike`` aliases exists on all boxes::

    echod -a /etc/aliases '
      root:   USER@DOMAIN.com
      mike:   USER@DOMAIN.com
      '

- Enable new aliases::

    newaliases

Mail tools
----------

Note: Ubuntu 16.04 uses mailutils now for the ``mail`` command.

- Install tools::

    agi mutt mailutils

    yi mutt mailx

  Provides enhanced ``mail`` command.

- [all]:

  - Test sending an email::

      date | mail -s 'testing' USER@DOMAIN.com

mail over ssh
-------------

- When on the road, can use ssh as follows:

  - Kill the local postfix::

      systemctl stop postfix

  - Uncomment ``mailman`` to use ``localhost`` in ``/etc/hosts``::

      127.0.0.1        mailman.DOMAIN.com mailman

  - Setup ssh tunnel, forwarding ports 25 and 993 for email::

      rSERVER -L 25:127.0.0.1:25 -L 993:127.0.0.1:993

  - Use Thunderbird normally.

- After return:

  - Comment ``mailman`` line in ``/etc/hosts``::

      #127.0.0.1        mailman.DOMAIN.com mailman

  - Restart postfix::

      systemctl restart postfix

  **Note** It's important to restart postfix, since ``/etc/init.d/postfix``
  copies files like ``/etc/hosts`` into ``/var/spool/postfix/etc/hosts``.


************
Productivity
************

KeePassx
========

- Ensure using version 2.x.  Note: CentOS needs ``keepassx2`` (v2.0.3) to get
  version 2.x; ``keepassx`` is older 1.x.  Sadly, command name is ``keepassx2``
  in this case.  The ``kee`` script deals with the inconsistent naming.

- Install::

    agi keepassx

    yi keepassx2

- Create mount points::

    sudo mkdir -p /mnt/{keepass,keemirror,keebackup}

- Setup mount points in ``/etc/fstab``::

    echod -a /etc/fstab '
    LABEL=KEEPASS   /mnt/keepass    vfat shortname=lower,user,noauto 0 0
    LABEL=KEEMIRROR /mnt/keemirror  vfat shortname=lower,user,noauto 0 0
    LABEL=KEEBACKUP /mnt/keebackup  vfat shortname=lower,user,noauto 0 0
    '

- Run KeePassX, then setup via Tools | Settings:

  - General:

    - Uncheck "Open previous databases on startup".

  - Security:

    - Check "Show passwords in cleartext by default".

- Use ``~/bin/kee`` script to maintain mirror/backup.

LibreOffice
===========

- Install::

    agi libreoffice

    yi libreoffice

- Install database tools (for ``*.mbd`` access)::

    agi mdbtools

gnucash
=======

References:

- https://wiki.gnucash.org/wiki/FAQ:

- Version compatibility:

  - Q: Can I exchange Gnucash file with any other version of GnuCash?

    A: Not quite.

        You can always upgrade from versions >= 1.8 to a higher release. In most
        cases you can go back-and-forth between adjacent major releases: You can
        read 2.0 in 1.8.x fine.  Reading 2.2 in 2.0 will not work, if you have
        scheduled transactions.  2.4 and 2.2 files are interchangeable 2.6 and
        2.4 files are interchangeable unless you use the Credit Notes feature in
        2.6

- Newer versions are available from GetDeb (see above).

- Install::

    # This gets version 2.6.19 on Ubuntu 18.04 (installed September 2018).
    agi gnucash gnucash-docs python-gnucash

- [not recommended] Install database back-end (not recommended at present
  because support is incomplete)::

    # agi libdbd-sqlite3

- Configure:

  - Edit | Preferences | Register Defaults | Check "Double Line mode".

anki
====

- Reference: https://apps.ankiweb.net/docs/manual.html

- Install::

    agi anki

kalarm
======

- Install::

    agi kalarm

tesseract optical character recognition (ocr) software
======================================================

- Install::

    agi tesseract-ocr

    yi tesseract

gocr - GNU optical character recognition (ocr) software
=======================================================

- **NOTE** It's pretty lousy.

- Install::

    agi gocr

    yi gocr

Dia
===

- Install::

    agi dia

    yi dia

xfig
====

- Install::

    agi xfig

    yi xfig

Fitbit synchronization
======================

- Reference:
  - https://bitbucket.org/benallard/galileo
  - http://linuxaria.com/article/how-to-sync-your-fitbit-under-linux

- Install Galileo::

    pipi galileo

**************
Remote Desktop
**************

Synergy
=======

- *NOTE* Probably needs firewall configuration (not tested).

- Install::

    agi synergy

    yi synergy-plus

- On client, no configuration necessary.

- On server, create configuration file::

    echod -o /etc/synergy.conf '
    section: screens
      LAPTOP-wired:
      SERVER:
    end

    section: links
      LAPTOP-wired:
        right = SERVER
        left  = SERVER

      SERVER:
        left  = LAPTOP-wired
        right = LAPTOP-wired
    end

    section: aliases
      LAPTOP-wired:
        LAPTOP-wired.DOMAIN.com
        LAPTOP.DOMAIN.com
        LAPTOP
      SERVER:
        SERVER.DOMAIN.com
    end
    '

- Configure firewall::

    ufw allow 24800/tcp

    firewall-cmd --add-port 24800/tcp
    firewall-cmd --permanent --add-port 24800/tcp

- Start server-side daemon:

  - [Ubuntu]::

      synergys

  - [Fedora]::

      synergys

- Start client and connect to desired server::

    synergyc servername

TeamViewer
==========

- [Ubuntu] Install:

  - Visit https://www.teamviewer.com/en/download/linux/

  - Download 64-bit .deb file and save in download area, e.g.::

      ~mike/download/internet/teamviewer/teamviewer_13.2.13582_amd64.deb

  - Install .deb file::

      dpkg -i ~mike/download/internet/teamviewer/teamviewer_13.2.13582_amd64.deb

- [Fedora] Install:

  - Install 32-bit library for xrandr::

      yi libXrandr-0:1.4.2-2.fc22.i686

  - Install::

      dnf install ~mike/download/internet/teamviewer/teamviewer_11.0.53191.i686.rpm

- Install on client machine as well (perhaps Windows VM).

- Execute TeamViewer tool and configure with account information::

    teamviewer &

- On Windows, may have problems seeing the mouse cursor.  Can change the mouse
  pointer scheme via Control Panel | Hardware and Sound | Devices and Printers |
  Mouse | Pointers | Scheme:

  - Set to "Windows Black (system scheme)" instead of default "Windows Aero
    (system scheme)".

  - Alternatively, keep the original scheme but fix the text cursor to be
    something like this:

      beam_i.cur

************
Search tools
************

ack
===

- Install::

    agi ack-grep

    yi ack

The Silver Searcher
===================

- Install::

    agi silversearcher-ag

    yi the_silver_searcher

The Platinum Searcher
=====================

- From:
  https://github.com/monochromegane/the_platinum_searcher

- Release area:
  https://github.com/monochromegane/the_platinum_searcher/releases

- Install::

    curl -L https://github.com/monochromegane/the_platinum_searcher/releases/download/v2.1.5/pt_linux_amd64.tar.gz | sudo tar -C /opt -zx
    sudo ln -sf /opt/pt_linux_amd64/pt /usr/local/bin/pt

Locate database (locatedb) for "all"
====================================

- The default database for the ``locate`` command is too encompassing, since it
  includes ``/snapshot``.

- Backup original configuration::

    cp /etc/updatedb.conf{,.dist}

- Enable ``PRUNENAMES`` and append ``/snapshot`` to ``PRUNEPATHS`` as follows::

    vim /etc/updatedb.conf

      PRUNENAMES = ".git .bzr .hg .svn"
      PRUNEPATHS="/tmp /var/spool /media /home/.ecryptfs /var/lib/schroot /snapshot"

- Create new cron job for ``mlocate-all`` by cloning original::

    cp -a /etc/cron.daily/mlocate{,-all}

- Edit new version to change lockfile and adjust invocation::

    vim /etc/cron.daily/mlocate-all

  Change this::

    flock --nonblock /run/mlocate.daily.lock $IONICE /usr/bin/updatedb.mlocate

  To this::

    flock --nonblock /run/mlocate-all.daily.lock \
      $IONICE /usr/bin/updatedb.mlocate \
      --prunenames '' \
      --prunepaths '/tmp /var/spool /media /home/.ecryptfs /var/lib/schroot' \
      --output /var/lib/mlocate/mlocate-all.db

- Create command to use new database::

    echod -o /usr/local/bin/locate-all '
      #!/bin/sh

      locate --database /var/lib/mlocate/mlocate-all.db "$@"
    '
    chmod +x /usr/local/bin/locate-all

- Invocation::

    # "Normal" search (excludes the snapshots):
    locate stuff

    # Exhaustive search (include the snapshots, .git/.svn/etc.):
    locate-all stuff

recoll desktop search
=====================

- Install::

    agi recoll

    yi recoll

zeal
====

- Install::

    agi zeal

    yi zeal

- Bound to Win-Z by default.

*******************
Network filesystems
*******************

wdfs
====

[Ubuntu]
--------

**NOTE** Need to install libneon27-gnutls-dev to avoid this bug:
http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=622140

- Get dependencies::

    agi libfuse-dev libneon27-gnutls libneon27-gnutls-dev \
      libglib2.0-dev libglib2.0-doc checkinstall

- Perform these steps to install::

    # As normal user:
    wget http://noedler.de/projekte/wdfs/wdfs-1.4.2.tar.gz
    tar -zxf wdfs-1.4.2.tar.gz
    cd wdfs-1.4.2
    ./configure
    make
    sudo checkinstall

  Answers to checkinstall questions:

  - Create default set of package docs?  Yes

  - Description of the package:

      WebDAV-based fuse filesystem.

  Using ``checkinstall`` instead of ``make install`` builds a .deb package
  (wdfs_1.4.2-1_amd64.deb in this case) and then installs the package.

[Fedora]
--------

- Install::

    yi wdfs

[all]
-----

- Optional test server:
  https://webdavserver.com/

  Visit above link, get URL of test data.

- Test the mount::

    mkdir -p ~/tmp/dav/

    # Use URL of some DAV server:
    wdfs https://webdavserver.com/User66f27c7 ~/tmp/dav/

    ls ~/tmp/dav/

- Unmount::

    fusermount -u ~/tmp/dav/

cadaver
=======

- Install::

    agi cadaver

    yi cadaver

davfs2
======

- Install::

    agi davfs2

    yi davfs2

  [Ubuntu] Allow unprivileged users to mount.

*********
Utilities
*********

xclip/xsel clipboard piping
===========================

- Install::

    agi xclip xsel

    yi xclip xsel

- Use::

    ls | xsel
    # middle-click somewhere to paste the result.

    xsel | grep string

    xclip-copyfile some/file
    xclip-pastefile

  Also has xclip, but it doesn't auto-detect piping.

- Also, setup a Bash script ``clip`` that just does::

    xsel -b "$@"

  That makes it easier to use the clipboard instead of the
  primary X selection.

lshw
====

Displays hardware-related information.

- Install::

    agi lshw lshw-gtk

    yi lshw lshw-gui

lstopo
======

Display computer block diagram graphically.

- Install::

    agi hwloc

    yi hwloc-gui

SMART disk support
==================

- Install::

    agi smartmontools

    yi smartmontools

hdparm
======

- Install::

    # Already installed on Ubuntu 18.04.
    agi hdparm

    yi hdparm

ranger (console-based file manager)
===================================

- Install::

    agi ranger

    yi ranger

Midnight commander
==================

- Install::

    agi mc

    yi mc

zsh
===

- Install::

    agi zsh

    yi zsh

fish
====

- Install::

    agi fish

autokey
=======

- Install::

    agi autokey-gtk

Most
====

- Install this 'more'-like, 'less'-like pager::

    agi most

    yi most

temperature sensors
===================

- Install::

    agi lm-sensors

    yi lm_sensors

- Install GUIs for sensors::

    agi conky xsensors

    yi conky xsensors

- Check::

    sensors

- Display GUI temperature::

    xsensors

stress
======

- Install::

    agi stress

    yi stress

- Invoke::

    stress --cpu 8 --io 4 --vm 2 --vm-bytes 128M

words (for ``/usr/share/dict``)
===============================

- Install::

    # Ubuntu: already installed.

    yi words

********************
Filesystem utilities
********************

USB Flash Drives
================

- See help at:
  https://help.ubuntu.com/community/RenameUSBDrive

- Ensure ``gparted`` is installed::

    agi gparted

    yi gparted

- If necessary, unmount partition of interest.

- Use ``gparted`` menu Partition | Label to change the label.

  Note that **the volume must not be mounted when changing the label**.

- For FAT volumes, should also be able to use ``mtools``.

- Pre-setup to ignore errors like this one::

    Total number of sectors (15638656) not a multiple of sectors per track (63)!

  by skipping the check::

    echo mtools_skip_check=1 >> ~/.mtoolsrc

- Check existing label::

    mlabel -i /dev/sde1 -s ::

- Change label::

    mlabel -i /dev/sde1 ::NEW_LABEL

archivemount
============

- Install::

    agi archivemount

    yi archivemount

- Use::

    sudo mkdir /mnt/archive
    archivemount archive.tar.gz /mnt/archive
    sudo umount /mnt/archive

usb-creator-kde
===============

- Install::

    agi usb-creator-kde

- Run::

    usb-creator-kde

unetbootin
==========

**TODO** No longer packaged under this name in Ubuntu.

- Install::

    agi unetbootin

- Format a USB drive with FAT32::

    fdisk /dev/sdh

    # Create partition 1 with type=c
    mkfs.vfat -F32 /dev/sdh1

- Mount the partition anywhere::

    mount /dev/sdh1 /mnt/tmp

- Run ``unetbootin``, browse for an .iso file, write to sdh1.

- Unmount the partition::

    umount /dev/sdh1

******************
Network apps/tools
******************

Google Chrome
=============

- Google Chrome home: http://www.google.com/chrome/

[Ubuntu]
--------

- Instructions: https://www.linuxbabe.com/ubuntu/install-google-chrome-ubuntu-18-04-lts

- Chrome sets up the cron job ``/opt/google/chrome/cron/google-chrome`` to
  periodically update ``/etc/apt/sources.list.d/google-chrome.list``.  To
  bootstrap the process, pre-create that file::

    echod -o /etc/apt/sources.list.d/google-chrome.list '
      deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main
    '

- Update and install stable version::

    agu
    agi google-chrome-stable

- **Until the defaults change**, always run with ``--password-store=detect`` to
  ensure stored passwords are encrypted.  Facilitated via a wrapper script
  ``~/bin/chrome`` that does::

    /opt/google/chrome/chrome --password-store=detect "$@"

Note: Installation of the open-source Chromium browser may be done via::

    # agi chromium-browser

For differences between Chrome and Chromium:
  http://code.google.com/p/chromium/wiki/ChromiumBrowserVsGoogleChrome

At present, chromium-browser for Ubuntu doesn't seem to have
kwallet support.

[Fedora]
--------

- Instructions from:
  http://www.if-not-true-then-false.com/2010/install-google-chrome-with-yum-on-fedora-red-hat-rhel/

- Create repo file for Google Chrome::

    echod -o /etc/yum.repos.d/google-chrome.repo '
      [google-chrome]
      name=google-chrome - \$basearch
      baseurl=http://dl.google.com/linux/chrome/rpm/stable/\$basearch
      enabled=1
      gpgcheck=1
      gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
    '

- Install::

    yi google-chrome-stable

[all]
-----

- Configure: Choose icon to right of URL bar | Settings (same as visiting
  the URL chrome://chrome/settings/):

  - Settings section (on the left):

    - "Appearance": Check "Show Home button", change from New Tab page to
      https://google.com/.

    - "On Startup": choose "Continue where you left off"

    - Choose "Show advanced settings".

    - Downloads | check "Ask where to save each file...".

Links
=====

- Install::

    agi links

    yi links

liferea RSS reader
==================

- Install::

    agi liferea

    yi liferea

Konversation IRC client
=======================

- Install::

    agi konversation

    yi konversation

portquiz.net
============

- Nice tool to verify outbound port connectivity, e.g.::

    $ telnet portquiz.net 8080
    Trying ...
    Connected to portquiz.net.
    Escape character is '^]'.

    $ nc -v portquiz.net 8080
    Connection to portquiz.net 8080 port [tcp/daytime] succeeded!

    $ curl portquiz.net:8080
    Port 8080 test successful!
    Your IP: 108.163.211.152

    $ wget -qO- portquiz.net:8080
    Port 8080 test successful!
    Your IP: 108.163.211.152

- How it works:

    Here is the content of /etc/iptables/rules.v4::


      # Generated by iptables-save v1.4.14 on Sun Aug 25 12:43:34 2013
      *nat
      :PREROUTING ACCEPT [0:0]
      :POSTROUTING ACCEPT [0:0]
      :OUTPUT ACCEPT [0:0]
      -A PREROUTING -i lo -j RETURN
      -A PREROUTING -p icmp -j RETURN
      -A PREROUTING -m state --state RELATED,ESTABLISHED -j RETURN
      -A PREROUTING -p tcp -m tcp --dport 22 -j RETURN
      -A PREROUTING -p tcp -m tcp --dport 21 -j RETURN
      -A PREROUTING -p tcp -m tcp --dport 25 -j RETURN
      -A PREROUTING -p tcp -m tcp --dport 80 -j RETURN
      -A PREROUTING -p tcp -m tcp --dport 443 -j RETURN
      -A PREROUTING -p tcp -j DNAT --to-destination :80
      COMMIT
      # Completed on Sun Aug 25 12:43:34 2013
      # Generated by iptables-save v1.4.14 on Sun Aug 25 12:43:34 2013
      *filter
      :INPUT ACCEPT [0:0]
      :FORWARD ACCEPT [0:0]
      :OUTPUT ACCEPT [0:0]
      -A INPUT -p icmp -j ACCEPT
      -A INPUT -i lo -j ACCEPT
      -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
      -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
      -A INPUT -p tcp -m state --state NEW -m tcp --dport 21 -j ACCEPT
      -A INPUT -p tcp -m state --state NEW -m tcp --dport 25 -j ACCEPT
      -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
      -A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
      -A INPUT -j DROP
      COMMIT
      # Completed on Sun Aug 25 12:43:34 2013

  The filter table is classical: ACCEPT a few services, then DROP the rest.

  Portquiz.net logic is in the nat table:

      Forward connection on any port to port 80 (DNAT –to-destination :80)
      Except for normal services (just RETURN for normal activity)

[Ubuntu] Netcat (OpenBSD variant)
=================================

- Install::

    # Ubuntu 18.04 appears to have this installed already.

    agi netcat-openbsd

  (Takes priority over netcat-traditional by default)

socat
=====

- Install::

    agi socat

    yi socat

traceroute
==========

- Install::

    agi traceroute

    yi traceroute

nmap
====

- Install::

    agi nmap

    yi nmap

socks proxy
===========

- Tunnel via ssh to setup a socks5 proxy::

    ssh -D1234 destination.server

  Now localhost:1234 is a SOCKS5 proxy.

- Install ``tsocks``::

    agi tsocks

- Configure::

    echod -o /etc/tsocks.conf '
      server = 127.0.0.1
      server_type = 5
      server_port = 1234
    '

- Wrap ``tsocks`` around any app, e.g.::

    tsocks apt-get install some-package

**NOTE** Some indications are that tsocks doesn't push DNS queries through the
socks proxy, so if DNS isn't working properly, this won't work either.

sshfs
=====

- Install::

    agi sshfs

    yi sshfs

- To mount ``/m`` from SERVER, for example::

    mkdir -p ~/tmp/m
    sshfs mike@SERVER:/m ~/tmp/m

    ls ~/tmp/m
    sudo umount ~/tmp/m

wireshark
=========

- Install::

    agi wireshark

    yi wireshark-gnome

  [Ubuntu] Allow non-superusers to capture packets.

telnet
======

- Install::

    agi telnet

    yi telnet

***********
Programming
***********

Python, setuptools and pypan
============================

- Install python3::

    # Ubuntu has python3 by default, but it's 3.6.5; but newer is available:
    agi python3.7

    yi python34

- Configure to use pypan:

  - Configuration info from:
    http://peak.telecommunity.com/DevCenter/EasyInstall#configuration-files

  - Create distutils configuration files for Python 2, Python 3::

      for i in /usr/lib*/python*/distutils; do
        echod -o $i/distutils.cfg '
          [easy_install]

          find_links = file:///m/sys/pypan/
        '
      done

    Ubuntu uses ``/usr/lib``; CentOS, Fedora use ``/usr/lib64``.

- Install pip::

    agi python-pip python3-pip

    yi python-pip python34-pip

- Configure pip for pypan (repeat for root and mike)::

    mkdir ~/.pip
    echod -o ~/.pip/pip.conf '
      [global]
      find-links = file:///m/sys/pypan/
    '

- **SKIPPING; BREAKS Ubuntu 18.04** Upgrade pip::

    # Install pip3 last so that ``pip`` refers to python3.
    pip2 install -U --force-reinstall pip
    pip3 install -U --force-reinstall pip

- Install virtualenv, virtualenvwrapper::

    pip2 install -U virtualenv virtualenvwrapper
    pip3 install -U virtualenv virtualenvwrapper

- Configure virtualenvwrapper using steps from
  https://virtualenvwrapper.readthedocs.org/en/latest/::

    echod -a ~/.bashrc '
      export WORKON_HOME=~/envs
      if [ -n "$(type -P virtualenvwrapper.sh)" ]; then
          source virtualenvwrapper.sh
      fi
    '
    mkdir ~/envs

    # Ignore ~/envs:
    vim ~/.gitignore

- Install "moo" as a test::

    # [Ubuntu]
    pip2 install oo
    agi python-pygame

    # [Fedora]
    pip install oo
    yi pygame

    python -moo

- Install from pypan::

    pip3 install findx
    pip3 install svnwrap
    pip3 install ptee

- Python development tools::

    pip3 install flake8 autopep8 pep8-naming pylint flake8-quotes black

    # Presumably the version < 0.2.0 is obsolete; verify this someday.
    # pip install flake8 autopep8 pep8-naming pylint 'flake8-quotes<0.2.0'

    # Note: the 0.2.0 restriction is hopefully temporary:
    # https://github.com/zheller/flake8-quotes/issues/23

ipython
=======

- IPython interactive python environment::

    agi ipython ipython3

    yi ipython ipython-gui ipython-doc

- Create configuration profile::

    ipython profile create

- Backup configuration::

    cp ~/.ipython/profile_default/ipython_config.py{,.dist}

- Adjust configuration::

    vim ~/.ipython/profile_default/ipython_config.py

    # [Obsolete; now 'Neutral' is the default] Only for light background:
    # c.TerminalInteractiveShell.colors = 'LightBG'

    c.TerminalInteractiveShell.confirm_exit = False

- Python graphical debugger::

    agi winpdb

    yi winpdb

- Python tools (e.g., ``2to3``)::

    yi python-tools

- Python documentation::

    agi python-doc python3-doc

    yi python-docs

Blessings
=========

This is a Python wrapper for curses.

- Reference: http://pypi.python.org/pypi/blessings/

- Install::

    pip3 install blessings

Python (old interpreters)
=========================

- [Ubuntu] Use the "deadsnakes" PPA:
  https://launchpad.net/~fkrull/+archive/ubuntu/deadsnakes

  - Add the PPA::

      sudo add-apt-repository ppa:deadsnakes/ppa

      # Pause to press Enter to accept the ppa.

      sudo apt-get update

  - Install older version, e.g.::

      agi python2.6 python2.6-dev

Development tools
=================

- Install general build tools::

    agi build-essential linux-headers-generic cmake ddd automake autopoint \
        make-doc manpages-dev perl-doc texinfo texi2html

    ygi 'Development Tools'
    yi scons cmake cmake-gui ddd texinfo gcc-c++

- [Fedora] [CentOS] RPM building tools::

    yi rpm-build

- Install Exuberant ctags::

    agi ctags

    yi ctags

- dos2unix, unix2dos tools::

    agi tofrodos

    yi dos2unix

  [Ubuntu] Create the traditional names::

    sudo ln -s /usr/bin/{todos,unix2dos}
    sudo ln -s /usr/bin/{fromdos,dos2unix}

- PyQt::

    agi python-qt4 python-qt4-dev python-qt4-doc qt4-doc qt4-dev-tools

    yi PyQt4 PyQt4-devel

- [Ubuntu] tree::

    agi tree

- General man pages::

    yi man-pages

- HTML tidy::

    agi tidy

    yi tidy

- [Fedora] [CentOS] RPM development tools::

    yi rpmdevtools

- strace::

    # Ubuntu: already installed.

    yi strace

- clang::

    agi clang-6.0

- PowerPC cross compiler::

    agi gcc-5-powerpc-linux-gnu

- Arm cross compiler::

    agi gcc-5-arm-linux-gnueabihf

- bmake (NetBSD make for POSIX compatibility testing).

32-bit libraries
================

- Install::

    agi gcc-multilib g++-multilib

Ruby
====

**NOTE** This causes some difficulties when compiling Vim, so skip this part
until Ruby is really needed, then figure out how to work around it.

- Reference:
  https://github.com/rvm/ubuntu_rvm

- TODO: Not sure it's best to use ``rvm`` package.  Requires a logout/login
  just so that user's gems are installed in system-owned directory.

- Add repository for Ubuntu package for ``rvm``::

    apt-add-repository -y ppa:rael-gc/rvm
    apt-get update
    agi rvm

- Add ``mike`` to ``rvm`` group::

    gpasswd -a mike rvm

- As normal user, login (to activate the ``rvm`` group and source the
  script ``etc/profile.d/rvm.sh``).

- Install latest ruby::

    rvm install ruby

- Setup to use version 2.4.1 by default::

    rvm use ruby-2.4.1 --default

  To remove the default::

    # NOTE: this doesn't work well yet.
    rvm alias delete default

- Generate documentation::

    rvm docs generate

- Install ``pry``::

    gem install pry

- Install Rubocop::

    gem install rubocop

Subversion
==========

- Install::

    agi subversion

    yi subversion

- Install ``svnshell``::

    agi python-subversion

- Configure (probably just take saved config file).

- Test with svnserve:

  - Edit ``path/to/repo/conf/svnserve.conf``::

      # Ensure this is uncommented:
      password-db = passwd

      # To disable anonymous access entirely, use these lines:
      anon-access = none
      auth-access = write

  - Setup dummy password file ``path/to/repo/conf/passwd``::

      [users]
      mike = password

  - Run ``svnserve`` in "daemon-mode", but in foreground with a log file::

      svnserve -d --foreground --log-file /tmp/svnserve.log

  - Use absolute path with ``svn:``.  For example::

      cd ~/tmp/svn
      svnadmin create test.repo
      svn ls svn://localhost/home/mike/tmp/svn/test.repo

Mercurial
=========

- Install::

    agi mercurial

    yi mercurial

Bazaar
======

- Install::

    agi bzr

    yi bzr

CVS
===

- Install::

    agi cvs

    yi cvs

Doxygen
=======

- Install Doxygen-related tools::

    agi doxygen doxygen-doc graphviz graphviz-doc mscgen

    yi doxygen graphviz mscgen

Bless hex editor
================

- Install::

    agi bless

Java
====

[Ubuntu]
--------

- https://help.ubuntu.com/community/Java

- Trying out OpenJDK for now (it's already installed).

- openjdk-8-jre is installed.  See other packages such as openjdk-8-jdk for more
  options.

[Fedora]
--------

Oracle Java
^^^^^^^^^^^

- Oracle Java instructions from:
  http://www.if-not-true-then-false.com/2014/install-oracle-java-8-on-fedora-centos-rhel/

- Download from:
  http://www.oracle.com/technetwork/java/javase/downloads/index.html

  Direct link (may not work unless accept license):
  http://download.oracle.com/otn-pub/java/jdk/8u45-b14/jdk-8u45-linux-x64.rpm

- Install::

    rpm -Uvh ~mike/download/programming/java/jdk-8u40-linux-x64.rpm

- Install alternatives::

    ## java ##
    alternatives --install /usr/bin/java java /usr/java/latest/bin/java 20000

    ## javaws ##
    alternatives --install /usr/bin/javaws javaws /usr/java/latest/bin/javaws 20000

    ## Java Browser (Mozilla) Plugin 64-bit ##
    alternatives --install /usr/lib64/mozilla/plugins/libjavaplugin.so libjavaplugin.so.x86_64 /usr/java/latest/jre/lib/amd64/libnpjp2.so 20000

    ## Install javac only if you installed JDK (Java Development Kit) package ##
    alternatives --install /usr/bin/javac javac /usr/java/latest/bin/javac 20000
    alternatives --install /usr/bin/jar jar /usr/java/latest/bin/jar 20000

- Choose alternative java implementation (one at a time)::

    alternatives --config java
    alternatives --config javaws
    alternatives --config libjavaplugin.so.x86_64
    alternatives --config javac
    alternatives --config jar

- A fix for ``failed to read link /usr/bin/javaws``:
  http://johnglotzer.blogspot.com/2012/09/alternatives-install-gets-stuck-failed.html

  Essentially, remove the contents of ``/var/lib/alternatives/SOMETHING``
  if the invocation of ``alternatives --install /usr/bin/SOMETHING`` fails,
  then try again (do the ``--install`` over again).

OpenJDK
^^^^^^^

- Doesn't work for chess.com.

- Can leave it installed anyway if needed to fulfill dependencies.

- Fedora FAQ recommends OpenJDK version:
  https://fedoraproject.org/wiki/Java/FAQ

- Install::

    yi java-1.7.0-openjdk{,-devel,-javadoc,-demo} icedtea-web

Haskell
=======

- Install Haskell Platform::

    agi haskell-platform haskell-platform-doc

    yi haskell-platform

golang
======

For the "Go" programming language.

- Install::

    agi golang

    yi golang

Diff tools
==========

- Install diff tools::

    agi kompare meld kdiff3 colordiff

    yi kdesdk-kompare xxdiff xxdiff-tools meld kdiff3 colordiff

- Setup colordiff settings::

    echod -o ~/.colordiffrc '
      # Settings for colordiff.
      plain=darkwhite
      newtext=darkgreen
      oldtext=darkred
      diffstuff=darkcyan
      cvsstuff=white
    '

  Add to git as well::

    git add -f ~/.colordiffrc

Emacs
=====

- Install::

    agi emacs

    yi emacs

Qt
==

- Install::

    agi qtbase5-dev qtbase5-examples

    yi qt5-qtbase-devel qt5-qtbase-examples

SDL
===

- Install::

    agi libsdl2-dev

uncrustify
==========

- Uncrustify source code beautifier:
  http://uncrustify.sourceforge.net/

- Install::

    agi uncrustify

    yi uncrustify

- Might also like UniversalIndentGUI (but it wasn't that great)::

    # [later]
    agi universalindentgui

Django
======

- Install::

    agi python-django

    yi python-django{,-doc}

Checkinstall
============

- Builds custom package, instead of just using ``make install``.

- Reference: https://help.ubuntu.com/community/CheckInstall

- Install::

    agi checkinstall

- After building a package, use ``sudo checkinstall`` instead of ``sudo make
  install`` to build a .deb file.

boost
=====

- Install::

    agi libboost-all-dev

    yi boost-devel

******
Markup
******

reStructuredText
================

- Install::

    agi python-docutils python-pygments python-sphinx

    yi python-docutils python-pygments python-sphinx

pandoc
======

- Markup conversions (e.g., Mediawiki to reStructuredText).

- Install::

    agi pandoc

    yi pandoc

- [optional] Install using cabal for latest version (Fedora 17 is too old)::

    sudo cabal update
    sudo cabal install pandoc

html2text
=========

- Install::

    agi html2text

linuxdoc-tools
==============

- For generating documentation (a.k.a. sgmltools on some distros).

- Install::

    agi linuxdoc-tools{,-latex,-text}

    yi linuxdoc-tools

YAML/JSON
=========

- Install YAML tooling::

    agi python-yaml python-yaml-dbg

    yi PyYAML

- Install JSON tooling::

    agi python-simplejson

    yi python-simplejson


**********
Multimedia
**********

Camera
======

- Create mount point::

    sudo mkdir /mnt/camera

- Setup mount point::

    echod -a /etc/fstab '
    LABEL=CAMERA /mnt/camera  vfat shortname=lower,user,noauto 0 0
    '

- Use gparted to label the volume.

Adobe Flash
===========

[Ubuntu]
--------

  - Requires ``extras`` to be enabled.

  - Install::

      agi flashplugin-installer

  - Restart the browser.

[Fedora]
--------

- Instructions from:
  http://www.if-not-true-then-false.com/2010/install-adobe-flash-player-10-on-fedora-centos-red-hat-rhel/

- Install Adobe yum repository and key::

    rpm -ivh http://linuxdownload.adobe.com/adobe-release/adobe-release-x86_64-1.0-1.noarch.rpm
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-adobe-linux

- Update repository list::

    dnf check-update

- Install flash itself::

    yi flash-plugin nspluginwrapper alsa-plugins-pulseaudio libcurl

- Verify installation via ``about:plugins`` URL in browser.  Should
  see ``application/x-shockwave-flash`` handled.

PDF viewers
===========

- Install various PDF viewers::

    agi xpdf zathura

    yi xpdf zathura zathura-plugins-all

Amarok
======

- Install::

    agi amarok

    yi amarok

- Since xdg directories are setup, automatically finds ``~/music``.

- Launch ``amarok`` and allow scanning of music directory.

- Settings | Configure Amarok | Notifications:

  - Check Use On-Screen-Display.

  - Drag display to desired location on screen.

  - Duration: 1000ms

  - Font scale: 75%

digikam
=======

- Install::

    agi digikam

devede
======

- Create DVDs from video files.

- Install::

    agi devede

**IMPORTANT** Change the default video format from PAL to NTSC!

Geeqie image viewer
===================

- Install::

    agi geeqie

    yi geeqie

- Run ``geeqie`` and configure via Edit | Preferences | Preferences (though at
  present, nothing to configure).

The GIMP
========

- Install::

    agi gimp

    yi gimp

Xpaint
======

- Install::

    agi xpaint

Pithos pandora client
=====================

- Install::

    agi pithos

    yi pithos

Video download
==============

- Install rtmpdump for saving video files::

    agi rtmpdump

    yi rtmpdump

- Sample invocation::

    rtmpdump -r 'rtmpe://video.infoq.com/cfx/st/presentations/11-sep-simplemadeeasy.mp4' -o simple_made_easy.mp4

- Instructions for figuring out the information:
  http://sandarenu.blogspot.com/2009/07/download-presentation-videos-from-infoq.html

  In essence:

  1. Open the site with the video you want to download, eg:
     http://www.infoq.com/presentations/Fowler-North-Crevasse-of-Doom

  2. Click on the play button and wait for the video to start.

  3. Stop the video and right click on it. In the Flash Player's context menu
     should be an entry named "Copy Log".

  4. After clicking on "Copy Log" you have some log entries on your clipboard.
     Paste these into a text editor of your choice. The entries look like this::

      22:09:42:333 --:--:-- NetConnection.Connect.Success ( rtmp://sxrl3x941kr4c.cloudfront.net/cfx/st/ )
      22:09:43:222 --:--:-- NetStream.Play.Reset ( presentations/08-feb-yawningcrevasseofdoom )
      22:09:43:223 --:--:-- NetStream.Play.Start ( presentations/08-feb-yawningcrevasseofdoom )
      22:09:43:224 --:--:-- NetConnection.URI ( rtmp://sxrl3x941kr4c.cloudfront.net:1935/cfx/st )
      22:09:43:224 --:--:-- NetStream.First.Byte
      22:09:45:232 00:00:00 NetStream.Buffer.Full
      22:09:46:920 00:00:02 NetStream.Pause.Notify
      22:09:46:920 00:00:02 NetStream.Buffer.Flush

  5. The essential parts are in the first and second line:
     rtmp://sxrl3x941kr4c.cloudfront.net/cfx/st/ and
     presentations/08-feb-yawningcrevasseofdoom We'll need those in Step 8.

  6. I've used the tool rtmpdump (http://rtmpdump.mplayerhq.hu/) to finally
     download the video. Maybe you could also use VLC but it didn't work for me.

  7. If you're installing rtmpdump on Linux, simply download the sources, run
     "make SYS=posix" and you're ready to go.

  8. Run "./rtmpdump -r rtmp://sxrl3x941kr4c.cloudfront.net/cfx/st/ -y
     presentations/08-feb-yawningcrevasseofdoom -o video.flv"

  9. Done. :-) Verify that the video is working with your favorite video player
     (I used mplayer).


recordMyDesktop
===============

**NOTE** No success with this tool

- Install::

    agi gtk-recordmydesktop

    yi qt-recordmydesktop

- Invocation::

    qt-recordMyDesktop

    gtk-recordMyDesktop

Pulseaudio tools
================

- Install volume control::

    agi pavucontrol

    yi pavucontrol

- Install preferences tool::

    agi paprefs

    yi paprefs

- Install volume meter::

    agi pavumeter

    yi pavumeter

Audacity
========

- Install::

    agi audacity

    yi audacity

mpg321
======

- Install::

    agi mpg321

sox
===

- Install::

    agi sox

    yi sox

ffmpeg
======

- Install::

    agi ffmpeg

    yi ffmpeg

- Converting a .mp4 file to better compression in two passes::

    ffmpeg -i infile.mp4 -strict -2 -vcodec libx264 -ac 1 -b:v 500k -bt 500k -pass 1 -passlogfile logfile.log outfile.mp4

    ffmpeg -i infile.mp4 -strict -2 -vcodec libx264 -ac 1 -b:v 500k -bt 500k -pass 2 -passlogfile logfile.log outfile.mp4

- Transcoding to make a video fit for devede (2 hours is too long for the
  version that I have on Ubuntu 16.04)::

    ffmpeg -i 1021_20180619000000.ts -target ntsc-dvd 1021_20180619000000.mpg

[Ubuntu] [alternative] custom build from source
-----------------------------------------------

Custom build for ffmpeg.

- Build dependencies::

    apt-get build-dep ffmpeg

- Acquire and build::

    cd ~/build
    git clone git://git.videolan.org/ffmpeg.git
    cd ffmpeg
    ./configure
    make

- Build the ``cws2fws`` tool (uncompresses .swf files)::

    make tools/cws2fws

- Execute::

    tools/cws2fws input.swf output.swf

swftools
========

- Site: http://www.swftools.org/

- Install::

    agi swftools

    yi swftools

[Ubuntu]
--------

- Optionally build from source.

- Dependencies::

    agi zlib1g-dev libgif-dev libfreetype6-dev libjpeg62-dev

- Acquire from git repository (but newest doesn't build)::

    cd ~/build
    git clone git://git.swftools.org/swftools

- Acquire and build released version::

    cd ~/build
    wget http://swftools.org/swftools-0.9.1.tar.gz
    tar -zxf swftools-0.9.1.tar.gz

- Build::

    cd swftools-0.9.1
    ./configure
    make

- Install::

    sudo checkinstall

[all]
-----

- Convert .swf file::

    swfrender file.swf -o file.png

OpenCV
======

- Install, with Python bindings::

    agi python-opencv

    yi opencv opencv-python

- Documentation:
  http://opencv.itseez.com/index.html

- Python bindings documentation:
  http://opencv.willowgarage.com/documentation/python/index.html

- Example using Python bindings:
  http://web.michaelchughes.com/how-to/watch-video-in-python-with-opencv

- StackOverflow question about generating movie from images:
  http://stackoverflow.com/questions/753190/programmatically-generate-video-or-animated-gif-in-python

- Fancy video effects tutorial:
  http://www.neuroforge.co.uk/index.php/input-and-output-with-open-cv

- Video Codecs:
  http://opencv.willowgarage.com/wiki/VideoCodecs

- StackOverflow question demonstrating how to overlay:
  http://stackoverflow.com/questions/5890738/overlaying-video-with-ffmpeg

  - Expand the left video to the final video dimensions::

      ffmpeg -i left.avi -vf "pad=640:240:0:0:black" left_wide.avi

  - Overlay the right video on top of the left one::

      ffmpeg -i left_wide.avi -vf "movie=right.avi [mv]; [in][mv] overlay=320:0" combined_video.avi

  - Question was about frame rate mis-match; should make sure input files
    have same fps.

avisynth
========

- Reference: http://avisynth.org/mediawiki/Main_Page

Screen Casting
==============

- The best tool so far that seems to work is ``ffmpeg``, though there is a bit
  of manual effort to determine the coordinates.

- This site explains options for Ubuntu screencasting:
  http://ubuntuguide.org/wiki/Screencasts

- The key point is to run ``pavucontrol`` (or ``vucontrol``), then begin a
  capture with your chosen software (``ffmpeg``, for example), then setup the
  recording device in ``pavucontrol``:

  - Make sure "Input Devices" includes "Monitor of Internal Audio Analog
    Stereo", and set this as the default.

  - Bring up "Recording" tab, and let it sit there while you begin a recording
    session with some application.

  - Once application is recording, the Recording tab should allow selection of
    the "Monitor of Internal Audio Analog Stereo" channel.

  - This should be remembered until next logout (or something like that).

- Presuming a rectangle at (1920, 0), with dimensions 980x500, the following
  invocation will record video and audio to an .mp4 file::

    ffmpeg -t 01:03:00 -f alsa -i pulse -f x11grab -s 980x500 -sameq -i :0.0+1920,0 screencast.mp4

  The ``-i`` switch limits the recording time.

mplayer
=======

- Install::

    agi mplayer mplayer-gui

    yi mplayer mplayer-gui gecko-mediaplayer mencoder

- [Ubuntu] Configure gnome-mplayer:

  - Launch via ``gnome-mplayer``.

  - Edit | Preferences (to configure).

- [Fedora] Configure mplayer-gui:

  - Launch via ``gmplayer``; don't use ``&`` to background it; seems to cause
    screw-ups.

  - Right-click window | Preferences | Misc:

    - Set DVD device to ``/dev/sr0``.

    - Set CD-ROM device to ``/dev/sr0``.

  - Execute mplayer GUI via ``gmplayer``.

vlc
===

- Video LAN Controller.  Playback and ripping of videos, DVDs, etc.

- Install vlc::

    agi vlc

    yi vlc vlc-extras

- Configure vlc:

  - Launch via ``vlc``.

  - Tools | Preferences | Input & Codecs:

    - Set Default optical device to ``/dev/sr0``.

MIDI playback
=============

- Reference: https://wiki.winehq.org/MIDI#Selecting_the_Output_-_the_MIDI_mapper

- Install timidity::

    agi timidity

- Verify MIDI ports are not yet available (run as normal user)::

    $ aconnect -o
    client 14: 'Midi Through' [type=kernel]
        0 'Midi Through Port-0'

- Original permissions on /etc/timidity::

    # ll -d /etc/timidity/
    drwxr-xr-x 2 root root 4096 Sep 18 09:43 /etc/timidity/

- Need write access to create /etc/timidity/.config/ when running as timidity
  user::

    sudo chown -h timidity /etc/timidity

- Restart the daemon::

    sudo systemctl restart timidity

- Verify timidity ports are available (run as normal user)::

    $ aconnect -o
    client 14: 'Midi Through' [type=kernel]
        0 'Midi Through Port-0'
    client 128: 'TiMidity' [type=user]
        0 'TiMidity port 0 '
        1 'TiMidity port 1 '
        2 'TiMidity port 2 '
        3 'TiMidity port 3 '

Sound converter
===============

- For converting sound and ripping CDs.

- Install::

    agi soundconverter

    yi soundconverter

- Install transcode (used for k3b-based ripping)::

    agi transcode

    yi transcode

- Software for ripping:
  https://help.ubuntu.com/community/CDRipping

abcde
=====

- A Better CD Encoder.  For ripping CDs.

- Install ``abcde``::

    agi abcde

    yi abcde

frei0r video plugins
====================

- Install::

    agi frei0r-plugins

    yi frei0r-plugins

kdenlive
========

- Insert clips, create video.

- Can extract audio, then re-import and render to new video file.

- Install::

    agi kdenlive

    yi kdenlive

xine
====

- Install::

    agi xine-ui

    yi xine

openshot
========

- Insert clips, create video.

- Install::

    agi openshot

    yi openshot

pitivi
======

- For video editing.

- Install::

    agi pitivi

avidemux
========

- Useful for trimming length of a video.

- Install::

    # From GetDeb repository:
    agi avidemux2.6-qt

    yi avidemux

Webcam video capture
====================

- Install cheese::

    agi cheese

- Install guvcview::

    agi guvcview

DVD burning
===========

k3b
---

- Install::

    agi k3b

brasero
-------

- Install::

    agi brasero

  **NOTE** Edit | Plugins, then uncheck ``File Checksum`` and ``Image Checksum``
  plugins (they take a very long time and don't do anything useful).

xfburn
------

- Install::

    agi xfburn

growisofs
---------

- Quite possibly already installed as a dependency of GUI-based burning tools.

- Install::

    agi growisofs

- Invoke::

    growisofs -dvd-compat -Z /dev/sr0=myimage.iso

*****
Games
*****

Maelstrom
=========

- Install::

    agi maelstrom

    yi Maelstrom

PySol Fan-Club edition
======================

- Install::

    agi pysolfc pysolfc-cardsets

    yi PySolFC{,-music,-cardsets}

Chess games
===========

- Install::

    agi xboard eboard dreamchess gnome-games knights

    yi xboard pychess knights

term2048
========

The "2048" game in your terminal.

- Install::

    pip install term2048

****
Math
****

speedcrunch
===========

- Install::

    agi speedcrunch

    yi speedcrunch

Reportlab
=========

- Install::

    agi python-reportlab{,-accel,-accel-dbg,-doc}

    yi python-reportlab python-reportlab-docs

- Browse documentation:
  http://www.reportlab.com/software/documentation/

Maxima
======

- Install::

    agi wxmaxima

    yi wxmaxima

Sage
====

- [Ubuntu]:

  - https://help.ubuntu.com/community/SAGE

  - Setup PPA::

      apt-add-repository -y ppa:aims/sagemath
      apt-get update

- Install::

    # Note: VERY SLOW MIRROR.
    agi sagemath-upstream-binary

    yi sagemath

*********
Emulation
*********

VirtualBox
==========

- Oracle instructions from:
  http://www.virtualbox.org/wiki/Linux_Downloads

[Ubuntu]
--------

- Instructions:

  - https://help.ubuntu.com/community/VirtualBox

  - https://askubuntu.com/questions/1029198/skipping-acquire-of-configured-file-contrib-binary-i386-packages-as-repository

  - https://askubuntu.com/questions/41478/how-do-i-install-the-virtualbox-version-from-oracle-to-install-an-extension-pack/41487#41487

- Add Oracle repository (with public key::

    add-apt-repository "deb [arch=amd64] https://download.virtualbox.org/virtualbox/debian $(lsb_release -cs) contrib"

    curl https://www.virtualbox.org/download/oracle_vbox_2016.asc | apt-key add -

    apt-get update

- Install virtualbox and extension pack::

    agi virtualbox-5.2

  **NOTE** virtualbox-ext-pack is apparently an Ubuntu-provided package that
  should not be mixed with virtualbox installed directly from Oracle's
  repository.

- For information on installing the extension pack:

  - https://askubuntu.com/questions/754815/can-i-install-the-virtualbox-extension-pack-from-the-ubuntu-repositories

- Based on the above, this will upgrade the extension pack::

    VBOXVERSION=`VBoxManage --version | sed -r 's/([0-9])\.([0-9])\.([0-9]{1,2}).*/\1.\2.\3/'`
    wget -q -N "http://download.virtualbox.org/virtualbox/$VBOXVERSION/Oracle_VM_VirtualBox_Extension_Pack-$VBOXVERSION.vbox-extpack"
    VBoxManage extpack install --replace Oracle*.vbox-extpack

    # Wait for license acceptance.

    rm Oracle*.vbox-extpack

- Verify extension pack::

    VBoxManage list extpacks

[Fedora]
--------

- Instructions from:
  http://www.if-not-true-then-false.com/2010/install-virtualbox-with-yum-on-fedora-centos-red-hat-rhel/

- Setup Oracle repository::

    cd /etc/yum.repos.d/
    wget http://download.virtualbox.org/virtualbox/rpm/fedora/virtualbox.repo

- Install prerequisites::

    yi gcc kernel-devel dkms

- Install::

    yi VirtualBox-5.0

- Install extension pack:
  https://www.virtualbox.org/wiki/Downloads

  Oracle_VM_VirtualBox_Extension_Pack-5.0.14-105127.vbox-extpack

  - Stupidly, this is now separate from VirtualBox itself and so it requires
    a manual installation of the extension pack.

  - Save Oracle_VM_VirtualBox_Extension_Pack-5.0.14-105127.vbox-extpack to
    download directory.

  - *As root*, execute virtualbox directly on the extension pack::

      sudo virtualbox Oracle_VM_VirtualBox_Extension_Pack-5.0.14-105127.vbox-extpack

    Alternatively, *as root*, run VirtualBox, choose menu File | Preferences |
    Extensions, click on blue diamond and browse to .vbox-extpack file to
    install.

[all]
-----

- Add user(s) to vboxusers group::

    sudo gpasswd -a mike vboxusers

- Logout/login, or ssh localhost for group ``vboxusers`` to take effect.

- To run::

    virtualbox

- Configure:

  - Choose menu File | Preferences:

    - General | Default Machine Folder, choose ``~/vms``.

- For using physical hard drive (including USB flash drive):

  - Create a copy of the raw device node::

      sudo mkdir -p /xdev
      for i in a b c d e; do
        sudo cp -a /dev/sd$i /xdev/raw_sd$i
      done
      sudo chown -hR mike: /xdev

  - Create a raw VirtualBox disk::

      for i in a b c d e; do
      VBoxManage internalcommands createrawvmdk \
        -filename /xdev/raw_sd$i.vmdk \
        -rawdisk /xdev/raw_sd$i
      done

- Create new Virtual Machine using now-existing disk ``sde.vmdk``.

- Might have to disable VT-x bit for Windows 2000 guest to fix this
  blue screen:

  - ntoskrnl.exe KMODE_EXCEPTION_NOT_HANDLED at boot

  Didn't help to turn on I/O APIC (as had been suggested online).

Compacting virtual disk
-----------------------

- http://eight.wikia.com/wiki/How_To_Shrink_VirtualBox_Windows_8_VDI_File

- Run administrator cmd.exe and perform degragment operation::

    defrag C: /U /V /X

- Run ``sdelete`` tool from
  http://technet.microsoft.com/en-us/sysinternals/bb897443::

    sdelete -z

- Shutdown the VM, then at command prompt::

    VBoxManage modifyvdi $PWD/windows8.vdi compact

dosbox
======

- Install::

    agi dosbox

    yi dosbox

- Run a program::

    dosbox /path/to/program.exe

wine
====

- Install::

    agi wine-stable

    yi wine

- [optional] Work-around for invalid ``.local`` domains that aren't just for
  zeroconf/bonjour stuff::

    vim /etc/nsswitch.conf

  Remove ``mdns4_minimal [NOTFOUND=return]`` from ``hosts:`` section::

    #hosts:      files dns mdns4_minimal
    hosts:      files dns

  Note: historically, ``mdns4_minimal`` was installed only as a dependency of
  ``wine``; but these days, it's often installed by default and already a
  problem for ``.local`` domains.  Not sure where these instructions ought to
  live now.

- Run a program::

    wine /path/to/winprog.exe

vice (commodore emulator)
=========================

- Install::

    agi vice

    yi vice

Virtual Machines using virsh, KVM
=================================

TODO: Verify these instructions.

- References:

  - https://help.ubuntu.com/community/KVM/Installation
  - https://help.ubuntu.com/community/KVM/Networking
  -
  - https://help.ubuntu.com/community/BridgingNetworkInterfaces

- Install::

    agi qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils \
      virt-viewer virt-manager

- Allow ``mike`` to operate VMs::

    adduser mike libvirtd

- Create persistent bridge::

    agi bridge-utils

- Edit ``/etc/network/interfaces`` and make these changes::

    ## The primary network interface
    #auto eno1
    #iface eno1 inet dhcp

    auto br0
    iface br0 inet dhcp
            bridge_ports eno1
            bridge_stp off
            bridge_fd 0
            bridge_maxwait 0

- Restart networking::

    systemctl restart networking

- Install a test VM::

    virt-install \
      --connect qemu:///system \
      --virt-type kvm \
      --name kvmtest \
      --memory 2048 \
      --disk kvmtest.qcow2,size=10 \
      --cdrom ~/tmp/iso/ubuntu-16.04-server-amd64.iso \
      --os-variant ubuntuutopic

- View the VM as it runs::

    virt-viewer -c qemu:///system kvmtest
    virt-viewer -c qemu+ssh://host-or-ip/system kvmtest

- View all VMs::

    virsh list --all

- Start, stop a VM::

    virsh start VM

- To support clipboard integration, install the following in the Linux-based
  guest::

    agi spice-vdagent



*********
Terminals
*********

tmux
====

- Install::

    agi tmux

    yi tmux

- Create ``~/.tmux.conf`` with contents::

    set-option -g prefix C-j
    unbind-key C-b
    bind-key C-j send-prefix
    bind-key -r Space next-layout

    # Global options.
    set-option -g default-terminal screen-256color

    # Enable extra key code for shifted functions keys, xterm-style.
    set-option -g xterm-keys on
    set-option -g display-panes-time 1500
    set-option -g display-time 1500
    set-option -g history-limit 100000
    set-option -g repeat-time 750
    set-option -g set-titles on

    # Server options.
    set-option -s escape-time 5

GNU Screen
==========

- Install::

    agi screen

    yi screen

- Create ``~/.screenrc`` with contents::

    # Change default ctrl-A to ctrl-@ for escape key.
    escape ^Jj

    # Kill unnecessary license message.
    startup_message off

GNOME Terminal
==============

- Install::

    agi gnome-terminal

rxvt
====

- Install::

    agi rxvt rxvt-unicode-256color

    yi rxvt rxvt-unicode

terminator
==========

- Install::

    agi terminator

    yi terminator

tack (terminfo action checker)
==============================

- Install::

    agi tack

    yi tack

************
Basic server
************

Setup /srv
==========

- [SERVER] Pull ``/srv`` from ``/m/srv``::

    rmdir /srv
    ln -s /m/srv /srv

John the Ripper password cracker
================================

- Install::

    agi john

    yi john

NTP for date and time
=====================

[Ubuntu]
--------

- Configure firewall::

    # [server].
    ufw allow ntp

- Install NTP server and documentation::

    agi ntp ntp-doc ntpdate

- Defaults to using ``*.ubuntu.pool.ntp.org`` as time servers; configure if
  desired::

    vim /etc/ntp.conf

[CentOS]
--------

- Enable and start::

    systemctl enable ntpd
    systemctl start ntpd

ipmi-related tools
==================

- Install::

    agi ipmitool freeipmi-tools

*********************
Certificate Authority
*********************

References
==========

- modssl.org ssl_faq.
- http://www.tc.umn.edu/~brams006/selfsign.html
- https://help.ubuntu.com/lts/serverguide/certificates-and-security.html
- https://gist.github.com/Soarez/9688998
- https://security.stackexchange.com/questions/165559/why-would-i-choose-sha-256-over-sha-512-for-a-ssl-tls-certificate
- https://askubuntu.com/questions/73287/how-do-i-install-a-root-certificate

Creating a Certificate Authority
================================

Certificates are stored permanently in ``/m/sys/etc/certs``, and
copied elsewhere as necessary.

Commands below are therefore run in ``/m/sys/etc/certs``.

**Note** Keep the name ``ca`` for the certificate authority so the signing
script will work.

- Use SHA-256 for hashing.

- Move to CA directory::

    cd /m/sys/etc/certs

- Create CA private key using strong password::

    openssl genrsa -des3 -out ca.key 4096

  Use standard strong password.

- View key just created::

    openssl rsa -noout -text -in ca.key

- Create self-signed CA Certificate (stored in X509 structure)
  using RSA key of the CA.  Output will be PEM-formatted::

    openssl req -new -sha256 -x509 -days 1825 \
      -key ca.key -out ca.crt

  Answer questions as follows:

  - Country code::
      US
  - State::
      Maryland
  - Locality::
      Severn
  - Organization name::
      DOMAIN CA
  - Organizational unit: <leave blank>

  - Common Name::

      DOMAIN CA

  - Email Address::
      USER@DOMAIN.com

- View details of new ca.crt file as follows::

    openssl x509 -noout -text -in ca.crt

- The ``sign.sh`` script (found at the bottom of this file) originally came from
  the openssl distribution (in the ``pkg.contrib/`` directory).  It's hard to
  find now.

  Copy this script into working directory.  Ensure ``ca.config`` file settings
  inside script include ``unique_subject = no`` to enable renewal of certs.

``DOMAIN CA`` is now ready to sign certificates, e.g.::

  ./sign.sh server.csr

Publish Certificate Authority
=============================

- Move to CA directory::

    cd /m/sys/etc/certs

- [ubuntu] Copy CA certificate and update::

    cp ca.crt /usr/local/share/ca-certificates/DOMAIN-ca.crt
    update-ca-certificates

sign.sh script
==============

Below is the modified ``sign.sh`` script.

Modifications:

- Change ``default_md`` from ``md5`` to ``sha256``.

Source::

  #!/bin/sh
  ##
  ##  sign.sh -- Sign a SSL Certificate Request (CSR)
  ##  Copyright (c) 1998-2001 Ralf S. Engelschall, All Rights Reserved.
  ##

  #   argument line handling
  CSR=$1
  if [ $# -ne 1 ]; then
      echo "Usage: sign.sign <whatever>.csr"; exit 1
  fi
  if [ ! -f $CSR ]; then
      echo "CSR not found: $CSR"; exit 1
  fi
  case $CSR in
     *.csr ) CERT="`echo $CSR | sed -e 's/\.csr/.crt/'`" ;;
         * ) CERT="$CSR.crt" ;;
  esac

  #   make sure environment exists
  if [ ! -d ca.db.certs ]; then
      mkdir ca.db.certs
  fi
  if [ ! -f ca.db.serial ]; then
      echo '01' >ca.db.serial
  fi
  if [ ! -f ca.db.index ]; then
      cp /dev/null ca.db.index
  fi

  #   create an own SSLeay config
  cat >ca.config <<EOT
  [ ca ]
  default_ca              = CA_own
  [ CA_own ]
  dir                     = .
  certs                   = \$dir
  new_certs_dir           = \$dir/ca.db.certs
  database                = \$dir/ca.db.index
  serial                  = \$dir/ca.db.serial
  RANDFILE                = \$dir/ca.db.rand
  certificate             = \$dir/ca.crt
  private_key             = \$dir/ca.key
  default_days            = 365
  default_crl_days        = 30
  default_md              = sha256
  preserve                = no
  policy                  = policy_anything
  unique_subject          = no
  [ policy_anything ]
  countryName             = optional
  stateOrProvinceName     = optional
  localityName            = optional
  organizationName        = optional
  organizationalUnitName  = optional
  commonName              = supplied
  emailAddress            = optional
  EOT

  #  sign the certificate
  echo "CA signing: $CSR -> $CERT:"
  openssl ca -config ca.config -out $CERT -infiles $CSR
  echo "CA verifying: $CERT <-> CA cert"
  openssl verify -CAfile ca.crt $CERT

  #  cleanup after SSLeay
  rm -f ca.config
  rm -f ca.db.serial.old
  rm -f ca.db.index.old

  #  die gracefully
  exit 0


***********
File server
***********

NFS server
==========

- NFS HOWTO:
  http://nfs.sourceforge.net/nfs-howto/ar01s06.html

- https://www.peterbeard.co/blog/post/setting-up-iptables-for-nfs-on-ubuntu/

- Ubuntu server guide for NFS:
  https://help.ubuntu.com/stable/serverguide/network-file-system.html

- Ubuntu NFS setup instructions:
  https://help.ubuntu.com/community/SettingUpNFSHowTo

- Arch Wiki NFS page:
  https://wiki.archlinux.org/index.php/NFS

- Fedora wiki:
  http://fedoraproject.org/wiki/User:Renich/HowTo/NFSv4

- CentOS info:
  https://www.howtoforge.com/nfs-server-and-client-on-centos-7

Ubuntu
------

- Debian-specific help from Securing NFS page:
  http://wiki.debian.org/SecuringNFS

- Install NFS server (includes client-side tools as well)::

    # [server]
    agi nfs-kernel-server

- Unfortunately, we need to lock down the ports so the firewall rules will work.
  To examine the currently chosen ports::

    rpcinfo -p

  With output::

    program vers proto   port  service
     100000    4   tcp    111  portmapper
     100000    3   tcp    111  portmapper
     100000    2   tcp    111  portmapper
     100000    4   udp    111  portmapper
     100000    3   udp    111  portmapper
     100000    2   udp    111  portmapper
     100005    1   udp  50455  mountd
     100005    1   tcp  53806  mountd
     100005    2   udp  43866  mountd
     100005    2   tcp  60395  mountd
     100005    3   udp  41739  mountd
     100005    3   tcp  47624  mountd
     100003    2   tcp   2049  nfs
     100003    3   tcp   2049  nfs
     100003    4   tcp   2049  nfs
     100227    2   tcp   2049
     100227    3   tcp   2049
     100003    2   udp   2049  nfs
     100003    3   udp   2049  nfs
     100003    4   udp   2049  nfs
     100227    2   udp   2049
     100227    3   udp   2049
     100021    1   udp  54335  nlockmgr
     100021    3   udp  54335  nlockmgr
     100021    4   udp  54335  nlockmgr
     100021    1   tcp  37346  nlockmgr
     100021    3   tcp  37346  nlockmgr
     100021    4   tcp  37346  nlockmgr

- The sunrpc(111) and nfs(2049) ports are fixed already, but mountd and nlockmgr
  are random and need to be locked down.  We choose the following port numbers
  for these purposes:

  - 2047  fs.nfs.nlm_tcpport/fs.nfs.nlm_udpport (for nlockmgr)
  - 2048  RPCMOUNTDOPTS (for mountd port)

- Lock down the kernel's nlockmgr ports::

    echod -o /etc/sysctl.d/30-nfs-ports.conf '
      fs.nfs.nlm_tcpport = 2047
      fs.nfs.nlm_udpport = 2047
    '

  And activate::

    sysctl --system

- Lock down the mountd port::

    vim /etc/default/nfs-kernel-server

    # RPCMOUNTDOPTS="--manage-gids"
    RPCMOUNTDOPTS="--manage-gids --port 2048"

  And restart nfs::

    systemctl restart nfs-config nfs-server

- Verify the port numbers are now fixed::

    rpcinfo -p

  With output::

     program vers proto   port  service
      100000    4   tcp    111  portmapper
      100000    3   tcp    111  portmapper
      100000    2   tcp    111  portmapper
      100000    4   udp    111  portmapper
      100000    3   udp    111  portmapper
      100000    2   udp    111  portmapper
      100005    1   udp   2048  mountd
      100005    1   tcp   2048  mountd
      100005    2   udp   2048  mountd
      100005    2   tcp   2048  mountd
      100005    3   udp   2048  mountd
      100005    3   tcp   2048  mountd
      100003    2   tcp   2049  nfs
      100003    3   tcp   2049  nfs
      100003    4   tcp   2049  nfs
      100227    2   tcp   2049
      100227    3   tcp   2049
      100003    2   udp   2049  nfs
      100003    3   udp   2049  nfs
      100003    4   udp   2049  nfs
      100227    2   udp   2049
      100227    3   udp   2049
      100021    1   udp   2047  nlockmgr
      100021    3   udp   2047  nlockmgr
      100021    4   udp   2047  nlockmgr
      100021    1   tcp   2047  nlockmgr
      100021    3   tcp   2047  nlockmgr
      100021    4   tcp   2047  nlockmgr

- Configure firewall::

    ufw allow nfs
    ufw allow sunrpc
    ufw allow 2047/udp
    ufw allow 2047/tcp
    ufw allow 2048/udp
    ufw allow 2048/tcp

- Verify::

    showmount -e SERVER

Fedora, CentOS
--------------

- Install packages::

    yi nfs-utils

- Configure firewall::

    firewall-cmd --permanent --zone=public --add-service=nfs
    firewall-cmd --permanent --zone=public --add-service=mountd
    firewall-cmd --permanent --zone=public --add-service=rpc-bind
    firewall-cmd --reload

- Enable service to start automatically, start now::

    systemctl enable  nfs-server
    systemctl restart nfs-server

All
---

- Create exports file::

    vim /etc/exports

    /m      1.2.3.0/24(rw,insecure,async,no_root_squash,no_subtree_check)
    /home   1.2.3.0/24(rw,insecure,async,no_root_squash,no_subtree_check)

- Re-export all filesystems::

    exportfs -r

- Verify that exported filesystems are available::

    showmount -e SERVER

    Export list for SERVER:
    /home 1.2.3.0/24
    /m    1.2.3.0/24

Samba
=====

- [Fedora] [CentOS] See manpage for Samba SELinux::

    man samba_selinux

- Install Samba::

    # [server]
    agi samba smbclient

    # [server]
    yi samba system-config-samba-docs samba-client

- Configure firewall::

    ufw allow samba

    firewall-cmd --add-service=samba
    firewall-cmd --permanent --add-service=samba
    firewall-cmd --add-service=samba-client
    firewall-cmd --permanent --add-service=samba-client

- Backup original configuration::

    cp -a /etc/samba/smb.conf{,.dist}

- Setup the following globals::

    vim /etc/samba/smb.conf

    [global]
       workgroup = TOYLAND
       netbios name = SERVER
       security = user
       hosts allow = 127. 1.2.3.
       preserve case = yes
       short preserve case = yes

       # With wide links = no (default), symbolic links won't be
       # followed outside the share root.
       # We need wide links = yes for /m/cifs/ symbolic links to work.
       # We also must turn off unix extensions when using wide links.

       wide links = yes
       unix extensions = no

       # If on corporate network, uncomment these lines:
       #   domain master = no
       #   local master = no
       #   preferred master = no
       #   os level = 0

- Setup shares at bottom of the configuration file::

    vim /etc/samba/smb.conf

    [M_DRIVE]
        comment = M Drive
        path = /m/cifs/m
        valid users = LIST OF USERS
        force group = data
        read only = No
        create mask = 0770
        force create mode = 0660
        directory mask = 0770
        force directory mode = 0770

    [S_DRIVE]
        comment = S Drive
        path = /m/cifs/s
        valid users = LIST OF USERS
        force group = data
        read only = No
        create mask = 0770
        force create mode = 0660
        directory mask = 0770
        force directory mode = 0770

    [ISO]
        comment = Mounted .iso image
        path = /mnt/iso
        valid users = LIST OF USERS
        force group = data
        read only = Yes

    [snapshot]
        comment = Snapshot backups
        path = /snapshot
        valid users = LIST OF USERS
        force group = data
        read only = Yes

- [Fedora] [CentOS] Ensure Samba shares have the correct SELinux file context::

    for i in /m /snapshot /mnt/iso; do
      semanage fcontext -a -t public_content_rw_t -r s0 "$i(/.*)?"
      restorecon -R $i
    done

  Note the use of ``-r s0`` to set the sensitivity level; this avoids the error
  ``libsepol.mls_from_string: invalid MLS context None (No such file or
  directory).``.

- [Fedora] [CentOS] Ensure Samba can write to anonymous ``public_content_t``
  files::

    setsebool -P smbd_anon_write 1

- Ensure ISO mount point is available::

    mkdir -p /mnt/iso

- Configure users and passwords::

    # One at a time:
    smbpasswd -a mike

- To view existing users::

    pdbedit -L

  To remove existing user::

    smbpasswd -x user

- [Ubuntu] Restart Samba::

    systemctl enable smbd
    systemctl start smbd

- [Fedora] Enable and start Samba::

    systemctl enable smb
    systemctl start smb

- [Fedora] Configure SELinux to allow exportation of home directories::

    setsebool -P samba_enable_home_dirs 1

- [Fedora] Configure SELinux to permit Samba to share everything read/write:

  Using ``setsebool``::

    setsebool -P samba_export_all_rw on

  Alternatively, the following GUI option may be used:

  - Run SELinux configuration tool::

      system-config-selinux

  - Select "Boolean".

  - In the Filter box, enter ``samba``.

  - Check ``samba_export_all_rw``.

External Hard Drive Backup
==========================

Backup drives:

Western Digital Blue 1TB drives
-------------------------------


# To be blanked:
/dev/disk/by-id/ata-WDC_WD10EZEX-08M2NA0_WD-WMC3F2355119

Samsung 2TB
-----------


- Create mount points and mark them immutable::

    mkdir /mnt/backup{1,2,3,4,5,6}
    chattr +i /mnt/backup{1,2,3,4,5,6}

- Create entries for the backups::

    vim /etc/fstab

    # 1 TB Western Digital WD1002FBYS (Silver in color)
    # /dev/disk/by-id/ata-WDC_WD1002FBYS-01A6B0_WD-WMATV0831115
    # /dev/disk/by-uuid/71c0d703-a93d-425a-9b84-11cdeb7e8374
    UUID="71c0d703-a93d-425a-9b84-11cdeb7e8374"     /mnt/backup1    ext4    defaults,noauto 0 0

    # 2 TB Samsung HD204UI
    # /dev/disk/by-id/ata-SAMSUNG_HD204UI_S2H7J90B712546
    # /dev/disk/by-uuid/74d2d33c-8a16-492e-ae58-a42ce79817dd
    UUID="74d2d33c-8a16-492e-ae58-a42ce79817dd"     /mnt/backup2    ext4    defaults,noauto 0 0

    # 1 TB Western Digital WD10EZEXSP (Blue in color)
    # /dev/disk/by-id/ata-WDC_WD10EZEX-08M2NA0_WD-WMC3F2355119
    # /dev/disk/by-uuid/e3948756-527c-4e1e-a3f3-ad1f629ed192
    UUID="e3948756-527c-4e1e-a3f3-ad1f629ed192"     /mnt/backup3    ext4    defaults,noauto 0 0

    # 1 TB Western Digital WD10EZEXSP (Blue in color)
    # /dev/disk/by-id/ata-WDC_WD10EZEX-22BN5A0_WD-WCC3F2851484
    # /dev/disk/by-uuid/ecb62152-395e-4ca0-8c91-2d978fe425b1
    UUID="ecb62152-395e-4ca0-8c91-2d978fe425b1"     /mnt/backup4    ext4    defaults,noauto 0 0

- For each mounted backup volume, ensure there is a backup directory::

    # Insert backup1 drive.
    mount /mnt/backup1
    mkdir /mnt/backup1/m
    umount /mnt/backup1
    # Remove backup1 drive.

    # Insert backup2 drive.
    mount /mnt/backup2
    mkdir /mnt/backup2/m
    umount /mnt/backup2
    # Remove backup2 drive.

- Ensure ``/m/sys/bin/backup-external`` is in crontab to run nightly::

    vim /etc/crontab

**********
Web server
**********

Apache
======

Apache Server Certificate
-------------------------

- Move to CA directory::

    cd /m/sys/etc/certs

- Create private key for server certificate::

    openssl genrsa -des3 -out DOMAIN.com.key 4096

- Remove encryption from this key as follows::

    openssl rsa -in DOMAIN.com.key \
        -out DOMAIN.com.key.unsecure

  This is necessary if you want to avoid typing in the web server's
  private key password when the server restarts.

- View the private key::

    openssl rsa -noout -text -in DOMAIN.com.key.unsecure

- Create a Certificate Signing Request (CSR) using server private key.
  Output will be PEM-formatted::

    openssl req -new -sha256 -key DOMAIN.com.key.unsecure \
        -out DOMAIN.com.csr

  Answer questions as follows:

  - Country code::
      US
  - State::
      Maryland
  - Locality::
      Severn
  - Organization name::
      DOMAIN
  - Organizational unit: <leave blank>

  - Common Name::

      *.DOMAIN.com

  - Email Address::
      USER@DOMAIN.com

  Skip any 'extra' attributes.

- View the CSR as follows::

    openssl req -noout -text -in DOMAIN.com.csr

- Sign this request using the CA::

    ./sign.sh DOMAIN.com.csr

  Creates DOMAIN.com.crt.

- View details of signed certificate::

    openssl x509 -noout -text -in DOMAIN.com.crt

Apache client certificate
-------------------------

- Move to CA directory::

    cd /m/sys/etc/certs

- Create private key for client certificate::

    openssl genrsa -des3 -out mike.key 4096

- View the private key::

    openssl rsa -noout -text -in mike.key

- Create a Certificate Signing Request (CSR) using server private key.
  Output will be PEM-formatted::

    openssl req -new -sha256 -key mike.key -out mike.csr

  Answer questions as follows:

  - Country code::
      US
  - State::
      Maryland
  - Locality::
      Severn
  - Organization name::
      DOMAIN
  - Organizational unit: <leave blank>

  - Common Name::

      Michael Richard Henry

  - Email Address::
      USER@DOMAIN.com

  Skip any 'extra' attributes.

- View the CSR as follows::

    openssl req -noout -text -in mike.csr

- Sign this request using the CA::

    ./sign.sh mike.csr

- Create PKCS12 certificate for browser importation::

    openssl pkcs12 -export -in mike.crt -inkey mike.key \
      -certfile ca.crt -name "Michael Richard Henry" -out mike.p12



[Ubuntu]
--------

- Configure firewall::

    ufw allow http
    ufw allow https

- Install Apache and associated libraries::

    agi apache2 apache2-doc

  Note: /manual URL doesn't show up until Apache restart

- Backup ``default`` and ``default-ssl`` sites::

    cp -a /etc/apache2/sites-available/000-default.conf{,.dist}
    cp -a /etc/apache2/sites-available/default-ssl.conf{,.dist}

- Create www root area (if necessary)::

    mkdir -p /srv/www/SERVER/htdocs

- Edit available sites::

    vim /etc/apache2/sites-available/{000-default.conf,default-ssl.conf}

  In ``default`` and ``default-ssl``, change ``DocumentRoot``::

        # DocumentRoot /var/www/html
        DocumentRoot /srv/www/SERVER/htdocs

- Change ``Directory`` location::

    vim /etc/apache2/apache2.conf

    # <Directory /var/www/>
    <Directory /srv/www/SERVER/htdocs/>

- Enable both ``default`` and ``default-ssl`` sites::

    a2ensite 000-default default-ssl

- Enable ``ssl`` module::

    a2enmod ssl

- Create directories for key and certificate::

    mkdir /etc/apache2/{certs,private}

- Copy key and cert files::

    cp /m/sys/etc/certs/DOMAIN.com.key.unsecure /etc/apache2/private
    cp /m/sys/etc/certs/DOMAIN.com.crt /etc/apache2/certs

- Configure certificate paths::

    vim /etc/apache2/sites-available/default-ssl.conf

    SSLCertificateFile    /etc/apache2/certs/DOMAIN.com.crt
    SSLCertificateKeyFile /etc/apache2/private/DOMAIN.com.key.unsecure

- Enable Apache to request a client certificate:

  - After publishing ``DOMAIN-ca.crt``, setup cert path::

      vim /etc/apache2/sites-available/default-ssl.conf

      SSLCACertificatePath /etc/ssl/certs

    And require a client certificate::

      SSLVerifyClient require
      SSLVerifyDepth 1

      **Note** This doesn't work yet on a per-location basis.  Running into a
      known bug in mod_ssl regarding use of 'SSLVerifyClient require' within a
      <Location> directive.

- Restart::

    systemctl restart apache2

- Place server certificate into ``htdocs`` area::

    cp /m/sys/etc/certs/ca.crt /srv/www/SERVER/htdocs/certs/DOMAIN-ca.crt

- Browse to install certificate authority::

    http://SERVER.DOMAIN.com/certs/

  Choose ``DOMAIN-ca.crt``.

- Verify server is trusted via https::

    https://SERVER.DOMAIN.com/

******************
Squid proxy server
******************

TODO: Verify these instructions.

[Ubuntu]
========

- http://wiki.xdroop.com/space/Linux/squid/Configuring+a+Yum+Cache
  contains an example configuration for yum proxying, and some
  hints:

  - Squid listens by default on port 3128.

  - To cache yum, edit ``/etc/yum.conf`` and add this line::

      proxy=http://$PROXY:3128

  - The default ``cache_dir`` limits cache to only 100 MB,
    so it must be changed.  The parameter is defined by::

      cache_dir ufs path_to_cache_directory Mbytes L1 L2

    - Mbytes: maximum disk space to use under this directory.

    - L1: Number of first-level directory entries.

    - L2: Number of second-level directory entries.

    Default is::

      cache_dir ufs /var/spool/squid 100 16 256

- Install::

    # [server]
    agi squid

- Backup configuration file::

    cp -a /etc/squid/squid.conf{,.dist}

- Stop squid (we'll be moving the cache directory)::

    service squid stop

- Adjust ``/etc/squid/squid.conf``::

    # Locate ``localnet`` definition and add ``clientnet``:
    acl localnet src 1.2.0.0/16 # RFC1918 possible internal network
    acl clientnet src 1.2.3.0/24

    # Uncomment this line:
    http_access deny to_localhost

    # Locate ``# INSERT YOUR OWN RULE(S) HERE`` and add:
    http_access allow clientnet

    # Locate ``http_port`` line, extend for more ports:
    # Squid normally listens to port 3128; use 1080 (SOCKS default) also.
    http_port 3128
    http_port 1080

    # Locate default line, customize below:
    # cache_dir ufs /var/spool/squid 100 16 256
    cache_dir ufs /m/tmp/squid 10000 16 256

    # Locate default line, customize below:
    # maximum_object_size 20480 KB
    maximum_object_size 800 MB

    # Insert these lines before the wildcard ``refresh_pattern .  `` line,
    # making sure to override related lines that may already exist:
    refresh_pattern (Release|Packages)(\.gz|\.bz2)?$        0       20%     2880
    # 1577846 = 3 years (in minutes)
    refresh_pattern (\.deb|\.udeb)$   1577846 100% 1577846
    refresh_pattern (\.rpm|\.drpm)$   1577846 100% 1577846
    # Existing wildcard pattern:
    refresh_pattern .              0       20%     4320

- Copy original cache directory to desired location::

    cp -a /var/spool/squid /m/tmp/squid

  Leave original in place to hold core dumps per the existing
  ``coredump_dir`` directive.

- Configure firewall::

    ufw allow 3128/tcp
    ufw allow 1080/tcp

- Restart squid::

    service squid restart

- Using Squid for caching Apt repository content:

  - Article on this concept:
    http://shadow-file.blogspot.com/2008/12/ditching-apt-cacher-ng-for-squid.html

  - Create file ``/etc/apt/apt.conf.d/squid`` with content::

      Acquire::http::Proxy "http://1.2.3.251:3128";

    **Note** Make sure there are no "Acquire" directives in any other apt
    configuration file below ``/etc/apt``, as some systems explicitly turn off
    proxy usage.

************
Media server
************

mediatomb
=========

- Serves DLNA content.

- Reference:
  http://mediatomb.cc/

Ubuntu
------

TODO: Verify these instructions.

- Install::

    agi mediatomb mediatomb-daemon

- Configure mediatomb::

    vim /etc/mediatomb/config.xml

  Change ``enabled`` to ``yes`` to turn on the user interface::

    <ui enabled="yes" show-tooltips="yes">

- Configure daemon::

    vim /etc/default/mediatomb

  Set options to configure the port::

    OPTIONS="--port 50500"

  Set interface::

    INTERFACE="eth0"

  Set group to ``data`` so mediatomb can read media files on SERVER::

    GROUP="data"

- Open firewall for TCP and UDP port MT_PORT and UDP port 1900 (for SSDP)::

    ufw allow 50500 1900/udp

- Restart the daemon::

    service mediatomb restart

Fedora
------

- Install::

    yi mediatomb

- Backup configuration file::

    cp -a /etc/mediatomb.conf{,.orig}

- Edit configuration file::

    vim /etc/mediatomb.conf

  Setup interface (based on ``ifconfig`` output, for example)::

    MT_INTERFACE="em1"

  Setup group to ``data`` so mediatomb can read media files on SERVER::

    MT_GROUP="data"

  Determine interface port (default value of ``50500`` should be OK)::

    MT_PORT="50500"

- Open firewall for TCP and UDP port MT_PORT and UDP port 1900 (for SSDP)::

    firewall-cmd --add-port 50500/tcp
    firewall-cmd --permanent --add-port 50500/tcp
    firewall-cmd --add-port 50500/udp
    firewall-cmd --permanent --add-port 50500/udp
    firewall-cmd --add-port 1900/udp
    firewall-cmd --permanent --add-port 1900/udp

- Enable and start service::

    systemctl enable mediatomb
    systemctl start mediatomb

All
---

- Keep an eye on the mediatomb log file::

    tailf /var/log/mediatomb
    tailf /var/log/mediatomb.log

- Visit the UI URL given in the log:
  http://1.2.3.251:50500/

- Choose "Filesystem", then navigate to ``/m/shared/music/Music`` and choose
  "add as autoscan dir" (the far-right "+" with a circle around it).
  This brings up a number of settings.

  - Scan Mode: Timed

  - Scan Level: Full

  - Recursive: <checked>

  - Scan Interval: 1800

  Then choose "Set" to activate the share.

**********
Git server
**********

- Install gitweb with dependencies::

    agi gitweb apache2

    yi gitweb httpd

- Consider these for later::

    # [later]
    agi git-email git-daemon-run gitosis

- For svn2git conversion::

    # [later]
    agi git-core git-svn ruby rubygems
    sudo gem install svn2git --source http://gemcutter.org

****
Misc
****

Uncategorized or new items.

- moreutils for ``errno`` and other utilities::

    agi moreutils
