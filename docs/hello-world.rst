Hello world!
------------

では初めてのCircleプログラムの作成を開始しましょう。これは次のようになります:

.. code-block:: c++
	:caption: main.cpp

	#include <circle/startup.h>	// EXIT_HALT のため
	#include <circle/actled.h>
	#include <circle/timer.h>

	int main (void)
	{
		CActLED ActLED;

		for (unsigned i = 1; i <= 10; i++)
		{
			ActLED.On ();
			CTimer::SimpleMsDelay (200);

			ActLED.Off ();
			CTimer::SimpleMsDelay (500);
		}

		return EXIT_HALT;
	}

プログラムは自明であると思います。 ``CTimer::SimpleMsDelay()`` は静的遅延関数で、システムに ``CTimer`` クラスのインスタンスがない場合に使用できます。

最初のテストとして *app/* ディレクトリにサブディレクトリを作成し、そこに *main.cpp* としてこのプログラムを保存してください。さらに、同じディレクトリに以下の *Makefile* を置く必要があります:

.. code-block:: make
	:caption: Makefile

	CIRCLEHOME = ../..

	OBJS	= main.o

	LIBS	= $(CIRCLEHOME)/lib/libcircle.a

	include $(CIRCLEHOME)/Rules.mk

	-include $(DEPS)

ではこのディレクトリで ``make`` を実行して、作成された *kernel\*.img* ファイルをSDカードにコピーしてください。Raspberry Piの電源を入れると緑色のActivity LEDが10回点滅し、システムが停止するはずです。

CKernel クラス
~~~~~~~~~~~~~~~~~

通常、アプリケーションはこのような単純なものではなく、プログラムに何らかの構造を適用する必要があります。これはすべてのCircleアプリケーションで使用されるはずです。C++では抽象化の手法はクラスですので、ここでアプリケーション用のメインクラスを定義したいと思います。Circleではこれは通常 ``CKernel`` と呼ばれます。クラスの定義と実装を分けるのは良い習慣なので、クラスはヘッダーファイル *kernel.h* で定義します:

.. code-block:: c++
	:caption: kernel.h

	#ifndef _kernel_h
	#define _kernel_h

	//#include <circle/memory.h>
	#include <circle/actled.h>
	#include <circle/types.h>

	enum TShutdownMode
	{
		ShutdownNone,
		ShutdownHalt,
		ShutdownReboot
	};

	class CKernel
	{
	public:
		CKernel (void);
		~CKernel (void);

		boolean Initialize (void);

		TShutdownMode Run (void);

	private:
		//CMemorySystem	m_Memory;	// not needed any more
		CActLED		m_ActLED;
	};

	#endif

*app/* 配下に新しいサブディレクトリを作成し、そこにこのファイルを保存してください。クラスのコンストラクタ ``CKernel()`` とデストラクタ ``~CKernel()`` の他にメソッド ``Initialize()`` と ``Run()`` があります。これはクラスのメンバに対して3段階の初期化を行うものであり、Circle全体で共通です。

1. コンストラクタ ``CKernel()`` はクラスメンバ変数の基本的な初期化を行います。
2. メソッド ``Initialize()`` はクラスメンバの初期化を完了し、初期化に成功した場合は ``TRUE`` を返します。
3. メソッド ``Run()`` はアプリケーションの実行を開始するために呼び出されます。 ``Run()`` が復帰すると ``TShutdownMode`` 型の戻り値に応じて、アプリケーションが停止するか、システムが再起動されます。多くのアプリケーションは ``Run()`` から復帰することはありません。

.. note::

	Circleは歴史的な理由により、値として  ``TRUE`` と ``FALSE`` を持つ ``boolean`` 型を使用します。代わりに ``bool``, ``true``, ``false`` を使うこともでき、これらは同じです。

.. note::

	以前のバージョンの Circle ではシステムメモリを初期化して管理する ``CMemorySystem`` クラスのメンバが ``CKernel`` に必要でした。現在では ``CMemorySystem`` のインスタンスは関数 ``main()`` が呼ばれる前に生成されるため、このメンバを ``CKernel`` に追加する必要はありません。互換性のために ``CKernel`` で ``CMemorySystem`` をインスタンス化することもできますが、これは非推奨です。

先ほどの "Hello world!" プログラムと同じ機能を持つ ``CKernel`` のクラス実装は以下のようになります:

.. code-block:: c++
	:caption: kernel.cpp

	#include "kernel.h"
	#include <circle/timer.h>

	CKernel::CKernel (void)
	{
	}

	CKernel::~CKernel (void)
	{
	}

	boolean CKernel::Initialize (void)
	{
		return TRUE;
	}

	TShutdownMode CKernel::Run (void)
	{
		for (unsigned i = 1; i <= 10; i++)
		{
			m_ActLED.On ();
			CTimer::SimpleMsDelay (200);

			m_ActLED.Off ();
			CTimer::SimpleMsDelay (500);
		}

		return ShutdownHalt;
	}

ここではクラスのコンストラクタ ``CKernel()`` とデストラクタ ``~CKernel()`` 、メソッド ``Initialize()`` はあまり使用されていませんが、実際のアプリケーションでは変更されるでしょう。メンバ変数 ``m_ActLED`` のコンストラクタは ``CKernel()`` の中で暗黙的に呼び出されることに注意してください。この呼び出しはコンパイラによって自動的に生成されます。

これで ``CKernel`` クラスの定義と実装が終わったので、次に ``main()`` 関数を用意します。この関数は上で示した3段階の処理を実装します。これは以下のように行うことができます:

.. code-block:: c++
	:caption: main.cpp

	#include "kernel.h"
	#include <circle/startup.h>

	int main (void)
	{
		CKernel Kernel;
		if (!Kernel.Initialize ())
		{
			halt ();
			return EXIT_HALT;
		}

		TShutdownMode ShutdownMode = Kernel.Run ();

		switch (ShutdownMode)
		{
		case ShutdownReboot:
			reboot ();
			return EXIT_REBOOT;

		case ShutdownHalt:
		default:
			halt ();
			return EXIT_HALT;
		}
	}

この *main.cpp* ファイルはほとんどのCircleプログラムで変更することなく使うことができます。

.. note::

	``CKernel`` で使用されるデストラクタの中には実装されていないものもあるため、 ``main()`` は実際には復帰せず、代わりに ``halt()`` や ``reboot()`` を呼び出します。ここでは *main.cpp* の一般的な実装を提供したいので、この小さな欠点は受け入れなければなりません。実際、説明した ``CKernel`` の実装では ``main()`` から復帰すすることもできますが、他の Circle アプリケーションではそうである必要はありません。

最後に、上記の *Makefile* に *kernel.o* を追加する必要があります:

.. code-block:: make
	:caption: Makefile

	CIRCLEHOME = ../..

	OBJS	= main.o kernel.o

	LIBS	= $(CIRCLEHOME)/lib/libcircle.a

	include $(CIRCLEHOME)/Rules.mk

	-include $(DEPS)

これで終わりです。これでCircleアプリケーションの基本構造ができ ``make`` を使ってビルドできるようになったはずです。
