USB
~~~

USB (Universal Serial Bus) サブシステムはUSB 2.0とUSB 3.0 (Raspberry Pi 4 のみ) のデバイスへのアクセスをサポートするサービスとデバイスドライバを提供します。ここでは、基本的に、Raspberry Pi 1-3とZeroのDWHCI OTG USBコントローラ（ホストモードとgadgetモード）とRaspberry Pi 4, 400, 5のxHCI USBコントローラ（ホストモードのみ）用のドライバ、USBデバイスクラスドライバ、いくつかのベンダー固有のUSBデバイスドライバ、これらすべてのドライバのサポートクラスを説明します。

このサブシステムのほとんどの操作は :ref:`Devices` セクションで説明するように、デバイスドライバインターフェースの背後に隠れていてアプリケーションからは見えないようになっています。USBを使用するアプリケーションはシステム起動時のUSBサポートの初期化とオプションですがシステムの実行中に新しく接続されたUSBデバイスの検出（USBプラグアンドプレイ）に対処しなければなりません。このセクションではこれらのトピックに限定して説明します。

Circleの（オプションの）USBプラグアンドプレイサポートに関する一般的な情報については :ref:`usb-plug-and-play` を参照してください。

.. important::

	CircleはOTGプロトコルをサポートしていません。そのため、USBコントローラは常にホストまたはガジェットモードで動作し、接続されたピアはその逆のモードで動作する必要があることにご注意ください。

CUSBController
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbcontroller.h>

.. cpp:class:: CUSBController

	このクラスは、CircleのすべてのUSB（ホストまたはガジェット）コントローラへのインターフェイスを定義します。これは、USBホスト/ガジェット双方のコントローラをサポートするアプリケーションで両コントローラのユニークな処理を可能にするために提供されます。

.. cpp:function:: virtual boolean CUSBController::Initialize (boolean bScanDevices = TRUE) = 0

	USB（ホストまたはガジェット）コントローラを初期化します。パラメータ ``bScanDevices`` は現在のところUSBホストコントローラにしか対応していません。説明については :cpp:func:`CUSBHCIDevice::Initialize()` を参照してください。成功すると ``TRUE`` を返します。

.. cpp:function:: virtual boolean CUSBController::UpdatePlugAndPlay (void) = 0

	接続されている USB デバイスの情報を更新します。このメソッドは、USBプラグアンドプレイが有効になっている場合、 ``TASK_LEVEL`` からて継続的に呼び出す必要があります。USBデバイスツリーが変更された可能性がある場合は ``TRUE`` を返します。この場合、アプリケーションは :cpp:func:`CDeviceNameService::GetDevice()` を呼び出してサポートしているデバイスが存在するかテストする必要があります。 ``UpdatePlugAndPlay()`` は最初の呼び出しでは常に ``TRUE`` を返します。

USBホストサポート
^^^^^^^^^^^^^^^^^^^

ホストモードではUSBコントローラは1つ以上のUSBデバイス（ガジェット、ペリフェラル）をサポートします。

CUSBHCIDevice
"""""""""""""

.. code-block:: cpp

	#include <circle/usb/usbhcidevice.h>

.. cpp:class:: CUSBHCIDevice : public CUSBHostController

	このクラスはCircleアプリケーションにおけるUSBホストサポートのベースとなるものです。USBをホストモードで使用するには、アプリケーションの ``CKernel`` クラス内にこのクラスのメンバを作成する必要があります。

.. note::

	実はCircleには ``CUSBHCIDevice`` というクラスは用意されていません。その代わり、対象となるRaspberry PiモデルのUSBホストコントローラ用として ``CDWHCIDevice`` と ``CXHCIDevice`` （これらは ``CUSBHostController`` を継承）、および ``CUSBSubSystem`` （Raspberry Pi 5用）の3つのクラスが存在し、 ``CUSBHCIDevice`` はこれらのクラスの別名としてマクロ定義されているだけです。Raspberry Piの各モデルに対応したアプリケーションを構築するために ``CUSBHCIDevice`` という名前だけを使用するべきです。

	``CUSBHCIDevice`` で利用できるメソッドの一部は基底クラスである :ref:`CUSBHostController` で定義されており ``CUSBHostController`` オブジェクトへのポインタを使用して呼び出すこともできます。

.. cpp:function:: CUSBHCIDevice::CUSBHCIDevice (CInterruptSystem *pInterruptSystem, CTimer *pTimer, boolean bPlugAndPlay = FALSE)

	このクラスのインスタンスを作成します。 ``pInterruptSystem`` は割り込みシステムオブジェクトへのポインタ、 ``pTimer`` はシステムタイマオブジェクトへのポインタです。 USBプラグアンドプレイのサポートを有効にするためには  ``bPlugAndPlay`` に ``TRUE`` を設定する必要があります。これはオプションであり、アプリケーションによる更なるサポートが必要です。

.. cpp:function:: boolean CUSBHCIDevice::Initialize (boolean bScanDevices = TRUE)

	USBホストサブシステムを初期化します。通常、これにはバススキャンと接続されているすべてのUSBデバイスの初期化が含まれ、これには時間がかかります。このクラスのコンストラクタでUSBプラグアンドプレイが有効になっている場合 (``bPlugAndPlay = TRUE``)、 ``bScanDevices`` をFALSEに設定することによりUSBの初期化のスピードを上げることができます。デバイスの初期化は後で ``UpdatePlugAndPlay()`` が呼び出されるまで延期されます。

.. cpp:function:: void CUSBHCIDevice::ReScanDevices (void)

	このメソッドを呼び出すことにより、このクラスのコンストラクを呼び出した際にUSBプラグアンドプレイサポートが有効になっていなかった場合 (``bPlugAndPlay = FALSE``)に新しく接続されたデバイスをUSBで再スキャンすることができます。

.. _CUSBHostController:

CUSBHostController
""""""""""""""""""

.. code-block:: cpp

	#include <circle/usb/usbhostcontroller.h>

.. cpp:class:: CUSBHostController : public CUSBController

	``CDWHCIDevice`` と ``CXHCIDevice`` （またの名は ``CUSBHCIDevice``）の基底クラスです。以下のメソッドはこれらのクラスのインスタンスでも呼び出すことができます。

.. cpp:function:: boolean CUSBHostController::IsPlugAndPlay (void)

	USBサブシステムでUSBプラグアンドプレイがサポートされている場合、 ``TRUE`` を返します。

.. cpp:function:: boolean CUSBHostController::UpdatePlugAndPlay (void)

	USBプラグアンドプレイが有効な場合、新しいデバイスが接続されたり、デバイスがUSBから取り外された場合に内部のUSBデバイスツリーを更新できるようにこのメソッドを ``TASK_LEVEL`` から継続して呼び出す必要があります。USBデバイスツリーが変更された可能性がある場合は ``TRUE`` を返します。この場合、アプリケーションは ``CDeviceNameService::GetDevice()`` を呼び出して自らがサポートしているデバイスの存在をテストする必要があります。 ``UpdatePlugAndPlay()`` は最初に呼び出した際は常に ``TRUE`` を返します。

.. cpp:function:: static boolean CUSBHostController::IsActive (void)

	USBサブシステムが利用可能な場合は ``TRUE`` を返します。

.. cpp:function:: static CUSBHostController *CUSBHostController::Get (void)

	システムにただ一つだけ存在する（シングルトン） ``CUSBHostController`` (またの名は ``CUSBHCIDevice``) のインスタンスへのポインタを返します。

USBガジェットサポート
^^^^^^^^^^^^^^^^^^^^^^

In gadget (aka device, peripheral) mode an USB controller supports the connection to exactly one USB host.

.. note::

	USB gadget support is currently not available for Raspberry Pi 5.

CUSBCDCGadget
"""""""""""""

.. code-block:: cpp

	#include <circle/usb/gadget/usbcdcgadget.h>

.. cpp:class:: CUSBCDCGadget : public CDWUSBGadget

	This class implements an USB serial CDC gadget, which can transfer data to/from the USB host via a serial interface. The device appears in the host system as a USB serial device (e.g. `/dev/ttyACM0`). To use USB for this purpose, you should create a member of this class in the ``CKernel`` class of your application. Only the constructor of this class is described here. More methods are described for the base class :cpp:class:`CDWUSBGadget`. The gadget driver automatically creates an instance of the interface device :cpp:class:`CUSBSerialDevice`, when the gadget is connected to an USB host.

.. note::

	The `test/usb-serial-cdc-gadget` is prepared to work as a serial CDC gadget. Please read the *README* file in the test's directory for information about the required configuration. You have to define your own USB vendor ID as system option ``USB_GADGET_VENDOR_ID``.

.. cpp:function:: CUSBCDCGadget::CUSBCDCGadget (CInterruptSystem *pInterruptSystem)

	Creates an instance of this class. ``pInterruptSystem`` is a pointer to the interrupt system object.

CUSBMIDIGadget
""""""""""""""

.. code-block:: cpp

	#include <circle/usb/gadget/usbmidigadget.h>

.. cpp:class:: CUSBMIDIGadget : public CDWUSBGadget

	This class implements an USB MIDI (v1.0) gadget, which can receive MIDI events from the USB host (e.g. from a sequencer program) and/or can send MIDI events to the host. To use USB for this purpose, you should create a member of this class in the ``CKernel`` class of your application. Only the constructor of this class is described here. More methods are described for the base class :cpp:class:`CDWUSBGadget`. The gadget driver automatically creates an instance of the interface device :cpp:class:`CUSBMIDIDevice`, when the gadget is connected to an USB host.

.. note::

	The `sample/29-miniorgan` is prepared to work as a MIDI gadget. Please read the *README* file in the sample's directory for information about the required configuration. Beside the define ``USB_GADGET_MODE``, which enables the gadget mode in the sample, you have to define your own USB vendor ID as system option ``USB_GADGET_VENDOR_ID``.

.. cpp:function:: CUSBMIDIGadget::CUSBMIDIGadget (CInterruptSystem *pInterruptSystem)

	Creates an instance of this class. ``pInterruptSystem`` is a pointer to the interrupt system object.

CDWUSBGadget
""""""""""""

.. code-block:: cpp

	#include <circle/usb/gadget/dwusbgadget.h>

.. cpp:class:: CDWUSBGadget : public CUSBController

	This class is the base class of all USB gadgets in Circle. It is supported for the Raspberry Pi models (3)A(+), Zero (2) (W) and 4B only. Only the methods, which are interesting for application usage, are described here. More methods are described for the base class :cpp:class:`CUSBController`.

.. note::

	USB gadgets always support USB plug-and-play in Circle.

.. cpp:function:: virtual const void *CDWUSBGadget::GetDescriptor (u16 wValue, u16 wIndex, size_t *pLength) = 0

	Returns a device-specific USB descriptor. ``wValue`` is a parameter from setup packet (descriptor type (MSB) and index (LSB)). ``wIndex`` is a parameter from setup packet (e.g. language ID for string descriptors). ``pLength`` is a pointer to a variable, which receives the descriptor size. Returns a pointer to the descriptor or ``nullptr``, if it is not available.

.. note::

	You may override this virtual method to provide user-specific (e.g. string) descriptors for your gadget.
