ファームウェアへのアクス
~~~~~~~~~~~~~~~~~~~~~~~~

デバイスドライバが特定の設定（フレームバッファのサイズなど）を行うためにVPUコプロセッサ上で動作するファームウェアと通信する必要がある場合があります。また、Circleではサポートされていない特定の設定を行う必要がある場合、アプリケーションコードからもこの通信が必要になることがあります。ファームウェアへのアクセスは `Mailboxプロパティインタフェース <https://github.com/raspberrypi/firmware/wiki/Mailbox-property-interface>`_ を使用して行うことができます。Circleでは `CBcmPropertyTags`` クラスでサポートされています。

.. note::

	この `Mailboxプロパティインタフェース <https://github.com/raspberrypi/firmware/wiki/Mailbox-property-interface>`_ Wikiにはサポートされているすべての機能が記載されているわけではありません。その他の機能に関する情報はLinuxのソースコードからしか入手できません。

	Raspberry Pi 5のファームウェアはこのWikiに記載されている機能（といくつかの追加機能）のうち、ごく一部のみしかサポートされていません。

CBcmPropertyTags
^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/bcmpropertytags.h>

.. cpp:class:: CBcmPropertyTags

.. cpp:function:: boolean CBcmPropertyTags::GetTag (u32 nTagId, void *pTag, unsigned nTagSize, unsigned nRequestParmSize = 0)

	単一タグを使用してファームウェアに対するてメールボックスプロパティの呼び出しを行います。 ``nTagId`` はタグ識別子です。Circle で使用される識別子は `circle/bcmpropertytags.h <https://github.com/rsta2/circle/blob/master/include/circle/bcmpropertytags.h>`_ に記載されています。 ``pTag`` はタグ構造体へのポインタ、 ``nTagSize`` はこの構造体のサイズです。このヘッダーファイルでは数多くのメールボックスプロパティ関数用のタグ構造体も定義されています。パラメータ ``nRequestParmSize`` はファームウェアへの入力パラメータとして渡されるタグ構造体のバイト数を指定します。ここで、 ``TPropertyTag`` ヘッダーはカウントしません。ファームウェアに入力パラメータを渡さないプロパティタグの場合、このパラメータは 0 になります。呼び出しが成功した場合、 ``GetTag()`` は ``TRUE`` を返します。

.. cpp:function:: boolean CBcmPropertyTags::GetTags (void *pTags, unsigned nTagsSize)

	ファームウェアに対して、一度に複数のタグを含む複数のタグを含むプロパティ呼び出しを行います。 ``pTags`` は複数のプロパティタグ構造体を連結したタグ構造体へのポインタです。 ``nTagsSize`` はこの構造体の合計サイズです。呼び出しが成功した場合、 ``GetTags()`` は ``TRUE`` を返します。

使用例
"""""""

以下は ``GetTag()`` を使ってファームウェからARM CPUの現在のクロックレートを取得するコードです。

.. code-block:: cpp

	#include <circle/bcmpropertytags.h>

	unsigned CMyClass::GetClockRate (void)
	{
		CBcmPropertyTags Tags;			// このクラス

		TPropertyTagClockRate TagClockRate;	// タグ構造体

		TagClockRate.nClockId = CLOCK_ID_ARM;	// 入力パラメタ

		if (!Tags.GetTag (PROPTAG_GET_CLOCK_RATE,
				  &TagClockRate, sizeof TagClockRate,
				  sizeof TagClockRate.nClockId))
		{
			return 0;			// 失敗した場合は0を返す
		}

		return TagClockRate.nRate;		// クロックレートを返す
	}

``GetTags()`` メソッドの使用例は `frame buffer driver <https://github.com/rsta2/circle/blob/master/lib/bcmframebuffer.cpp>`_ にあります。
