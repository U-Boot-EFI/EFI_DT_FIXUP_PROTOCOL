U-Boot patches
==============

The patches were created against branch origin/master commit 5a1a8a63be8f.

To build the test tool dtbdump.efi you need to select::

    CONFIG_EFI_SELFTEST=y

Copy lib/efi_selftest/dtbdump.efi to a partition accessible from U-Boot.
Assuming that the tool is in the root directory off partition *host 0:1*
you can run it with::

    load host 0:1 $kernel_addr_r dtbdump.efi
    bootefi $kernel_addr_r

The tool can only read from and write to the same partition from which it
was loaded.

A dtbdump.efi session could look like::

    DTB Dump
    ========

    => load test.dtb
    device-tree installed
    => save fixed-up.dtb
    fixed-up.dtb written
    => exit
