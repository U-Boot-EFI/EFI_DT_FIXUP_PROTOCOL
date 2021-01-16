GRUB process overview
---------------------

GRUB's *devicetree* command is implemented by function grub_cmd_devicetree() in
grub-core/loader/efi/fdt.c.

At a later stage grub_fdt_load() is called by finalize_params_linux().
grub_fdt_load() uses either the device-tree loaded by function
grub_cmd_devicetree() or if none has been loaded the device-tree configuration
table and copies it to a new buffer. finalize_params_linux() applies fix-ups
and calls grub_fdt_install().

To make use of the EFI_DT_FIXUP_PROTOCOL we should invoke it with
EFI_DT_APPLY_FIXUPS in grub_cmd_devicetree() and with
EFI_DT_RESERVE_MEMORY | EFI_DT_INSTALL_TABLE in grub_fdt_install().
