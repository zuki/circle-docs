割り込み
~~~~~~~~~~

このセクションではCircleにおける低レベルのハードウェア割り込みサポートについて説明します。これは、独自のデバイスドライバやドライバ類似関数を開発したい場合にしか参考にならないはずです。

ARMアーキテクチャにはIRQとFIQという2種類の割り込み要求があります。IRQは基本的な割り込み要求タイプであり、すべてのRaspberry Piモデルで複数のアクティブソースを持つことができ、ほとんどの割り込み駆動のデバイスの制御に使用されます。FIQ (Fast Interrupt Request) は低レイテンシの割り込みに使用され、Raspberry Pi 1-3とZeroではアクティブな割り込みソースは一度に1つしか持つことができません。Raspberry Pi 4には新しい割り込みコントローラ (GIC-400) が搭載されており、理論的には複数のFIQソースを同時にサポートすることができますが、均質なソリューションを実現するために、現在のところCircleではサポートしていません。そのため、通常、システムにはIRQを使用する複数のアクティブな割り込みソースが存在しますが、FIQを使用するのは1つだけです。

.. important::

	現在のところ、CircleはRaspberry Pi 5の割り込みターゲットレベルとしてFIQをサポートしていません。

CInterruptSystem
^^^^^^^^^^^^^^^^

クラス ``CInterruptSystem`` は Circle においてハードウェア割り込みのサポートを提供者です。今ではハードウェア割り込みのサポートはすべてのアプリケーションで必須であり、このクラスのインスタンスはシステムの初期化で生成されます。互換性のために、このクラスの2番目のインスタンスを作成することが可能です。2番目のインスタンスのメソッドの呼び出しは自動的に1番目のインスタンスにルーティングされます。

.. code-block:: c++

	#include <circle/interrupt.h>

.. cpp:class:: CInterruptSystem

.. cpp:function:: boolean CInterruptSystem::Initialize (void)

	割り込みシステムを初期化します。成功すると ``TRUE`` を返します。

.. note::

	割り込みシステムには 2 段階の初期化が必要です。ステップ 1 は ``CInterruptSystem`` のコンストラクタで行い、ステップ 2 は ``Initialize()`` で行います。

.. cpp:function:: void CInterruptSystem::ConnectIRQ (unsigned nIRQ, TIRQHandler *pHandler, void *pParam)

	割り込みハンドラを IRQ ソース (ベクタ) に接続します。既知の割り込みソースは Raspberry Pi 1-3 と Zero 用は ``<circle/bcm2835int.h>`` で、Raspberry Pi 4 と 5 用は ``<circle/bcm2711int.h>`` で、RP1 サウスブリッジからの割り込みソース用は ``<circle/rp1int.h>`` で定義されています。IRQハンドラのプロトタイプは以下の通りです。

.. c:type:: void IRQHandler (void *pParam)

	``pParam`` には任意のユーザーパラメータを指定することができ、この IRQ ソース用の :cpp:func:`CInterruptSystem::ConnectIRQ()` の呼び出しで指定した値を取得します。

.. note::

	Raspberry Pi 5について、CircleはIRQ番号に新しいスキームを採用しています。これにより、メインのGIC-400割り込みコントローラに直接接続されているIRQ割り込みソースと、RP1サウスブリッジからGICにルーティングされている割り込みソースを区別することができます。

	ビット11が設定されているIRQ番号は、RP1からGICにルーティングされる第2レベルのIRQです。その他のIRQはすべてGICに直接接続されます。ビット10がセットされているIRQ番号はエッジトリガ（それ以外はレベルトリガ）です。現在のところ、エッジトリガの割り込みはRP1からのみサポートされています。

.. cpp:function:: void CInterruptSystem::DisconnectIRQ (unsigned nIRQ)

	指定したIRQソースの割り込みハンドラを解放します。

.. cpp:function:: void CInterruptSystem::ConnectFIQ (unsigned nFIQ, TFIQHandler *pHandler, void *pParam)

	FIQソースに割り込みハンドラを接続します。現時点ではアクティブなFIQソースは1つしか許されていません。FIQハンドラはIRQハンドラと同じプロトタイプを持ちます（上を参照）。

.. cpp:function:: void CInterruptSystem::DisconnectFIQ (void)

	指定したFIQソースの割り込みハンドラを解放します。

.. cpp:function:: static CInterruptSystem *CInterruptSystem::Get (void)

	``CInterruptSystem`` の唯一のインスタンスへのポインタを返します。

.. important::

	システムの1つ以上の IRQ ハンドラが浮動小数点レジスタを使用している場合、システムオプショ ン ``SAVE_VFP_REGS_ON_IRQ`` を有効にする必要があります。FIQハンドラに対する ``SAVE_VFP_REGS_ON_FIQ`` も同様です。これらのシステムオプションは、GNU-C 12.1以降に基づくツールチェーンをCircleのビルドに使用している場合、デフォルトで有効になります。
