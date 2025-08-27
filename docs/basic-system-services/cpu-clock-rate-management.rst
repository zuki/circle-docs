CPUクロックレート管理
~~~~~~~~~~~~~~~~~~~~~~~~~

Raspberry Piのほとんどのモデルでは最大限のパフォーマンスを発揮するためにベアメタルアプリケーションによるCPUクロックレートの管理が必要です。この管理ではCPU（実際にはSoC）の現在の温度を継続的に測定して、温度が高くなりすぎた場合にARM CPUのクロックレートを下げるように調整します。

許容されるCPU温度の絶対最大値は85℃です。ファームウェアは自動的にこの制限を超えないようにしています。温度がこの値に近づくとファームウェアは画面右上に警告アイコンを表示します。詳しくは `Frequency management and thermal control <https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#frequency-management-and-thermal-control>`_ を参照してください。

最大CPUクロックレートはRaspberry Piのモデルによって異なり、起動後に設定されるARM CPUの周波数も異なります。

==============  ======================  =========================  ======================
Raspberry Pi    最大CPUクロックレート   起動時のCPUクロックレート  コメント
==============	======================  =========================  ======================
1               700 MHz                 700 MHz                    管理不要
2               900 MHz                 600 MHz
Zero            1000 MHz                700 MHz
Zero 2          000 MHz                 600 MHz
3 Model B       1200 MHz                600 MHz
3 Model A+/B+   1400 MHz                600 MHz
4               1500 MHz                600 MHz                    Rev. 1.4: 1800 MHz
400             1800 MHz                600 MHz                    Head sink included
5               2400 MHz                1500 MHz
==============	======================  =========================  ======================

Circle は以下で説明するように ``CCPUThrottle`` クラスを使って CPU のクロックレート管理を実装しています。サンプルプログラム `26-cpustress` はその使い方を示しています。

.. important::

	ブート後のARM CPUのCPUクロックレートは、ほとんどのRaspberry Piモデルで許容される最大値に設定されていません。このままではベアメタルアプリケーションは最大のパフォーマンスで動作しません。

	アプリケーションが常に最大のパフォーマンスが必要であるが、CPUがクロックダウンしている際に対処できない場合は、ヘッドシンクやファンを取り付ける必要があるかもしれません。

	``CCPUThrottle`` はI2CやSPIによる転送を行うコードと一緒に使用しないでください。CPU クロックのクロックレートが変化するとCORE クロックにも影響するため、転送速度が変化する可能性があるからです。

.. note::

	CPUのパフォーマンスを最大レベルに保つためにケースファンを使用することができます。特に公式のRaspberry Pi 4ケースには専用のファンが存在します。このファンには制御ラインがあり、GPIOピンに接続する必要があります。このようなファンを ``CCPUThrottle`` クラスで使用するには :ref:`cmdline` ファイルに ``gpiofanpin=PIN`` オプションを追加する必要があります。ここで、PIN はファンの制御ラインを接続した GPIO ピン番号（ヘッダーの位置ではなく SoC の番号）です。このオプションを使用した場合、CPU速度はスロットルされません。

	Raspberry Pi 5には専用のPWMファンコネクタがあり、Raspberry Pi 5用の公式ケースまたはActive Coolerソリューションと組み合わせて使用することができます。どちらも冷却ファンを搭載しています。これらのソリューションのいずれかを使用する場合は :ref:`cmdline` ファイルに ``gpiofanpin=45`` オプションを指定する必要があります。Raspberry Pi 4のケースファン同様 ``CCPUThrottle`` クラスは現在のSoC温度に応じてファンのオン/オフを切り替えます。PWMはまだ使っていません。

CCPUThrottle
^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/cputhrottle.h>

.. cpp:class:: CCPUThrottle

.. cpp:function:: CCPUThrottle::CCPUThrottle (TCPUSpeed InitialSpeed = CPUSpeedUnknown)

	Creates the class ``CCPUThrottle``. ``InitialSpeed`` is the CPU speed to be set initially, with these possible values:

	* CPUSpeedLow
	* CPUSpeedMaximum
	* CPUSpeedUnknown

	If ``CPUSpeedUnknown`` is selected as initial speed and the parameter ``fast=true`` is set in the file `cmdline.txt <https://github.com/rsta2/circle/blob/master/doc/cmdline.txt>`_, the resulting setting will be ``CPUSpeedMaximum``, or ``CPUSpeedLow`` if not set.

.. cpp:function:: static CCPUThrottle *CCPUThrottle::Get (void)

	Returns a pointer to the only ``CCPUThrottle`` object in the system (if any).

.. cpp:function:: boolean CCPUThrottle::IsDynamic (void) const

	Returns if CPU clock rate change is supported. Other Methods can be called in any case, but may be nop's or return invalid values, if ``IsDynamic()`` returns ``FALSE``.

.. cpp:function:: unsigned CCPUThrottle::GetClockRate (void) const

	Returns the current CPU clock rate in Hz or zero on failure.

.. cpp:function:: unsigned CCPUThrottle::GetMinClockRate (void) const

	Returns the minimum CPU clock rate in Hz.

.. cpp:function:: unsigned CCPUThrottle::GetMaxClockRate (void) const

	Returns the maximum CPU clock rate in Hz.

.. cpp:function:: unsigned CCPUThrottle::GetTemperature (void) const

	Returns the current CPU (SoC) temperature in degrees Celsius or zero on failure.

.. cpp:function:: unsigned CCPUThrottle::GetMaxTemperature (void) const

	Returns the maximum CPU (SoC) temperature in degrees Celsius.

.. cpp:function:: TCPUSpeed CCPUThrottle::SetSpeed (TCPUSpeed Speed, boolean bWait = TRUE)

	Sets the CPU speed. ``Speed`` selects the speed to be set and overwrites the initial value. Possible values are:

	* CPUSpeedLow
	* CPUSpeedMaximum

	``bWait`` must be ``TRUE`` to wait for new clock rate to settle before return. Returns the previous setting or ``CPUSpeedUnknown`` on error.

.. cpp:function:: boolean CCPUThrottle::SetOnTemperature (void)

	Sets the CPU speed depending on current SoC temperature. Call this repeatedly all 2 to 5 seconds to hold the temperature down! Throttles the CPU down when the SoC temperature reaches 60 degrees Celsius Returns ``TRUE`` if the operation was successful.

.. note::

	The default temperature limit of 60 degrees Celsius may be too small for continuous operation with maximum performance. The limit can be increased with the parameter ``socmaxtemp`` in the file `cmdline.txt <https://github.com/rsta2/circle/blob/master/doc/cmdline.txt>`_.

.. cpp:function:: boolean CCPUThrottle::Update (void)

	Same function as ``SetOnTemperature()``, but can be called as often as you want, without checking the calling interval. Additionally checks for system throttled conditions, if a system throttled handler has been registered with ``RegisterSystemThrottledHandler()``. Returns ``TRUE`` if the operation was successful.

.. important::

	You have to repeatedly call ``SetOnTemperature()`` or ``Update()``, if you use this class!

.. cpp:function:: void CCPUThrottle::RegisterSystemThrottledHandler (unsigned StateMask, TSystemThrottledHandler *pHandler, void *pParam = 0)

	Registers the callback ``pHandler``, which is invoked from ``Update()``, when a system throttled condition occurs, which is given in ``StateMask``. ``pParam`` is any user parameter to be handed over to the callback function. ``StateMask`` can be composed from these bit masks by or'ing them together:

	* SystemStateUnderVoltageOccurred
	* SystemStateFrequencyCappingOccurred
	* SystemStateThrottlingOccurred
	* SystemStateSoftTempLimitOccurred

.. cpp:function:: void CCPUThrottle::DumpStatus (boolean bAll = TRUE)

	Dumps some information on the current CPU status to the :ref:`System log`. Set ``bAll`` to ``TRUE`` to dump all information. Only the current clock rate and temperature will be dumped otherwise.
