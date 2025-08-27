デバイス管理
~~~~~~~~~~~~~~~~~

サークルではほとんどのデバイスは以下の2つで表現されます。

* デバイス固有のオブジェクトによる。これは ``CDevice`` クラスから派生したクラスのインスタンスです。
* c文字列のデバイス名による。これは ``CDeviceNameService`` クラスにより実装されているCircleデバイス名サービスを使ってデバイスオブジェクトへのポインタを取得することができるようにされています。

.. note::

	CircleのI/OシステムはたとえばLinuxのI/Oシステムほどには統一されていません。デバイスクラスの中には よく知られている ``Read()`` や ``Write()`` インターフェースとは異なる特定のインターフェースを持ち、 ``CDevice`` から派生したものではないものがあります。

CDevice
^^^^^^^

.. code-block:: c++

	#include <circle/device.h>

.. cpp:class:: CDevice

	このクラスはCircleのほとんどのデバイスクラスの基底クラスです。

.. cpp:function:: virtual int CDevice::Read (void *pBuffer, size_t nCount)

	デバイスから ``pBuffer`` に最大 ``nCount`` バイトの読み込み操作を行います。読み込んだバイト数を返します。失敗した場合は < 0 を返します。

.. cpp:function:: virtual int CDevice::Write (const void *pBuffer, size_t nCount)

	デバイスに ``pBuffer`` から最大 ``nCount`` バイトの書き込み操作を行います。書き込んだバイト数を返します。失敗した場合は < 0 を返します。

.. cpp:function:: virtual u64 CDevice::Seek (u64 ullOffset)

	デバイスの読み書きポインタの位置をバイトオフセット ``ullOffset`` に設定します。設定されたオフセットを返します。失敗した場合は ``(u64) -1`` を返します。このメソッドはブロックデバイスに対してのみ実装されており、キャラクタデバイスは常に失敗を返します。

.. cpp:function:: virtual u64 CDevice::GetSize (void) const

	ブロックデバイスの合計バイトサイズを返します。失敗した場合は ``(u64) -1`` を返します。このメソッドはブロックデバイスに対してのみ実装されており、キャラクタデバイスは常に失敗を返します。

.. cpp:function:: virtual int CDevice::IOCtl (unsigned long ulCmd, void *pData)

	デバイスの I/O 制御コマンド ``ulCmd`` をコマンド固有のデータ ``pData`` と共に呼び出します。 ``pData`` はコマンド固有のデータを返すことに使用できます。成功した場合は0を、失敗した場合はエラーコードを返します。現在、このメソッドはCircle自体では使用されていません。ユーザによる拡張のために定義されています。

.. cpp:function:: virtual boolean CDevice::RemoveDevice (void)

	擬似プラグアンドプレイ用にシステムからデバイスを取り外すことを要求します。これはUSBデバイス（USB大容量記憶デバイスなど）に対してのみ実装されています。デバイスの取り外しに成功したら ``TRUE`` を返します。

.. cpp:function:: CDevice::TRegistrationHandle CDevice::RegisterRemovedHandler (TDeviceRemovedHandler *pHandler, void *pContext = 0)

	このデバイスがホットプラグでシステムから取り外されたときに呼び出されるコールバックを登録します。デバイスオブジェクトが削除される前に ``pHandler`` が呼び出されます。 ``pContext`` はユーザポインタであり、ハンドラに渡されます。:cpp:func:`CDevice::UnregisterRemovedHandler()` に渡すハンドルを返します。このメソッドは特定のデバイスに対して複数回呼び出すことができ、その場合、登録されたハンドラは逆順で呼び出されます。このメソッドを ``pHandler = 0`` で呼び出すことで登録を解除することはサポートされなくなりました。

.. code-block:: c++

	void TDeviceRemovedHandler (CDevice *pDevice, void *pContext);

.. cpp:function:: void CDevice::UnregisterRemovedHandler (TRegistrationHandle hRegistration)

	デバイス削除ハンドラの登録を元に戻します。 ``hRegistration`` は :cpp:func:`CDevice::RegisterRemovedHandler()` により返されたハンドルです。

.. note::

	CircleによるUSBプラグアンドプレイのサポートに関する詳細は :doc:`../ref/usb-plug-and-play` を参照してください。

CDeviceNameService
^^^^^^^^^^^^^^^^^^

.. code-block:: c++

	#include <circle/devicenameservice.h>

.. cpp:class:: CDeviceNameService

	Circleではデバイスを名前で登録して、後でその名前で検索することができます。これは ``CDeviceNameService`` クラスで実装されています。

.. note::

	通常、デバイス名はアルファベットの名前プレフィックスとそれに続く10進数のデバイスインデックス番号（1以上）で構成されます。ブロックデバイスのパーティションには別のパーティションインデックスがあり、これも1以上です。サウンドデバイスにはデバイスインデックス番号はありません。以下はデバイス名の例です。

	==============  ====================================================
	デバイス名      説明
	==============  ====================================================
	tty1            最初のスクリーンデバイス
	ukbd1           最初のUSBキーボードデバイス
	umsd1           最初のUSB大容量格納デバイス（フラッシュドライブなど）
	umsd1-1         最初のUSB大容量格納デバイスの最初のパーティション
	sndpwm          PWMサウンドデバイス
	null            ヌルデバイス
	==============  ====================================================

.. cpp:function:: static CDeviceNameService *CDeviceNameService::Get (void)

	システムに1つだけある ``CDeviceNameService`` インスタンスへのポインタを返します。

.. cpp:function:: CDevice *CDeviceNameService::GetDevice (const char *pName, boolean bBlockDevice)

	名前が ``pName`` でデバイスタイプが ``bBlockDevice`` のデバイスのデバイスオブジェクトへのポインタを返します。デバイスが存在しない場合は 0 を返します。  ``bBlockDevice``  はこれがブロックデバイスの場合は  ``TRUE`` 、そうでなければキャラクタデバイスです。

.. cpp:function:: CDevice *CDeviceNameService::GetDevice (const char *pPrefix, unsigned nIndex, boolean bBlockDevice)

	 名前プリフィックスが ``pName`` でデバイスインデックスが ``nIndex``  、デバイスタイプが ``bBlockDevice`` のデバイスのデバイスオブジェクトへのポインタを返します。デバイスが存在しない場合は 0 を返します。  ``bBlockDevice`` はこれがブロックデバイスの場合は  ``TRUE`` 、そうでなければキャラクタデバイスです。検索されるデバイス名は名前プリフィックスとそれに続く10進のデバイスインデックスで構成されます（たとえば、最初のUSB大容量ストレージデバイスは ``umsd1`` です）。

.. cpp:function:: void CDeviceNameService::ListDevices (CDevice *pTarget)

	デバイス名のリストをテキストで生成し、デバイス ``pTarget`` に書き込みます。
	Generates a textual device name listing and writes it to the device ``pTarget``.

.. cpp:function:: boolean CDeviceNameService::EnumerateDevices (boolean (*pCallback) (CDevice *pDevice, const char *pName, boolean bBlockDevice, void *pParam), void *pParam)

	すべてのデバイスをエヌメレートして各デバイスに対して ``pCallback`` を呼び出します。 ``pParam`` はコールバックに渡すユーザ定義のポインタです。コールバックが ``FALSE`` を返してエヌメレーションがキャンセルされた場合は ``FALSE`` を返します。

.. cpp:function:: void CDeviceNameService::AddDevice (const char *pName, CDevice *pDevice, boolean bBlockDevice)

	``pName`` という名前のデバイスオブジェクトへのポインタ ``pDevice`` をデバイス名レジストリに追加します。 ``bBlockDevice`` はこれがブロックデバイスの場合は  ``TRUE`` 、そうでなければキャラクタデバイスです。通常、このメソッドはデバイスドライバクラスでしか使用されません。

.. cpp:function:: void CDeviceNameService::AddDevice (const char *pPrefix, unsigned nIndex, CDevice *pDevice, boolean bBlockDevice)

	名前プリフィックスが ``pName`` でデバイスインデックスが ``nIndex`` のデバイスオブジェクトへのポインタ ``pDevice`` をデバイス名レジストリに追加します。 ``bBlockDevice`` はこれがブロックデバイスの場合は  ``TRUE`` 、そうでなければキャラクタデバイスです。登録されるデバイス名は名前プリフィックスとそれに続く10進のデバイスインデックスで構成されます（たとえば、最初のUSB大容量ストレージデバイスは ``umsd1`` です）。通常、このメソッドはデバイスドライバクラスでしか使用されません。

.. cpp:function:: void CDeviceNameService::RemoveDevice (const char *pName, boolean bBlockDevice)

	デバイス名 ``pName`` でデバイスタイプが ``bBlockDevice`` のデバイスをデバイス名のレジストリから削除します。 ``bBlockDevice`` はこれがブロックデバイスの場合は  ``TRUE`` 、そうでなければキャラクタデバイスです。通常、このメソッドはデバイスドライバクラスでしか使用されません。

.. cpp:function:: void CDeviceNameService::RemoveDevice (const char *pPrefix, unsigned nIndex, boolean bBlockDevice)

	名前プリフィックスが ``pName`` でデバイスインデックスが ``nIndex`` のデバイスをデバイス名レジストリから削除します。 ``bBlockDevice`` はこれがブロックデバイスの場合は  ``TRUE`` 、そうでなければキャラクタデバイスです。削除されるデバイス名は名前プリフィックスとそれに続く10進のデバイスインデックスで構成されます（たとえば、最初のUSB大容量ストレージデバイスは ``umsd1`` です）。通常、このメソッドはデバイスドライバクラスでしか使用されません。
