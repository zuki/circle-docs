.. _libraries:

ライブラリ
~~~~~~~~~~~

この付録ではCircleプロジェクトが提供するライブラリのリストを提供します。

基本ライブラリ
^^^^^^^^^^^^^^

基本ライブラリはCircleのプロジェクトルートで ``./makeall`` を実行するとビルドされます。

==============================  ===========================================================  ============================
ライブラリ lib/...              説明                                                         libに依存
==============================  ===========================================================  ============================
libcircle.a                     基本的なシステムサービスとドライバ
usb/libusb.a                    USBホストコントローラとドライバクラス                        circle, input, fs, (sched)
usb/gadget/libusbgadget.a       USBガジェットドライバ                                        circle, usb
input/libinput.a                汎用入力デバイスサービス                                     circle
fs/libfs.a                      基本的なファイルシステムサービス (パーティションマネージャ)  circle
fs/fat/libfatfs.a               FATファイルシステムドライバ [#fs]_                           circle, fs
sched/libsched.a                協調型マルチタスクのサポート                                 circle
net/libnet.a                    TCP/IPネットワーク                                           circle, sched
sound/libsound.a                サウンドドライバ                                             circle, usb, (sched)
==============================  ===========================================================  ============================

.. note::

	USBライブラリとサウンドライブラリはシステムオプション ``NO_BUSY_WAIT`` が定義されている場合、スケジューラライブラリだけに依存します。

アドオンライブラリ
^^^^^^^^^^^^^^^^^^^

アドオンライブラリは対象ディレクトリで ``make`` を実行するとビルドされます。この付録では利用可能なアドオンライブラリの一部のみをリストアップしています。提供されているすべてのアドオンモジュールは `ここにリストアップされています <https://github.com/rsta2/circle/blob/master/addon/README>`_.

==============================  =========================================
ライブラリ addon/...            説明
==============================  =========================================
SDCard/libsdcard.a              EMMC/SDHOST SDカードドライバ
fatfs/libfatfs.a                `FatFs file system module`_ [#fs]_
Properties/libproperties.a      プロパティファイル (.ini) のサポート
linux/liblinuxemu.a             Linuxカーネルドライバとpthreadのエミュレーション
vc4/vchiq/libvchiq.a            VCHIQインタフェースドライバ
vc4/sound/libvchiqsound.a       VCHIQ (HDMI) サウンドドライバ
ugui/libugui.a                  `uGUI graphics library`_
lvgl/liblvgl.a                  `LVGL graphics library`_
==============================  =========================================

.. _FatFs file system module: http://elm-chan.org/fsw/ff/00index_e.html
.. _uGUI graphics library: http://embeddedlightning.com/ugui
.. _LVGL graphics library: https://lvgl.io

以下のライブラリは *addon/vc4/interface/* にある Raspberry Pi 1-3 と Zero (32-bit のみ) 用のアクセラレーテッドグラフィックスサポートを提供します。

* bcm_host/libbcm_host.a
* khronos/libkhrn_client.a
* vmcs_host/libvmcs_host.a
* vcos/libvcos.a

.. rubric:: Footnotes

.. [#fs] 基本ライブラリのファイルシステムサポートは制限されています(サブディレクトリなし、短いファイル名)。 *addon/fatfs/* にあるFatFsファイルシステムモジュールは全機能をサポートします。
