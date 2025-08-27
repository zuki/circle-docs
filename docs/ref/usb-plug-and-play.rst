.. _usb-plug-and-play:

USBプラグ・アンド・プレイ
-------------------------

CircleはリリースStep43からUSBプラグ・アンド・プレイ（USB PnP）に対応しました。
このファイルはUSB PnPを使用するためのヒントを提供します。Circleアプリケーションは
USB PnPを特別にサポートする必要がありますが、これは必須ではありません。アプリケーションは
USB PnPに対応しなくても変更することなく、これまでと同じように動作することが
できます。

Raspberry Pi 1-3やZeroでUSB PnPを使うには`include/circle/sysconfig.h`で
システムオプション`USE_USB_SOF_INTR`が定義されていることが前提になります。
USB PnPはこのオプションなしでも動作するかもしれませんが、信頼性はないので
推奨しません。

USB PnPの有効化
^^^^^^^^^^^^^^^

既存の非USB PnPのアプリケションをUSB PnP化するには次のような変更が必要です。

* CKernelのUSBホストコントローラデバイスm_USBHCIのメンバーイニシャライザが第3パラメータを取得するようにする。USB PnPを有効にするためには、このパラメータにTRUEをセットする必要があります。FALSE（デフォルト値）をセットした場合、アプリケーションはUSB-PnPを認識しません。

.. code-block:: cpp

  CKernel::CKernel (void)
  :	...
    m_USBHCI (&m_Interrupt, &m_Timer, TRUE),
    ...


* アプリケーションはコア0のメインループで`m_USBHCI.UpdatePlugAndPlay()`を継続的に呼び出す必要があります。これにより、USBデバイスツリーの更新やUSBデバイスの着脱の検出が可能になります。このメソッドはIRQ_LEVELから呼んではいけません。

.. code-block:: cpp

	boolean bUpdated = m_USBHCI.UpdatePlugAndPlay ();


* ``m_USBHCI.UpdatePlugAndPlay()`` は真偽値を返し、USBデバイスツリーが更新された可能性がある場合はTRUEを返します。この場合、アプリケーションは新しいUSBデバイスが出現したことを期待できます。アプリケーションは制御したいデバイスオブジェクトへのポインタを取得する必要があります。

  .. code-block:: cpp

    if (bUpdated && m_pDevice == 0)
    {
      m_pDevice = m_DeviceNameService.GetDevice ("devname", FALSE or TRUE);
      if (m_pDevice != 0)
      {
        m_pDevice->RegisterRemovedHandler (DeviceRemovedHandler, this);
        ...

  これはデバイスポインタがまだ未知の場合にのみ必要です。そのために、CKernelにはデバイスポインタを保持するメンバ変数が必要であり、ポインタが既知でない場合は0になります。このポインタは"volatile"で定義する必要があります。

  .. code-block:: cpp

    class CKernel
    {
      ...
      static void DeviceRemovedHandler (CDevice *pDevice, void *pContext);
      ...
      CDevice * volatile m_pDevice;
      ...

* USB PnPが有効な場合、USBコネクタから対象のデバイスが取り外されると、USBデバイスオブジェクトが（ ``UpdatePlugAndPlay()`` メソッド内で）破壊されることがあります。アプリケーションはstaticな ``DeviceRemovedHandler()`` 関数を定義し、デバイスオブジェクトが削除されるときに呼び出されるハンドラとして登録する必要があります（上記参照）。

  .. code-block:: cpp

    void CKernel::DeviceRemovedHandler (CDevice *pDevice, void *pContext)
    {
      CKernel *pThis = (CKernel *) pContext;

      pThis->m_pDevice = 0;
    }

  この場合、削除されたハンドラは`m_pDevice`ポインタを0にリセットするだけなので
  アプリケーションは現在利用可能なデバイスオブジェクトがないことを認識する
  ことができます。

* これによりアプリケーションはポインタが0でなければ`m_pDevice`ポインタ（実際のデバイスクラスへのポインタにキャストする必要があります）を使用して制御するデバイスにアクセスできるようになります。

注意点
^^^^^^

* ``sample/README`` ファイルで ``[PnP]`` マークが付いたサンプルは、USB PnPが有効なサンプルです。
  あなたのアプリケーションでUSB PnPを有効にする方法の詳細についてはこれらをご覧ください。

* USBデバイスが取り外さると現在処理中のUSBリクエストは失敗し、USBドライバからエラー
  メッセージが表示されることがあります。これらのメッセージは通常、無視することができます。

* USB大容量記憶装置（フラッシュドライブなど）は、データの損失を防ぐためにデバイスを
  取り外す前にアンマウントする必要があります。これにはユーザによる操作が必要に
  なります。ユーザがユーザインターフェイスを介してデバイスの取り外しを要求したら
  アプリケーションがファイルシステムをアンマウントし、デバイスを物理的に取り外す
  ことができるようになった旨の情報をユーザーに提供します。

* 現在、USB netデバイスはUSB PnPをサポートしていません。これらのデバイスは
  オンボードに固定されており取り外すことができないため、これは必要ありません。
