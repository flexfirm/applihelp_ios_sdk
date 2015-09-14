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
1. [Installation](#Installation)
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
- iOS5.1.1 or later (iOS 9確認済み)
- Xcode7.0 or later
- iOS SDK9.0 or later

**[[⬆]](#TOC)**

<a name="Installation">Installation</a>
--------------------------------------------------
### SDKについて
SDKを利用するためには事前に[Applihelp](http://console.applihelp.com)のアカウントを取得し、以下の情報を登録する必要があります。

- iOSアプリケーションAppID(例：333903271)  
- Apple Push Certificate(.p12)(PUSH通知を利用する場合)  

Installationについて[導入動画（音声付き）](https://www.youtube.com/watch?v=C50B35sf22M&feature=youtu.be)をご用意しておりますのでご利用ください。
[![アプリヘルプ導入動画](https://i.ytimg.com/vi/C50B35sf22M/sddefault.jpg)](https://www.youtube.com/watch?v=C50B35sf22M&feature=youtu.be)

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

- AssetsLibrary.framework
- CoreFoundation.framework
- CoreGraphics.framework
- CoreTelephony.framework
- Foundation.framework
- libsqlite3.0.tbd **※**
- MobileCoreServices.framework
- SystemConfiguration.framework
- UIKit.framework

**※** Xcode 6以前の場合はlibsqlite3.0.dylibを追加してください。

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
- `APHResources/images/aph-bubble-min@3x.png`
- `APHResources/images/aph-bubble-min@2x.png`
- `APHResources/images/aph-bubble-min.png`

#### テーマ
- `APHResources/themes/APHTheme.plist`

### iOS 9における設定
Xcode 7.0以降ではInfo.plistに以下の設定を追加してください。
また、FAQで画像を使う場合はそのドメイン用の設定を加えてください。

```xml
<key>NSAppTransportSecurity</key>
<dict>
	<key>NSExceptionDomains</key>
	<dict>
		<key>applihelp.com</key>
		<dict>
			<key>NSIncludesSubdomains</key>
			<true/>
			<key>NSTemporaryExceptionRequiresForwardSecrecy</key>
			<false/>
			<key>NSExceptionMinimumTLSVersion</key>
			<string>TLSv1.0</string>
		</dict>
	</dict>
</dict>
```

設定を追加するとXcode上の表示は以下のようになります。

![iOS 9設定変更](https://raw.githubusercontent.com/flexfirm/applihelp_ios_sdk/master/img/applihelp_info_plist_ios9.png)

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
**※** "Your-Application-AppID"　は iTunes Connect の Manage Your Apps で発行される9桁の数字です。

**<a name="Sandbox">SANDBOX環境を利用する場合</a>**  
PUSH通知に開発用SSL証明書を利用する場合はSANDBOX環境を利用して下さい。
SANDBOX環境を利用する場合は setAppId の引数 AppID　に　"-sandbox" を付与します。下記の例のようにマクロを定義してデバッグビルド、リリースビルドで切り替えます。

##### 例） "Your-Application-AppID" が "999999999" の場合
```objc
#ifdef DEBUG
// デバッグビルド
[ApplihelpSDK setAppId:@"999999999-sandbox"];
#else
// リリースビルド
[ApplihelpSDK setAppId:@"999999999"];
#endif
}
```

**※** デバッグビルドをインストールした端末に、リリースビルドをインストールするとエラーが発生する可能性があります。そのため、一度アンインストールを行ってから再インストールしてください。

### RootViewControllerの設定
Applihelpの各画面はウィンドウに対するモーダルViewとして表示されます。そのため各画面を呼出す前には必ずウインドウのrootViewControllerプロパティに値を設定してください。


### FAQ画面、もしくは問合せ履歴を表示
FAQが登録されていればFAQ画面を表示します。  
登録されていなければ問合せ履歴を表示します。  

```objc
[ApplihelpSDK showApphelp];
```

### FAQ画面表示
ApplihelpのFAQ画面を表示します。

```objc
[ApplihelpSDK showFaq];
```

### お問い合わせ履歴画面表示
Applihelpのお問い合わせ履歴画面を表示します。  
```objc
[ApplihelpSDK showIssueHistory];
```

### お問い合わせ画面表示
Applihelpのお問い合わせ画面を表示します。  
ユーザ名が未設定の場合はプロフィール入力欄が表示されます。

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

### カスタム情報を設定する  
アプリで取得できる任意の情報（以降、「カスタム情報」）を、ApplihelpのWebコンソールで確認することができます。  
任意のkeyとvalueの組み合わせで複数のカスタム情報を以下のように設定できます。  
この情報は、以下のメソッドを呼び出した後のユーザーの問合せ送信ごと、もしくはメッセージ送信ごとに一緒に送信されます。  
送信した情報は、Messages画面で最後に受信した情報のみ確認できます。

```objc
[ApplihelpSDK addCustomInfoKey:@"user_id" value:@"3000"];
[ApplihelpSDK addCustomInfoKey:@"name" value:@"たけし"];
[ApplihelpSDK addCustomInfoKey:@"appli_point" value:@"4"];
[ApplihelpSDK addCustomInfoKey:@"in_app_purchase" value:@"true"];
```
カスタム情報をクリアする場合は以下のメソッドを呼び出します。  
```objc
[ApplihelpSDK clearCustomInfo];
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

開発用SSL証明書を利用する場合はSANDBOX用の AppID が設定されている必要があります。詳しくは以下を参考にしてください。  
<a href="#Sandbox">SANDBOX環境を利用する場合</a>

#### APNs認証
`AppDelegate`クラスの`didFinishLaunchingWithOptions`メソッドでAPNsにデバイストークンを要求します。   **デバイストークンはAppleの仕様によりシミュレータでは取得できません。**
iOS 8からプッシュ通知のデバイストークンの登録方法が変更になりました。下記のコードは、iOS8 SDK以降とiOS7 SDK以前でのコンパイルに対応しています。

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    BOOL iOS8OrLater = NO;

// SDKのバージョンの判定 iOS 8 SDK以降だったら
#if __IPHONE_OS_VERSION_MAX_ALLOWED >= 80000
    // iOS 8以降だったら
    if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0) {
        iOS8OrLater = YES;

        // Remote Notificationを受信するためにAPNSへデバイスを登録(iOS8から)
        UIUserNotificationSettings *notificationSettings = [UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeSound |
                                                                                                         UIUserNotificationTypeAlert |
                                                                                                         UIUserNotificationTypeBadge)
                                                                                             categories:nil];

        [[UIApplication sharedApplication] registerUserNotificationSettings:notificationSettings];
        [[UIApplication sharedApplication] registerForRemoteNotifications];
    }
#endif

	// iOS8未満だったら
    if (!iOS8OrLater) {
        // Remote Notificationを受信するためにAPNSへデバイスを登録(iOS7まで)
        [[UIApplication sharedApplication] registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge
                                                                               | UIRemoteNotificationTypeSound
                                                                               | UIRemoteNotificationTypeAlert)];
    }

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

*(例：Xcode7の場合)*  
`File`タブ->`New`を選択->`File...`を選択->`Resource`の`Strings File`を選択し`Next`->ファイル名に`Localizable.strings`を入力し`Create`を選択

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

### カスタマイズ
Applihelpが使用するテーマはリソースファイル`APHResources/themes/APHTheme.plist`に定義されています。  
文字色や背景色の変更、画面回転の制御などをしたい場合はファイルを編集してください。

#### 画面回転の制御
画面回転は、`APHResources/themes/APHTheme.plist`と、Xcodeプロジェクトの設定で決定されます。  
**APHTheme.plistの設定を変更する場合は、Rotateから始まる項目のうち、少なくともどれか一つをYESに設定してください。**  
**また、リソースファイルとプロジェクトの設定で矛盾があった場合は、アプリケーションが強制終了してしまう場合があります。**  
なおApplihelpでは、PortraitUpSideDown(縦画面でホームボタンが上)はサポートされていません。  
またiOS 5.1.1では、画像アップロード時の画像選択画面での画面回転の制御はサポートされていません。

- ShouldAutorotate  
アプリヘルプの画面で、回転を許可します。(iOS 6以降)  
初期値 YES  

- Rotate Portrait  
アプリヘルプの画面で、縦画面(ホームボタンが下)を許可します。  
初期値 YES  

- Rotate LandscapeLeft  
アプリヘルプの画面で、横画面(ホームボタンが左)を許可します。  
初期値 YES  

- Rotate LandscapeRight  
アプリヘルプの画面で、横画面(ホームボタンが右)を許可します。  
初期値 YES  

#### ブランド表記の削除
通常Applihelpの全画面にはブランド表記がありますが、任意で削除することが可能です。  
もし削除したい場合は`APHResources/themes/APHTheme.plist`の`Common Attributes`に定義されている`Footer view height`に0を入力してください。  

**[[⬆]](#TOC)**

<a name="Changelogs">Changelogs</a>
--------------------------------------------------
- [Ver.1.4.1]Released on Sep 15, 2015
  - iOS 9に対応しました。また、iOS 9で必要な設定を追記しました。
  - 特定の条件で空のお問い合わせや、メッセージが送れてしまう問題を修正しました。
  - iPadでアプリヘルプ画面が最大表示されてしまうことがある問題を修正しました。


- [Ver.1.4.0]非公開のバージョン


- [Ver.1.3.1]Released on Oct 10, 2014
  - プッシュ通知に必要なデバイストークンが更新されていてもサーバに反映されない問題を修正しました。
  - SANDBOX環境についての説明を追記しました。
  - 画面回転に関する注意書きを追記しました。
  - SDK導入動画を追加しました。


- [Ver.1.3.0]Released on Oct 10, 2014
  - Webコンソールで設定したFAQを表示できる機能を追加しました。


- [Ver.1.2.2]Released on Oct 9, 2014
  - 新着通知数取得時に応答を返さないことがある問題を修正しました。


- [Ver.1.2.1]Released on Oct 7, 2014
  - iOS 8に対応しました。
  - 画面回転の制御に対応しました。


- [Ver.1.2.0]Released on Aug 18, 2014
  - カスタム情報を設定する機能を追加しました。  


- [Ver.1.1.1]Released on April 23, 2014
 - アプリで利用するライブラリによって duplicate symbol エラーが発生する不具合を修正しました。
 - 64bitのシミュレータに対応しました。


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
© 2013-2015 [KSK Co., Ltd.](http://www.flexfirm.jp) All rights reserved.
