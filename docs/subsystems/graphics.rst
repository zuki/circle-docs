グラフィックス
~~~~~~~~~~~~~~

Circleでは、グラフィカルユーザインターフェース（GUI）の実装や、接続されているHDMIやコンポジットテレビディスプレイ、SPI/I2Cインタフェースを備えた外部ディスプレイへのピクセルグラフィックスとベクターグラフィックスの出力を行ういくつかのオプションが用意されています。これらのオプションについてこのセクションで説明します。

C2DGraphics
^^^^^^^^^^^

``C2DGraphics`` クラスはCircleの基本ライブラリの一部であり、  :cpp:class:`CDisplay` クラスにより提供されるドットマトリクスディスプレイ上のピクセルグラフィックスを生成するために使用できます。

.. code-block:: cpp

	#include <circle/2dgraphics.h>

.. cpp:class:: C2DGraphics

	このクラスはVSyncとハードウェアアクセラレーションによるダブルバッファリング機能を備えたソフトウェアグラフィックスライブラリです。

.. note::

	The double buffering does not work on the Raspberry Pi 5 and on displays with SPI/I2C interface.

.. cpp:type:: T2DColor

	Same as :cpp:type:`CDisplay::TColor`.

.. c:macro:: COLOR2D(red, green, blue)

	Same as :c:macro:`DISPLAY_COLOR`.

.. cpp:function:: C2DGraphics::C2DGraphics (CDisplay *pDisplay)

	Creates on instance of this class. ``pDisplay`` is a pointer to the display driver.

.. note::

	このコンストラクタではVSyncはサポートされていません。

.. cpp:function:: C2DGraphics::C2DGraphics (unsigned nWidth, unsigned nHeight, boolean bVSync = TRUE, unsigned nDisplay = 0)

	Creates on instance of this class and uses the class :cpp:class:`CBcmFrameBuffer` as display driver. ``nWidth`` is the screen width in pixels (0 to detect). ``nHeight`` is the screen height in pixels (0 to detect). Set ``bVSync`` to ``TRUE`` to enable VSync and HW double buffering. ``nDisplay`` is the zero-based display number (for Raspberry Pi 4).

.. cpp:function:: boolean C2DGraphics::Initialize (void)

	Initializes the screen. Returns ``TRUE`` on success.

.. cpp:function:: boolean C2DGraphics::Resize (unsigned nWidth, unsigned nHeight)

	Initializes the screen again with a new size. ``nWidth`` is the new screen width and ``nHeight`` the new screen height in number of pixels. Returns ``TRUE`` on success. When ``FALSE`` is returned, the width and/or height are not supported. The object is in an uninitialized state then and must not be used, but ``Resize()`` can be called again with other parameters.

.. cpp:function:: unsigned C2DGraphics::GetWidth (void) const

	Returns the screen width in pixels.

.. cpp:function:: unsigned C2DGraphics::GetHeight (void) const

	Returns the screen height in pixels.

.. cpp:function:: void C2DGraphics::ClearScreen (T2DColor Color)

	Clears the screen. ``Color`` is the color used to clear the screen.

.. cpp:function:: void C2DGraphics::DrawRect (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, T2DColor Color)

	Draws a filled rectangle. ``nX`` is the start X coordinate. ``nY`` is the start Y coordinate. ``nWidth`` is the rectangle width. ``nHeight`` is the rectangle height. ``Color`` is the rectangle color.

.. cpp:function:: void C2DGraphics::DrawRectOutline (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, T2DColor Color)

	Draws an unfilled rectangle (inner outline). ``nX`` is the start X coordinate. ``nY`` is the start Y coordinate. ``nWidth`` is the rectangle width. ``nHeight`` is the rectangle height. ``Color`` is the rectangle color.

.. cpp:function:: void C2DGraphics::DrawLine (unsigned nX1, unsigned nY1, unsigned nX2, unsigned nY2, T2DColor Color)

	Draws a line. ``nX1`` is the start position X coordinate. ``nY1`` is the start position Y coordinate. ``nX2`` is the end position X coordinate. ``nY2`` is the end position Y coordinate. ``Color`` is the line color.

.. cpp:function:: void C2DGraphics::DrawCircle (unsigned nX, unsigned nY, unsigned nRadius, T2DColor Color)

	Draws a filled circle. ``nX`` is the circle X coordinate. ``nY`` is the circle Y coordinate. ``nRadius`` is the circle radius. ``Color`` is the circle color.

.. cpp:function:: void C2DGraphics::DrawCircleOutline (unsigned nX, unsigned nY, unsigned nRadius, T2DColor Color)

	Draws an unfilled circle (inner outline). ``nX`` is the circle X coordinate. ``nY`` is the circle Y coordinate. ``nRadius`` is the circle radius. ``Color`` is the circle color.

.. cpp:function:: void C2DGraphics::DrawImage (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, const void *PixelBuffer)

	Draws an image from a pixel buffer. ``nX`` is the image X coordinate. ``nY`` is the image Y coordinate. ``nWidth`` is the image width. ``nHeight`` is the image height. ``PixelBuffer`` is a pointer to the pixels.

.. cpp:function:: void C2DGraphics::DrawImageTransparent (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, const void *PixelBuffer, T2DColor TransparentColor)

	Draws an image from a pixel buffer with transparent color. ``nX`` is the image X coordinate. ``nY`` is the image Y coordinate. ``nWidth`` is the image width. ``nHeight`` is the image height. ``PixelBuffer`` is a pointer to the pixels. ``TransparentColor`` is the color to use for transparency.

.. cpp:function:: void C2DGraphics::DrawImageRect (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, unsigned nSourceX, unsigned nSourceY, const void *PixelBuffer)

	Draws an area of an image from a pixel buffer. ``nX`` is the image X coordinate. ``nY`` is the image Y coordinate. ``nWidth`` is the image width. ``nHeight`` is the image height. ``nSourceX`` is the source X coordinate in the pixel buffer. ``nSourceY`` is the source Y coordinate in the pixel buffer. ``PixelBuffer`` is a pointer to the pixels.

.. cpp:function:: void C2DGraphics::DrawImageRectTransparent (unsigned nX, unsigned nY, unsigned nWidth, unsigned nHeight, unsigned nSourceX, unsigned nSourceY, unsigned nSourceWidth, unsigned nSourceHeight, const void *PixelBuffer, T2DColor TransparentColor)

	Draws an area of an image from a pixel buffer with transparent color. ``nX`` is the image X coordinate. ``nY`` is the image Y coordinate. ``nWidth`` is the image width. ``nHeight`` is the image height. ``nSourceX`` is the source X coordinate in the pixel buffer. ``nSourceY`` is the source Y coordinate in the pixel buffer. ``nSourceWidth`` is the source image width. ``nSourceHeight`` is the source image height. ``PixelBuffer`` is a pointer to the pixels. ``TransparentColor`` is the color to use for transparency.

.. cpp:function:: void C2DGraphics::DrawPixel (unsigned nX, unsigned nY, T2DColor Color)

	Draws a single pixel. ``nX`` is the pixel X coordinate. ``nY`` is the pixel Y coordinate. ``Color`` is the pixel color.

.. note::

	If you need to draw a lot of pixels, consider using :cpp:func:`C2DGraphics::GetBuffer()` for better speed.

.. cpp:function:: void C2DGraphics::DrawText (unsigned nX, unsigned nY, T2DColor Color, const char *pText, TTextAlign Align = AlignLeft, const TFont &rFont = DEFAULT_FONT, CCharGenerator::TFontFlags FontFlags = CCharGenerator::FontFlagsNone)

	Draws a horizontal ISO8859-1 text string, using the font ``rFont`` with the flags ``FontFlags``. ``nX`` is the text X coordinate. ``nY`` is the text Y coordinate. ``Color`` is the text color. The background is transparent. ``pText`` is a zero-terminated C-string. ``Align`` specifies the horizontal text alignment, with these possible values:

.. cpp:enum:: C2DGraphics::TTextAlign

	* AlignLeft
	* AlignRight
	* AlignCenter

.. cpp:function:: void *C2DGraphics::GetBuffer (void)

	Gets raw access to the drawing buffer. Returns a pointer to the buffer, which contains the pixels in the physical color format (depends on the display).

.. cpp:function:: CDisplay *C2DGraphics::GetDisplay (void)

	Returns a pointer to the display, we are working on.

.. cpp:function:: void C2DGraphics::UpdateDisplay (void)

	Once everything has been drawn, updates the display to show the contents on screen. If VSync is enabled, this method is blocking until the screen refresh signal is received (every 16ms for 60 FPS refresh rate).

.. cpp:class:: C2DImage

	:cpp:class:`C2DGraphics` インスタンス上に表示されるスプライト画像です。

.. cpp:function:: C2DImage::C2DImage (C2DGraphics *p2DGraphics)

	Creates an instance of ``C2DImage``. ``p2DGraphics`` is the :cpp:class:`C2DGraphics` instance to be used.

.. cpp:function:: void C2DImage::Set (unsigned nWidth, unsigned nHeight, const T2DColor *pData)

	Initializes the image with a width of ``nWidth`` pixels, the height of ``nHeight`` pixels and the color information for the pixels in logical format at ``pData``.

.. cpp:function:: unsigned C2DImage::GetWidth (void) const

	Returns the width of the image in number of pixels.

.. cpp:function:: unsigned C2DImage::GetHeight (void) const

	Returns the height of the image in number of pixels.

.. cpp:function:: const void *C2DImage::GetPixels (void) const

	Returns a pointer to the color information of the image pixels in physical (hardware) format.

.. _LVGL:

LVGL
^^^^

Circleでは `Light and Versatile Graphics Library <https://lvgl.io/>`_ (LVGL) v9.4.0 が利用できます。このライブラリはC言語ベースのAPIを提供しています。詳細については `LVGLのドキュメント <https://docs.lvgl.io/9.4/>`_ を参照してください。

.. code-block:: cpp

	#include <lvgl/lvgl.h>

.. cpp:class:: CLVGL

	このクラスはLVGLのラッパーであり、このグラフィックスライブラリを使用するにはインスタンス化する必要があります。このラッパークラスはUSBマウスとタッチスクリーン入力をサポートしています。

.. cpp:function:: CLVGL::CLVGL (CScreenDevice *pScreen)
.. cpp:function:: CLVGL::CLVGL (CDisplay *pDisplay)

	Create an instance of this class. ``pScreen`` or ``pDisplay`` reference the display to be used.

.. cpp:function:: boolean CLVGL::Initialize (void)

	Initializes to LVGL support. Returns ``TRUE`` on success.

.. cpp:function:: void CLVGL::Update (boolean bPlugAndPlayUpdated = FALSE)

	Updates the display. This has to be called continuously from the application main loop at ``TASK_LEVEL``. ``bPlugAndPlayUpdated`` must be set to ``TRUE``, if the application supports USB plug-and-play and :cpp:func:`CUSBHostController::UpdatePlugAndPlay()` returned ``TRUE`` too.

μGUI
^^^^

Circleでは `μGUIライブラリ <http://embeddedlightning.com/ugui/>`_ を利用できます。このライブラリはC言語をベースとしたAPIを提供しています。詳細については `リファレンスガイド <http://embeddedlightning.com/download/reference-guide/>`_ をダウンロードしてください。

.. note::

	このライブラリは現在のところRaspberry Pi 5ではサポートされていません。

.. code-block:: cpp

	#include <ugui/uguicpp.h>

.. cpp:class:: CUGUI

	このクラスはμGUIのラッパークラスであり、このグラフィックスライブラリを使用するにはインスタンス化する必要があります。このラッパークラスは、USBマウスとタッチスクリーン入力をサポートしています。

.. cpp:function:: CUGUI::CUGUI (CScreenDevice *pScreen)

	Creates an instance of this class. ``pScreen`` references the display to be used.

.. cpp:function:: boolean CUGUI::Initialize (void)

	Initializes to μGUI support. Returns ``TRUE`` on success.

.. cpp:function:: void CUGUI::Update (boolean bPlugAndPlayUpdated = FALSE)

	Updates the display. This has to be called continuously from the application main loop at ``TASK_LEVEL``. ``bPlugAndPlayUpdated`` must be set to ``TRUE``, if the application supports USB plug-and-play and :cpp:func:`CUSBHostController::UpdatePlugAndPlay()` returned ``TRUE`` too.

Accelerated graphics
^^^^^^^^^^^^^^^^^^^^

The accelerated graphics support is described in the :ref:`VC4` subsystem section.
