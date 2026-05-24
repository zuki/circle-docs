ディスプレイデバイス
~~~~~~~~~~~~~~~~~~~~

このセクションではHDMI、SPI、I2Cの各インタフェースを介してさまざまなドットマトリクスディスプレイを制御するために使用されるデバイスドライバクラスについて説明します。これらのクラスは独自のインターフェースを持ち、  :cpp:class:`CDevice` クラスを継承していません。

CDisplay
^^^^^^^^

.. code-block:: cpp

	#include <circle/display.h>

.. cpp:class:: CDisplay

	ドットマトリクスディスプレイのサポートは ``CDisplay`` クラスに基づいています。このクラスは、様々な（論理的、物理的）色モデル間の色変換を行うメソッドとディスプレイ上にピクセル情報（単一のピクセルまたは（矩形の）ピクセル領域）を表示するための汎用インターフェースを構成する仮想メソッドを提供します。

	このクラスは論理色型 (``TColor``)を定義しています。それはRGB888形式であり、いくつかの事前定義された色が用意されています。この型の色は、ディスプレイハードウェアで使用されるさまざまな色モデルに変換可能です。これらの色は ``TRawColor`` 型により表現されます。現在、以下の色モデルがサポートされています。

.. cpp:enum:: CDisplay::TColorModel

	物理的な（ハードウェア）ディスプレイは以下のいずれかの物理色モデルを使用する必要があります。

	* RGB565 (0bRRRRRGGG'GGGBBBBB)
	* RGB565_BE (0bGGGBBBBB'RRRRRGGG, big endian)
	* ARGB8888 (0bAAAAAAAA'RRRRRRRR'GGGGGGGG'BBBBBBBB)
	* I1 (白黒)
	* I8 (パレットのインデックス)

.. cpp:enum:: CDisplay::TColor

	以下の論理色（RGB888）を事前定義しています。

	* Black
	* Red
	* Green
	* Yellow
	* Blue
	* Magenta
	* Cyan
	* White
	* BrightBlack
	* BrightRed
	* BrightGreen
	* BrightYellow
	* BrightBlue
	* BrightMagenta
	* BrightCyan
	* BrightWhite

	以下の論理色のエイリアスも定義されています。

	* NormalColor (BrightWhite)
	* HighColor (BrightRed)
	* HalfColor (Blue)

.. c:macro:: DISPLAY_COLOR(red, green, blue)

	論理表示色（RGB888）を定義します。パラメータの値は0から255までの範囲で指定できます。

.. cpp:type:: CDisplay::TRawColor

	（色モデルに合致した）物理職

.. cpp:struct:: CDisplay::TArea

	次の（0ベースの）座標でディスプレイ上のピクセル領域を定義します。

	* x1
	* x2
	* y1
	* y2

.. cpp:function:: CDisplay::CDisplay (TColorModel ColorModel)

	Creates an instance of ``CDisplay``. ``ColorModel`` is the physical color model to be used by the hardware.

.. cpp:function:: TColorModel CDisplay::GetColorModel (void) const

	Returns the used physical color model.

.. cpp:function:: TRawColor CDisplay::GetColor (TColor Color) const

	Converts the logical display color (RGB888) ``Color`` to a raw physical color value for the used color model and returns it.

.. cpp:function:: TColor CDisplay::GetColor (TRawColor Color) const

	Converts the raw physical display color ``Color`` for the used color model to a logical color value and returns it. Returns ``Black``, if the raw color is not predefined.

.. cpp:function:: virtual unsigned CDisplay::GetWidth (void) const = 0

	Returns the number of horizontal pixels on the display.

.. cpp:function:: virtual unsigned CDisplay::GetHeight (void) const = 0

	Returns the number of vertical pixels on the display.

.. cpp:function:: virtual unsigned CDisplay::GetDepth (void) const = 0

	Returns the number of bits, which is assigned to each pixel.

.. cpp:function:: virtual void CDisplay::SetPixel (unsigned nPosX, unsigned nPosY, TRawColor nColor) = 0

	Sets one pixel at the (0-based) position ``nPosX``, ``nPosY`` to the raw physical color
	``nColor``. The raw color value must match the color model.

.. cpp:function:: virtual void CDisplay::SetArea (const TArea &rArea, const void *pPixels, TAreaCompletionRoutine *pRoutine = nullptr, void *pParam = nullptr) = 0

	Sets the area (rectangle) ``rArea`` on the display to the raw physical colors in the array referenced by ``pPixels``. ``pRoutine`` is a pointer to a routine to be called on completion or ``nullptr`` for a synchronous call. ``pParam`` is an user parameter to be handed over to the completion routine:

.. cpp:type:: void CDisplay::TAreaCompletionRoutine (void *pParam)

.. note:: 一部のディスプレイドライバでは、この関数の非同期的な使用が実装されておらず、このメソッドから戻る直前に完了ルーチンを直接呼び出しています。

.. cpp:function:: virtual CDisplay *CDisplay::GetParent (void) const

	Returns a pointer to the parent display or ``nullptr``, if there is none. This is used to implement the class :cpp:class:`CWindowDisplay`.

.. cpp:function:: virtual unsigned CDisplay::GetOffsetX (void) const

	Returns the X-offset in pixels of this window display in the parent display or 0, if there is none.

.. cpp:function:: virtual unsigned CDisplay::GetOffsetY (void) const

	Returns the Y-offset in pixels of this window display in the parent display or 0, if there is none.

CWindowDisplay
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/windowdisplay.h>

.. cpp:class:: CWindowDisplay

	クラス ``CWindowDisplay`` は ``CDisplay`` 内の :cpp:class:`CDisplay` インスタンスであり、ディスプレイ上で複数の（重ならない）ウィンドウを使用できるようにします。このウィンドウには  :cpp:class:`CTerminalDevice`, :cpp:class:`C2DGraphics`, :cpp:class:`CLVGL` のインスタンスを表示することができます。

	このクラスが提供するメソッドのほとんどはその基底クラスである :cpp:class:`CDisplay` で説明されています。

	`sample/43-multiwindow` はマルチコアアプリケーションにおけるこのクラスの使用例を示しています。

.. cpp:function:: CWindowDisplay::CWindowDisplay (CDisplay *pDisplay, const TArea &rArea)

	``pDisplay`` is the display, this window is displayed on. ``rArea`` is the area on ``pDisplay``, which is covered by this window.

CBcmFrameBuffer
^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/bcmframebuffer.h>

.. cpp:class:: CBcmFrameBuffer : public CDisplay

	このクラスはRaspberry Piのファームウェアにより提供されているフレームバッファデバイス用のドライバです。Raspberry Pi 4、400、Compute Module 4は複数のフレームバッファデバイスに対応していますが、その他のモデルは1つのみに対応しています。フレームバッファとは基本的にはメインメモリ内のアドレス範囲のことであり、HDMIやコンポジット接続のテレビディスプレイに表示されるよう、ファームウェアがバックグラウンドで継続的に読み取りを行います。このメモリアドレス範囲に書き込むと表示される画像が変更されます。Raspberry Piのファームウェアは様々な幅、高さ、ピクセル情報の深度を持つフレームバッファをサポートしています。フレームバッファにテキストを表示したい場合はソフトウェアの文字生成機能を使用して文字を生成する必要があります。ファームウェア自体はテキスト表示をサポートしていません。

.. note::

	複数のフレームバッファデバイスを使用するにはSDカードの *config.txt* ファイルでオプション ``max_framebuffers=N`` (N > 1) を指定する必要があります。

クラス ``CBcmFrameBuffer`` はクラス :cpp:class:`CDisplay` のメソッドに加え、以下のメソッドを提供します。

.. cpp:function:: CBcmFrameBuffer::CBcmFrameBuffer (unsigned nWidth, unsigned nHeight, unsigned nDepth, unsigned nVirtualWidth = 0, unsigned nVirtualHeight = 0, unsigned nDisplay = 0, boolean bDoubleBuffered = FALSE)

	``nWidth`` * ``nHeight`` ピクセルのフレームバッファデバイスオブジェクトを作成します。両方のパラメータが 0 の場合、フレームバッファはデフォルトサイズで自動的に作成されます。デフォルトサイズは通常、接続されているディスプレイがサポートする最大サイズとなります。各ピクセルの色深度は ``nDepth`` ビット（4、8、16、24、32のいずれか）です。

	フレームバッファのメモリ範囲は表示される物理ディスプレイのサイズよりも大きくても構いません。これは表示画像を素早く切り替えるために使用できます(:cpp:func:`SetVirtualOffset()` を参照 )。仮想ディスプレイサイズ ``nVirtualWidth`` * ``nVirtualHeight`` ピクセルはオプションです。 ``bDoubleBuffered`` が ``TRUE`` の場合、 ``nVirtualWidth`` と ``nVirtualHeight`` が 0 に指定されていると、仮想ディスプレイの高さは自動的に物理ディスプレイサイズの 2 倍に設定されます。

	``nDisplay`` はフレームバッファデバイスの0から始まるID番号であり、Raspberry Pi 4、400、Compute Module 4で指定のディスプレイを選択するためにファームウェアに渡されます。

.. note::

	On the Raspberry Pi 5 only depths 16 and 32 are supported. For depth 32 you need the following settings in the file *config.txt*:

	* framebuffer_depth=32
	* framebuffer_ignore_alpha=1

.. cpp:function:: void CBcmFrameBuffer::SetPalette (u8 nIndex, u16 nRGB565)
.. cpp:function:: void CBcmFrameBuffer::SetPalette32 (u8 nIndex, u32 nRGBA)

	Set the entry ``nIndex`` of the color palette to ``nRGB565`` or ``nRGBA``. The color palette is only used in in 4-bit or 8-bit pixel depth mode. The color palette must be set before :cpp:func:`Initialize()` is called, but can be updated later.

.. c:macro:: PALETTE_ENTRIES

	The maximum number of entries in the color palette in 4-bit or 8-bit depth mode (256). ``nIndex`` must be below this.

.. cpp:function:: boolean CBcmFrameBuffer::Initialize (void)

	Initializes the frame buffer device and starts the display. Returns ``TRUE`` on success.

.. note::

	このメソッドはRaspberry Pi 1～3とZeroではディスプレイが接続されていない場合でも成功しますが、Raspberry Pi 4、400、Compute Module 4では失敗します。

.. cpp:function:: u32 CBcmFrameBuffer::GetWidth (void) const
.. cpp:function:: u32 CBcmFrameBuffer::GetHeight (void) const
.. cpp:function:: u32 CBcmFrameBuffer::GetVirtWidth(void) const
.. cpp:function:: u32 CBcmFrameBuffer::GetVirtHeight(void) const

	Return the physical or virtual size of the frame buffer in number of pixels.

.. cpp:function:: u32 CBcmFrameBuffer::GetPitch (void) const

	Returns the size of one pixel line in memory in number of bytes and may contain padding bytes.

.. cpp:function:: u32 CBcmFrameBuffer::GetDepth (void) const

	Returns the size of one pixel in memory in number of bits.

.. cpp:function:: u32 CBcmFrameBuffer::GetBuffer (void) const
.. cpp:function:: u32 CBcmFrameBuffer::GetSize (void) const

	Return the address and total size of the frame buffer in main memory.

.. cpp:function:: boolean CBcmFrameBuffer::UpdatePalette (void)

	Updates the color palette, after modifying it using :cpp:func:`SetPalette()` or :cpp:func:`SetPalette32()`. Returns ``TRUE`` on success. This method should be used only with a pixel depth of 4 or 8 bits.

.. cpp:function:: boolean CBcmFrameBuffer::SetVirtualOffset (u32 nOffsetX, u32 nOffsetY)

	Sets the offset of the top-left corner of the physically displayed image in a larger virtual frame buffer to [``nOffsetX``, ``nOffsetY``]. Returns ``TRUE`` on success.

.. cpp:function:: boolean CBcmFrameBuffer::WaitForVerticalSync (void)

	Waits for the next vertical synchronization (VSYNC) blanking gap. Returns ``TRUE`` on success.

.. cpp:function:: boolean CBcmFrameBuffer::SetBacklightBrightness(unsigned nBrightness)

	Sets the backlight brightness level of the display to ``nBrightness``. This has been tested with the Official 7" Raspberry Pi touchscreen only. The brightness level can be about 0..180 there. Returns ``TRUE`` on success.

.. cpp:function:: static unsigned CBcmFrameBuffer::GetNumDisplays (void)

	Returns to number of available displays, which is always 1 on models other than the Raspberry Pi 4, 400 or Compute Module 4.

CST7789Display
^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <display/st7789display.h>

.. cpp:class:: CST7789Display : public CDisplay

	This class is a driver for dot-matrix displays with ST7789 controller and SPI interface. It provides the methods, defined by its base-class :cpp:class:`CDisplay`, and the following additional methods. Other methods, which are not listed here, are deprecated.

.. cpp:function:: CST7789Display::CST7789Display (CSPIMaster *pSPIMaster, unsigned nDCPin, unsigned nResetPin = None, unsigned nBackLightPin = None, unsigned nWidth = 240, unsigned nHeight = 240, unsigned CPOL = 0, unsigned CPHA = 0, unsigned nClockSpeed = 15000000, unsigned nChipSelect = 0, boolean bSwapColorBytes = TRUE)

	``pSPIMaster`` is a pointer to the SPI master object to be used. ``nDCPin`` is the GPIO pin number (SoC number, not header position) for the DC pin, ``nResetPin`` is the GPIO pin number for the Reset pin (optional), ``nBackLightPin`` is the GPIO pin number for backlight pin (optional). ``nWidth`` is the display width in number of pixels (default 240), ``nHeight`` is the display height in number of pixels (default 240). ``CPOL`` is the SPI clock polarity (0 or 1, default 0), ``CPHA`` is the SPI clock phase (0 or 1, default 0). ``nClockSpeed`` is the SPI clock frequency in Hz (default 15 MHz). ``nChipSelect`` is the SPI chip select (if connected, otherwise don't care). Set ``bSwapColorBytes`` to ``TRUE`` to use big endian colors (RGB565_BE) instead of RGB565.

.. note::

	Optional GPIO pin numbers have to be set to ``None``, if they are not connected. If the SPI chip select is not connected, ``CPOL`` is probably 1. The default physical color model is RGB565_BE for compatibility reasons.

.. cpp:function:: boolean CST7789Display::Initialize (void)

	Initializes and clears the display and switches it on. Returns ``TRUE`` on success.

.. cpp:function:: void CST7789Display::SetRotation (unsigned nRot)

	Sets the global rotation of the display. ``nRot`` can have the value 0, 90, 180 or 270 (degrees counterclockwise).

.. cpp:function:: unsigned CST7789Display::GetRotation (void) const

	Returns the global rotation in degrees (0, 90, 180 or 270).

.. cpp:function:: void CST7789Display::On (void)

	Switches the display on.

.. cpp:function:: void CST7789Display::Off (void)

	Switches the display off.

.. cpp:function:: void CST7789Display::Clear (TST7789Color Color = ST7789_BLACK_COLOR)

	Clears the entire display to ``Color`` (default black).

CSSD1306Display
^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <display/ssd1306display.h>

.. cpp:class:: CSSD1306Display : public CDisplay

	This class is a driver for monochrome dot-matrix displays with SSD1306 controller and I2C interface. It provides the methods, defined by its base-class :cpp:class:`CDisplay`, and the following additional methods.

.. cpp:function:: CSSD1306Display::CSSD1306Display (CI2CMaster *pI2CMaster, unsigned nWidth = 128, unsigned nHeight = 32, u8 uchI2CAddress = 0x3C, unsigned nClockSpeed = 0)

	``pI2CMaster`` is a pointer to the I2C master to be used. ``nWidth`` is the display width in pixels (128 only), ``nHeight`` is the display height in pixels (32 or 64, default 32). ``uchI2CAddress`` is the I2C slave address of the display controller (default 0x3C). ``nClockSpeed`` is the I2C clock frequency in Hz or 0 to use the system default.

.. cpp:function:: boolean CSSD1306Display::Initialize (void)

	Initializes and clears the display and switches it on. Returns ``TRUE`` on success.

.. cpp:function:: void CSSD1306Display::SetRotation (unsigned nDegrees)

	Sets the global rotation of the display to ``nDegrees`` (0 or 180). This method must be called before :cpp:func:`CSSD1306Display::Initialize()`. The default rotation is 0.

.. cpp:function:: unsigned CSSD1306Display::GetRotation (void) const

	Returns the global rotation in degrees (0 or 180).

.. cpp:function:: void CSSD1306Display::On (void)

	Switches the display on.

.. cpp:function:: void CSSD1306Display::Off (void)

	Switches the display off.

.. cpp:function:: void CSSD1306Display::Clear (TRawColor nColor = 0)

	Clears the entire display to ``nColor`` (0 or 1, default black);

CILI9341Display
^^^^^^^^^^^^^^^

.. code-block:: cpp

	#include <display/ili9341display.h>

.. cpp:class:: CILI9341Display : public CDisplay

	This class is a driver for dot-matrix displays with ILI9341 controller and SPI interface. It provides the methods, defined by its base-class :cpp:class:`CDisplay`, and the following additional methods.

.. cpp:function:: CILI9341Display::CILI9341Display (CSPIMaster *pSPIMaster, unsigned nDCPin, unsigned nResetPin = None, unsigned nBackLightPin = None, unsigned nWidth = 240, unsigned nHeight = 320, unsigned nCPOL = 0, unsigned nCPHA = 0, unsigned nClockSpeed = 15000000, unsigned nChipSelect = 0, boolean bSwapColorBytes = TRUE)

	``pSPIMaster`` is a pointer to the SPI master object to be used. ``nDCPin`` is the GPIO pin number (SoC number, not header position) for the DC pin, ``nResetPin`` is the GPIO pin number for the Reset pin (optional), ``nBackLightPin`` is the GPIO pin number for backlight pin (optional). ``nWidth`` is the display width in number of pixels (default 240), ``nHeight`` is the display height in number of pixels (default 320). ``CPOL`` is the SPI clock polarity (0 or 1, default 0), ``CPHA`` is the SPI clock phase (0 or 1, default 0). ``nClockSpeed`` is the SPI clock frequency in Hz (default 15 MHz). ``nChipSelect`` is the SPI chip select (if connected, otherwise don't care). Set ``bSwapColorBytes`` to ``TRUE`` to use big endian colors (RGB565_BE) instead of RGB565.

.. note::

	Optional GPIO pin numbers have to be set to ``None``, if they are not connected. Width/height are valid at rotation 0 (may be swapped with rotation 90 and 270). Big endian colors are supported by the hardware and are displayed quicker.

.. cpp:function:: boolean CILI9341Display::Initialize (void)

	Initializes and clears the display and switches it on. Returns ``TRUE`` on success.

.. cpp:function:: void CILI9341Display::SetRotation (unsigned nDegrees)

	Sets the global rotation of the display to ``nDegrees`` counterclockwise (0, 90, 180 or 270). This method must be called before :cpp:func:`CILI9341Display::Initialize()`. The default rotation is 0.

.. cpp:function:: unsigned CILI9341Display::GetRotation (void) const

	Returns the global rotation in degrees (0, 90, 180 or 270).

.. cpp:function:: void CILI9341Display::On (void)

	Switches the display on.

.. cpp:function:: void CILI9341Display::Off (void)

	Switches the display off.

.. cpp:function:: void CILI9341Display::Clear (TRawColor nColor = 0)

	Clears the entire display to ``nColor`` (default black).
