始めよう
---------------

Circleの使用を開始するには、プロジェクトとツールチェーン [#tc]_1 をダウンロードし、ターゲットプラットフォームに合わせてにCircleを構成し、Circleライブラリとアプリケーション2をビルドし [#ap]_、ビルドしたバイナリイメージ（カーネルイメージ） [#ki]_ をファームウェアファイルと共にSDカードにインストールする必要があります。さらに、SDカードには設定ファイル *config.txt*  も必要です。以下の注意事項では開発ホストとしてLinuxが動作するx86_64 PCを必要とします。 `doc/windows-build.txt <https://github.com/rsta2/circle/blob/master/doc/windows-build.txt>`_ ファイルにはLinuxの代わりにWindowsを使用する方法について記載されています。

ダウンロード
~~~~~~~~~~~~~

Circleは次のように *Git* を使ってダウンロードすることができます:

.. code-block:: shell

	cd /path/to/your/projects
	git clone https://github.com/rsta2/circle.git

Circleアプリケーションをビルドするための推奨のツールチェーンは `here <https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads>`_ からダウンロードすることができます。32ビット（AArch32, 通常は *arm-none-eabi-* ）用と64ビット（AArch64, 通常は *aarch64-none-elf-* ）用のツールチェーンは異なることに注意してください。

構成
~~~~~~~~~~~~~

.. important::

	Raspberry Pi 5 は64ビットカーネルイメージでしか動作しません。

Circleはプロジェクトのルートディレクトリにあるファイル *Config.mk* を追加って構成します。このファイルは次のオプションを持つ ``configure`` スクリプトを使って作成することができます:

.. code-block:: none

	-r <number>, --raspberrypi <number>
	                   Raspberry Piモデル番号 (1, 2, 3, 4, 5, デフォルト: 1)
	-p <string>, --prefix <string>
	                   ツールチェーンコマンドのPrefix
	                   (デフォルト: arm-none-eabi-, -r 5 では aarch64-none-elf-)
	--multicore        マルチコアアプリケーションを許可する
	--realtime         IRQレイテンシを改善するためにリアルタイムモードを有効にする
	--keymap <country> デフォルトのUSBキーマップを設定する (DE, ES, FR, IT, UK, US)
	--qemu             QEMU配下で実行するようにビルドする
	-d <option>, --define <option>
	                   システムオプションを定義する
	--c++17            コンパイルに C++17 規格を使用するstandard for compiling (デフォルトは C++14)
	-f, --force        既存の Config.mk ファイルを上書きする
	-h, --help         利用法を示す

CircleをRaspberry Pi 3用にシステムの ``PATH`` 変数にあるツールチェーンパスに存在するデフォルトのツールチェーンPrefix ``arm-none-eabi-`` で構成したい場合、Circleのプロジェクトルートから次のように入力するだけです:

.. code-block:: shell

	./configure -r 3

ファイル *Config.mk* は自分で作成することもできます。通常の32ビット用の構成ファイルは次のようになります:

.. code-block:: make

	PREFIX = /path/to/your/toolchain/bin/arm-none-eabi-
	AARCH = 32
	RASPPI = 3

これはツールチェーンのパスと名前、Raspberry Pi [#pi]_ コンピュータのアーキテクチャとモデルを設定します。

.. note::

	`include/circle/sysconfig.h <https://github.com/rsta2/circle/blob/master/include/circle/sysconfig.h>`_ ファイルに記述されている設定可能なシステムオプションは、そこで定義するか、次のように *Config.mk* ファイルで定義することができます:

	``DEFINE += -DOPTION_NAME``

	デフォルトで有効になっているシステムオプションは次のようにして無効にすることができます:

	``DEFINE += -DNO_OPTION_NAME``

通常の64ビット用の構成ファイルは次のようになります:

.. code-block:: make

	PREFIX64 = /path/to/your/toolchain/bin/aarch64-none-elf-
	AARCH = 64
	RASPPI = 3

64ビット操作は Raspberry Pi 3, 4, 5 と Zero 2 oでのみ可能です。

ビルド
~~~~~~~~

After configuring Circleを構成したら、Circleプロジェクトのルートディレクトリに移動して次のように入力してください:

.. code-block:: shell

	./makeall clean
	./makeall

デファルトではサンプルプログラムはビルドされません。サンプルをビルドしたい場合は ``./makeall`` した後にビルドしたいサンプルのサブディレクトリに移動して ``make`` を実行してください。

インストール
~~~~~~~~~~~~

Raspberry Pi ファームウェア (*boot/* サブディレクトリから、このディレクトリで ``make`` すると取得できます) ファイルと *kernel*.img* (*sample/* サブディレクトリから) を FAT ファイルシステムの SDカードにコピーしてください。

boot/* サブディレクトリから *config32.txt* (32ビット動作用、AArch32) または *config64.txt* (64ビット動作用、AArch64) ファイルを SD カードにコピーし、*config.txt* にリネームすることを勧めます。

Raspberry Pi 4でFIQを使用する場合は、Circle固有のARMスタブファイル（32ビット動作の場合は *armstub7-rpi4.bin* 、64ビット動作の場合は *armstub8-rpi4.bin* ）を追加する必要があります。このファイルはファームウェアによりロードされます。このARMスタブは *boot/* サブディレクトリでビルドすることができます。これらのファイルのビルド方法については `boot/README <https://github.com/rsta2/circle/blob/master/boot/README>`_ を参照してください。

SDカードをRaspberry Piに入れて電源を入れてください。

.. rubric:: Footnotes

.. [#tc] ここでいうツールチェーンとは、特定のプラットフォーム上で動作し、別の（通常は異なる）プラットフォーム用のバイナリをビルドする、ツールやライブラリを追加したクロスコンパイラのことです。
.. [#ap] 手始めに提供されている `サンプルプログラム <https://github.com/rsta2/circle/blob/master/sample/README>`_ を使うことができます。
.. [#ki] Raspberry Piのモデルと対象アーキテクチャ（32ビットか64ビットか）によりバイナリイメージのファイル名は *kernel.img*, *kernel7.img*, *kernel7l.img*, *kernel8.img*, *kernel8-rpi4.img*, *kernel_2712.img* のいずれかとなります。
.. [#pi] Raspberry Pi Zero と Zero W ではターゲットを ``RASPPI = 1`` と構成する必要があります。Raspberry Pi Zero 2 Wではターゲットを ``RASPPI = 3`` と構成する必要があります。
