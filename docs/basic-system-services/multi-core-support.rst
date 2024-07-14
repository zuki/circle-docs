.. _Multi-core support:

マルチコアサポート
~~~~~~~~~~~~~~~~~~

Raspberry Pi 2から1つのCortex-A CPUが4つのコアを搭載するようになりました。Circleでは、割り込み処理などのすべてのシステム処理をプライマリコア0で実行するという風に、プライマリコア0とセカンダリコア1から3を区別しています。セカンダリコアはアプリケーションで自由に使用することができます。これにより、割り込みやその他のシステム機能に邪魔されることなく、セカンダリコア上でタイムクリティカルな処理や時間のかかる処理を実行することができます。オプションのスケジューラとすべてのタスクはコア0でも実行されます (:ref:`Multitasking` を参照してください) 。

Circle では、セカンダリコアの起動を :ref:`CMultiCoreSupport` クラスと同期クラス :ref:`CSpinLock` および :ref:`Memory Barriers` で処理することによりマルチコアアプリケーションをサポートしています。

``CMultiCoreSupport`` クラスによるマルチコアサポートを使用するにはシステムオプション ``ARM_ALLOW_MULTI_CORE`` を定義する必要があります。パフォーマンス上の理由から、シングルコアのアプリケーションではこのシステムオプションは定義しないでください。

マルチコアサポートの使用に関する詳細は `doc/multicore.txt <https://github.com/rsta2/circle/blob/master/doc/multicore.txt>`_ ファイルに記載されています。

サンプルプログラム `17-fractal` と `26-cpustress` はマルチコア対応でビルドすることができます。より複雑なマルチコアの例として、プロジェクト `MiniSynth Pi <https://github.com/rsta2/minisynth/>`_ があります。

.. _CMultiCoreSupport:

CMultiCoreSupport
^^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/multicore.h>

.. cpp:class:: CMultiCoreSupport

アプリケーションでセカンダリCPUコアを使用したい場合は ``CMultiCoreSupport`` クラスを継承したユーザクラスを定義する必要があります。

.. cpp:function:: CMultiCoreSupport::CMultiCoreSupport (CMemorySystem *pMemorySystem)

	``CMultiCoreSupport`` のインスタンスを作成します。定義されたユーザークラスのイニシャライザリストの先頭で呼び出す必要があります。パラメータ ``pMemorySystem`` には  ``<circle/memory.h>`` からインクルードできる ``CMemorySystem::Get()`` を設定する必要があります。

.. cpp:function:: boolean CMultiCoreSupport::Initialize (void)

	マルチコアサポートを初期化し、セカンダリーコアを開始します。このメソッドが呼ばれるのは、他のシステムの初期化がすでに終了済みであることが重要です。通常は ``CKernel::Run()`` の最後のメソッドとして呼び出します。

.. cpp:function:: virtual void CMultiCoreSupport::Run (unsigned nCore) = 0

	アプリケーションにセカンダリコア（1～3）のエントリーを定義するにはこの仮想メソッドをオーバーロードします。 このメソッドは 実行するCPUコアの番号である ``nCore`` を
	引数に3回（各セカンダリコアごとに1回）呼び出されます。

.. important::

	セカンダリコアが ``Run()`` から復帰するとCPUコアは自動的に停止し、スリープ状態になります。使用しないコアについては単にこのメソッドからリターンすれば良いです。

.. note::

	このメソッドはデフォルトではプライマリCPUコア0からは実行されません。すべてのCPUコアを同時に処理したい場合は ``CKernel::Run()`` から継承したユーザ定義のマルチコアクラスの ``Run()`` メソッドをパラメータ 0 で明示的に呼び出す必要があります。

.. cpp:function:: static unsigned CMultiCoreSupport::ThisCore (void)

	このメソッドを呼び出したCPUコアの番号（0、1、2、3）を返します。

.. cpp:function:: static void CMultiCoreSupport::HaltAll (void)

	マルチコア環境において、このメソッドはすべてのCPUコアを停止させます。現在の実行はプロセッサ間割り込み（IPI）を使って割り込みをかけられ、各コアは順番に ``halt()`` 関数を呼び出します。

.. cpp:function:: static void CMultiCoreSupport::SendIPI (unsigned nCore, unsigned nIPI)

	コア ``nCore`` (0, 1, 2, 3) に番号 ``nIPI`` のプロセッサ間割り込み (IPI) を送信します。このテクニックをアプリケーションで使用する場合、 ``nIPI`` には ``IPI_USER`` から ``IPI_MAX`` までのユーザ定義の値を設定することができます。

.. cpp:function:: virtual void CMultiCoreSupport::IPIHandler (unsigned nCore, unsigned nIPI)

	他のCPUコアからプロセッサ間割り込み（IPI）を受信するにはこの仮想メソッドをオーバーロードしてください。 ``nCore`` はIPIを受信して ``IPIHandler()`` を実行するCPUコアの番号です。 ``nIPI`` は ``CMultiCoreSupport::SendIPI()`` の呼び出しで指定された IPI 番号です。

.. important::

	``nIPI < IPI_USER`` の場合は ``CMultiCoreSupport::IPIHandler()`` をさらに呼び出す場合はこのメソッドと同じパラメータを渡すようにしてください。そうしないとシステムパニック状態(アボート例外、アサーション失敗)の際に呼び出される ``CMultiCoreSupport::HaltAll()`` メソッドは動作しません。
