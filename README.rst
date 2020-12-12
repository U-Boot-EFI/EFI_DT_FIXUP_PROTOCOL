.. SPDX-License-Identifier: CC-BY-ND-4.0
.. Copyright (c) 2020 Heinrich Schuchardt

EFI Device Tree Fixup Protocol Specification
============================================

Draft, 2020-12-12

Background
----------

Device-trees are used to convey information about hardware to the operating
system. Some of the properties are only known at boot time. (One example of such
a property is the number of the boot hart on RISC-V systems.) Therefore the
firmware applies fix-ups to the original device-tree. Some nodes and properties
are added or altered.

Typically the software support for hardware is introduced in small steps and not
in a big bang approach. The device-tree used to describe the hardware is
developped in parallel. Sometimes backwards compatibility is broken. New
versions of operating systems cannot boot with old device-trees or old
versions of operating systems cannot boot with new device-trees.

This leads to operating systems providing version specific device-trees. It can
be desirable that a boot manager (e.g. GRUB) can provide the device-tree that
best matches the chosen operating system. But the device-tree provided by the
operating system will lack the fix-ups that the firmware would apply.

Objective
---------

The objective of this specification is to define a UEFI protocol implemented by
firmware that a boot manager can call to apply fix-ups to device-trees.

EFI_DT_FIXUP_PROTOCOL
---------------------

This section provides a detailed description fo the EFI device-tree fix-up
protocol.

Summary
~~~~~~~

The EFI device-tree fix-up protocol provides a function to let the firmware
apply fix-ups.

Furthermore the function can be used to adjust the UEFI memory map according
to the reservations defined in the device-tree.

As a final step the protocol allow to install the device-tree as a configuration
table.

GUID
~~~~

.. code-block:: c

    #define EFI_DT_FIXUP_PROTOCOL_GUID \
     { 0xe617d64c, 0xfe08, 0x46da, \
     { 0xf4, 0xdc, 0xbb, 0xd5, 0x87, 0x0c, 0x73, 0x00 } }

Revision Number
~~~~~~~~~~~~~~~

.. code-block:: c

    #define EFI_DT_FIXUP_PROTOCOL_REVISION 0x00010000

Protocol Interface Structure
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: c

    typedef struct _EFI_DT_FIXUP_PROTOCOL {
    UINT64         Revision;
    EFI_DT_FIXUP   Fixup;
    } EFI_DT_FIXUP_PROTOCOL;

Parameters
~~~~~~~~~~

Revision
    The version of the EFI_DT_FIXUP_PROTOCOL. The version specified by this
    specification is 0x00010000. All future revisions must be backwards
    compatible. If a new version of the specification breaks backwards
    compatibility, a new GUID must be defined.

Fixup
    Applies fix-ups to a device-tree and installs it as a configuration table.

EFI_DT_FIXUP_PROTOCOL.Fixup()
-----------------------------

Summary
~~~~~~~

Applies fix-ups to a device-tree, makes memory reservations, and installs the
device-tree as a configuration table.

Prototype
~~~~~~~~~

.. code-block:: c

    typedef EFI_STATUS
    (EFIAPI *EFI_DT_FIXUP) (
        IN EFI_DT_FIXUP_PROTOCOL *This,
        IN VOID                  *Fdt,
        IN OUT UINTN             *BufferSize,
        IN UINT32                Flags,
        );

Parameters
~~~~~~~~~~

This
    Pointer to the protocol

Fdt
    Buffer with the device-tree. This shall be memory of type
    **EfiACPIReclaimMemory** if *Flags* comprises **EFI_DT_INSTALL_TABLE**.

BufferSize
    Pointer to the size of the buffer including trailing unused bytes for
    fix-ups. If the buffer size is too small, the required buffer size is
    returned.

Flags
    Bitmap containing at least one of the values

    * **EFI_DT_APPLY_FIXUPS**
    * **EFI_DT_RESERVE_MEMORY**
    * **EFI_DT_INSTALL_TABLE**

    Indicates the actions to be applied to the device-tree.

Related Definitions
~~~~~~~~~~~~~~~~~~~

.. code-block:: c

    /* Add nodes and update properties */
    #define EFI_DT_APPLY_FIXUPS    0x00000001
    /*
     * Reserve memory according to the /reserved-memory node
     * and the memory reservation block
     */
    #define EFI_DT_RESERVE_MEMORY  0x00000002
    /* Install the device-tree as configuration table */
    #define EFI_DT_INSTALL_TABLE   0x00000004

Description
~~~~~~~~~~~

The **Fixup()** function is called by a UEFI binary that has loaded a
device-tree to let the firmware apply firmware specific fix-ups, adjust memory
reservations, and eventually install the the device-tree as a configuration
table.

The caller provides the device-tree in a buffer of memory type
**EfiACPIReclaimMemory** as this is the memory type to be used for device-trees
provided as configuration table to the operating system.

If all or only sub-set of these actions shall be executed is determined by the
*Flags* parameter. The selected actions indicated in *Flags* are applied in the
sequence:

* Add nodes and update properties.
* Reserve memory according to the /reserved-memory node and the memory
  reservation block
* Install the device-tree as configuration table

Memory is reserved as **EfiBootServicesData** if the reservation does not carry
the **no-map** property and as **EfiReservedMemoryType** if it is marked as
**no-map**.

If the value pointed to by *BufferSize* exceeds the value of the *totalsize*
field of the device-tree header upon entry to the service, the *totalsize* field
is set to the value pointed to by *BufferSize*.

If the buffer is too small, **EFI_BUFFER_TOO_SMALL** is returned,
the device-tree is unmodified and the value pointed to by *BufferSize* is
updated with the required buffer size for the provided device-tree.

The required buffer size when called with **EFI_DT_APPLY_FIXUPS** should enforce
at least 4 KiB unused space for additional fix-ups by the operating system or
the caller. The available space in the device-tree shall be determined using the
device-tree header fields::

    Available = header->totalsize
              - header->off_dt_strings
              - header->size_dt_strings;

(The strings block is always last in the flattened device-tree. There
might be more space between blocks but not all device-tree libraries can
use it.)

The required buffer size when called without **EFI_DT_APPLY_FIXUPS** shall be
the value of the *totalsize* field of the flattened device-tree header.

If any other error code is returned, the state of the device-tree is undefined.
The caller should discard the buffer content.

The extent to which the validity of the device-tree is checked is implementation
dependent. But a buffer without the correct value of the *magic* field of the
flattened device-tree header should always be rejected.

The protocol implementation is not required to check if the device-tree is in
memory of type **EfiACPIReclaimMemory**.

Status codes returned
~~~~~~~~~~~~~~~~~~~~~

+---------------------------+-------------------------------------------------+
| **EFI_INVALID_PARAMETER** | This is NULL or does not point to a valid       |
|                           | EFI_DT_FIXUP_PROTOCOL implementation.           |
+---------------------------+-------------------------------------------------+
| **EFI_INVALID_PARAMETER** | Fdt or BufferSize is NULL                       |
+---------------------------+-------------------------------------------------+
| **EFI_INVALID_PARAMETER** | \*Fdt is not a valid device-tree                |
|                           | (e.g. incorrect value of magic)                 |
+---------------------------+-------------------------------------------------+
| **EFI_INVALID_PARAMETER** | Invalid value of Flags (zero or unknown bit)    |
+---------------------------+-------------------------------------------------+
| **EFI_BUFFER_TOO_SMALL**  | The buffer is too small to apply the fix-ups.   |
+---------------------------+-------------------------------------------------+
| **EFI_SUCCESS**           | All steps succeeded                             |
+---------------------------+-------------------------------------------------+
