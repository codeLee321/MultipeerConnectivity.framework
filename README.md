# MultipeerConnectivity.framework
MultipeerConnectivity.framework：iOS7开始引入的新的蓝牙通讯开发框架，用于取代GameKit。
前面已经说了GameKit相关的蓝牙操作类从iOS7已经全部过期，苹果官方推荐使用MultipeerConnectivity代替。但是应该了解，MultipeerConnectivity.framework并不仅仅支持蓝牙连接，准确的说它是一种支持Wi-Fi网络、P2P Wi-Fi已经蓝牙个人局域网的通信框架，它屏蔽了具体的连接技术，让开发人员有统一的接口编程方法。通过MultipeerConnectivity连接的节点之间可以安全的传递信息、流或者其他文件资源而不必通过网络服务。此外使用MultipeerConnectivity进行近场通信也不再局限于同一个应用之间传输，而是可以在不同的应用之间进行数据传输（当然如果有必要的话你仍然可以选择在一个应用程序之间传输）。
要了解MultipeerConnectivity的使用必须要清楚一个概念：广播（Advertisting）和发现（Disconvering），这很类似于一种Client-Server模式。假设有两台设备A、B，B作为广播去发送自身服务，A作为发现的客户端。一旦A发现了B就试图建立连接，经过B同意二者建立连接就可以相互发送数据。在使用GameKit框架时，A和B既作为广播又作为发现，当然这种情况在MultipeerConnectivity中也很常见。
A.广播
无论是作为服务器端去广播还是作为客户端去发现广播服务，那么两个（或更多）不同的设备之间必须要有区分，通常情况下使用MCPeerID对象来区分一台设备，在这个设备中可以指定显示给对方查看的名称（display name）。另外不管是哪一方，还必须建立一个会话MCSession用于发送和接受数据。通常情况下会在会话的-(void)session:(MCSession )session peer:(MCPeerID )peerID didChangeState:(MCSessionState)state代理方法中跟踪会话状态（已连接、正在连接、未连接）;在会话的-(void)session:(MCSession )session didReceiveData:(NSData )data fromPeer:(MCPeerID *)peerID代理方法中接收数据;同时还会调用会话的-(void)sendData: toPeers:withMode: error:方法去发送数据。
广播作为一个服务器去发布自身服务，供周边设备发现连接。在MultipeerConnectivity中使用MCAdvertiserAssistant来表示一个广播，通常创建广播时指定一个会话MCSession对象将广播服务和会话关联起来。一旦调用广播的start方法周边的设备就可以发现该广播并可以连接到此服务。在MCSession的代理方法中可以随时更新连接状态，一旦建立了连接之后就可以通过MCSession的connectedPeers获得已经连接的设备。
B.发现
前面已经说过作为发现的客户端同样需要一个MCPeerID来标志一个客户端，同时会拥有一个MCSession来监听连接状态并发送、接受数据。除此之外，要发现广播服务，客户端就必须要随时查找服务来连接，在MultipeerConnectivity中提供了一个控制器MCBrowserViewController来展示可连接和已连接的设备（这类似于GameKit中的GKPeerPickerController），当然如果想要自己定制一个界面来展示设备连接的情况你可以选择自己开发一套UI界面。一旦通过MCBroserViewController选择一个节点去连接，那么作为广播的节点就会收到通知，询问用户是否允许连接。由于初始化MCBrowserViewController的过程已经指定了会话MCSession，所以连接过程中会随时更新会话状态，一旦建立了连接，就可以通过会话的connected属性获得已连接设备并且可以使用会话发送、接受数据。

在两个程序中无论是MCBrowserViewController还是MCAdvertiserAssistant在初始化的时候都指定了一个服务类型“cmj-photo”，这是唯一标识一个服务类型的标记，可以按照官方的要求命名，应该尽可能表达服务的作用。需要特别指出的是，如果广播命名为“cmj-photo”那么发现节点只有在MCBrowserViewController中指定为“cmj-photo”才能发现此服务
