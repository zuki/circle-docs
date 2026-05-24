フォントサービス
~~~~~~~~~~~~~~~~~

Circleは、ISO-8859-1（Latin1）の文字コードをドットマトリクスディスプレイの文字表示用のピクセル情報に変換する、シンプルなディスプレイフォントサービスを提供しています。

システムフォント
^^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/font.h>

.. cpp:struct:: TFont

	この構造体は定義済みのシステムフォントを表す基本的な記述子です。利用可能なフォントは以下の通りです。

	===============	=======	=======	===============	============ ==========
	Font		Width	Height	Extra height	Total height Remarks
	===============	=======	=======	===============	============ ==========
	Font6x7		6	7	1		8
	Font8x8		8	8	2		10
	Font8x10	8	10	2		12
	Font8x12	8	12	3		15
	Font8x14	8	14	3		17
	Font8x16	8	16	3		19
	Font12x22	12	22	4		26	     ASCII only
	===============	=======	=======	===============	============ ==========

	WidthとHeightはピクセル単位で指定されています。Extra heigtは、下線（カーソル）用に確保されたスペースです。

.. c:macro:: DEFAULT_FONT

	このマクロはデフォルトのシステムフォントを定義します。通常、 ``Font8x16`` です。このデフォルト設定はシステムオプション ``DEFAULT_FONT`` で上書きすることができます。

CCharGenerator
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/chargenerator.h>

.. cpp:class:: CCharGenerator

	This class provides easy access to system fonts.

.. cpp:function:: CCharGenerator::CCharGenerator (const TFont &rFont = DEFAULT_FONT, TFontFlags Flags = FontFlagsNone)

	Creates an instance of `CCharGenerator`. ``rFont`` is the font descriptor of the font to be used. ``Flags`` modifies the font with the following possible options:

.. cpp:enum:: CCharGenerator::TFontFlags

	* FontFlagsNone
	* FontFlagsDoubleWidth
	* FontFlagsDoubleHeight
	* FontFlagsDoubleBoth

.. cpp:function:: static CCharGenerator::TFontFlags CCharGenerator::MakeFlags (boolean bDoubleWidth, boolean bDoubleHeight)

	Makes font flags from separate options.

.. cpp:function:: unsigned CCharGenerator::GetCharWidth (void) const

	Returns the horizontal number of pixels per character.

.. cpp:function:: unsigned CCharGenerator::GetCharHeight (void) const

	Returns the vertical number of pixels per character including the underline space.

.. cpp:function:: unsigned CCharGenerator::GetUnderline (void) const

	Returns the vertical pixel start line of the underline space.

.. cpp:function:: CCharGenerator::TPixelLine CCharGenerator::GetPixelLine (char chAscii, unsigned nPosY) const

	Returns the horizontal pixel information for character code (normally ISO-8859-1) ``chAscii`` for the pixel line ``nPosY`` inside the character (0-based) as the following type:

.. cpp:type:: CCharGenerator::TPixelLine

.. cpp:function:: boolean CCharGenerator::GetPixel (unsigned nPosX, TPixelLine Line) const

	Returns ``TRUE``, if the pixel at the horizontal position ``nPosX`` (left is 0) is set inside the pixel line ``Line``, which has been fetched using :cpp:func:`CCharGenerator::GetPixelLine()`.

.. cpp:function:: boolean CCharGenerator::GetPixel (char chAscii, unsigned nPosX, unsigned nPosY) const

	Returns ``TRUE``, if the pixel at the horizontal position ``nPosX`` (left is 0) and vertical position ``nPosY`` (0-based) is set for character code (normally ISO-8859-1) ``chAscii``.
