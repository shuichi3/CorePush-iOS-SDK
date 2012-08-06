# Core Push iOS SDK

##概要

Core Push iOS SDK は、プッシュ通知ASPサービス「CORE PUSH」の iOS用のSDKになります。 ドキュメントは CORE PUSH Developer サイトに掲載しております。

 
■公式サイト

CORE PUSH：<a href="http://core-asp.com">http://core-asp.com</a>

CORE PUSH Developer（開発者向け）：<a href="http://developer.core-asp.com">http://developer.core-asp.com</a>
* Xcodeのプロジェクトのターゲットを選択し、Build Phases の Link Binary With Libraries から SDK/CorePush.framework を追加してください。

アプリケーションの動作状態に応じて通知をハンドリングするために、CorePushManagerDelegateプロトコルを実装した
クラスを CorePushManager#setDelegate で指定します。
CorePushManager#registerForRemoteNotifications を呼び出すことで APNSサーバからデバイストークンを取得し、
デバイストークンを CORE PUSH に送信します。












UIApplication#application:didFinishLaunchingWithOptions にて、以下のメソッドを呼び出します。

	[[CorePushManager shared] handleLaunchingNotificationWithOption:launchOptions];

アプリケーションが動作していない状態で通知から起動した場合はCorePushManagerDelegate#handleLaunchingNotification が呼び出されます。

