.. _stdlib-support:

STDLIBのサポート
----------------

Circleはベアメタルアプリケーションの構築にCとC++の標準ライブラリ機能を使用可能と
するソリューションに対するいくつかのサポートを提供します。これはビルドオプション
``STDLIB_SUPPORT`` を使用して構成され、Config.mkまたはRules.mkファイルで次のように
定義されています。

* STDLIB_SUPPORT=0: 外部ライブラリを使用せずにビルドします(古い設定)。
* STDLIB_SUPPORT=1: libgcc.aだけを使ってビルドします(現在のデフォルト)
* STDLIB_SUPPORT=2: C標準ライブラリも使ってビルドします．
* STDLIB_SUPPORT=3: C++の標準ライブラリも使ってビルドします。

STDLIB_SUPPORT=2とSTDLIB_SUPPORT=3を実装するにはCircleは外部プロジェクトのサポートが
必要であることに注意してください。これは主にStephan Muehlstrasser氏による次の
プロジェクト（Circleへのnewlibの移植）により実現されています。

	https://github.com/smuehlst/circle-stdlib

CircleはこのプロジェクトにGitサブモジュールとして含まれており、CircleでC標準
ライブラリ関数を使用するために必要なすべてのライブラリ（Circleライブラリを含む）を
ビルドするためのconfigureスクリプトとmakefileが提供されています。また、この
プロジェクトにはこのサポートがどのように利用できるかを示す具体的なサンプル
プログラムも含まれています。詳しくは、このプロジェクトのドキュメントをご覧ください。

ツールチェーンの中にはnewlibを使ってビルドされたC++標準ライブラリのサポートが
提供されているものもあります。それらは次のサイトでダウンロードできます。

	https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads

これらのツールチェーンとStephan Muehlstrasser氏によるnewlib移植版のCircleを使う
ことで、C++標準ライブラリの機能を使用したCircleアプリケーションを開発することが
できます。これはcircle-stdlibプロジェクトのconfigureスクリプトでもサポートされて
おり、デフォルト設定になっています。アプリケーションのビルド時に問題が発生した場合
（たとえばRaspbianでのビルド時）、build.bashのオプション`--no-cpp`を使って、
C標準ライブラリだけを使うようにすることができます。
