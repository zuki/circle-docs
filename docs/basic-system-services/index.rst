.. _Basic system services:

基本システムサービス
---------------------

このセクションではCircleベースライブラリ `libcircle.a` がアプリケーションに提供する基本的なシステムサービスについて説明します。ここではアプリケーションにより直接使用されるクラスについてのみ説明します。Circleのすべてのクラスは `doc/classes.txt <https://github.com/rsta2/circle/blob/master/doc/classes.txt>`_ にリストアップされています。

Circleプロジェクトは集中型の単一C++ヘッダーファイルを提供しません。代わりに、特定のクラス、関数またはマクロ定義に含める必要があるヘッダーファイルは関連するサブセクションで指定されます。

.. toctree::
	:maxdepth: 1

	system-information
	memory
	synchronization
	system-log
	interrupts
	time
	direct-memory-access
	gpio-access
	multi-core-support
	cpu-clock-rate-management
	firmware-access
	direct-hardware-access
	utilities
	debugging-support
