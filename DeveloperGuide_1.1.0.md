Applihelp SDK for iOS
==================================================
[Applihelp](http://www.applihelp.com) |
[Facebook](http://facebook.com/applihelp) |
[Twitter](http://twitter.com/applihelp)

---

<a name="TOC">Table of Contents</a>
--------------------------------------------------
1. [Introduction](#Introduction)
1. [Requirements](#Requirements)
1. [Installtion](#Installtion)
1. [Usage](#Usage)
1. [Changelogs](#Changelogs)
1. [License](#License)

<a name="Introduction">Introduction</a>
--------------------------------------------------
ApplihelpはあなたのiOSアプリケーションにヘルプサポート機能を提供します。  
Applihelp SDKはお問い合わせメッセージの送信と回答の受信をサポートします。

**[[⬆]](#TOC)**

<a name="Requirements">Requirements</a>
--------------------------------------------------
- iOS5.1.1 or later
- Xcode5.0 or later
- iOS SDK7.0 or later

**[[⬆]](#TOC)**

<a name="Installtion">Installation</a>
--------------------------------------------------
### SDKについて
SDKを利用するためには事前に[Applihelp](http://console.applihelp.com)のアカウントを取得し、以下の情報を登録する必要があります。

- iOSアプリケーションAppID(例：333903271)  
- Apple Push Certificate(.p12)(PUSH通知を利用する場合)  

SDKの構成は下記の通りです。

- `Document` APIリファレンスです。
- `SampleApplication` サンプルアプリケーションです。PUSH通知の受信は行えません。
- `SDK`
	- `libapphelp.a` ApplihelpのStatic Libraryです。
	- `ApplihelpSDK.h` Static Libraryのヘッダファイルです。
	- `APHResources` SDKで利用するリソースファイルです。

### Applihelp Static Library追加
アプリケーションプロジェクトのビルド設定`Link Binary With Libraries`に`libapphelp.a`を追加してください。

### Applihelp ヘッダファイル追加
アプリケーションプロジェクトへヘッダファイル`ApplihelpSDK.h`を追加してください。

### フレームワーク追加
Applihelp SDKが必要とする以下のフレームワークをアプリケーションプロジェクトに追加してください。

- CoreFoundation.framework
- CoreGraphics.framework
- CoreTelephony.framework
- Foundation.framework
- libsqlite3.0.dylib
- SystemConfiguration.framework
- UIKit.framework

### リンカフラグ設定
アプリケーションプロジェクトのビルド設定`Other Linker Flags`に以下2つの設定を追加してください。
Static Libraryに含まれるカテゴリクラスを利用するために必要な設定です。

 - `-all_load` **※**
 - `-ObjC`

**※** アプリケーションで使用しているライブラリによっては "duplicate symbol" エラーが発生する場合があります。その場合は替わりに `-force_load {環境に応じたパス}/libapphelp.a ` を指定してください。

### リソースファイル配置
`APHResources`に含まれる全てのリソースファイルをアプリケーションプロジェクトのビルド設定`Copy Bundle Resources`に追加してください。SDKが使用する画像および言語ファイルが含まれています。
**ファイル名は変更しないでください。**

#### 言語
- `APHResources/languages/en.lproj/APHLocalizable.strings`
- `APHResources/languages/ja.lproj/APHLocalizable.strings`

#### 画像
- `APHResources/images/aph-bubble-min@2x.png`
- `APHResources/images/aph-bubble-min.png`

#### テーマ
- `APHResources/themes/APHTheme.plist`

**[[⬆]](#TOC)**

<a name="Usage">Usage</a>
--------------------------------------------------
### AppID設定
**Applihelpの各機能を利用する前に必ず実行してください。**  
iTunes Connectで発行したアプリケーションのAppIDを指定してください。  
`viewDidLoad`などで実行することを推奨します。

```objc
[ApplihelpSDK setAppId:@"Your-Application-AppID"];
```

### メイン画面表示
Applihelpのメイン画面を表示します。

```objc
[ApplihelpSDK showApphelp];
```

### お問い合わせ画面表示
Applihelpのお問い合わせ画面を表示します。  
ユーザ名が未設定の場合はプロフィール入力画面が表示されます。

```objc
[ApplihelpSDK showRegisterIssue];
```

### ユーザ名取得・設定
キャッシュに保存されているユーザ名を取得します。  

```objc
NSString *userName = [ApplihelpSDK userName];
```

お問い合わせ時のユーザ名をあらかじめ設定することが可能です。  
初回お問い合わせ時にユーザ名の入力を求められます。しかし、事前にユーザ名を設定することによって、それをスキップすることが可能です。

```objc
[ApplihelpSDK setUserName:@"User-Name"];
```

### 新着通知数取得
1度も取得されていない未読メッセージの件数を取得します。

```objc
[ApplihelpSDK notificationCountWithSuccessBlock: ^(int count) {
    // 成功
    UIAlertView *alert = [[UIAlertView alloc] init];
    alert.title = @"完了";
    alert.message = [NSString stringWithFormat:@"新着通知数：%d", count];
    [alert addButtonWithTitle:@"閉じる"];
    [alert show];
} failureBlock: ^{
    // 失敗
    UIAlertView *alert = [[UIAlertView alloc] init];
    alert.title = @"エラー";
    alert.message = @"新着通知数取得失敗";
    [alert addButtonWithTitle:@"閉じる"];
    [alert show];
}];
```

### PUSH通知受信
お問い合わせに対して新着回答がある場合、PUSH通知を受信することが可能です。  

アプリでPUSH通知を有効にするためには Apple Developer での設定が必要です。設定は以下を参考にしてください。  
[Local および Push Notification プログラミングガイド | Apple Developer](https://developer.apple.com/jp/devcenter/ios/library/documentation/RemoteNotificationsPG.pdf)  

#### APNs認証
`AppDelegate`クラスの`didFinishLaunchingWithOptions`メソッドでAPNsにデバイストークンを要求します。   **デバイストークンはAppleの仕様によりシミュレータでは取得できません。**

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	// Remote Notificationを受信するためにAPNSへデバイスを登録
	[[UIApplication sharedApplication] registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge
	                                                                       | UIRemoteNotificationTypeSound
	                                                                       | UIRemoteNotificationTypeAlert)];
	return YES;
}
```

#### デバイストークン登録
`AppDelegate`クラスでデバイストークン登録成功時に実行されるデリゲート`didRegisterForRemoteNotificationsWithDeviceToken`メソッドを実装します。  
取得したデバイストークンは`ApplihelpSDK registerDeviceToken:tokenId`でApplihelpに登録します。  

```objc
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)devToken {
	// デバイストークン
	NSMutableString *tokenId = [[NSMutableString alloc] initWithString:[NSString stringWithFormat:@"%@", devToken]];
	[tokenId setString:[tokenId stringByReplacingOccurrencesOfString:@" " withString:@""]];
	[tokenId setString:[tokenId stringByReplacingOccurrencesOfString:@"<" withString:@""]];
	[tokenId setString:[tokenId stringByReplacingOccurrencesOfString:@">" withString:@""]];
	NSLog(@"DeviceToken: %@", tokenId);
    
	// アプリヘルプのサーバにデバイストークンを登録
	[ApplihelpSDK registerDeviceToken:tokenId];
    
	UIAlertView *view = [[UIAlertView alloc] init];
	view.title = @"デバイストークン";
	view.message = tokenId;
	[view addButtonWithTitle:@"閉じる"];
	[view show];
}
```

#### デバイストークン登録失敗
念のため`AppDelegate`クラスでデバイストークン登録失敗時の処理を実装しておくことをお勧めします。デリゲート`didFailToRegisterForRemoteNotificationsWithError`メソッドを実装します。  

```objc
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)err {
	NSLog(@"Error in registration:%@", err);
	UIAlertView *view = [[UIAlertView alloc] init];
	view.title = @"デバイストークン";
	view.message = @"デバイストークン発行に失敗しました";
	[view addButtonWithTitle:@"閉じる"];
	[view show];
}
```

#### Push通知受信時
`AppDelegate`クラスでPUSH通知受信時に実行されるデリゲート`didReceiveRemoteNotification`メソッドや`didFinishLaunchingWithOptions`メソッドを実装します。

```objc
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
#if !TARGET_IPHONE_SIMULATOR
	NSDictionary *apsInfo = userInfo[@"aps"];
	NSString *alert = apsInfo[@"alert"];
	NSString *sound = apsInfo[@"sound"];
	NSString *badge = apsInfo[@"badge"];
    
	NSLog(@"Remote Notification: %@", [userInfo description]);
	NSLog(@"Remote Notification Alert: %@", alert);
	NSLog(@"Remote Notification Sound: %@", sound);
	NSLog(@"Remote Notification Badge: %@", badge);
    
	if (UIApplicationStateActive == application.applicationState) {
		// アプリケーションがフォアグラウンドで実行中
	}
	else if (UIApplicationStateInactive == application.applicationState) {
		// アプリケーションがバッググラウンドで起動している状態
	}
    
	// アプリヘルプの画面を開く
	[ApplihelpSDK showApphelpWithRemoteNotificationInfo:userInfo viewController:self.window.rootViewController];
    
#endif
}
```

`[ApplihelpSDK showApphelpWithRemoteNotificationInfo]`メソッドを利用することで,新着回答があるお問い合わせのメッセージ画面を開くことが可能です。

#### Localizable.stringsの編集
PUSH通知受信時にiOS端末の通知センターに表示されるアラートメッセージ文字列は、iOSアプリケーションプロジェクトの`Localizable.strings`ファイルから参照します。  
`Localizable.strings`ファイルに以下の2項目を追加してください。  
**ファイルが存在しない、またはファイルに項目が未定義の場合は`aph New message`または`aph Please review this application`という文字列がメッセージとして表示されます。**

```xml
// 通常メッセージのPUSH通知受信時に表示するメッセージ
"aph New message" = "New message.";
// レビューリクエストメッセージのPUSH通知受信時に表示するメッセージ
"aph Please review this application" = "Please review this application.";
```

`Localizable.strings`ファイルが存在しない場合は、以下の手順で新規作成してください。  

*(例：Xcode5.0.2の場合)*  
`File`タブ->`New`を選択->`File...`を選択->`iOS Resource`の`Strings File`を選択し`Next`->ファイル名に`Localizable.strings`を入力し`Create`を選択

#### Push通知受信データ形式
PUSH通知で受信するデータ形式は以下の通りです。  

キー			| 値			| 説明 			
:-------- 	| :-------: | :------------------------------------------
data_type 	| string  	| `Applihelp`固定。ApplihelpからのPushを示すキー
issue_id 	| int  		| 問合せID
type 		| int  		| メッセージタイプ（0:通常、1:レビューリクエスト）

### 文字列
Applihelpが使用する文字列はリソースファイル`APHLocalizable.strings`に定義されています。英語および日本語をサポートしています。

#### 文言の変更
画面に表示される文字列を変更したい場合はリソースファイルを修正してください。  
例えば問合せ完了後に表示されるメッセージを変更したい場合は以下の項目を変更してください。
```xml
"success_alert_message_register_issue" = "送信が完了しました。\nお問い合わせありがとうございます。";
```

#### 多言語対応(i18n)
多言語対応(i18n)については`APHLocalizable.strings`ファイルの`Localization`設定を変更し、ファイルを編集してください。

多言語対応については以下を参考してください。  
[国際化およびローカライズ | Apple Developer](https://developer.apple.com/jp/internationalization/)

### 外観
Applihelpが使用するテーマはリソースファイル`APHResources/themes/APHTheme.plist`に定義されています。  
文字色や背景色など、外観を変更したい場合はファイルを編集してください。

#### ブランド表記の削除
通常Applihelpの全画面にはブランド表記がありますが、任意で削除することが可能です。  
もし削除したい場合は`APHResources/themes/APHTheme.plist`の`Common Attributes`に定義されている`Footer view height`に0を入力してください。  

**[[⬆]](#TOC)**

<a name="Changelogs">Changelogs</a>
--------------------------------------------------
- [Ver.1.1.0]Released on Feb 10, 2013

**[[⬆]](#TOC)**

<a name="License">License and Credits</a>
--------------------------------------------------
このソフトウェアは、以下のオープンソースソフトウェアを使用しています。  

**[AFNetworking](https://github.com/AFNetworking/AFNetworking) © 2013 AFNetworking (http://afnetworking.com/) All rights reserved.**  
[MIT License.](http://opensource.org/licenses/MIT)

**[fmdb](https://github.com/ccgus/fmdb) © 2008 Flying Meat Inc. All rights reserved.**  
[MIT License.](http://opensource.org/licenses/MIT)

**[HPGrowingTextView](https://github.com/HansPinckaers/GrowingTextView) © 2011 Hans Pinckaers All rights reserved.**  
[MIT License.](http://opensource.org/licenses/MIT)

**[JSMessagesViewController](https://github.com/jessesquires/MessagesTableViewController) © 2013 Jesse Squires All rights reserved.**  
[MIT License.](http://opensource.org/licenses/MIT)

**[OHAttributedLabel](https://github.com/AliSoftware/OHAttributedLabel) © 2010 Olivier Halligon All rights reserved.**  
[MIT License.](http://opensource.org/licenses/MIT)

**[Reachability](https://developer.apple.com/library/ios/samplecode/reachability/Introduction/Intro.html) © Apple Inc. All rights reserved.**  

**[SVProgressHUD](https://github.com/samvermette/SVProgressHUD) © 2011 Sam Vermette All rights reserved.**  
Copyright (c) 2011 Sam Vermette

Permission is hereby granted, free of charge, to any person
obtaining a copy of this software and associated documentation
files (the "Software"), to deal in the Software without
restriction, including without limitation the rights to use,
copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the
Software is furnished to do so, subject to the following
conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
OTHER DEALINGS IN THE SOFTWARE.

A different license may apply to other ressources included in this package, 
including Joseph Wain's Glyphish Icons. Please consult their 
respective headers for the terms of their individual licenses.

**[[⬆]](#TOC)**

---
© 2013-2014 [KSK Co., Ltd.](http://www.flexfirm.jp) All rights reserved.
