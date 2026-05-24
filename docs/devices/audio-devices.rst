.. _Audio devices:

オーディオデバイス
~~~~~~~~~~~~~~~~~~~~

Circleは複数のハードウェアインタフェース（PWM、I2S、HDMI、USB）とソフトウェアインタフェース（VCHIQ）を介したサウンド生成をサポートしています。I2SハードウェアインタフェースやUSBオーディオストリーミングデバイスを介したサウンドデータの取得も可能です。さらに、USBとシリアルインタフェース（UART）を介したMIDIデータの送受信も行うことができます。後者については :cpp:class:`CSerialDevice` を使ってアプリケーションで実装する必要があります。

.. important::

	USBオーディオストリーミングデバイスのサポートはRaspberry Pi 4、400、5、Compute Module 4でしか利用できません。

	HDMIとVCHIQオーディオインタフェースのサポートは現在のところ、Raspberry Pi 5では利用できません。

すべてのサウンド生成およびキャプチャデバイスの基底クラスは ``CSoundBaseDevice`` です。以下の表に様々なインタフェース用に用意されているクラスを示します。高レベルサポートではさらに例として様々なフォーマットのサウンドデータの変換関数が提供されており、これは他のサウンドクラスにも容易に適用できます。

==============	======================	======================	====================
Interface	Connector		Low level support	Higher level support
==============	======================	======================	====================
PWM		3.5" headphone jack	CPWMSoundBaseDevice	CPWMSoundDevice
I2S		GPIO header		CI2SSoundBaseDevice
HDMI		HDMI(0)			CHDMISoundBaseDevice
USB		Jack of USB device	CUSBSoundBaseDevice
VCHIQ		HDMI or headphone jack	CVCHIQSoundBaseDevice	CVCHIQSoundDevice
==============	======================	======================	====================

.. note::

	The class ``CUSBSoundBaseDevice`` depends on more lower level drivers (e.g. class ``CUSBAudioStreamingDevice``) in the USB library, which are normally not accessed directly by an application. Technical details of the USB audio streaming architecture are described in the file *lib/usb/usbaudiostreaming.cpp*.

いくつかのサンプルプログラムでさまざまなオーディオデバイスの機能を紹介しています。

* sample/12-pwmsound （PWMサウンドデバイスを使用して短いサウンドサンプルを再生）
* sample/29-miniorgan （PWM、HDMI、I2S、USBサウンドデバイス、USBまたはシリアルMIDIを使用し、サウンドコントローラで音量を調整）
* sample/34-sounddevices （1つのアプリケーションに複数のサウンドデバイスを統合）
* sample/42-soundinput (I2SまたはUSBからPWMサウンドデータへの変換および録音)
* addon/vc4/sound/sample (VCHIQインタフェースによるHDMIまたはPWMサウンドのサポート)
* test/sound-controller (複数のサウンドデバイスのサウンドコントローラの制御設定)

別のプロジェクトである `MiniSynth Pi <https://github.com/rsta2/minisynth>`_ はマルチコア環境においてPWM、I2S、USBの各インタフェースを介してサウンドを生成し、USBまたはシリアルMIDIストリームで制御するアプリケーションのより詳細な例です。

CSoundBaseDevice
^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/soundbasedevice.h>

.. cpp:class:: CSoundBaseDevice : public CDevice

	このクラスはCircleにおけるすべてのサウンド生成、サウンドキャプチャクラスの基底クラスです。通常、アプリケーションではこのクラスを直接使用するのではなく、使用するインターフェースに対応する派生クラスをインスタンス化して使用します。この基底クラスはすべてのサウンドクラスに共通するインタフェースを定義しているため、ここで最初に説明します。

	このクラスはサウンドの出力と入力を開始・停止するためのメソッド、各方向ごとに1つあるサウンドキューを設定・操作するためのメソッドを提供します。アプリケーションはこれらのキューを使用して ``Write()``, ``Read()`` メソッドでサウンドデータを提供・取得することができます。あるいは、 ``GetChunk()``, ``PutChunk()`` メソッドをオーバーライドすることで指定したDMAバッファに対してオーディオサンプルを直接書き込み・読み取りを行うことも可能です。

.. important::

	マルチコア環境では、特に断りがない限り、すべてのメソッドはCPUコア0で呼び出されるか、（コールバックの場合は）CPUコア0で呼び出されることになります。

デバイス情報
""""""""""""""""""

.. cpp:function:: unsigned CSoundBaseDevice::GetHWTXChannels (void) const

	ハードウェア出力チャネルの数を返します。このメソッドは、任意のCPUコアから呼び出すことができます。

.. cpp:function:: unsigned CSoundBaseDevice::GetHWRXChannels (void) const

	ハードウェア入力チャネルの数を返します。このメソッドは、任意のCPUコアから呼び出すことができます。

デバイスのアクティベーション
"""""""""""""""""""""""""""""

.. cpp:function:: virtual boolean CSoundBaseDevice::Start (void)

	サウンドデータの送信を開始します。最初の呼び出し時にはデバイスを初期化します。操作が成功した場合、 ``TRUE`` を返します。

.. cpp:function:: virtual void CSoundBaseDevice::Cancel (void)

	サウンドデータの送信を中止します。中止は少し遅れて有効になります。

.. cpp:function:: virtual boolean CSoundBaseDevice::IsActive (void) const

	現在、サウンドデータの送信が実行されている場合、 ``TRUE`` を返します。このメソッドは任意のCPUコアから呼び出すことができます。

出力キュー
""""""""""""

以下のメソッドは書き込みキューを使ったサウンドの出力に使用されます。 ``GetChunk()`` がオーバライドされている場合はこれらは使用されません。

.. cpp:function:: boolean CSoundBaseDevice::AllocateQueue (unsigned nSizeMsecs)

	``Write()`` で使用されるキューを割り当てます。 ``nSizeMsecs`` はストリームのミリ秒単位の継続時間で表したキューのサイズです。

.. cpp:function:: boolean CSoundBaseDevice::AllocateQueueFrames (unsigned nSizeFrames)

	``Write()`` で使用されるキューを割り当てます。 ``nSizeFrames`` はオーディオフレーム数で表したキューのサイズです。

.. cpp:function:: void CSoundBaseDevice::SetWriteFormat (TSoundFormat Format, unsigned nChannels = 2)

	``Write()`` に渡されるサウンドデータのフォーマットを ``Format`` に設定します。 ``nChannels`` は論理チャンネルの数であり、1 から 32 までの値を指定できます。オーディオデバイスが指定された値よりも多くのハードウェアチャンネルをサポートしている場合、残りのチャンネルにはヌルレベルが送信されます。オーディオデバイスが指定された値よりも少ないハードウェアチャンネルしかサポートしていない場合、書き込まれた残りのサウンドサンプルは無視されます。以下の（インターリーブされたリトルエンディアン形式の）書き込みフォーマットが使用可能です。

	* SoundFormatUnsigned8
	* SoundFormatSigned16
	* SoundFormatSigned24 (3バイトを占める)
	* SoundFormatSigned24_32 (4バイトを占める)

.. cpp:function:: int CSoundBaseDevice::Write (const void *pBuffer, size_t nCount)

	``pBuffer`` にあるオーディオサンプルを出力キューに追加します。 ``nCount`` はバイト単位のバッファサイズであり、フレームサイズの倍数でなければなりません。バッファから読み込まれ、正常に処理されたバイト数を返します。この値は ``nCount`` より小さくなる場合があります。その場合は一部のフレームが無視されたことになります。このメソッドは任意のCPUコアから呼び出すことができます。

.. cpp:function:: unsigned CSoundBaseDevice::GetQueueSizeFrames (void)

	出力キューのサイズをフレーム数で返します。このメソッドは任意のCPUコアから呼び出すことができます。

.. cpp:function:: unsigned CSoundBaseDevice::GetQueueFramesAvail (void)

	出力キューの現在利用可能であり、ハードウェアインタフェースへの送信を待機しているフレームの数を返します。このメソッドは任意のCPUコアから呼び出すことができます。

.. cpp:function:: void CSoundBaseDevice::RegisterNeedDataCallback (TSoundDataCallback *pCallback, void *pParam)

	コールバック関数 ``pCallback`` を登録します。この関数はさらなるサウンドデータが必要になったとき、すなわち、キューの少なくとも半分が空になったときに呼び出されます。 ``pParam`` はコールバックに渡されるユーザパラメータです。コールバック関数のプロトタイプは以下の通りです。

.. c:type:: void TSoundDataCallback (void *pParam)

	``pParam`` はユーザバラメタで ``RegisterNeedDataCallback()`` に渡されます。.

入力キュー
"""""""""""

以下のメソッドは読み取りキューを使ったサウンドデータの入力に使用されます。 ``PutChunk()`` がオーバライドさている場合はこれらは使用されません。

.. cpp:function:: boolean CSoundBaseDevice::AllocateReadQueue (unsigned nSizeMsecs)

	``Read()`` で使用されるキューを割り当てます。 ``nSizeMsecs`` はストリームのミリ秒単位の継続時間で表したキューのサイズです。

.. cpp:function:: boolean CSoundBaseDevice::AllocateReadQueueFrames (unsigned nSizeFrames)

	``Read()`` で使用されるキューを割り当てます。 ``nSizeFrames`` はオーディオフレーム数で表したキューのサイズです。

.. cpp:function:: void CSoundBaseDevice::SetReadFormat (TSoundFormat Format, unsigned nChannels = 2, boolean bLeftChannel = TRUE)

	``Read()`` に渡されるサウンドデータのフォーマットを ``Format`` に設定します。 ``nChannels`` は論理チャンネルの数であり、1 から 32 までの値を指定できます。オーディオデバイスが指定された値よりも多くのハードウェアチャンネルをサポートしている場合、残りのチャンネルは無視されます。オーディオデバイスが指定された値よりも少ないハードウェアチャンネルしかサポートしていない場合、残りのreadサウンドサンプルはヌルレベルを返します。 ``bLeftChannel`` が ``TRUE`` の場合、 ``nChannels == 1`` であれば、 ``Read()`` は左チャンネルを返します。以下の（インターリーブされたリトルエンディアン形式の）読み取り形式が使用可能です。

	* SoundFormatUnsigned8
	* SoundFormatSigned16
	* SoundFormatSigned24 (3バイトを占める)
	* SoundFormatSigned24_32 (4バイトを占める)

.. cpp:function:: int CSoundBaseDevice::Read (void *pBuffer, size_t nCount)

	入力キューから最大 ``nCount`` バイトのオーディオサンプルを ``pBuffer`` に転送し、転送されたバイト数を返します。この値は常にフレームサイズの倍数となりますが、データがなかった場合は 0 を返します。 ``nCount`` はフレームサイズの倍数でなければなりません。このメソッドは任意のCPUコアから呼び出すことができます。

.. cpp:function:: unsigned CSoundBaseDevice::GetReadQueueSizeFrames (void)

	入力キューのサイズをフレーム数で返します。このメソッドは任意のCPUコアから呼び出すことができます。

.. cpp:function:: unsigned CSoundBaseDevice::GetReadQueueFramesAvail (void)

	入力キューにある現在利用可能であり、アプリケーションによる読み取りを待機しているフレームの数を返します。このメソッドは任意のCPUコアから呼び出すことができます。

.. cpp:function:: void CSoundBaseDevice::RegisterHaveDataCallback (TSoundDataCallback *pCallback, void *pParam)

	コールバック関数 ``pCallback`` を登録します。この関数は ``Read()`` を実行するのに十分なサウンドデータが利用可能になったとき、すなわち、キューの少なくとも半分が埋まったときに呼び出されます。 ``pParam`` はコールバックに渡されるユーザパラメータです。このコールバック関数のプロトタイプは :c:func:`TSoundDataCallback` です。

代替インタフェース
"""""""""""""""""""

アプリケーションは必要に応じて出力キューや入力キューをバイパスして、直接、バッファからオーディオサンプルを提供したり、バッファへオーディオサンプルを格納したりすることができます。このバッファは、コールバックメソッドである ``GetChunk()``, ``PutChunk()`` に渡されます。この代替インタフェースを使用するにはこれらのメソッドをオーバーライドする必要があります。サンプルのフォーマットは使用されるハードウェア/ソフトウェアインタフェースによって異なります。

==============  ==============================================  ====================================================
インタフェース    フォーマット               備考
==============  ==============================================  ====================================================
PWM             SoundFormatUnsigned32                           Range MaxはサンプリングレートとPWMクロックレートによる
I2S             SoundFormatSigned24_32                          4バイトを占める
HDMI            SoundFormatIEC958                               特別なフレームフォーマット (S/PDIF)
USB             SoundFormatSigned16 または SoundFormatSigned24
VCHIQ           SoundFormatSigned16                             4バイトを占める
==============	==============================================  ====================================================

.. cpp:function:: virtual int CSoundBaseDevice::GetRangeMin (void) const
.. cpp:function:: virtual int CSoundBaseDevice::GetRangeMax (void) const

	1サンプルの最小値/最大値を返します。これらのメソッドは任意のCPUコアから呼び出すことができます。

.. cpp:function:: boolean CSoundBaseDevice::AreChannelsSwapped (void) const

	アプリケーションが ``GetChunk()`` で右チャンネルを先にバッファに書き込む必要がある場合、 ``TRUE`` を返します。

.. cpp:function:: virtual unsigned CSoundBaseDevice::GetChunk (s16 *pBuffer, unsigned nChunkSize)
.. cpp:function:: virtual unsigned CSoundBaseDevice::GetChunk (u32 *pBuffer, unsigned nChunkSize)

	これらのメソッドのいずれかをオーバーライドして、サウンドサンプルを提供することができます。最初のメソッドはVCHIQインタフェースとUSBインタフェースで使用され、2番目のメソッドはその他のすべてのインタフェース（各サンプルが3バイトを占める24ビット解像度のUSBを含む）で使用されます。 ``pBuffer`` はサンプルを格納するバッファへのポインタです。 ``nChunkSize`` はワード単位のバッファサイズです。バッファに書き込まれたワード数を返します。この値は通常 ``nChunkSize`` ですが、転送を停止する場合は 0 を返します。各サンプルは ``GetHWTXChannels()`` 個のワードで構成されます。各ワードは ``GetRangeMin()`` から ``GetRangeMax()`` の範囲である必要があります。HDMI インタフェースではここで特別なフレーム形式が必要となり ``ConvertIEC958Sample()`` を使用して適用できます。

.. cpp:function:: virtual void CSoundBaseDevice::PutChunk (const s16 *pBuffer, unsigned nChunkSize)
.. cpp:function:: virtual void CSoundBaseDevice::PutChunk (const u32 *pBuffer, unsigned nChunkSize)

	これらのメソッドをオーバーライドして、受信したサウンドサンプルを処理することができます。最初のメソッドはUSBインタフェース用であり、2番目のメソッドはI2S（または、各サンプルが3バイトを占める24ビット解像度の場合はUSB）用です。 ``pBuffer`` はサンプルが格納されているバッファへのポインタです。 ``nChunkSize`` はワード単位のバッファサイズです。各サンプルは ``GetHWRXChannels()`` 個のワードで構成されます。

.. cpp:function:: u32 CSoundBaseDevice::ConvertIEC958Sample (u32 nSample, unsigned nFrame)

	このメソッドは ``GetChunk()`` から呼び出して、IEC958 (S/PDIF) 形式のサンプルにフレーミングを適用することができます。 ``nSample`` は ``u32`` 型の 24ビット符号付きサンプル値であり、上位ビットは無視されます。 ``nFrame`` はIEC958フレームの数であり、 (0..191) の間の数です。

.. _Sound controller:

サウンドコントローラ
""""""""""""""""""""

サウンドデバイスはオプションでサウンドコントローラを提供することができ、以下の機能を提供します。

* デバイスの出力/入力のプロパティに関する情報を返します。
* 複数のコネクタやコネクタ構成を持つサウンドデバイスにおいて、特定のジャックを有効にします。
* 特定のジャックを無効にします（マルチジャック操作のみ）。
* サウンドの出力/入力に影響を与えるオーディオコントロール（音量など）に関する情報を返す。
* オーディオコントロールの特定の値を設定する（ミュートのオン/オフなど）。

.. cpp:function:: virtual CSoundController *CSoundBaseDevice::GetController (void)

	This method returns a pointer to the sound controller object of a sound device or ``nullptr``, if a sound controller is not supported or not (yet) available. The sound controller is only available, after :cpp:func:`CSoundBaseDevice::Start()` has been called for the sound device, and only while the device is active.

.. code-block:: cpp

	#include <circle/sound/soundcontroller.h>

.. cpp:class:: CSoundController

	This class represents the interface of a sound controller to an application. A pointer to a sound controller object can be fetched by calling :cpp:func:`CSoundBaseDevice::GetController()` on a created driver object for a sound device.

	Please note that the enum values, given below, are valid in the name space of the class ``CSoundController`` only, so you have to use the prefix ``CSoundController::`` on them.

.. important::

	Methods of the sound controller can be called only at ``TASK_LEVEL``.

.. cpp:function:: u32 CSoundController::GetOutputProperties (void) const
.. cpp:function:: u32 CSoundController::GetInputProperties (void) const

	Returns a bit-mask with values defined in :cpp:enum:`CSoundController::TProperty` or'ed together. The first method returns the properties of the output direction of the controlled sound device, the second method the properties of the input direction of the device.

.. cpp:enum:: CSoundController::TProperty

	The follwing values are defined:

	* PropertyDirectionSupported (Is the respective output / input direction supported?)
	* PropertyMultiJackOperation (Is it possible to enable multiple jacks at once for this direction?)

.. cpp:function:: boolean CSoundController::EnableJack (TJack Jack)

	Enables the specified ``Jack`` of the sound device. Returns ``TRUE`` on success. This method can be called multiple times for different jacks, if ``PropertyMultiJackOperation`` is available. Otherwise a call to this method automatically disables the previously active jack.

.. cpp:enum:: CSoundController::TJack

	Output jacks:

	* JackDefaultOut (default or currently active output jack)
	* JackLineOut
	* JackSpeaker
	* JackHeadphone
	* JackHDMI
	* JackSPDIFOut

	Input jacks:

	* JackDefaultIn (default or currently active input jack)
	* JackLineIn
	* JackMicrophone

.. cpp:function:: boolean CSoundController::DisableJack (TJack Jack)

	Disables a specific ``Jack`` of the sound device. Returns ``TRUE`` on success. This method always fails without ``PropertyMultiJackOperation`` available.

.. cpp:function:: const CSoundController::TControlInfo CSoundController::GetControlInfo (TControl Control, TJack Jack, TChannel Channel) const

	Returns information about a specific ``Control`` of a specific ``Jack`` and ``Channel`` of a sound device. Please note that a control may be supported for ``ChannelAll``, but not for ``ChannelLeft`` and ``ChannelRight``.

.. cpp:enum:: CSoundController::TControl

	* ControlMute (mute value is 0 (disable) or 1 (enable))
	* ControlVolume (volume value in dB)
	* ControlALC (Automatic Level Control, 0 (disable) or 1 (enable))

.. cpp:enum:: CSoundController::TChannel

	* ChannelAll (all channels)
	* ChannelLeft = Channel1
	* ChannelRight = Channel2
	* Channel3
	* ...
	* Channel32

.. cpp:struct:: CSoundController::TControlInfo

.. code:: c++

	struct TControlInfo
	{
		boolean	Supported;	// Is control supported?
		int	RangeMin;	// Minimum allowed value
		int	RangeMax;	// Maximum allowed value
	};

.. cpp:function:: boolean CSoundController::SetControl (TControl Control, TJack Jack, TChannel Channel, int nValue)

	Sets the value ``nValue`` of a specific ``Control`` of a specific ``Jack`` and affected ``Channel`` of a sound device. Returns ``TRUE`` on success.

CPWMSoundBaseDevice
^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/pwmsoundbasedevice.h>

.. cpp:class:: CPWMSoundBaseDevice : public CSoundBaseDevice

	このクラスはPWMサウンドインタフェース用のドライバです。生成されたサウンドはほとんどのRaspberry Piモデルに搭載されている3.5mmヘッドフォンジャックから出力されます。このクラスで使用可能なメソッドのほとんどは基底クラスである :cpp:class:`CSoundBaseDevice` によって提供されています。このクラス固有のメソッドはコンストラクタだけです。このデバイスはデバイス名サービス（キャラクタデバイス）において ``"sndpwm"`` という名前で登録されています。

.. note::

	ヘッドフォンジャックを搭載していないRaspberry Pi 5とZeroでは、GPIOヘッダーを介してPWMサウンドインタフェースからの出力を利用できます。これには通常、GPIO12/13に接続する `このような <https://learn.adafruit.com/adding-basic-audio-ouput-to-raspberry-pi-zero>`_ 外部インターフェースが必要です。Raspberry Pi Zeroでこれを行うにはシステムオプション ``USE_PWM_AUDIO_ON_ZERO`` を定義する必要があります。詳細については `include/circle/sysconfig.h <https://github.com/rsta2/circle/blob/master/include/circle/sysconfig.h>`_ ファイルを参照してください。

.. cpp:function:: CPWMSoundBaseDevice::CPWMSoundBaseDevice (CInterruptSystem *pInterrupt, unsigned nSampleRate = 44100, unsigned nChunkSize = 2048)

	このクラスのインスタンスを作成します。インスタンスは1つしか存在できません。 ``pInterrupt`` は割り込みシステムオブジェクトへのポインタです。 ``nSampleRate`` はHz単位のサンプルレートです。 ``nChunkSize`` は ``GetChunk()`` を1回呼び出すごとに処理されるサンプル数（ワード数）の2倍です（ステレオチャンネルごとに1ワード）。この値を小さくすると、このインタフェースのレイテンシは減少しますが、CPUコア0へのIRQ負荷が増加します。

CPWMSoundDevice
^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/pwmsounddevice.h>

.. cpp:class:: CPWMSoundDevice : public CPWMSoundBaseDevice

	このクラスはメインメモリ上に存在するサウンドデータを再生するためのPWM再生デバイスです。 :cpp:class:`CPWMSoundBaseDevice` クラスを継承していますが、独自のインターフェースを持っています。サンプリングレートは44100 Hzに固定されています。

.. cpp:function:: CPWMSoundDevice::CPWMSoundDevice (CInterruptSystem *pInterrupt)

	このクラスのインスタンスを作成します。インスタンスは1つしか存在できません。 ``pInterrupt`` は割り込みシステムオブジェクトへのポインタです。

.. cpp:function:: void CPWMSoundDevice::Playback (void *pSoundData, unsigned nSamples, unsigned nChannels, unsigned  nBitsPerSample)

	PWMサウンドデバイスを使用して、 ``pSoundData`` のサウンドデータの再生を開始します。 ``nSamples`` はサンプル数であり、ステレオの場合、L/Rサンプルが1つとしてカウントされます。 ``nChannels`` はモノラルなら1、ステレオなら2です。 ``nBitsPerSample`` は 8（unsigned char形式のサウンドデータ）か 16（signed short形式のサウンドデータ）です。

.. cpp:function:: boolean CPWMSoundDevice::PlaybackActive (void) const

	再生中の場合は ``TRUE`` を返します。

.. cpp:function:: void CPWMSoundDevice::CancelPlayback (void)

	再生を中止します。この操作はわずかな遅延を経て有効になり、その後、 ``PlaybackActive()`` は  ``FALSE`` を返します。

CI2SSoundBaseDevice
^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/i2ssoundbasedevice.h>

.. cpp:class:: CI2SSoundBaseDevice : public CSoundBaseDevice

	This class is a driver for the I2S sound interface. The generated sound is available via the GPIO header in the format: two 32-bit wide channels with 24-bit signed data. Most of the methods, available for using this class, are provided by the base class :cpp:class:`CSoundBaseDevice`. Only the constructor is specific to this class. This device has the name ``"sndi2s"`` in the device name service (character device).

.. note::

	The following GPIO pins have to be connected (SoC numbers, not header positions):

	==============	==============	===============	==================================
	Name		Pin number	On early models	Description
	==============	==============	===============	==================================
	PCM_CLK		GPIO18		GPIO28		Bit clock (output or input)
	PCM_FS		GPIO19		GPIO29		Frame clock (output or input)
	PCM_DIN		GPIO20		GPIO30		Data input (not for TX only mode)
	PCM_DOUT	GPIO21		GPIO31		Data output (not for RX only mode)
	==============	==============	===============	==================================

	The clock pins are outputs in master mode, or inputs in slave mode. On early models the signals are exposed on the separate P5 header.

	The Raspberry Pi 5 can be used in an 8 channel mode (TX master only), with this GPIO pin assignment:

	==============	==============	==================================
	Name		Pin number	Description
	==============	==============	==================================
	I2S0_SCLK	GPIO18		Bit clock (output)
	I2S0_WS		GPIO19		Frame clock (output)
	I2S0_SDO[0]	GPIO21		Data output (channels 0 and 1)
	I2S0_SDO[1]	GPIO23		Data output (channels 2 and 3)
	I2S0_SDO[2]	GPIO25		Data output (channels 4 and 5)
	I2S0_SDO[3]	GPIO27		Data output (channels 6 and 7)
	==============	==============	==================================

	The clock signals are shared between the data lines. This mode can be used with the HifiBerry DAC8x.

.. note::

	This driver class supports several I2S interfaces. Some interfaces require an additional I2C connection to work. The following interfaces are known to work:

	* pHAT DAC (with PCM5102A DAC)
	* PiFi DAC+ v2.0 (with PCM5122 DAC)
	* `Adafruit I2S Audio Bonnet <https://www.adafruit.com/product/4037>`_ (with UDA1334A DAC)
	* `Adafruit I2S 3W Class D Amplifier Breakout <https://www.adafruit.com/product/3006>`_ (with MAX98357A DAC)
	* `Waveshare WM8960 Audio HAT <https://www.waveshare.com/wm8960-audio-hat.htm>`_ (with WM8960 DAC, sample rates 44100 and 48000 only)

.. cpp:function:: CI2SSoundBaseDevice::CI2SSoundBaseDevice (CInterruptSystem *pInterrupt, unsigned nSampleRate = 192000, unsigned nChunkSize = 8192, boolean bSlave = FALSE, CI2CMaster *pI2CMaster = 0, u8 ucI2CAddress = 0, TDeviceMode DeviceMode  = DeviceModeTXOnly, unsigned nHWChannels = 2)

	Constructs an instance of this class. There can be only one. ``pInterrupt`` is  a pointer to the interrupt system object. ``nSampleRate`` is the sample rate in Hz. ``nChunkSize`` is twice the number of samples (words) to be handled with one call to ``GetChunk()`` (one word per stereo channel). Decreasing this value also decreases the latency on this interface, but increases the IRQ load on CPU core 0.

	``bSlave`` enables the slave mode (PCM clock and FS clock are inputs). ``pI2CMaster`` is a pointer to an I2C master object (0 if no I2C DAC initialization is required). ``ucI2CAddress`` is the I2C slave address of the DAC (0 for auto probing the addresses 0x4C, 0x4D and 0x1A). ``DeviceMode`` selects, which transfer direction will be used, with these supported values:

	* DeviceModeTXOnly (output)
	* DeviceModeRXOnly (input)
	* DeviceModeTXRX (output and input)

	``nHWChannels`` specifies the number of hardware channels (normally 2, can be 8 on the Raspberry Pi 5 for output).

.. note::

	On the Raspberry Pi 5 only the following sample rates are supported: 8000, 11025, 16000, 22050, 32000, 44100, 48000, 88200, 96000, 176400, 192000.

CUSBSoundBaseDevice
^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/usbsoundbasedevice.h>

.. cpp:class:: CUSBSoundBaseDevice : public CSoundBaseDevice

	This class is a driver for USB audio streaming devices. Most of the methods, available for using this class, are provided by the base class :cpp:class:`CSoundBaseDevice`. Only the constructor is specific to this class. The first device has the name ``"sndusb"`` in the device name service (character device). If there are multiple instances of this class, the second instance has the name ``"sndusb1"`` on so on.

.. important::

	The support for USB audio streaming devices is only available on the Raspberry Pi 4, 400 and Compute Module 4.

.. note::

	Circle supports USB audio streaming devices with up to 32 PCM channels and 16-bit (default) or 24-bit resolution. For the latter the option ``soundopt=24`` has to be specified in the file *cmdline.txt*. The number of channels has to be selected with the option `usbsoundchannels=TX,RX <https://github.com/rsta2/circle/blob/master/doc/cmdline.txt#L37>`_ in the same file.

.. cpp:function:: CUSBSoundBaseDevice::CUSBSoundBaseDevice (unsigned nSampleRate = 48000, TDeviceMode DeviceMode = DeviceModeTXOnly, unsigned nDevice = 0)

	Constructs an instance of this class. ``nSampleRate`` is the sample rate in Hz. The selected value must be supported by the attached USB audio streaming device (48000 should work with most devices). ``DeviceMode`` selects, which transfer direction will be used, with these supported values:

	* DeviceModeTXOnly (output)
	* DeviceModeRXOnly (input)
	* DeviceModeTXRX (output and input)

	Theoretically there may be multiple instances of this class at once. ``nDevice`` selects the attached USB audio streaming device to be accessed (0 is the first one found in USB device enumeration).

.. important::

	The class ``CUSBSoundBaseDevice`` must be instantiated, when the USB host controller is initialized already. Therefore it cannot be a class member of the class ``CKernel``. Use a pointer to the driver object instead and create it with the ``new`` operator.

CHDMISoundBaseDevice
^^^^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/sound/hdmisoundbasedevice.h>

.. cpp:class:: CHDMISoundBaseDevice : public CSoundBaseDevice

	このクラスはオーディオ対応のHDMIディスプレイ用のドライバです。ハードウェアに直接アクセスするため、 :ref:`Multitasking` のサポートやシステムに  :ref:`VCHIQ driver` を必要としません。このクラスで使用可能なメソッドのほとんどは基底クラスである :cpp:class:`CSoundBaseDevice` で提供されています。このデバイスはデバイス名サービス（キャラクタデバイス）において ``"sndhdmi"`` という名前で登録されています。

.. note::

	このドライバは2チャンネル（ステレオ）しかサポートしていません。

	このドライバはRaspberry Pi 4、5、400のHDMI1はサポートしていません（HDMI0のみです）。

	このドライバはDMAモードとポーリングモードをサポートしています。後者は、割り込みを使用できない、非常に時間的制約が厳しく、キャッシュの影響を受けやすいアプリケーションを対象としています。

.. note::

	Circleの44.5より前のリリースではこのドライバはステレオ信号のチャンネルを反転させていました。この問題は本リリース以降で修正されています。

.. cpp:function:: CHDMISoundBaseDevice::CHDMISoundBaseDevice (CInterruptSystem *pInterrupt, unsigned nSampleRate = 48000, unsigned nChunkSize = 384 * 10)

	DMAモードで動作するこのクラスのインスタンスを作成します。インスタンスは1つしか存在できません。 ``pInterrupt`` は割り込みシステムオブジェクトへのポインタです。 ``nSampleRate`` はHz単位のサンプルレートです。 ``nChunkSize``は ``GetChunk()`` の1回の呼び出しで処理されるサンプル数（ワード数）の2倍です（ステレオチャンネルごとに1ワード、384の倍数でなければなりません）。この値を小さくすると、このインタフェースのレイテンシは減少しますが、CPUコア0へのIRQ負荷が増加します。

.. cpp:function:: CHDMISoundBaseDevice::CHDMISoundBaseDevice (unsigned nSampleRate = 48000)

	ポーリングモードで動作するこのクラスのインスタンスを作成します。インスタンスは1つしか存在できません。 ``nSampleRate`` はHz単位のサンプルレートです。

.. cpp:function:: boolean CHDMISoundBaseDevice::IsWritable (void)

	データFIFOに少なくとも1つのサンプルを書き込むための空き容量があるか否かを返します。このメソッドはポーリングモードでしか呼び出すことができません。

.. cpp:function:: void CHDMISoundBaseDevice::WriteSample (s32 nSample)

	データFIFOにサンプルを1つ書き込みます。 ``nSample`` は書き込み対象となる24ビットの符号付きサンプルです。このメソッドはポーリングモードでしか呼び出すことができず、かつ、事前に :cpp:func:`IsWritable()` が ``TRUE`` を返した場合にしか呼び出すことができません。各フレームにつき2回（左チャンネルと右チャンネル用）呼び出す必要があります。

CVCHIQSoundBaseDevice
^^^^^^^^^^^^^^^^^^^^^

.. note::

	このクラスはRaspberry Pi 5では利用できません。

.. code-block:: cpp

	#include <vc4/sound/vchiqsoundbasedevice.h>

.. cpp:class:: CVCHIQSoundBaseDevice : public CSoundBaseDevice

	このクラスは、VCHIQサウンドサービスへの低レベルなアクセスを提供します。VCHIQサウンドサービスはオーディオ対応のHDMIディスプレイと3.5インチヘッドフォンジャックを備えたRaspberry Piモデルでサウンドを出力することができます。このクラスを使用するには、システムに :ref:`Multitasking` サポートと :ref:`VCHIQ driver` がインストールされている必要があります。このクラスで使用可能なメソッドのほとんどは基底クラスである :cpp:class:`CSoundBaseDevice` で提供されています。このクラスの説明ではこのクラス固有のメソッドについてのみ説明します。このデバイスはデバイス名サービス（キャラクタデバイス）において ``"sndvchiq"`` という名前を持ちます。

.. cpp:function:: CVCHIQSoundBaseDevice::CVCHIQSoundBaseDevice (CVCHIQDevice *pVCHIQDevice, unsigned nSampleRate = 44100, unsigned nChunkSize  = 4000, TVCHIQSoundDestination Destination = VCHIQSoundDestinationAuto)

	このクラスのインスタンスを作成します。インスタンスは1つしか存在できません。
	``pVCHIQDevice`` は、VCHIQインタフェースデバイスへのポインタです。 ``nSampleRate`` は、
	Hz 単位のサンプリングレート（44100～48000）です。 ``nChunkSize`` は、一度に転送される
	サンプル数です。 ``Destination`` は、サウンドデータが送信される宛先デバイスです
	（ ``VCHIQSoundDestinationAuto`` の場合は自動的に検出されます）。以下の値が指定可能です。

.. c:enum:: TVCHIQSoundDestination

	* VCHIQSoundDestinationAuto
	* VCHIQSoundDestinationHeadphones
	* VCHIQSoundDestinationHDMI
	* VCHIQSoundDestinationUnknown

.. cpp:function:: void CVCHIQSoundBaseDevice::SetControl (int nVolume, TVCHIQSoundDestination Destination = VCHIQSoundDestinationUnknown)

	出力音量を ``nVolume`` （-10000～400、1/100 dB単位）に設定し、必要に応じて出力先を
	``Destination`` に設定します（ ``VCHIQSoundDestinationUnknown`` の場合は変更されません）。
	このメソッドは、サウンドデータが送信中の間も呼び出すことができます。音量を指定するために以下の
	マクロが定義されています。

.. c:macro:: VCHIQ_SOUND_VOLUME_MIN
.. c:macro:: VCHIQ_SOUND_VOLUME_DEFAULT
.. c:macro:: VCHIQ_SOUND_VOLUME_MAX

.. note::

	:ref:`Sound controller` はサウンドデバイスのコンロール設定を行うためのより汎用的なソリューションを提供しています。

CVCHIQSoundDevice
^^^^^^^^^^^^^^^^^

.. note::

	このクラスはRaspberry Pi 5では利用できません。

.. code-block:: cpp

	#include <vc4/sound/vchiqsounddevice.h>

.. cpp:class:: CVCHIQSoundDevice : private CVCHIQSoundBaseDevice

	このクラスはメインメモリ上に存在するサウンドデータ用のVCHIQ再生デバイスです。 :cpp:class:`CVCHIQSoundBaseDevice` クラスを拡張していますが独自のインターフェースを持っています。サンプリングレートは44100 Hzに固定されています。

.. cpp:function:: CVCHIQSoundDevice::CVCHIQSoundDevice (CVCHIQDevice *pVCHIQDevice, TVCHIQSoundDestination Destination = VCHIQSoundDestinationAuto)

	このクラスのインスタンスを作成します。インスタンスは1つしか存在できません。 ``pVCHIQDevice`` は
	VCHIQインタフェースデバイスへのポインタです。 ``Destination`` はサウンドデータが送信される
	ターゲットデバイスです（利用可能なオプションについては :c:enum:`TVCHIQSoundDestination` を
	参照してください）。

.. cpp:function:: boolean CVCHIQSoundDevice::Playback (void *pSoundData, unsigned nSamples, unsigned nChannels, unsigned nBitsPerSample)

	VCHIQサウンドデバイス経由で ``pSoundData`` のサウンドデータの再生を開始します。
	``nSamples`` はサンプル数であり、ステレオの場合、L/Rのサンプルは1つとしてカウントされます。
	``nChannels`` は、モノラルなら1、ステレオなら2です。 ``nBitsPerSample`` は8
	（unsigned charのサウンドデータ）または16（singed shortのサウンドデータ）です。
	成功した場合は ``TRUE`` を返します。

.. cpp:function:: boolean CVCHIQSoundDevice::PlaybackActive (void) const

	再生中の場合は ``TRUE`` を返します。

.. cpp:function:: void CVCHIQSoundDevice::CancelPlayback (void)

	再生をキャンセルします。キャンセルされるには少し時間がかかります。これ以後、The operation takes  ``PlaybackActive()`` は ``FALSE`` を返します。

.. cpp:function:: void CVCHIQSoundDevice::SetControl (int nVolume, TVCHIQSoundDestination Destination = VCHIQSoundDestinationUnknown)

	:cpp:func:`CVCHIQSoundBaseDevice::SetControl()` を参照してください.

CUSBMIDIDevice
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/usb/usbmidi.h>

.. cpp:class:: CUSBMIDIDevice : public CDevice

	このクラスはUSBオーディオクラスMIDI 1.0デバイス用のインタフェースデバイスです。USBデバイスのエヌメレーション処理において互換性のあるデバイスが検出されると、このクラスのインスタンスが自動的に作成されます。そのため、ここでは初期化に使用されるメソッドではなく、アプリケーションがUSB MIDIデバイスを使用するために必要なクラスメソッドだけを説明します。このデバイスはデバイス名サービス（キャラクタデバイス）において ``"umidiN"`` (N >= 1) という名前を持ちます。

.. note::

	USB MIDIパケットと仮想MIDIケーブルに関する情報は `Universal Serial Bus Device Class Definition for MIDI Devices, Release 1.0 <https://usb.org/document-library/usb-midi-devices-10>`_ を参照してください。

.. cpp:function:: void CUSBMIDIDevice::RegisterPacketHandler (TMIDIPacketHandler *pPacketHandler)

	Registers a callback function, which is called, when a MIDI packet arrives. ``pPacketHandler`` is a pointer to the function, which has the following prototype:

.. c:type:: void TMIDIPacketHandler (unsigned nCable, u8 *pPacket, unsigned nLength)

	``nCable`` is the number of the virtual MIDI cable (0..15). ``pPacket`` is a pointer to one received MIDI packet. ``nLength`` is the number of valid bytes in the packet (1..3).

.. cpp:function:: void CUSBMIDIDevice::RegisterPacketHandler (TMIDIPacketHandlerEx *pPacketHandler, void *pParam)

	Alternative version of ``RegisterPacketHandler()``, which gets an additional user parameter, which is handed over to this callback function:

.. c:type:: void TMIDIPacketHandlerEx (unsigned nCable, u8 *pPacket, unsigned nLength, unsigned nDevice, void *pParam)

.. cpp:function:: boolean CUSBMIDIDevice::SendEventPackets (const u8 *pData, unsigned nLength)

	Sends one or more packets in the encoded USB MIDI event packet format. ``pData`` is a pointer to the packet buffer. ``nLength`` is the length of the packet buffer in bytes, which must be a multiple of 4. Returns ``TRUE``, if the operation has been successful. This function fails, if ``nLength`` is not a multiple of 4 or the send function is not supported. The format of the USB MIDI event packets is not validated.

.. cpp:function:: boolean CUSBMIDIDevice::SendPlainMIDI (unsigned nCable, const u8 *pData, unsigned nLength, unsigned nChunkSize = 0)

	Sends one or more messages in plain MIDI message format. ``nCable`` is the number of the virtual MIDI cable (0..15). ``pData`` is a pointer to the message buffer. ``nLength`` is the length of the message buffer in bytes. The MIDI data is sent in ``nChunkSize`` number of bytes (multiple of 4), if this parameter is not 0. Returns ``TRUE``, if the operation has been successful. This function fails, if the message format is invalid or the send function is not supported.

.. cpp:function:: void CUSBMIDIDevice::SetAllSoundOffOnUSBError (boolean bEnable)

	If this method has been called with ``bEnable`` equal to ``TRUE``, the driver generates MIDI Control Change "All Sound Off" (120) messages for each MIDI channel (1-16) on MIDI cable 0, when an USB error is detected by the driver.
