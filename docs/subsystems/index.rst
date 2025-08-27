サブシステム
-------------

このセクションではCircleのサブシステムについて説明します。サブシステムとは特定の目的のためのサービスを実装するクラスのグループのことであり、 :ref:`Basic system services` とは異なり、通常は、独自のライブラリ (:ref:`libraries` を参照) によって提供されます。ここではアプリケーションにより直接使用されるクラスについてのみ説明します。Circleのすべてのクラスは `doc/classes.txt <https://github.com/rsta2/circle/blob/master/doc/classes.txt>`_ にリストアップされています。

Circleプロジェクトは集中型の単一C++ヘッダーファイルを提供しません。代わりに、特定のクラス、関数またはマクロ定義に含める必要があるヘッダーファイルは関連するサブセクションで指定されます。

.. toctree::
	:maxdepth: 1

	multitasking
	usb
	filesystems
	tcp-ip-networking
	graphics
	vc4
