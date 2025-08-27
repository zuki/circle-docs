DMA
~~~

CircleはRaspberry PiプラットフォームのDMAコントローラを使ってDMA (Direct Memory Access) をサポートしています。これは ``CDMAChannel`` クラスで実装されています。Raspberry Pi 5にはRP1サウスブリッジに別のDMAコントローラがあり、これは ``CDMAChannelRP1`` クラスで制御されます。

CDMAChannel
^^^^^^^^^^^

.. code-block:: c++

	#include <circle/dmachannel.h>

.. cpp:class:: CDMAChannel

.. cpp:function:: CDMAChannel::CDMAChannel (unsigned nChannel, CInterruptSystem *pInterruptSystem = 0)

	``CDMAChannel`` のインスタンスを生成し、プラットフォームのDMAコントローラのチャネルを割り当てます。 ``nChannel`` には ``DMA_CHANNEL_NORMAL`` (通常の DMA エンジン)、 ``DMA_CHANNEL_LITE`` (ライト (または通常の) DMA エンジン)、 ``DMA_CHANNEL_EXTENDED`` (Raspberry Pi 4 と 5 のみの "ラージアドレス" DMA4 エンジン)、または明示的なチャンネル番号 (0-15) のいずれかを指定します。 ``pInterruptSystem`` は ``CInterruptSystem`` のインスタンスへのポインタであり、割り込み処理用にのみ必要です。

.. note::

	現在のところ Raspberry Pi 5では "ラージアドレス" DMA4エンジンだけがサポートされています。動的 DMA チャネルの割り当て (明示的でないチャネル番号) にはこれらのエンジンのいずれかを使用します。定義されている ``DREQSource*`` の値は、現在のところ、Raspberry Pi 1-4 と Zero でのみ有効です。

.. cpp:function:: void CDMAChannel::SetupMemCopy (void *pDestination, const void *pSource, size_t nLength, unsigned nBurstLength = 0, boolean bCached = TRUE)

	``pSource`` から ``pDestination`` への長さ ``nLength`` のDMA メモリコピー転送を設定します。 0 より大きな ``nBurstLength`` は速度を向上させますが、システムバスが混雑させる可能性があります。 ``bCached`` は、送信元と宛先のアドレス範囲がキャッシュメモリにあるか否かを決定します。

.. cpp:function:: void CDMAChannel::SetupIORead (void *pDestination, uintptr nIOAddress, size_t nLength, TDREQ DREQ)

	I/Oポート ``nIOAddress`` から ``pDestination`` への長さ ``nLength`` の DMA読み込み転送を設定します。 ``DREQ`` は以下のデバイスからの転送をペース配分します。

* DREQSourceNone (no wait)
* DREQSourceEMMC
* DREQSourcePCMRX
* DREQSourceSMI
* DREQSourceSPIRX
* DREQSourceUARTRX

.. cpp:function:: void CDMAChannel::SetupIOWrite (uintptr nIOAddress, const void *pSource, size_t nLength, TDREQ DREQ)

	 ``pSource`` からI/Oポート ``nIOAddress`` への長さ ``nLength`` の DMA書き込み転送を設定します。 ``DREQ`` は以下のデバイスからの転送をペース配分します。

* DREQSourceNone (no wait)
* DREQSourceEMMC
* DREQSourcePCMTX
* DREQSourcePWM
* DREQSourcePWM1 (on Raspberry Pi 4 only)
* DREQSourceSMI
* DREQSourceSPITX
* DREQSourceUARTTX

.. cpp:function:: void CDMAChannel::SetupMemCopy2D (void *pDestination, const void *pSource, size_t nBlockLength, unsigned nBlockCount, size_t nBlockStride, unsigned nBurstLength = 0)

	Setup a 2D DMA memory copy transfer of ``nBlockCount`` blocks of ``nBlockLength`` length from ``pSource`` to ``pDestination``. Skip ``nBlockStride`` bytes after each block on destination. Source is continuous. The destination cache, if any, is not touched. ``nBurstLength`` > 0 increases speed, but may congest the system bus. This method can be used to copy data to the framebuffer and is not supported with ``DMA_CHANNEL_LITE``.

.. cpp:function:: void CDMAChannel::SetCompletionRoutine (TDMACompletionRoutine *pRoutine, void *pParam)

	割り込み操作用の DMA 完了ルーチンを設定します。転送が完了すると ``pRoutine`` が呼び出されます。 ``pParam`` は完了ルーチンに渡されるユーザパラメータです。 ``TDMACompletionRoutine`` は以下のプロトタイプを持ちます。

.. code-block:: c++

	void TDMACompletionRoutine (unsigned nChannel, boolean bStatus, void *pParam);

``nChannel`` はチャネル番号r. ``bStatus`` 転送が成功裏に完了した場合 ``TRUE`` です。

.. cpp:function:: void CDMAChannel::Start (void)

	事前に設定したDMA転送を開始します。Starts the DMA transfer, which has been setup before.

.. cpp:function:: boolean CDMAChannel::Wait (void)

	DMA転送の完了を待ちます（完了ルーチンを持たない同期非インターラプトオペレーション用）。転送が成功したら ``TRUE`` を返します。

.. cpp:function:: boolean CDMAChannel::GetStatus (void)

	最後の転送が成功した場合、 ``TRUE`` を返します。

CDMAChannelRP1
^^^^^^^^^^^^^^

.. code-block:: c++

	#include <circle/dmachannel-rp1.h>

.. cpp:class:: CDMAChannelRP1

	このクラスはRaspberry Pi 5のRP1 DMAコントローラを制御します。通常、これはRP1サウスブリッジのペリフェラルとシステムメモリ間のデータ転送に使用されます。
	This class controls the RP1 DMA controller of the Raspberry Pi 5, which is normally used to transfer data between peripherals in the RP1 southbridge and the system memory.

.. cpp:function:: CDMAChannelRP1::CDMAChannelRP1 (unsigned nChannel, CInterruptSystem *pInterruptSystem)

	Creates an instance of ``CDMAChannelRP1`` for RP1 DMA channel ``nChannel`` (0-7). There is currently no dynamic channel allocation for RP1 DMA channels. ``pInterruptSystem`` is a pointer to the interrupt system object.

.. cpp:function:: void CDMAChannelRP1::SetupMemCopy (void *pDestination, const void *pSource, size_t ulLength, boolean bCached = TRUE)

	Setup a DMA memory copy transfer from ``pSource`` to ``pDestination`` with length ``nLength`` bytes. ``bCached`` determines, if the source and destination address ranges are in cached memory.

.. cpp:function:: void CDMAChannelRP1::SetupIORead (void *pDestination, uintptr ulIOAddress, size_t ulLength, TDREQ DREQ, unsigned nIORegWidth = 4)

	Setup a DMA read transfer from the I/O port ``ulIOAddress`` (ARM-side or bus address) to ``pDestination`` with length ``nLength`` bytes. ``nIORegWidth`` specifies the width of the accessed I/O register (1 or 4 bytes). ``DREQ`` paces the transfer from these devices:

	* DREQSourceNone (no wait)
	* DREQSourceSPI0RX
	* DREQSourceSPI0TX
	* DREQSourceSPI1RX
	* DREQSourceSPI1TX
	* DREQSourceSPI2RX
	* DREQSourceSPI2TX
	* DREQSourceSPI3RX
	* DREQSourceSPI3TX
	* DREQSourceSPI5RX
	* DREQSourceSPI5TX
	* DREQSourcePWM0
	* DREQSourceI2S0RX
	* DREQSourceI2S0TX
	* DREQSourceI2S1RX
	* DREQSourceI2S1TX

.. cpp:function:: void CDMAChannelRP1::SetupCyclicIORead (void *ppDestinations[], uintptr ulIOAddress, unsigned nBuffers, size_t ulLength, TDREQ DREQ, unsigned nIORegWidth = 4)

	Setup a cyclic DMA read transfer from the I/O port ``ulIOAddress`` (ARM-side or bus address) for ``nBuffers`` concatenated DMA buffers (max. 4) at ``ppDestinations`` (pointer to array of pointers) with length ``nLength`` bytes per buffer. ``nIORegWidth`` specifies the width of the accessed I/O register (1 or 4 bytes). ``DREQ`` paces the transfer (see :cpp:func:`CDMAChannelRP1::SetupIORead` for the possible devices). The transfer starts from first buffer again, when last buffer has been filled.

.. cpp:function:: void CDMAChannelRP1::SetupIOWrite (uintptr ulIOAddress, const void *pSource, size_t ulLength, TDREQ DREQ, unsigned nIORegWidth = 4)

	Setup a DMA write transfer to the I/O port ``ulIOAddress`` (ARM-side or bus address) from ``pSource`` with length ``nLength`` bytes. ``nIORegWidth`` specifies the width of the accessed I/O register (1 or 4 bytes). ``DREQ`` paces the transfer (see :cpp:func:`CDMAChannelRP1::SetupIORead` for the possible devices).

.. cpp:function:: void CDMAChannelRP1::SetupCyclicIOWrite (uintptr ulIOAddress, const void *ppSources[], unsigned nBuffers, size_t ulLength, TDREQ DREQ, unsigned nIORegWidth = 4)

	Setup a cyclic DMA write transfer to the I/O port ``ulIOAddress`` (ARM-side or bus address) for ``nBuffers`` concatenated DMA buffers (max. 4) at ``ppSources`` (pointer to array of pointers) with length ``nLength`` bytes per buffer. ``nIORegWidth`` specifies the width of the accessed I/O register (1 or 4 bytes). ``DREQ`` paces the transfer (see :cpp:func:`CDMAChannelRP1::SetupIORead` for the possible devices). The transfer starts from first buffer again, when last buffer has been sent.

.. cpp:function:: void CDMAChannelRP1::SetCompletionRoutine (TCompletionRoutine *pRoutine, void *pParam)

	Sets a DMA completion routine for interrupt operation. ``pRoutine`` is called, when a transfer is completed. ``pParam`` is a user parameter, which is handed over to the completion routine. For cyclic transfer the completion routine is called after each buffer again. ``TCompletionRoutine`` has the following prototype:

.. cpp:type:: void CDMAChannelRP1::TCompletionRoutine (unsigned nChannel, unsigned nBuffer, boolean bStatus, void *pParam)

	``nChannel`` is the index of the RP1 DMA channel (0-7). ``nBuffer`` is the index of the cyclic buffer (0-N, 0 if not cyclic). ``bStatus`` is ``TRUE`` for a successful transfer, ``FALSE`` on error. ``pParam`` is the user parameter, handed over to ``SetCompletionRoutine()``.

.. cpp:function:: void CDMAChannelRP1::Start (void)

	Starts the DMA transfer, which has been setup before.

.. cpp:function:: void CDMAChannelRP1::Cancel (void)

	Cancels a running DMA transfer and waits for its termination.

.. _dma-buffers:

DMAバッファ
^^^^^^^^^^^

.. code-block:: c++

	#include <circle/synchronize.h>

.. c:macro:: DMA_BUFFER(type, name, num)

	DMA 転送に使用する ``type`` を ``num`` 要素持つ ``name`` バッファを定義します。

	DMAバッファに関するより詳細な情報は :ref:`dma-buffer-requirements` を参照してください。

キャッシュメンテナンス
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: c++

	#include <circle/synchronize.h>

.. c:function:: void CleanAndInvalidateDataCacheRange (uintptr nAddress, size_t nLength)

	データキャッシュのメモリ範囲を消去して無効にします。
