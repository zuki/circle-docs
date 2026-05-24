Direct hardware access
~~~~~~~~~~~~~~~~~~~~~~

該当するデバイスのドライバがまだ存在しない場合、Circleアプリケーションはハードウェアデバイスのレジスタを直接読み書きする必要がある場合があります。この節では、ハードウェアレジスタへのアクセスに関するCircleのサポートについて説明します。

関数
^^^^^^^^^

.. code-block:: c

	#include <circle/memio.h>

.. c:function:: u8 read8 (uintptr nAddress)
.. c:function:: u16 read16 (uintptr nAddress)
.. c:function:: u32 read32 (uintptr nAddress)

	メモリマップドI/Oアドレス ``nAddress`` から指定のビットサイズの値を読み取って返します。

.. c:function:: void write8 (uintptr nAddress, u8 uchValue)
.. c:function:: void write16 (uintptr nAddress, u16 usValue)
.. c:function:: void write32 (uintptr nAddress, u32 nValue)

	メモリマップドI/Oアドレス ``nAddress`` に指定のビットサイズの値を書き込みます。 ``uchValue``, ``usValue``, ``nValue`` は対応する値です。

.. note::

	通常、メモリマップドI/Oデバイスレジスタへのアクセスはアクセスサイズにアラインしていなければなりません。

マクロ
^^^^^^

Raspberry Piの各種ハードウェアデバイスに関する詳細な定義についてここですべてを記載することはできません。詳細についてはそれぞれのヘッダーファイルをご参照ください。

.. code-block:: c

	#include <circle/bcm2835.h>

このヘッダーファイルは `BCM2835 ARM Peripherals <https://datasheets.raspberrypi.com/bcm2835/bcm2835-peripherals.pdf>`_ ドキュメントに記載されているすべてのRaspberry Piモデル向けのメモリマップドI/Oアドレスのマクロ定義を提供しています。特に以下のマクロが重要です。

.. c:macro:: ARM_IO_BASE

	各Raspberry PiモデルのARM CPUにおいて有効な16MBサイズの主たるメモリマップドI/Oブロックのベースアドレスです。通常、このアドレスはCircleアプリケーションから使用されます。

.. c:macro:: GPU_IO_BASE

	GPUコプロセッサ上で有効な16MBサイズの主たるメモリマップドI/Oブロックのベースアドレス。このアドレスは、GPUまたは接続されたデバイス（DMAコントローラなど）によって実行される操作で使用されます。

.. note::

	Raspberry Piには複数の処理ユニットが搭載されています。ここでは、Circleアプリケーションが実行されているARM CPUとファームウェアや高速グラフィックス処理などが実行されているその他の処理ユニットだけを区別します。後者のプロセッサをGPUまたはVPUと呼びます。なお、起動順序の観点からは、ARM CPUはセカンダリコプロセッサである点にご注意ください。

.. c:macro:: GPU_MEM_BASE

	GPUおよび接続デバイス（DMAコントローラなど）で有効な、下位（ARM CPUではアドレス0x0から始まる）1GBのメモリアドレス範囲のベースアドレス。たとえば、レガシープラットフォームのDMAコントローラはデータ転送においてこのアドレス空間でしかアクセスできません。

.. c:macro:: BUS_ADDRESS(address)

	ARM CPU上で有効なメモリアドレス ``address`` を、GPUおよび接続デバイス上で有効なGPUバスアドレスに変換します。

.. code-block:: c

	#include <circle/bcm2836.h>

このヘッダーファイルは `BCM2835 ARM Peripherals <https://datasheets.raspberrypi.com/bcm2835/bcm2835-peripherals.pdf>`_ ドキュメントに記載されているRaspberry Pi 2〜4とcompatibleモデル向けのメモリマップドI/Oアドレスのマクロ定義を提供します。特に以下のマクロが需要です。

.. c:macro:: ARM_LOCAL_BASE

	256 MBのローカルメモリマップドI/Oブロックのベースアドレスです。このブロック内のレジスタは各ARM CPUコアにローカルに割り当てられています。

.. code-block:: c

	#include <circle/bcm2711.h>

This header file provides macro definitions of memory-mapped I/O addresses for the Raspberry Pi 4 and compatible models, described in the `BCM2711 ARM Peripherals <https://datasheets.raspberrypi.com/bcm2711/bcm2711-peripherals.pdf>`_ document.

.. code-block:: c

	#include <circle/bcm2712.h>

This header file provides macro definitions of memory-mapped I/O addresses for the Raspberry Pi 5, mostly described in the `RP1 Peripherals <https://datasheets.raspberrypi.com/rp1/rp1-peripherals.pdf>`_ document.

I/Oバリア
^^^^^^^^^^^^

以下のI/Oバリアは特にRaspberry Pi 1とZeroで必要です。その他のRaspberry Piモデルでは、これらは機能しません。

.. code-block:: c

	#include <circle/synchronization.h>

.. c:macro:: PeripheralEntry()

	If your code directly accesses memory-mapped hardware registers, you should insert this special barrier before the first access to a specific hardware device.

.. c:macro:: PeripheralExit()

	If your code directly accesses memory-mapped hardware registers, you should insert this special barrier after the last access to a specific hardware device.

.. note::

	Most programs would work without ``PeripheralEntry()`` and ``PeripheralExit()``, but to be sure, it should be used as noted. In a few tests there have been issues on the Raspberry Pi 1, where invalid data was read from hardware registers, without these barriers inserted.

	You do not need to care about this, when you access hardware devices using a Circle device driver class, because this is handled inside the driver.
