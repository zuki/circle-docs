メモリ
~~~~~~

Circleはメモリ管理ユニット（MMU）を有効にして、動作の高速化のためにCPUのデータキャッシュを利用することができますが、特定のシステム機能を実装するために仮想メモリを利用することはありません。物理アドレスと仮想アドレスのマッピングは使用されるメモリ空間全体で1対1です。 [#ma]_  様々なシステム構成のメモリレイアウトは :ref:`memorymap` で見ることができます。

new と delete
^^^^^^^^^^^^^^

Circleはシステムヒープメモリをサポートしています。メモリは C++の通常の ``new`` 演算子で確保し、 ``delete`` 演算子で解放することができます。メモリブロックの確保と解放は ``TASK_LEVEL`` と ``IRQ_LEVEL`` でサポートされていますが、 ``FIQ_LEVEL`` でははサポートされていません。 [#el]_ 割り当てられたメモリブロックは常にシステムのキャッシュラインの最大サイズにアライメントされます。 [#al]_

.. note::

	Circleは様々なサイズのメモリブロックを管理するために数多くのリンクリストを保持しています。サポートされるブロックサイズはシステムオプション ``HEAP_BLOCK_BUCKET_SIZES`` で定義されています。デフォルトで管理可能な最大ブロックサイズは 512 KByte です。それ以上のサイズのメモリブロックを確保することもできますが ``delete`` した後に再利用することができません。

 ``new`` 演算子には割り当てるメモリのタイプを指定するパラメタを指定できます。

==============  =================================================================
パラメタ        説明
==============  =================================================================
HEAP_LOW        1 GByte以下のメモリ
HEAP_HIGH       1 GByte以上のメモリ (Raspberry Pi 4 と 5 のみ)
HEAP_ANY        1 GByte以上のメモリ (利用可能な場合)、または、1 GByte以下のメモリ (その他)
HEAP_DMA30      30ビットのDMA可能なメモリ (HEAP_LOWのエイリアス)
==============  =================================================================

これは異なるSDRAMメモリ領域をサポートするRaspberry Pi 4と5では特に重要です。たとえば、1GByte以上の領域から256バイトのメモリブロックを割り当てるように指定することができます。

.. code-block:: c++

	#include <circle/new.h>

	unsigned char *p = new (HEAP_HIGH) unsigned char[256];

メモリタイプパラメタの仕様に関するさらなる情報は :ref:`new-operator` にあります。

CMemorySystem
^^^^^^^^^^^^^

.. code-block:: c++

	#include <circle/memory.h>

.. cpp:class:: CMemorySystem

クラス ``CMemorySystem`` はCircleのメモリ管理機能のほとんどを実装しています。通常、このクラスのインスタンスは各Circleアプリケーションに1つだけ存在し、Circleシステムの初期化コードで生成されます。以前のバージョンのCircleでは ``CKernel`` でこのインスタンスを明示的に作成する必要がありました。これは現在では非推奨ですがそうしても問題はありません。もう一つ ``CMemorySystem`` のインスタンスが作成された場合、それは最初に作成されたインスタンスのエイリアスになります。

アプリケーションから呼び出せるメソッドは以下の通りです。

.. cpp:function:: size_t CMemorySystem::GetMemSize (void) const

	Returns the total memory size available to the application, as reported by the firmware.

.. cpp:function:: size_t CMemorySystem::GetHeapFreeSpace (int nType) const

	Returns the free space on the heap of the given type, according to the memory type ``HEAP_LOW``, ``HEAP_HIGH`` or ``HEAP_ANY``. Does not cover memory blocks, which have been freed.

.. cpp:function:: static CMemorySystem *CMemorySystem::Get (void)

	Returns a pointer to the instance of ``CMemorySystem``.

.. cpp:function:: static void CMemorySystem::DumpStatus (void)

	Dumps some memory allocation status information. Requires ``HEAP_DEBUG`` to be defined.

CClassAllocator
^^^^^^^^^^^^^^^

The class ``CClassAllocator`` allows to define a class-specific allocator for a class, using a pre-allocated store of memory blocks. This can speed up memory allocation, if the maximum number of instances of the class is known and a class instance does not occupy too much memory space. If you want to use this technique for your own class, the class definition has to look like this:

.. code-block:: c++
	:caption: myclass.h

	#include <circle/classallocator.h>

	class CMyClass
	{
	...

		DECLARE_CLASS_ALLOCATOR
	};

You have to add the following to the end of the class implementation file:

.. code-block:: c++
	:caption: myclass.cpp

	#include "myclass.h"

	...

	IMPLEMENT_CLASS_ALLOCATOR (CMyClass)

Before an instance of your class can be created, one of these (macro-) functions have to be executed:

.. code-block:: c++

	#include "myclass.h"

	INIT_CLASS_ALLOCATOR (CMyClass, Number);			// or:

	INIT_PROTECTED_CLASS_ALLOCATOR (CMyClass, Number, Level);

The second variant initializes a class-specific allocator, which is protected with a spin-lock for concurrent use. *Number* is the number of pre-allocated memory blocks and *Level* the maximum execution level, from which ``new`` or ``delete`` for this class will be called. [#el]_ This variant can be called multiple times with the same *Level* parameter. The class store will be extended then by the given number of objects.

C functions
^^^^^^^^^^^

Circle provides the following C standard library functions for memory allocation:

.. code-block:: c

	#include <circle/alloc.h>

	void *malloc (size_t nSize);
	void *calloc (size_t nBlocks, size_t nSize);
	void *realloc (void *pBlock, size_t nSize);
	void free (void *pBlock);

.. rubric:: Footnotes

.. [#ma] There is one exception from this rule. On the Raspberry Pi 4 the memory mapped I/O register space of the xHCI USB controller, which is connected using a PCIe interface, is re-mapped into the 4 GByte 32-bit address space, because it is physically located above the 4 GByte boundary, and would not be accessible in 32-bit mode otherwise.

.. [#el] System execution levels (e.g. ``TASK_LEVEL``) are described in the section :ref:`synchronization`.

.. [#al] 32 bytes on the Raspberry Pi 1 and Zero, 64 bytes otherwise
