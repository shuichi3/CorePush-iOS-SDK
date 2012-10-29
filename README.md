# Core Push iOS SDK

##概要

Core Push iOS SDK は、プッシュ通知ASPサービス「CORE PUSH」の iOS用のSDKになります。 ドキュメントは CORE PUSH Developer サイトに掲載しております。

 
■公式サイト

CORE PUSH：<a href="http://core-asp.com">http://core-asp.com</a>

CORE PUSH Developer（開発者向け）：<a href="http://developer.core-asp.com">http://developer.core-asp.com</a>## 動作条件* iOS4.0以上が動作対象になります。
* Xcodeのプロジェクトのターゲットを選択し、Build Phases の Link Binary With Libraries から SDK/CorePush.framework を追加してください。
* リッチ通知をご利用の場合は、SDK/CorePush.framework と SDK/CorePushSDKResources.bundle を追加してください。リッチ通知のサンプルは、CorePushRichSampleプロジェクトをご参照ください。	##アプリの通知設定###CORE PUSHの設定キーの指定Core Push管理画面 にログインし、ホーム画面からiOSアプリの設定キーを確認してください。 この設定キーをCorePushManager#setConfigKey で指定します。
	[[CorePushManager shared] setConfigKey:@"XXXXXXXXXX"];###CorePushManagerクラスのデリゲートクラスの指定
アプリケーションの動作状態に応じて通知をハンドリングするために、CorePushManagerDelegateプロトコルを実装した
クラスを CorePushManager#setDelegate で指定します。	 [[CorePushManager shared] setDelegate:self];     ##デバイスの通知登録解除デバイスが通知を受信できるようにするには、CORE PUSH にデバイストークンを送信します。またデバイスが通知を受信できないようにするには、CORE PUSH からデバイストークンを削除します。###通知登録
CorePushManager#registerForRemoteNotifications を呼び出すことで APNSサーバからデバイストークンを取得し、
デバイストークンを CORE PUSH に送信します。
	[[CorePushManager shared] registerForRemoteNotifications];
本メソッドはアプリの初回起動時かON/OFFスイッチなどで通知をONにする場合に使用してください。	###通知解除
CorePushManager#unregisterDeviceToken を呼び出すことで CORE PUSH からデバイストークンを削除します。
	[[CorePushManager shared] unregisterDeviceToken];
本メソッドはON/OFFスイッチなどで通知をOFFにする場合に使用してください。		##通知受信後の動作設定
アプリケーションの動作状態に応じて通知をハンドリングすることができます。	###バックグランド状態で動作中に通知から起動した場合
UIApplication#application:didReceiveRemoteNotification: にて、以下のメソッドを呼び出します。
	[[CorePushManager shared] handleRemoteNotification:userInfo]		  アプリケーションがバックグランド状態で動作中の場合は、CorePushManagerDelegate#handleBackgroundNotification が呼び出されます。###フォアグラウンド状態で動作中に通知を受信した場合
UIApplication#application:didReceiveRemoteNotification: にて、以下のメソッドを呼び出します。
	[[CorePushManager shared] handleRemoteNotification:userInfo]
アプリケーションがフォアグラウンド状態で動作中の場合は、CorePushManagerDelegate#handleForegroundNotification が呼び出されます。
###アプリケーションが動作していない状態で通知から起動した場合
UIApplication#application:didFinishLaunchingWithOptions にて、以下のメソッドを呼び出します。

	[[CorePushManager shared] handleLaunchingNotificationWithOption:launchOptions];

アプリケーションが動作していない状態で通知から起動した場合はCorePushManagerDelegate#handleLaunchingNotification が呼び出されます。

##通知履歴の表示


###通知履歴の取得
CorePushNotificationHistoryManager#requestNotificationHistory を呼び出すことで通知履歴を最大100件取得できます。

    [[CorePushNotificationHistoryManager shared] requestNotificationHistory];

取得した通知履歴のオブジェクトの配列は、CorePushNotificationHistoryManager#notificationHistoryModelArray に格納されます。

    [[CorePushNotificationHistoryManager shared] notificationHistoryModelArray];
  
上記の配列により、個々の通知履歴の CorePushNotificationHistoryModel オブジェクトを取得できます。CorePushNotificationHistoryModelオブジェクトには、履歴ID、通知メッセージ、通知日時、リッチ通知URLが格納されます。

	NSString* historyId = notificationHistoryModel.historyId;
	NSString* message = notificationHistoryModel.message;
	NSString* url = notificationHistoryModel.url;
	NSString* regDate = notificationHistoryModel.regDate;
	
###通知履歴の未読管理
CorePushNotificationHistoryManager#setRead を呼び出すことで通知履歴の履歴ID毎に未読を管理することができます。以下は、ある履歴IDの
通知メッセージを既読に設定する例になります。

    //タップされた場合、該当する通知メッセージを既読に設定する。
    [[CorePushNotificationHistoryManager shared] setRead:historyId];

また、CorePushNotificationHistoryManager#getUnreadNumber を呼び出すことで 通知履歴の配列全体の未読数を取得することができます。

	[[CorePushNotificationHistoryManager shared] getUnreadNumber]
	
取得した未読数は、タブのバッジ数やアイコンのバッジ数などに用いることができます。

	 //タブのバッジ数に未読数を設定する場合
	 self.tabBarItem.badgeValue = [NSString stringWithFormat:@"%d", unreadNumber];
	 
	 //タブのバッジ数に未読数を設定する場合
	 [CorePushManager setApplicationIconBadgeNumber:unreadNumber];	
##リッチ通知画面の表示

リッチ通知を受信した場合は、通知オブジェクト内にリッチ通知用のURLが含まれます。
リッチ通知用のURLは、以下の方法で取得できます。

	NSString* url = (NSString*) [userInfo objectForKey:@"url"];
	
上記のURLをリッチ通知画面で表示するには、SDKのCorePushPopupViewオブジェクトを使用します。
以下は CorePushRichSampleプロジェクトからの抜粋になります。

    UIWindow* window = (UIWindow*)[[UIApplication sharedApplication] delegate].window;
    CorePushPopupView* popupView=[[CorePushPopupView alloc] initWithFrame:CGRectMake(0, 100, 300, 400)
                                                           withParentView:window];
    NSString* appName = [[[NSBundle mainBundle] infoDictionary] objectForKey:@"CFBundleDisplayName"];
    popupView.titleBarText = [NSString stringWithFormat:@"%@からのお知らせ", appName];
    popupView.center = window.center;
    [popupView buildLayoutSubViews];
    UIWebView* webView =[[UIWebView alloc] initWithFrame:CGRectMake(0,0,
									popupView.contentView.frame.size.width,
									popupView.contentView.frame.size.height)];
    NSString *urlAddress = self.richUrl;
    NSURL *url = [NSURL URLWithString:urlAddress];
    NSURLRequest *requestObj = [NSURLRequest requestWithURL:url];
    [webView loadRequest:requestObj];
    webView.scalesPageToFit=YES;
    [popupView.contentView addSubview:webView];
    [webView release];
    
    UIButton* safariLaunchButton =[UIButton buttonWithType:UIButtonTypeCustom];
    safariLaunchButton.frame = CGRectMake(0,0,
                                          popupView.contentView.frame.size.width,
                                          popupView.contentView.frame.size.height);
    safariLaunchButton.backgroundColor=[UIColor clearColor];
    [safariLaunchButton addTarget:self action:@selector(safariLaunch:) forControlEvents:UIControlEventTouchUpInside];
    
    [popupView.contentView addSubview:safariLaunchButton];
    
    [popupView show];

	
	