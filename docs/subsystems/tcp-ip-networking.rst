.. _TCP/IP networking:

TCP/IPネットワーキング
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

このセクションではCircleのTCP/IPネットワーキングのサポートについて説明します。初期化と構成、ソケットAPI、利用可能な上位層プロトコルのクライアントとサーバ、ユーティリティクラスが対象です。TCP/IPネットワークサポートにはシステムにスケジューラが必要です（ :ref:`Multitasking` を参照）。

以下のサンプルプログラムはTCP/IPネットワークの機能を示しています。

==============	=========================================
サンプル		説明
==============	=========================================
18-ntptime	NTPサーバからシステム時刻を設定する
20-tcpsimple	TCPエコーサーバ
21-webserver	シンプルなHTTPサーバ
31-webclient	シンプルなHTTPクライアント (レファレンス用途のみ)
33-syslog	UDP syslogサーバにログメッセージを送信する
35-mqttclient	MQTTクライアント
38-bootloader	HTTPとTFTPベースのbootloader
==============	=========================================

.. note::

	Circleは現在のところ、RFC1112に基づき、IPマルチキャストサポートレベル1（送信のみ）をサポートしています。

CNetSubSystem
^^^^^^^^^^^^^

.. code-block:: cpp

	#include <circle/net/netsubsystem.h>

.. cpp:class:: CNetSubSystem

	このクラスはCircleにおけるTCP/IPサポートを表します。このクラスのインスタンスは1つだけしか存在できません。

.. cpp:function:: CNetSubSystem::CNetSubSystem (const u8 *pIPAddress = 0, const u8 *pNetMask = 0, const u8 *pDefaultGateway = 0, const u8 *pDNSServer = 0, const char *pHostname = "raspberrypi", TNetDeviceType DeviceType = NetDeviceTypeEthernet)

	``CNetSubSystem`` のインスタンスを生成します。パラメータ ``pIPAddress``, ``pNetMask``, ``pDefaultGateway`` と ``pDNSServer`` は 4 バイトの配列を指し、ネットワーク構成を定義します（例: IP アドレス {192, 168, 0, 42}）。これらのポインタをすべて0に設定するとDHCPを使用した動的なネットワーク設定が有効になります。 ``pHostname`` はDHCPサーバーに送信されるホスト名を指定します(無効にするには0とする)。 ``DeviceType`` には ``NetDeviceTypeEthernet`` (デフォルト) または ``NetDeviceTypeWLAN`` を指定できます。

.. note::

	WLAN にアクセスするには ``DeviceType`` に ``NetDeviceTypeWLAN`` を設定するだけでは十分でありません。 ``CBcm4343Device`` (WLAN ハードウェアドライバ)、 ``CNetSubSystem``、 ``CWPASupplicant`` (セキュアな WLAN アクセスをサポートするタスク) の各クラスをこの順番でインスタンス化して初期化する必要があります。詳細はサブセクション :ref:`WLAN support` と `WLAN sample <https://github.com/rsta2/circle/tree/master/addon/wlan/sample/hello_wlan>`_ を参照してください。

.. cpp:function:: boolean CNetSubSystem::Initialize (boolean bWaitForActivate = TRUE)

	ネットワークサブシステムを初期化します。通常、このメソッドは DHCP が有効になっている場合、ネットワークコンフィギュレーションが割り当てられた後に復帰します。これにはDHCPサーバに到達できる必要があり、時間がかかります。ネットワークの初期化を早くしたい場合は、パラメータ ``bWaitForActivate`` に ``FALSE`` を設定します。そうすると、このメソッドは初期化した後、すぐに復帰します。ただし、ネットワークにアクセスする前に、 ``IsRunning()`` メソッドを使ってネットワークが利用可能かどうかを自分でテストする必要があります。

.. cpp:function:: boolean CNetSubSystem::IsRunning (void) const

	TCP/IPネットワークが利用可能で構成されている場合、 ``TRUE`` を返します。DHCPが有効になっている場合、これは既にIPアドレスが割り当てられていることを意味します。

.. cpp:function:: const char *CNetSubSystem::GetHostname (void) const

	コンストラクタに渡されたホスト名を返します。

.. cpp:function:: CNetConfig *CNetSubSystem::GetConfig (void)

	クラス ``CNetConfig`` のインスタンスとして保存されているネットワーク構成へのポインタを返します。通常、このポインタは動的に割り当てられた構成をユーザに通知するために使用されます。このポインタを使用して構成を操作しようとしてはいけません。

.. cpp:function:: static CNetSubSystem *CNetSubSystem::Get (void)

	``CNetSubSystem`` のインスタンスへのポインタを返します.

CNetConfig
^^^^^^^^^^

.. code-block:: cpp

	#include <circle/net/netconfig.h>

.. cpp:class:: CNetConfig

	このクラスのインスタンスは TCP/IP ネットワーキングサブシステムの構成を保持します。このインスタンスへのポインタは ``CNetSubSystem::GetConfig()`` を使用して要求することができます。以下のメソッドを使用してさまざまな構成項目を取得することができます。

.. cpp:function:: boolean CNetConfig::IsDHCPUsed (void) const

	ネットワークがDHCPを使って動的に構成されている場合、 ``TRUE`` を返します。

.. cpp:function:: const CIPAddress *CNetConfig::GetIPAddress (void) const

	自身のIPアドレスを返します。

.. cpp:function:: const u8 *CNetConfig::GetNetMask (void) const

	接続しているローカルネットワークのネットマスクを返します。

.. cpp:function:: const CIPAddress *CNetConfig::GetDefaultGateway (void) const

	インターネットへのデフォルトゲートウェイのIPアドレスを返します。

.. cpp:function:: const CIPAddress *CNetConfig::GetDNSServer (void) const

	DNSさーばのIPアドレスを返します。

.. cpp:function:: const CIPAddress *CNetConfig::GetBroadcastAddress (void) const

	接続しているローカルネットワークで有効な（指定の）ブロードキャストアドレスを返します。

CSocket
^^^^^^^

.. code-block:: cpp

	#include <circle/net/socket.h>
	#include <circle/net/in.h>		// IPPROTO_*, MSG_DONTWAIT 用
	#include <circle/netdevice.h>		// FRAME_BUFFER_SIZE 用

.. cpp:class:: CSocket : public CNetSocket

	このクラスはCircleにおけるTCP/IPネットワークアクセスのためのAPIを形成します。

.. note::

	CircleにおけるソケットAPIのポート番号はホストのバイトオーダーです。これは、API関数にリトルエンディアンの番号を指定する際に、バイトオーダをネットワークオーダに入れ替える必要はないことを意味します。

	操作にはブロッキングとノンブロッキングがあります。ブロッキング操作では関数の完了を待って復帰します。のンプロッキング操作は直ちに復帰します。これは、大量のデータを送信する場合などでシステムが混雑していないことを自分で確認する必要があることを意味します。

.. cpp:function:: CSocket::CSocket (CNetSubSystem *pNetSubSystem, int nProtocol)

	Circleにおいて1つのTCP/IPコネクションを表す ``CSocket`` オブジェクトを作成します。 ``pNetSubSystem`` はネットワークサブシステムへのポインタ、 ``nProtocol`` は ``IPPROTO_TCP``  か ``IPPROTO_UDP`` のいずれかです。

.. cpp:function:: CSocket::~CSocket (void)

	``CSocket`` オブジェクトを破棄してアクティブなコネクションを終了させます。

.. cpp:function:: int CSocket::Bind (u16 usOwnPort)

	ポート番号 ``usOwnPort`` をこのソケットにバインドします。成功した場合は 0、エラーの場合は < 0 を返します。

.. cpp:function:: int CSocket::Connect (const CIPAddress &rForeignIP, u16 usForeignPort)

	外部のホスト/ポートに接続する (TCP) 、または、外部のホスト/ポートアドレスを設定します (UDP)。 ``rForeignIP`` は接続するホストのIPアドレスです。 ``usForeignPort`` は接続するポート番号です。成功した場合は 0、エラーの場合は < 0 を返します。

.. cpp:function:: int CSocket::Listen (unsigned nBackLog = 4)

	着信コネクションをリッスンします (TCPのみ). これより先に ``Bind()`` を呼び出す必要があります。 ``nBackLog`` は ``Accept()`` により連続して受け付けることができる同時接続の最大数です（最大32）。成功した場合は 0、エラーの場合は < 0 を返します。

.. cpp:function:: CSocket *CSocket::Accept (CIPAddress *pForeignIP, u16 *pForeignPort)

	着信コネクションを受け付けます (TCPのみ). これより先に ``Listen()`` を呼び出す必要があります。 ``pForeignIP`` はリモートホストのIPアドレスを受信する ``CIPAddress`` オブジェクトへのポインタです。リモートポート番号は ``*pForeignPort`` で返されます。リモートホストと通信する際に使用する新規作成されたソケットを返します。エラーの場合は 0 を返します。

.. cpp:function:: int CSocket::Send (const void *pBuffer, unsigned nLength, int nFlags)

	リモートホストにメッセージを送信します。 ``pBuffer`` はメッセージへのポインタであり、 ``nLength`` はそのバイト長であす。 ``nFlags`` には ``MSG_DONTWAIT`` (ノンブロッキング操作) または 0 (ブロッキング操作) を指定することができます。送信したメッセージの長さを返します、エラーの場合は < 0 を返します。

.. cpp:function:: int CSocket::Receive (void *pBuffer, unsigned nLength, int nFlags)

	リモートホストからメッセージを受信します。 ``pBuffer`` はメッセージバッファへのポインタで、 ``nLength`` はそのバイト長です。 ``nLength`` は少なくとも ``FRAME_BUFFER_SIZE`` である必要があります。そうでないとデータが消失する可能性があります。 ``nFlags`` には ``MSG_DONTWAIT`` (ノンブロッキング操作) または 0 (ブロッキング操作) を指定することができます。受信したしたメッセージの長さを返します。 ``MSG_DONTWAIT`` を指定しており、利用可能なメッセージがない場合は 0 を返します。エラーの場合は < 0 を返します。

.. cpp:function:: int CSocket::SendTo (const void *pBuffer, unsigned nLength, int nFlags, const CIPAddress &rForeignIP, u16 nForeignPort)

	指定したリモートホストにメッセージを送信します。 ``pBuffer`` はメッセージへのポインタ、 ``nLength`` はそのバイト長です。 ``nFlags`` には ``MSG_DONTWAIT`` (ノンブロッキング操作) または 0 (ブロッキング操作) を指定することができます。 ``rForeignIP`` は送信先のホストの IP アドレスです (TCP ソケットでは無視されます)。 ``nForeignPort`` は送信先のポート番号です (TCP ソケットでは無視されます)。送信したメッセージの長さを返します。エラーの場合は < 0 を返します。

.. cpp:function:: int CSocket::ReceiveFrom (void *pBuffer, unsigned nLength, int nFlags, CIPAddress *pForeignIP, u16 *pForeignPort)

	リモートホストからメッセージを受信し、リモートホストのホスト/ポートを返します。 ``pBuffer`` はメッセージバッファへのポインタ、 ``nLength`` はそのバイト長です。 ``nLength`` は少なくとも ``FRAME_BUFFER_SIZE`` である必要があります。そうでないとデータが消失する可能性があります。 ``nFlags`` には ``MSG_DONTWAIT`` (ノンブロッキング操作) または 0 (ブロッキング操作) を指定することができます。 ``pForeignIP`` は ``CIPAddress`` オブジェクトへのポインタで、メッセージを送信したホストのIPアドレスが設定されます。メッセージが送信されたポート番号は ``*pForeignPort`` に設定されます。受信したメッセージの長さを返します。 ``MSG_DONTWAIT`` を指定しており、利用可能なメッセージがない場合は 0 を返します。エラーの場合は < 0 を返します。

.. cpp:function:: int CSocket::SetOptionBroadcast (boolean bAllowed)

	``bAllowed`` はこのソケットでブロードキャストメッセージの送受信を許可するか否を指定します (デフォルトは ``FALSE``)。 ``Bind()`` または ``Connect()`` の後に ``bAllowed = TRUE`` を指定してこのメソッドを呼び出すと、ブロードキャストメッセージを送受信できるようになります (TCP ソケットでは無視されます)。成功した場合は 0、エラーの場合は < 0 を返します。

.. cpp:function:: const u8 *CSocket::GetForeignIP (void) const

	接続したリモートホストのIPアドレス（4バイト）へのポインタを返します。ソケットが接続されていない場合は 0 を返します。

Clients
^^^^^^^

CDNSClient
""""""""""

.. code-block:: cpp

	#include <circle/net/dnsclient.h>

.. cpp:class:: CDNSClient

	This class supports the resolve of an Internet domain host name to an IP address.

.. cpp:function:: CDNSClient::CDNSClient (CNetSubSystem *pNetSubSystem)

	Creates a ``CDNSClient`` object. ``pNetSubSystem`` is a pointer to the network subsystem.

.. cpp:function:: boolean CDNSClient::Resolve (const char *pHostname, CIPAddress *pIPAddress)

	Resolves the host name ``pHostname`` to an IP address, returned in ``*pIPAddress``. ``pHostname`` can be a dotted IP address string (e.g. "192.168.0.42") too, which will be converted. Returns ``TRUE`` on success.

CHTTPClient
"""""""""""

.. code-block:: cpp

	#include <circle/net/httpclient.h>
	#include <circle/net/http.h>		// for THTTPStatus

.. cpp:class:: CHTTPClient

	This class can be used to generate requests to a HTTP server.

.. note::

	In the Internet of today there are only a few webservers any more, which provide plain HTTP access. For HTTPS (HTTP over TLS) access with Circle you can use the `circle-stdlib <https://github.com/smuehlst/circle-stdlib>`_ project, which includes Circle as a submodule.

.. cpp:function:: CHTTPClient::CHTTPClient (CNetSubSystem *pNetSubSystem, CIPAddress &rServerIP, u16 usServerPort = HTTP_PORT, const char *pServerName = 0)

	Creates a ``CHTTPClient`` object. ``pNetSubSystem`` is a pointer to the network subsystem. ``rServerIP`` is the IP address of the server and ``usServerPort`` the server port to connect. ``pServerName`` is the host name of the server, which is required for the access to virtual servers (multiple websites with different host names, hosted on the same server).

.. cpp:function:: THTTPStatus CHTTPClient::Get (const char *pPath, u8 *pBuffer, unsigned *pLength)

	Sends a GET request to the server. ``pPath`` is the absolute path of the requested document, optionally with appended parameters:

	``/path/filename.ext[?name=value[&name=value...]]``

	The received content will be returned in ``pBuffer``. ``*pLength`` is the buffer size in bytes on input and the received content length on output. Returns the HTTP status (``HTTPOK`` on success).

.. cpp:function:: THTTPStatus CHTTPClient::Post (const char *pPath, u8 *pBuffer, unsigned *pLength, const char *pFormData)

	Sends a POST request to the server. ``pPath`` is the absolute path of the requested document, optionally with appended parameters:

	``/path/filename.ext[?name=value[&name=value...]]``

	The received content will be returned in ``pBuffer``. ``*pLength`` is the buffer size in bytes on input and the received content length on output. ``pFormData`` are the posted parameters in this format:

	``name=value[&name=value...]``

	Returns the HTTP status (``HTTPOK`` on success).

CNTPClient
""""""""""

.. code-block:: cpp

	#include <circle/net/ntpclient.h>

.. cpp:class:: CNTPClient

	This class can be used to request the current time from a Network Time Protocol server.

.. cpp:function:: CNTPClient::CNTPClient (CNetSubSystem *pNetSubSystem)

	Creates a ``CNTPClient`` object. ``pNetSubSystem`` is a pointer to the network subsystem.

.. cpp:function:: unsigned CNTPClient::GetTime (CIPAddress &rServerIP)

	Requests the current time from a NTP server. ``rServerIP`` is the IP address from the NTP server, which can be resolved using the class ``CDNSClient``. Returns the current time in seconds since 1970-01-01 00:00:00 UTC, or 0 on error.

CNTPDaemon
""""""""""

.. code-block:: cpp

	#include <circle/net/ntpdaemon.h>

.. cpp:class:: CNTPDaemon : public CTask

	This class is a background task, which continuously (all 15 minutes) updates the Circle system time from a NTP server. It uses the class ``CNTPClient``.

.. cpp:function:: CNTPDaemon::CNTPDaemon (const char *pNTPServer, CNetSubSystem *pNetSubSystem)

	Creates the ``CNTPDaemon`` task. ``pNTPServer`` is the host name of the NTP server (e.g. "pool.ntp.org"). ``pNetSubSystem`` is a pointer to the network subsystem. This object must be created using the ``new`` operator.

CMQTTClient
"""""""""""

.. code-block:: cpp

	#include <circle/net/mqttclient.h>

.. cpp:class:: CMQTTClient : public CTask

	This class is a client for the MQTT protocol, according to the `MQTT v3.1.1 specification <http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.pdf>`_. It is implemented as a task. To use this class, you have to derive a user defined class from ``CMQTTClient`` and override its virtual methods. The task must be created with the ``new`` operator.

.. warning::

	This implementation does not support multi-byte-characters in strings.

.. cpp:function:: CMQTTClient::CMQTTClient (CNetSubSystem *pNetSubSystem, size_t nMaxPacketSize = 1024, size_t nMaxPacketsQueued = 4, size_t nMaxTopicSize = 256)

	Creates a ``CMQTTClient`` task. ``pNetSubSystem`` is a pointer to the network subsystem. ``nMaxPacketSize`` is the maximum allowed size of a MQTT packet sent or received (topic size + payload size + a few bytes protocol overhead). ``nMaxPacketsQueued`` is the maximum number of MQTT packets queue-able on receive. If processing a received packet takes longer, further packets have to be queued. ``nMaxTopicSize`` is the maximum allowed size of a received topic string.

.. cpp:function:: boolean CMQTTClient::IsConnected (void) const

	Returns ``TRUE`` if an active connection to the MQTT broker exists.

.. cpp:function:: void CMQTTClient::Connect (const char *pHost, u16 usPort = MQTT_PORT, const char *pClientIdentifier = 0, const char *pUsername = 0, const char *pPassword = 0, u16 usKeepAliveSeconds = 60, boolean bCleanSession = TRUE, const char *pWillTopic = 0, u8 uchWillQoS = 0, boolean bWillRetain = FALSE, const u8 *pWillPayload = 0, size_t nWillPayloadLength = 0)

	Establishes a connection to the MQTT broker ``pHost`` (host name or IP address as a dotted string). ``usPort`` is the port number of the MQTT broker service (default 1883). ``pClientIdentifier`` is the identifier string of this client (if 0 set internally to ``raspiNNNNNNNNNNNNNNNN``, N = hex digits of the serial number). ``pUsername`` is the user name for authorization (0 if not required). ``pPassword`` is the password for authorization (0 if not required). ``usKeepAliveSeconds`` is the duration of the keep alive period in seconds (default 60). ``bCleanSession`` specifies, if this should be a clean MQTT session. (default TRUE).

	``pWillTopic`` is the topic string for the last will message (no last will message if 0). ``uchWillQoS`` is the QoS setting for last will message (default unused). ``bWillRetain`` is the retain parameter for last will message (default unused). ``pWillPayload`` is a pointer to the last will message payload (default unused). ``nWillPayloadLength`` is the length of the last will message payload (default unused).

.. cpp:function:: void CMQTTClient::Disconnect (boolean bForce = FALSE)

	Closes the connection to a MQTT broker. ``bForce`` forces a TCP disconnect only and does not send a MQTT DISCONNECT packet.

.. cpp:function:: void CMQTTClient::Subscribe (const char *pTopic, u8 uchQoS = MQTT_QOS2)

	Subscribes to the MQTT topic ``pTopic`` (may include wildchars). ``uchQoS`` is the maximum QoS value for receiving messages with this topic (default QoS 2).

.. cpp:function:: void CMQTTClient::Unsubscribe (const char *pTopic)

	Unsubscribes from the MQTT topic ``pTopic``.

.. cpp:function:: void CMQTTClient::Publish (const char *pTopic, const u8 *pPayload = 0, size_t nPayloadLength = 0, u8 uchQoS = MQTT_QOS1, boolean bRetain = FALSE)

	Publishes the MQTT topic ``pTopic``. ``pPayload`` is a pointer to the message payload (default unused). ``nPayloadLength`` is the length of the message payload (default 0). ``uchQoS`` is the QoS value for sending the PUBLISH message (default QoS 1). ``bRetain`` is the retain parameter for the message (default FALSE).

.. cpp:function:: virtual void CMQTTClient::OnConnect (boolean bSessionPresent)

	This is a callback entered when the connection to the MQTT broker has been established. ``bSessionPresent`` specifies, if a session was already present on the server for this client.

.. cpp:function:: virtual void CMQTTClient::OnDisconnect (TMQTTDisconnectReason Reason)

	This is a callback entered when the connection to the MQTT broker has been closed. ``Reason`` is the reason for closing the connection, which can be:

.. code-block:: cpp

	enum TMQTTDisconnectReason
	{
		MQTTDisconnectFromApplication			= 0,

		// CONNECT errors
		MQTTDisconnectUnacceptableProtocolVersion	= 1,
		MQTTDisconnectIdentifierRejected		= 2,
		MQTTDisconnectServerUnavailable			= 3,
		MQTTDisconnectBadUsernameOrPassword		= 4,
		MQTTDisconnectNotAuthorized			= 5,

		// additional errors
		MQTTDisconnectDNSError,
		MQTTDisconnectConnectFailed,
		MQTTDisconnectFromPeer,
		MQTTDisconnectInvalidPacket,
		MQTTDisconnectPacketIdentifier,
		MQTTDisconnectSubscribeError,
		MQTTDisconnectSendFailed,
		MQTTDisconnectPingFailed,
		MQTTDisconnectNotSupported,
		MQTTDisconnectInsufficientResources,

		MQTTDisconnectUnknown
	};

.. cpp:function:: virtual void CMQTTClient::OnMessage (const char *pTopic, const u8 *pPayload, size_t nPayloadLength, boolean bRetain)

	This is a callback entered when a PUBLISH message has been received for a subscribed topic. ``pTopic`` is the topic of the received message. ``pPayload`` is a pointer to the payload of the received message. ``nPayloadLength`` is the length of the payload of the received message. ``bRetain`` is the retain parameter of the received message.

.. cpp:function:: virtual void CMQTTClient::OnLoop (void)

	This is a callback regularly entered from the MQTT client task.

.. _CSysLogDaemon:

CSysLogDaemon
"""""""""""""

.. code-block:: cpp

	#include <circle/net/syslogdaemon.h>

.. cpp:class:: CSysLogDaemon : public CTask

	This class is a background task, which sends the messages from the :ref:`System log` to a RFC5424/RFC5426 syslog server via UDP.

.. cpp:function:: CSysLogDaemon::CSysLogDaemon (CNetSubSystem *pNetSubSystem, const CIPAddress &rServerIP, u16 usServerPort = SYSLOG_PORT)

	Creates the ``CSysLogDaemon`` task. ``pNetSubSystem`` is a pointer to the network subsystem. ``rServerIP`` is the IP address of the syslog server. ``usServerPort`` is the port number of the syslog server (default 514). This object must be created using the ``new`` operator.

Servers
^^^^^^^

CHTTPDaemon
"""""""""""

.. code-block:: cpp

	#include <circle/net/httpdaemon.h>
	#include <circle/net/http.h>		// for THTTPStatus

.. cpp:class:: CHTTPDaemon : public CTask

	This class implements a simple HTTP server as a task. You have to derive a user class from it, override the virtual methods and create it using the ``new`` operator to start it.

.. note::

	This class uses a listener/worker model. The initially created task listens for incoming requests (listener) and spawns a child task (worker), which processes the request and terminates afterwards.

.. cpp:function:: CHTTPDaemon::CHTTPDaemon (CNetSubSystem *pNetSubSystem, CSocket *pSocket = 0, unsigned nMaxContentSize = 0, u16 nPort = HTTP_PORT, unsigned nMaxMultipartSize = 0)

	Creates the ``CHTTPDaemon`` task. ``pNetSubSystem`` is a pointer to the network subsystem. ``pSocket`` is 0 for first created instance (listener). ``nMaxContentSize`` is the buffer size for the content of the created worker tasks. Set this parameter to the maximum length in bytes of a webpage, which is generated by your server. ``nPort`` is the port number to listen on (default 80). ``nMaxMultipartSize`` is the buffer size for received multipart form data. If your server receives requests, which include multipart form data, this parameter must be set to the maximum length of this data, which you want to process.

.. cpp:function:: virtual CHTTPDaemon *CHTTPDaemon::CreateWorker (CNetSubSystem *pNetSubSystem, CSocket *pSocket) = 0

	Creates a worker instance of your derived webserver class. ``pNetSubSystem`` is a pointer to the network subsystem. ``pSocket`` is the socket that manages the incoming connection. Both parameters have to be handed over to the constructor of your derived webserver class, to be passed to ``CHTTPDaemon::CHTTPDaemon``. See this example:

.. code-block:: cpp
	:caption: mywebserver.h

	class CMyWebServer : public CHTTPDaemon
	{
	public:
		CMyWebServer (CNetSubSystem *pNetSubSystem,
			      CActLED	    *pActLED,	   // some user data
			      CSocket	    *pSocket = 0); // 0 for first instance

		CHTTPDaemon *CreateWorker (CNetSubSystem *pNetSubSystem,
					   CSocket	 *pSocket);

		...
	};

.. code-block:: cpp
	:caption: mywebserver.cpp

	#define MAX_CONTENT_SIZE	4000	// maximum content size of your pages

	CMyWebServer::CMyWebServer (CNetSubSystem *pNetSubSystem,
				    CActLED	  *pActLED,
				    CSocket	  *pSocket)
	:	CHTTPDaemon (pNetSubSystem, pSocket, MAX_CONTENT_SIZE),
		m_pActLED (pActLED)
	{
	}

	CHTTPDaemon *CMyWebServer::CreateWorker (CNetSubSystem *pNetSubSystem,
						 CSocket       *pSocket)
	{
		return new CMyWebServer (pNetSubSystem, m_pActLED, pSocket);
	}


.. cpp:function:: virtual THTTPStatus CHTTPDaemon::GetContent (const char *pPath, const char *pParams, const char *pFormData, u8 *pBuffer, unsigned *pLength, const char **ppContentType) = 0

	Define this method to provide your own content. ``pPath`` is the path of the file to be sent (e.g. "/index.html", can be "/" too). ``pParams`` are the GET parameters ("" for none). ``pFormData`` contains the parameters from the form data from POST ("" for none). Copy your content to ``pBuffer``. ``*pLength`` is the buffer size in bytes on input and the content length on output. ``*ppContentType`` must be set to the MIME type, if it is not "text/html". This method has to return the HTTP status (``HTTPOK`` on success).

.. cpp:function:: virtual void CHTTPDaemon::WriteAccessLog (const CIPAddress &rRemoteIP, THTTPRequestMethod RequestMethod, const char *pRequestURI, THTTPStatus Status, unsigned nContentLength)

	Overwrite this method to implement your own access logging. ``rRemoteIP`` is the IP address of the client. ``RequestMethod`` is the method of the request and ``pRequestURI`` its URI. ``Status`` and ``nContentLength`` specify the returned HTTP status number and the length of the sent content. The default implementation of this method writes a message to the :ref:`System log`.

.. cpp:function:: boolean CHTTPDaemon::GetMultipartFormPart (const char **ppHeader, const u8 **ppData, unsigned *pLength)

	This method can be called from ``GetContent()`` and returns the next part of multipart form data (``TRUE`` if available). This data is not available after returning from ``GetContent()`` any more. ``*ppHeader`` returns a pointer to the part header. ``*ppData`` returns a pointer to part data. ``*pLength`` returns the part data length.

CTFTPDaemon
"""""""""""

.. code-block:: cpp

	#include <circle/net/tftpdaemon.h>

.. cpp:class:: CTFTPDaemon : public CTask

	This class provides a server task for the TFTP protocol. You have to implement the pure virtual methods in a derived class, start the task with the ``new`` operator and will be able to receive and handle TFTP requests. This server can handle only one connection at a time, and works in binary mode only. The `TFTP fileserver sample <https://github.com/rsta2/circle/tree/master/addon/tftpfileserver/sample>`_ demonstrates the usage of this class.

.. cpp:function:: CTFTPDaemon::CTFTPDaemon (CNetSubSystem *pNetSubSystem)

	Creates the ``CTFTPDaemon`` task. ``pNetSubSystem`` is a pointer to the network subsystem.

.. cpp:function:: virtual boolean CTFTPDaemon::FileOpen (const char *pFileName) = 0

	Virtual method entered to open a file for read to be sent via TFTP. ``pFileName`` is the file name sent by the client. Returns ``TRUE`` on success.

.. cpp:function:: virtual boolean CTFTPDaemon::FileCreate (const char *pFileName) = 0

	Virtual method entered to create a file for write to be received via TFTP. ``pFileName`` is the file name sent by the client. Returns ``TRUE`` on success.

.. cpp:function:: virtual boolean CTFTPDaemon::FileClose (void) = 0

	Virtual method entered to close the currently open file. Returns ``TRUE`` on success.

.. cpp:function:: virtual int CTFTPDaemon::FileRead (void *pBuffer, unsigned nCount) = 0

	Virtual method entered to read ``nCount`` bytes from the currently open file into ``pBuffer``. Returns the number of bytes read, or < 0 on error.

.. cpp:function:: virtual int CTFTPDaemon::FileWrite (const void *pBuffer, unsigned nCount) = 0

	Virtual method entered to write ``nCount`` bytes from ``pBuffer`` into the currently open file. Returns the number of bytes written, or < 0 on error.

.. cpp:function:: virtual void CTFTPDaemon::UpdateStatus (TStatus Status, const char *pFileName)

	Virtual method entered to inform the derived class about the progress of an ongoing transfer. ``pFileName`` is the name of the transferred file, if available. ``Status`` can have the following values:

.. code-block:: cpp

	enum TStatus
	{
		StatusIdle,		// pFileName not available
		StatusReadInProgress,
		StatusWriteInProgress,
		StatusReadCompleted,
		StatusWriteCompleted,
		StatusReadAborted,
		StatusWriteAborted
	};

Utilities
^^^^^^^^^

CIPAddress
""""""""""

.. code-block:: cpp

	#include <circle/net/ipaddress.h>

.. c:macro:: IP_ADDRESS_SIZE

	The size of an IP (v4) address (4 bytes).

.. cpp:class:: CIPAddress

	This class encapsulates an IP (v4) address.

.. cpp:function:: CIPAddress::CIPAddress (void)
.. cpp:function:: CIPAddress::CIPAddress (u32 nAddress)
.. cpp:function:: CIPAddress::CIPAddress (const u8 *pAddress)
.. cpp:function:: CIPAddress::CIPAddress (const CIPAddress &rAddress)

	Creates an ``CIPAddress`` object. Initialize it from different address formats.

.. cpp:function:: boolean CIPAddress::operator== (const CIPAddress &rAddress2) const
.. cpp:function:: boolean CIPAddress::operator!= (const CIPAddress &rAddress2) const
.. cpp:function:: boolean CIPAddress::operator== (const u8 *pAddress2) const
.. cpp:function:: boolean CIPAddress::operator!= (const u8 *pAddress2) const
.. cpp:function:: boolean CIPAddress::operator== (u32 nAddress2) const
.. cpp:function:: boolean CIPAddress::operator!= (u32 nAddress2) const

	Compares this IP address with a second IP address in different formats.

.. cpp:function:: CIPAddress &CIPAddress::operator= (u32 nAddress)

	Assign a new IP address ``nAddress``.

.. cpp:function:: void CIPAddress::Set (u32 nAddress)
.. cpp:function:: void CIPAddress::Set (const u8 *pAddress)
.. cpp:function:: void CIPAddress::Set (const CIPAddress &rAddress)

	Sets the IP address in different formats.

.. cpp:function:: void CIPAddress::SetBroadcast (void)

	Sets the IP address to the broadcast address (255.255.255.255).

.. cpp:function:: CIPAddress::operator u32 (void) const

	Returns the IP address as ``u32`` value.

.. cpp:function:: const u8 *CIPAddress::Get (void) const

	Returns a pointer to the IP address as an array with 4 bytes.

.. cpp:function:: void CIPAddress::CopyTo (u8 *pBuffer) const

	Copy the IP address to a buffer (4 bytes).

.. cpp:function:: boolean CIPAddress::IsNull (void) const

	Returns ``TRUE``, if the IP address components are all zero (0.0.0.0).

.. cpp:function:: boolean CIPAddress::IsBroadcast (void) const

	Returns ``TRUE`` if the IP address is the broadcast address (255.255.255.255).

.. cpp:function:: boolean CIPAddress::IsMulticast (void) const

	Returns ``TRUE`` if the IP address is a multicast address (224.x.x.x to 239.x.x.x).

.. cpp:function:: unsigned CIPAddress::GetSize (void) const

	Returns the size of an IP (v4) address (4).

.. cpp:function:: void CIPAddress::Format (CString *pString) const

	Sets ``*pString`` to the dotted string representation of the IP address.

.. cpp:function:: boolean CIPAddress::OnSameNetwork (const CIPAddress &rAddress2, const u8 *pNetMask) const

	Returns ``TRUE``, if this IP address is on the same network as ``rAddress2`` with ``pNetMask`` applied.

CMACAddress
"""""""""""

.. code-block:: cpp

	#include <circle/macaddress.h>

.. note::

	This class is belongs to the Circle base library, because it is needed there to implement non-USB network device drivers.

.. c:macro:: MAC_ADDRESS_SIZE

	The size of an (Ethernet) MAC address (6 bytes).

.. cpp:class:: CMACAddress

	This class encapsulates an (Ethernet) MAC address.

.. cpp:function:: CMACAddress::CMACAddress (void)

	Creates an ``CMACAddress`` object. The address is initialized as "invalid" and must be set, before it can be read.

.. cpp:function:: CMACAddress::CMACAddress (const u8 *pAddress)

	Creates an ``CMACAddress`` object. Set it from ``pAddress``, which points to an array with 6 bytes.

.. cpp:function:: boolean CMACAddress::operator== (const CMACAddress &rAddress2) const
.. cpp:function:: boolean CMACAddress::operator!= (const CMACAddress &rAddress2) const

	Compares this MAC address with a second MAC address.

.. cpp:function:: void CMACAddress::Set (const u8 *pAddress)

	Sets the MAC address to ``pAddress``, which points to an array with 6 bytes.

.. cpp:function:: void CMACAddress::SetBroadcast (void)

	Sets the MAC address to the (Ethernet) broadcast address (FF:FF:FF:FF:FF:FF).

.. cpp:function:: const u8 *CMACAddress::Get (void) const

	Returns a pointer to the MAC address as an array with 6 bytes.

.. cpp:function:: void CMACAddress::CopyTo (u8 *pBuffer) const

	Copy the MAC address to a buffer (6 bytes).

.. cpp:function:: boolean CMACAddress::IsBroadcast (void) const

	Returns ``TRUE`` if the MAC address is the (Ethernet) broadcast address (FF:FF:FF:FF:FF:FF).

.. cpp:function:: unsigned CMACAddress::GetSize (void) const

	Returns the size of an (Ethernet) MAC address (6).

.. cpp:function:: void CMACAddress::Format (CString *pString) const

	Sets ``*pString`` to the string representation of the MAC address.

.. _WLAN support:

WLANサポート
^^^^^^^^^^^^

CircleにおけるWLANのサポートは次の3つの要素に基づいています:

1. WLANハードウェア用のドライバクラス :cpp:class:`CBcm4343Device`
2. :ref:`TCP/IP networking` サブシステム, クラス :cpp:class:`CNetSubSystem` からインスタンス課されます
3. *WPA Supplicant* ライブラリ, サブモジュール `hostap <https://github.com/rsta2/hostap/tree/hostap_0_7_0-circle>`_ でビルドされており、ラッパークラス  :cpp:class:`CWPASupplicant` を通じてインスタンス化されます

CircleでWLANサポートを有効にするにはこれらの要素をこの順で作成し、初期化する必要があります。これは `WLAN sample <https://github.com/rsta2/circle/tree/master/addon/wlan/sample/hello_wlan>`_ で示されています。3番目の要素はセキュアなWLANネットワークを使用する場合にのみ必要です。

.. note::

	TCP/IPネットワーキングサブシステムがWLANデバイスを使用するように構成されており (``NetDeviceTypeWLAN``) 、DHCPサーバからのIPアドレスを待たないように初期化されている (パラメータ ``FALSE``) 必要があります。DHCP プロトコルが動作するためには *WPA Supplicant* が必要だからです。そうでないと :cpp:func:`CNetSubSystem::Initialize()` は決して復帰しません。

CWPASupplicant
""""""""""""""

.. code-block:: cpp

	#include <wlan/hostap/wpa_supplicant/wpasupplicant.h>

.. cpp:class:: CWPASupplicant

	このクラスはライブラリとしてCircleに移植されている有名な *WPA Supplicant* アプリケーションのラッパーです。セキュアな（つまりWPA2）WLANネットワークに接続するためにはこのクラスのインスタンスが必要です。 *WPA Supplicant* が初期化される前にすでにWLAN ハードウェアドライバ :cpp:class:`CBcm4343Device` と :ref:`TCP/IP networking` サブシステムが動作している必要があります。

.. cpp:function:: CWPASupplicant::CWPASupplicant (const char *pConfigFile)

	このクラスのインスタンスを作成します。 ``pConfigFile`` は構成ファイルのパス (たとえば、"SD:/wpa_supplicant.conf") です。.

.. cpp:function:: boolean CWPASupplicant::Initialize (void)

	Initializes the *WPA Supplicant* モジュールを初期化して、構成ファイルで設定されているWLANネットワークへの接続を自動的に開始します。

.. cpp:function:: boolean CWPASupplicant::IsConnected (void) const

	設定したWLANネットワークへの接続が現在アクティブの場合、 ``TRUE`` を返します。
