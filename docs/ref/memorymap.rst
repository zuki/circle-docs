.. _memorymap:

メモリマップ
------------

Raspberry Pi 3 (AArch64)
^^^^^^^^^^^^^^^^^^^^^^^^

::

	------------------------------------------------------------------------------
	Base		Size		Contents		Remarks
	------------------------------------------------------------------------------
	00000000	256 Bytes	ARM stub		Core 1-3 spin here

	00000100	variable	ATAGS			unused
	...
	00080000	max. 2 MByte	Kernel image
					.init			startup code
					.text
					.rodata
					.init_array		static constructors
					.ARM.exidx		unused
					.eh_frame		C++ exception frames
					.data
					.bss
	...
					Kernel stacks
	00280000	128 KByte	 for Core 0
	002A0000	128 KByte	 for Core 1		may be unused
	002C0000	128 KByte	 for Core 2		may be unused
	002E0000	128 KByte	 for Core 3		may be unused

					Exception stacks
	00300000	32 KByte	 for Core 0
	00308000	32 KByte	 for Core 1		may be unused
	00310000	32 KByte	 for Core 2		may be unused
	00318000	32 KByte	 for Core 3		may be unused

	00500000	1 MByte		Coherent region		for property mailbox, VCHIQ
	00600000	variable	Heap allocator		malloc()
	????????	16 MByte	Page allocator		palloc()

	????????	variable	GPU memory
	3F000000	16 MByte	Peripherals
	40000000			Local peripherals
	...
	------------------------------------------------------------------------------
