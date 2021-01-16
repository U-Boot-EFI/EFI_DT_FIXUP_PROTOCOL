U-Boot
======

Configuration
-------------

The EFI_DT_FIXUP_PROTOCOL is enabled via::

    CONFIG_EFI_DT_FIXUP=y

The patches in this directory are already merged in U-Boot v2021.04. They are
applicable to U-Boot v2021.01.

dtbdump.efi
-----------

dtbdump.efi is a tool provided with U-Boot for testing the
EFI_DT_FIXUP_PROTOCOL.

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
