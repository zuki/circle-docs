.. _VC4:

VC4
~~~

`addon/vc4 <https://github.com/rsta2/circle/tree/master/addon/vc4>`_ にあるVC4サブシステムはRaspberry Piファームウェアが提供するオーディオとアクセレイテッドグラフィックスサービスへのインターフェースとしてVCHIQ ドライバを提供します。アクセレイテッドグラフィックス機能はRaspberry Pi 4と5では利用できません。また、 ``AARCH = 32`` でしか利用できません。このセクションではVC4サブシステムの構成要素について説明します。

.. _VCHIQ driver:

VCHIQドライバ
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <vc4/vchiq/vchiqdevice.h>

.. cpp:class:: CVCHIQDevice : public CLinuxDevice

	このクラスはVCHIQ (VideoCore Host Interface Queue) 用のドライバです。Raspberry Piコンピュータのビデオ処理ユニット（VPU）上で実行されている数多くのサービスプロセスへのインタフェースを実装しています。このドライバはLinuxから移植されたものであるため、 `addon/linux <https://github.com/rsta2/circle/tree/master/addon/linux>`_ ディレクトリにあるLinuxカーネルデバイスドライバエミュレーションコードに基づいています。VCHIQドライバのAPIはC言語に基づいていますがここでは扱いません。

.. cpp:function:: CVCHIQDevice::CVCHIQDevice (CMemorySystem *pMemory, CInterruptSystem *pInterrupt)

	VCHIQドライバクラスのインスタンスを作成します。これは1つしか存在できません。
	``pMemory`` と ``pInterrupt`` は、Circle のメモリと割り込みシステムオブジェクトへのポインタです。

.. cpp:function:: boolean CVCHIQDevice::Initialize (void)

	VCHIQドライバを初期化します。成功した場合は ``TRU`` Eを返します。このメソッドは基底クラス ``CLinuxDevice`` から継承されています。

VCHIQサウンド
^^^^^^^^^^^^^^

VCHIQサウンドドライバクラスである :cpp:class:`CVCHIQSoundBaseDevice` については :ref:`Audio devices` セクションで説明しています。

アクセレイテッドグラフィックス
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

 `addon/vc4/interface <https://github.com/rsta2/circle/tree/master/addon/vc4/interface>`_ にあるアクセレイテッドグラフィックス機能はRaspberry Pi OS（旧Raspbian）のユーザランドライブラリから移植されたものであり、以下の API を実装しています。

* EGL 1.4
* OpenGL ES 1.1 and 2.0
* OpenVG 1.1
* Dispmanx (proprietary)

最初の3つのAPIの詳細については `このwebsite <https://www.khronos.org/>`_ をご覧ください。これらはCircle固有のものではなく、C言語をベースとしています。

.. note::

	アクセレイテッドグラフィックス機能はRaspberry Pi 4と5では利用できません。また、 ``AARCH = 32`` でしか利用できません。
