ネットワークデバイス
~~~~~~~~~~~~~~~~~~~~

ネットワークデバイスはネットワークインターフェースへの低レベルアクセスを可能にし、ネットワークフレームを送受信する方法やネットワークアクセスを管理する追加機能を提供します。CircleはIEEE 802.3イーサネットとIEEE 802.11無線LAN（WLAN）インターフェースへのアクセスをサポートしています。すべてのネットワークデバイスクラスはネットワークアプリケーション用の低レベルAPIを定義している基本クラス :cpp:class:`CNetDevice` から派生したものです。通常、アプリケーションは :cpp:class:`CSocket` クラスが提供する高レベルの TCP/IP ソケットインターフェースを使用することに注意してください。

CNetDevice
^^^^^^^^^^

.. code-block:: cpp

	#include <circle/netdevice.h>

.. cpp:class:: CNetDevice

	このクラスはすべてのネットワークデバイスドライバクラスの基本クラスであり、ネットワークインタフェースを介してフレームを直接交換したい特定のネットワークアプリケーションのための低レベル API を定義しています。ネットワークデバイスはデバイス名サービスには登録されず、 :cpp:func:`CNetDevice::GetNetDevice()` メソッドを使用して見つけることができます。

.. c:macro:: FRAME_BUFFER_SIZE

	このマクロはネットワークインターフェース上で送信または受信されるフレームの最大サイズを定義します。Circleでは通常、ネットワークバッファのサイズはこの値です。

.. cpp:function:: virtual TNetDeviceType CNetDevice::GetType (void)

	このネットワークデバイスのタイプを返します。次のいずれかです。

.. c:type:: TNetDeviceType

	* NetDeviceTypeEthernet
	* NetDeviceTypeWLAN

.. cpp:function:: virtual const CMACAddress *CNetDevice::GetMACAddress (void) const

	このネットワークインターフェースデバイスにおける自身のアドレスを保持しているMACアドレスオブジェクトへのポインタを返します。

.. cpp:function:: virtual boolean CNetDevice::IsSendFrameAdvisable (void)

	``SendFrame()``を呼び出すことが望ましい場合は ``TRUE`` を返します。

.. note::

	``SendFrame()`` はいつでも呼び出すことができますが、TXキューが満杯の場合は失敗することがあります。このメソッドは ``SendFrame()`` を呼び出すことが望ましいかどうかのヒントを与えます。

.. cpp:function:: virtual boolean CNetDevice::SendFrame (const void *pBuffer, unsigned nLength)

	有効なフレームをネットワークに送信します。 ``pBuffer`` はフレームへのポインタであり、フレームチェックシーケンス（FCS）は含みません。 ``nLength`` はバイト単位のフレーム長です。アプリケーションはフレームをパディングする必要はありません。

.. cpp:function:: virtual boolean CNetDevice::ReceiveFrame (void *pBuffer, unsigned *pResultLength)

	ネットワークインターフェース経由で受信したフレームをポーリングします。 ``pBuffer`` はフレームが置かれるバッファへのポインタであり、 :c:macro:`FRAME_BUFFER_SIZE` のサイズでなければなりません。 ``pResultLength`` は有効なフレーム長を受け取る変数へのポインタです。バッファにフレームが返された場合は ``TRUE`` を返し、何も受信しなかった場合は ``FALSE`` を返します。

.. cpp:function:: virtual boolean CNetDevice::IsLinkUp (void)

	物理リンク (PHY) がアクティブの場合 ``TRUE`` を返します。

.. cpp:function:: virtual TNetDeviceSpeed CNetDevice::GetLinkSpeed (void)

	物理リンク (PHY) がアクティブの場合はその速度を返します。わからない場合は ``NetDeviceSpeedUnknown`` を返します。次のリンク速度が定義されています。

.. c:type:: TNetDeviceSpeed

	* NetDeviceSpeed10Half
	* NetDeviceSpeed10Full
	* NetDeviceSpeed100Half
	* NetDeviceSpeed100Full
	* NetDeviceSpeed1000Half
	* NetDeviceSpeed1000Full
	* NetDeviceSpeedUnknown

.. cpp:function:: virtual boolean CNetDevice::UpdatePHY (void)

	物理リンク (PHY) の状態に応じてデバイスの設定を更新します。この関数がサポートされていない場合は ``FALSE`` を返します。

.. note::

	このメソッドは :ref:`TCP/IP networking` サブシステムのPHYタスクによって2秒ごとに継続的に呼び出されます。このサブシステムを使わない場合は自身でこのメソッドを呼び出す必要があります。

.. cpp:function:: static const char *CNetDevice::GetSpeedString (TNetDeviceSpeed Speed)

	通常 ``GetLinkSpeed()`` から返されるリンク速度の値 ``Speed`` の説明を返します。

.. cpp:function:: static CNetDevice *CNetDevice::GetNetDevice (unsigned nDeviceNumber)

	ネットワークデバイスのゼロ始まりの番号 ``nDeviceNumber`` に対するネットワークデバイスオブジェクトへのポインタを返します。そのデバイスが利用できない場合は 0 を返します。

.. cpp:function:: static CNetDevice *CNetDevice::GetNetDevice (TNetDeviceType Type)

	特定のネットワークデバイスタイプ（ :cpp:func:`CNetDevice::GetType()` を参照）か、任意のネットワークデバイスを検索する場合は ``NetDeviceTypeAny`` であるタイプ ``Type`` の最初のネットワークデバイスオブジェクトへのポインタを返します。

CSMSC951xDevice
^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/smsc951x.h>

.. cpp:class:: CSMSC951xDevice : public CUSBFunction, CNetDevice

	This class is a driver for the SMSC9512 and SMSC9514 Ethernet network interface devices, which are attached to the internal USB hub of Raspberry Pi 1, 2 and 3 Model B boards. This class is automatically instantiated in the USB device enumeration process, when a device of this type is found. This class does not provide specific methods, its API is defined by the base class :cpp:class:`CNetDevice`.

CLAN7800Device
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/lan7800.h>

.. cpp:class:: CLAN7800Device : public CUSBFunction, CNetDevice

	このクラスは、Raspberry Pi 3 Model B+ ボードの内部 USB ハブに接続されている LAN7800 Gigabit Ethernet ネットワークインターフェースデバイス用のドライバです。このクラスはUSBデバイスのエヌメレーション処理中にこのタイプのデバイスが見つかると自動的にインスタンス化されます。このクラスは固有のメソッドは提供しません。そのAPIは基底クラス :cpp:class:`CNetDevice` で定義されています。

CUSBCDCEthernetDevice
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbcdcethernet.h>

.. cpp:class:: CUSBCDCEthernetDevice : public CUSBFunction, CNetDevice

	このクラスは、QEMU がサポートする USB CDC イーサネットネットワークインターフェイスデバイス用のドライバです。このクラスはUSBデバイスのエヌメレーション処理中にこのタイプのデバイスが見つかると自動的にインスタンス化されます。このクラスは固有のメソッドは提供しません。そのAPIは基底クラス :cpp:class:`CNetDevice` で定義されています。

CBcm54213Device
^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/bcm54213.h>

.. cpp:class:: CBcm54213Device : public CNetDevice

	This class is a driver for the BCM54213PE Gigabit Ethernet Transceivers of the Raspberry Pi 4, 400 and Compute Module 4. It is instantiated in the :ref:`TCP/IP networking` subsystem, but has to be manually instantiated by applications, which do not use this subsystem. This class does not provide specific methods, its API is defined by the base class :cpp:class:`CNetDevice`.

CMACBDevice
^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/macb.h>

.. cpp:class:: CMACBDevice : public CNetDevice

	This class is a driver for the MACB / GEM Gigabit Ethernet Transceiver of the Raspberry Pi 5. It is instantiated in the :ref:`TCP/IP networking` subsystem, but has to be manually instantiated by applications, which do not use this subsystem. This class does not provide specific methods, its API is defined by the base class :cpp:class:`CNetDevice`.

CBcm4343Device
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <wlan/bcm4343.h>

.. cpp:class:: CBcm4343Device : public CNetDevice

	このクラスはRaspberry Pi 3, 4, 5, Zero (2) WのBCM4343x WLANインタフェース
	デバイス用のドライバーです。マニュアルでインスタンス化する必要があり、通常は
	ref:`TCP/IP ネットワーク` サブシステムの :cpp:class:`CNetSubSystem` クラスと
	サブモジュール `hostap <https://github.com/rsta2/hostap/tree/hostap_0_7_0-circle>`_ 
	を継承した :cpp:class: `CWPASupplicant`クラスと組み合わせて使用します。
	このクラスは、基底クラス :cpp:class:`CNetDevice` で定義されているインタフェースを
	提供し、WLANアクセスポイント (AP) とのアソシエーションを管理するために必要な
	追加のメソッドを提供します。以下の説明はこのクラスに固有のメソッドだけを扱います。

.. cpp:function:: CBcm4343Device::CBcm4343Device (const char *pFirmwarePath)

	このクラスのインスタンスを作成します。 ``pFirmwarePath`` にはWLANコントローラ用の
	ファームウェアファイルが存在するパス（"SD:/firmware/" など）を指定します。

.. cpp:function:: boolean CBcm4343Device::Initialize (void)

	WLANコントローラをドライバをインスタンス化します。成功時には ``TRUE`` を返します。

.. cpp:function:: void CBcm4343Device::RegisterEventHandler (TBcm4343EventHandler *pHandler, void *pContext)

	指定のWLANイベント（APからの切断など）時に呼び出されるイベントハンドラ
	``pHandler`` を登録します。 ``pContext`` はイベントハンドラに渡されるユーザポインタです。
	``pHandler`` に0を指定するとイベントハンドラの登録を解除することができます。

.. cpp:function:: boolean CBcm4343Device::Control (const char *pFormat, ...)

	デバイス固有のコントロールコマンド ``pFormat`` をオプションのパラメータと共にWLAN
	デバイスドライバに送信します。成功時には ``TRUE`` を返します。

.. cpp:function:: boolean CBcm4343Device::ReceiveScanResult (void *pBuffer, unsigned *pResultLength)

	スキャン結果メッセージを受信するためにポーリングします。 ``pBuffer`` はメッセージが
	格納されるバッファへのポインタです。バッファのサイズは :c:macro:`FRAME_BUFFER_SIZE` 
	でなければなりません。 ``pResultLength`` は有効なメッセージ長を受け取る変数への
	ポインタです。バッファにメッセージが返された場合は ``TRUE`` 、何も受信しなかった場合は
	``FALSE`` を返します。

.. cpp:function:: const CMACAddress *CBcm4343Device::GetBSSID (void)

	接続したAPのBSSIDを返します。

.. cpp:function:: boolean CBcm4343Device::JoinOpenNet (const char *pSSID)

	SSIDが ``pSSID`` の公開WLANネットワークに参加します。成功時には ``TRUE`` を返します。

.. cpp:function:: boolean CBcm4343Device::CreateOpenNet (const char *pSSID, int nChannel, bool bHidden)

	チャネル ``nChannel`` 上にSSIDが ``pSSID`` の公開WLANネットワーク（APモード）を
	作成します。 ``bHidden`` が ``TRUE`` の場合、SSIDは非公開になります。成功時には 
	``TRUE`` を返します。

.. cpp:function:: boolean CBcm4343Device::DestroyOpenNet (void)

	作成した公開WLANネットワークを廃棄します。このネットワークは後で再度作成することが
	できます。成功時には ``TRUE`` を返します。
