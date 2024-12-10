GRUB
----

Currently grub only loads the Linux kernel and the initial file system
before booting.

On the arm and arm64 architectures the device tree is also needed for
booting. The device tree is Linux kernel version dependent. So we can
not rely on the initial bootloader to provide the correct device tree
when calling grub.

0001-10_linux-support-loading-device-trees.patch
  The patch changes the 10_linux script to look in sequence for files
  ${dirname}/dtb-${version} and ${dirname}/dtb. If one of the files is
  present, a devicetree command

	devicetreee ${rel_dirname}/${dtb}

  is added to the Linux boot entry.

0001-efi-EFI-Device-Tree-Fixup-Protocol.patch
  With the patch the EFI_DT_FIXUP_PROTOCOL is used to fix-up device trees
  loaded via the devicetree command.
