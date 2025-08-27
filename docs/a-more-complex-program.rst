.. _a-more-complex-program:

より複雑なプログラム
----------------------

Circleアプリケーションの基本的な構造がどのようなものかわかったところで、さらによく使われるクラスとその機能を追加したいと思います。以下のプログラムは、*sample/04-timer* に基づいています。試すにはRaspberry Piに接続されたHDMIディスプレイかシリアルターミナルが必要です。

まず *app/* の下に新しいサブディレクトリを作成し、先に説明したプログラムから *main.cpp* と *Makefile* をコピーしてください。これらのファイルは変更しません。 ``CKernel`` クラスだけを変更・拡張します。クラス定義は以下のようになります。

.. code-block:: c++
	:caption: kernel.h

	#ifndef _kernel_h
	#define _kernel_h

	#include <circle/actled.h>
	#include <circle/koptions.h>
	#include <circle/devicenameservice.h>
	#include <circle/screen.h>
	#include <circle/serial.h>
	#include <circle/exceptionhandler.h>
	#include <circle/interrupt.h>
	#include <circle/timer.h>
	#include <circle/logger.h>
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
		static void TimerHandler (TKernelTimerHandle hTimer,
					  void *pParam, void *pContext);

	private:
		CActLED			m_ActLED;
		CKernelOptions		m_Options;
		CDeviceNameService	m_DeviceNameService;
		CScreenDevice		m_Screen;
		CSerialDevice		m_Serial;
		CExceptionHandler	m_ExceptionHandler;
		CInterruptSystem	m_Interrupt;
		CTimer			m_Timer;
		CLogger			m_Logger;
	};

	#endif

以下のクラスをメンバオブジェクトとして ``CKernel`` に追加しました。

======================  ======================================================
クラス                  目的
======================  ======================================================
CKernelOptions          *cmdline.txt* から共通のラインオプションを提供します
CDeviceNameService      デバイス名をデバイスオブジェクトへのポインタにマップします
CScreenDevice           HDMIディスプレイ (screen) にアクセスします
CSerialDevice           シリアルインタフェース (UART) にアクセスします
CExceptionHandler       デバッグ用にシステムエラー (abort exceptions) を報告します
CInterruptSystem        割り込み (IRQ と FIQ) を処理します
CTimer                  タイムサービスを提供します
CLogger                 システムロギング機能です
======================	======================================================

さらに、プライベートな静的関数 ``TimerHandler()`` コールバック関数が追加され、 ``CTimer`` クラスで実装されているカーネルタイマーの機能を表示するために使用されます。ファイル *kernel.cpp* は次のようにに更新されました。

.. code-block:: c++
	:caption: kernel.cpp

	#include "kernel.h"

	static const char FromKernel[] = "kernel";

	CKernel::CKernel (void)
	:	m_Screen (m_Options.GetWidth (), m_Options.GetHeight ()),
		m_Timer (&m_Interrupt),
		m_Logger (m_Options.GetLogLevel (), &m_Timer)
	{
		m_ActLED.Blink (5);
	}

	CKernel::~CKernel (void)
	{
	}

``CKernel`` のコンストラクタにおいて ``CScreenDevice`` メンバは SD カード上の設定ファイル *cmdline.txt* に指定されているディスプレイの幅と高さを指定して明示的に初期化されます。ディスプレイの解像度はこのファイルの最初の行でたとえば次のように選択できます: ``width=640 height=480`` 。 ``CTimer`` メンバは100 Hzのシステムティックを実装するために割り込み(IRQ)を使用します。そのため、 ``CInterruptSystem`` メンバオブジェクトへのポインタを取得しています。

.. note::

	*cmdline.txt* 二指定できるすべてのCircleオプションは :ref:`cmdline` にリストされています。すべてのオプションは最初の行にスペースで区切って指定する必要があります。

システムロギング機能 ``CLogger`` は希望するロギングレベルとタイマーへのポインタで初期化されます。そのため、システムタイムをログ出力することができます。ロギングレベルは *cmdline.txt* に ``loglevel=N`` を追加することで設定することができます。ここでNは 0（パニック）から4（デバッグ、デフォルト）の間の数字です。この値以下の重大度のログメッセージだけが主力されます。

.. code-block:: c++
	:caption: kernel.cpp (continued)

	boolean CKernel::Initialize (void)
	{
		boolean bOK = TRUE;

		if (bOK)
		{
			bOK = m_Screen.Initialize ();
		}

		if (bOK)
		{
			bOK = m_Serial.Initialize (115200);
		}

		if (bOK)
		{
			CDevice *pTarget = m_DeviceNameService.GetDevice (
						m_Options.GetLogDevice (), FALSE);
			if (pTarget == 0)
			{
				pTarget = &m_Screen;
			}

			bOK = m_Logger.Initialize (pTarget);
		}

		if (bOK)
		{
			bOK = m_Interrupt.Initialize ();
		}

		if (bOK)
		{
			bOK = m_Timer.Initialize ();
		}

		return bOK;
	}

``Initialize()``メソッドでは、クラスメンバの初期化の第2ステップが行われます。 ``m_Logger.Initialize()``の呼び出しではパラメータとしてロギングデバイスへのポインタを取ります。そのデフォルトは ``&m_Screen`` です。 *cmdline.txt* に ``logdev=ttyS1`` を追加すると接続されているシリアルターミナルでメッセージを読むことができます。デバイス名からデバイスオブジェクトポインタへのマッピングは ``m_DeviceNameService.GetDevice()`` で行われます。その際、デバイス名が見つからない場合は 0 を返します。

.. important::

	初期化の順序は重要です。同様に、 *kernel.h* のクラス定義におけるコンストラクタとメンバオブジェクトの順序も重要です。

.. code-block:: c++
	:caption: kernel.cpp (continued)

	TShutdownMode CKernel::Run (void)
	{
		m_Logger.Write (FromKernel, LogNotice,
				"An exception will occur after 15 seconds from now");

		m_Timer.StartKernelTimer (15 * HZ, TimerHandler);

		unsigned nTime = m_Timer.GetTime ();
		while (1)
		{
			while (nTime == m_Timer.GetTime ())
			{
				// just wait a second
			}

			nTime = m_Timer.GetTime ();

			m_Logger.Write (FromKernel, LogNotice, "Time is %u", nTime);
		}

		return ShutdownHalt;
	}

``m_Logger.Write()`` は指定された重要度のメッセージをシステムログに書き込みます。 ``FromKernel`` はメッセージのソースを指定します（上の定義を参照）。 ``m_Timer.StartKernelTimer()`` は15秒後に ``TimerHandler()`` が呼び出されるようにトリガします。 ``m_Timer.GetTime()`` は1970-01-01 00:00:00からの現在のローカルシステム時間を秒単位で返します。リアルタイムクロックを使用していないので、実際の時刻はシステムの稼働時間に等しいことになります。このプログラムは、ロギングデバイスとして選択されている画面またはシリアルターミナルに1秒ごとにログメッセージを生成します。

.. code-block:: c++
	:caption: kernel.cpp (continued)

	void CKernel::TimerHandler (TKernelTimerHandle hTimer,
				    void *pParam, void *pContext)
	{
		void (*pInvalid) (void) = (void (*) (void)) 0x500000;

		(*pInvalid) ();
	}

15秒後に ``TimerHandler()`` が呼び出され、0x500000番地にジャンプして "Prefetch abort "例外を発生させます。このアドレス範囲は "実行不可" とマークされているからです。

付録 :ref:`analyzing-exceptions` では、このプログラムを使ってアボート例外が発生したときに表示される情報をどのように解析できるかを説明しています。
