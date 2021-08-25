---
title: iOSとFlutterを活用したアプリから電話へのコール
description: Flutterを使用してiOSアプリケーションを構築し、VonageクライアントSDKを用いてモバイルアプリケーションから電話をかけられるようにします。
author: igor-wojda
published: true
published_at: 2021-08-25T08:35:09.354Z
updated_at: 2021-08-25T08:35:09.397Z
category: tutorial
tags:
  - conversation-api
  - flutter
  - ios
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
*This is translated from an English blog. In case of any inconsistency between the English version and the Japanese version, the English version shall prevail. ）
本文は英語版からの翻訳となります。日本語版において意味または文言に相違があった場合、英語版が優先するものとします。
https://learn.vonage.com/blog/2021/04/01/make-app-to-phone-call-using-ios-and-flutter/*

本日は、[Flutter](https://flutter.dev/)を使用して`iOS`アプリケーションを構築し、[VonageクライアントSDK](https://developer.nexmo.com/client-sdk/overview)を用いて[Vonage Conversation API](https://www.vonage.com/communications-apis/conversation/)により、モバイルアプリケーションから電話をかけられるようにします。アプリケーションは3つの画面（3つのUIステート）で構成されます。

![UI states: logon, make a call, and end call](/content/blog/make-app-to-phone-call-using-ios-and-flutter/ui-states.png)

## 前提条件

`Flutter iOS`アプリケーションのソースコードは、[GitHub](https://github.com/nexmo-community/client-sdk-voice-app-to-phone-flutter)で公開されています。
 
`iOS`デバイス向けに`Flutter`アプリケーションを構築する前に、以下の前提条件を満たす必要があります：

* コールコントロールオブジェクト（[NCCO](https://developer.nexmo.com/voice/voice-api/guides/ncco)）を作成
* Vonage CLI（旧Nexmo CLI）をインストール
* Vonageアプリケーションを設定
* Flutter SDKをインストール
* Flutterプロジェクトを作成


## Vonageアプリケーション

### NCCOを作成

[コールコントロールオブジェクト（NCCO)](https://developer.nexmo.com/voice/voice-api/ncco-reference)は、Voice API callのフローをコントロールするために使用するJSON配列です。
NCCOは公開され、インターネットからアクセスできる必要があります。そのためにこのチュートリアルでは、GitHub Gistを使って構成をホストする便利な方法を紹介します。それでは新しいgistを追加しましょう：

1. https://gist.github.com/（Githubにログイン）
2. ncco.jsonをファイル名にして、新しいgistを作成します
3. 以下のJSONオブジェクトをgistにコピー＆ペーストします：

```json
[
    {
        "action": "talk",
        "text": "Please wait while we connect you."
    },
    {
        "action": "connect",
        "endpoint": [
            {
                "type": "phone",
                "number": "PHONE_NUMBER"
            }
        ]
    }
]
```

4. PHONE_NUMBERをあなたの電話番号に置き換えます（Vonageの番号はE.164形式で、+と-は有効ではありません。電話番号を入力する際には、必ず国コードを指定してください。例：US：14155550100、UK：447700900001) [Vonage numbers format](https://developer.nexmo.com/concepts/guides/glossary#e-164-format)  
5. Create secret gistボタンをクリックします
6. Rawボタンをクリックします
7.  次のステップで使用するので、ブラウザに表示されているURLをメモします


### Vonage CLIをインストール

[Vonage CLI](https://developer.nexmo.com/application/nexmo-cli)は、コマンドラインを使用して多くの操作を実行することができます。アプリケーションの作成、会話の作成、Vonage番号の購入などのタスクを実行したい場合は、Vonage CLIをインストールする必要があります。

Vonage CLIはNode.jsが必要ですので、[まずNode.jsをインストール](https://nodejs.org/en/download/)します。

npmでCLIのベータ版をインストールするには、以下のコマンドを実行します：

```cmd
npm install nexmo-cli@beta -g
```

Vonage API KeyとAPI Secretを使用するためにVonage CLIを設定します。ダッシュボードの[設定ページ](https://dashboard.nexmo.com/settings)から設定できます。

以下のターミナルのコマンドを実行し、API_KEYとAPI_SECRET[をダッシュボード](https://dashboard.nexmo.com/settings)の値にリプレースします：

```cmd
nexmo setup API_KEY API_SECRET
```

### Vonageアプリケーションを設定

1. プロジェクトディレクトリを作成し、次のターミナルのコマンドを実行します：

```cmd
mkdir vonage-tutorial
```

2. プロジェクトディレクトリに移動します：:

```cmd
cd vonage-tutorial
```

3. 下記のコマンドをターミナルにコピー＆ペーストして、Vonageアプリケーションを作成します。GIST-URLを前のステップのgistのURLにリプレースすることで、引数--voice-answer-urlの値を変更します。

```
nexmo app:create "App to Phone Tutorial" --capabilities=voice --keyfile=private.key --voice-event-url=https://example.com/ --voice-answer-url=GIST-URL
```

アプリケーションの作成時に、ターミナルにエコーされるApplication IDをメモしておきます。

> 注：`.nexmo-app`という名前の隠しファイルがプロジェクトディレクトリに作成され、新しく作成された`Vonage Application ID`と秘密鍵が含まれます。また、`private.key`という名前の秘密鍵ファイルが既存フォルダに作成されます。

### ユーザーを作成

各参加者は[User](https://developer.nexmo.com/conversation/concepts/user)オブジェクトで表され、Client SDKによって認証される必要があります。本番アプリケーションでは、通常、ユーザー情報をデータベースに保存します。

次のコマンドを実行して`Alice`というユーザーを作成します：

```cmd
nexmo user:create name="Alice"
```

### JWTを生成

`JWT`はユーザーの認証に使用されます。ターミナルで以下のコマンドを実行し、ユーザー`Alice`のJWTを生成します。以下のコマンドでは、`APPLICATION_ID`をアプリケーションのIDにリプレースしてください。


```
nexmo jwt:generate sub=Alice exp=$(($(date +%s)+86400)) acl='{"paths":{"/*/users/**":{},"/*/conversations/**":{},"/*/sessions/**":{},"/*/devices/**":{},"/*/image/**":{},"/*/media/**":{},"/*/applications/**":{},"/*/push/**":{},"/*/knocking/**":{},"/*/legs/**":{}}}' application_id=APPLICATION_ID
```

上記のコマンドでは、`JWT`の有効期限を最大の1日後に設定しています。

`Alice`用に生成した`JWT`をメモしておきます。

> 注：本番環境では、アプリケーションは、クライアントのリクエストごとに`JWT`を生成するエンドポイントを公開する必要があります。

## Xcodeをインストール

AppStoreを開いて[Xcode](https://developer.apple.com/xcode/)をインストールします。

## Flutterを設定

### Flutter SDKをインストール

`Flutter SDK`をダウンロードしてインストールします。

この手順は、`MacOS`、`Win`、`Linux`で異なりますが、一般的には特定のOS用の`Flutter SDK`をダウンロードし、`Flutter SDK`ファイルを解凍して、sdk╲binフォルダをシステムの`PATH`変数に追加します。プラットフォームごとの詳細な説明は[こちら](https://flutter.dev/docs/get-started/install)をご覧ください。 

幸いなことに、`Flutter`には、`SDK`と全ての必要な「コンポーネント」が存在し、正しく構成されているか確認できるツールが付属しています。次のコマンドを実行してください：

```cmd
flutter doctor
```

`Flutter Doctor`が、`Flutter SDK`がインストールされ、その他のコンポーネントもインストールされていて、正しく構成されているかどうか確認します。

## Flutterプロジェクトを作成

ターミナルを使用して`Flutter`プロジェクトを作成します：

```cmd
flutter create app_to_phone_flutter
```

上記のコマンドで、`Flutter`プロジェクトを含む`app_to_phone_flutter`フォルダを作成しま

> 「`Flutter`プロジェクトには、`iOS`プロジェクトを含む`ios`フォルダ、`Android`プロジェクトを含む`android`フォルダ、そして`web`プロジェクトを含む`web`フォルダがあります。」

`pubspec.yaml`ファイルを開き、`permission_handler`の依存関係を追加します（`sdk: flutter`のすぐ下）：

```yaml
dependencies:
  flutter:
    sdk: flutter
  
  permission_handler: ^6.0.1+1
```

> 「`yaml`ファイルではインデントが重要なので、`permission_handlerがflutter`: アイテムと同じインデントレベルであることを確認してください。」

ここで次のコマンドを実行して（パスは`Flutter`プロジェクトのルート）、上記の依存関係をダウンロードします。

```cmd
flutter pub get
```

上記のコマンドは`ios`サブフォルダに`Podfile`も作成します。`ios╲Podfile`を開き、`platform`の行をアンコメントして、`platform`のバージョンを11にアップデートします：

```
platform :ios, '11.0'
```

同じファイルの終わりに`pod 'NexmoClient'`を追加します：


```
target 'Runner' do
  use_frameworks!
  use_modular_headers!
  pod 'NexmoClient'
```

ターミナルで`app_to_phone_flutter/ios`フォルダを開き、ポッドをインストールします：

```cmd
pod install
```

上記のコマンドは、`Flutter`、パーミッションハンドラー、`Client SDK`など、必要な全ての依存関係をダウンロードします。

`XcodeでRunner.xcworkspace`を開き、アプリを実行して、上記の設定が正しく行われたことを確認します。

## Flutter/iOSの双方向コミュニケーション

現在、`Client SDKはFlutter`のパッケージとしては提供されていないので、`Android`ネイティブのクライアント`SDK`を使用し、`iOS`と`Flutter`の間で[MethodChannel](https://api.flutter.dev/flutter/services/MethodChannel-class.html)を使って通信する必要があります。これにより、FlutterはAndroidのメソッドを呼び出し、iOSはFlutterのメソッドを呼び出します。

`Flutter`のコードはlib/main.dartファイルに格納され、`iOS`のネイティブコードは`ios/Runner/AppDelegate.swift`ファイルに格納されます。

## Flutterアプリケーションを起動

Flutterアプリケーションは、[Dart](https://dart.dev/)というプログラミング言語を使って構築されています。

`lib/main.dart`ファイルを開き、コンテンツを全て以下のコードにリプレースします：

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      home: CallWidget(title: 'app-to-phone-flutter'),
    );
  }
}

class CallWidget extends StatefulWidget {
  CallWidget({Key key, this.title}) : super(key: key);
  final String title;

  @override
  _CallWidgetState createState() => _CallWidgetState();
}

class _CallWidgetState extends State<CallWidget> {
  SdkState _sdkState = SdkState.LOGGED_OUT;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            SizedBox(height: 64),
            _updateView()
          ],
        ),
      ),
    );
  }

  Widget _updateView() {
    if (_sdkState == SdkState.LOGGED_OUT) {
      return ElevatedButton(
          child: Text("LOGIN AS ALICE")
      );
    }
  }

  Future<void> _loginUser() async {
      // Login user
  }

  Future<void> _makeCall() async {
      // Make call
  }

  Future<void> _endCall() async {
      // End call
  }
}

enum SdkState {
  LOGGED_OUT,
  LOGGED_IN,
  WAIT,
  ON_CALL,
  ERROR
}
```

上記のコードには、アプリケーションの状態を管理する役割(ユーザーのロギングとコールの管理)を担うカスタム`CallWidget`が含まれています。`SdkState`の列挙型は`Vonage Client SDK`の可能な状態を表します。この列挙型は、`Dart`を使用した`Flutter`用と`Swift`を使用した`iOS`用で2回定義されます。ウィジェットには、`SdkState`の値に基づいてUIを変更する`_updateView`メソッドが含まれています。

`Xcode`からアプリケーションを実行します：

![Running the application from xcode](/content/blog/make-app-to-phone-call-using-ios-and-flutter/run-xcode.png)

`Login as Alice`ボタンが表示されます：

![Logged out screen showing Login as Alice button](/content/blog/make-app-to-phone-call-using-ios-and-flutter/loggedout.png)

### ログイン画面

The `Login as Alice` button is disabled so now add `onPressed` handler to the `ElevatedButton` to allow logging in:

```dart
Widget _updateView() {
    if (_sdkState == SdkState.LOGGED_OUT) {
      return ElevatedButton(
          onPressed: () { _loginUser(); },
          child: Text("LOGIN AS ALICE")
      );
    }
  }
```

ネイティブコードと通信し、ユーザーをログインさせるために、`_loginUser`メソッドのボディをアップデートします：

```dart
Future<void> _loginUser() async {
    String token = "ALICE_TOKEN";

    try {
      await platformMethodChannel.invokeMethod('loginUser', <String, dynamic>{'token': token});
    } on PlatformException catch (e) {
      print(e);
    }
  }
```

`ALICE_TOKEN`を、先ほど`Vonage CLI`から取得したJWTトークンにリプレースし、会話アクセスのためにユーザー`Alice`を認証します。`Flutter`は`loginUser`メソッドを呼び出し、`token`を引数として渡します。`loginUser`メソッドは`MainActivity`クラスで定義されています（後ほど説明します）。このメソッドを`Flutter`から呼び出すには、`MethodChannel`を定義する必要があります。`_CallWidgetState`クラスの先頭に`platformMethodChannel`フィールドを追加します：


`_CallWidgetState`クラスの先頭に`platformMethodChannel`フィールドを追加します：

```dart
class _CallWidgetState extends State<CallWidget> {
  SdkState _sdkState = SdkState.LOGGED_OUT;
  static const platformMethodChannel = const MethodChannel('com.vonage');
```

`com.vonage`の文字列は、`iOS`のネイティブコード（`AppDelegate`クラス）でも参照される、固有のチャンネル`ID`を表しています。次に、このメソッドコールを`iOS`ネイティブ側で処理する必要があります。

`FlutterMethodChannel`への参照を保持する`ios/Runner/AppDelegate`クラスと`vonageChannel`プロパティを開きます：

```swift
@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  var vonageChannel: FlutterMethodChannel?
    
...
```

To listen for method calls originating from `Flutter` add `addFlutterChannelListener` method inside `AppDelegate` class (same level as above `application` method):

```swift
func addFlutterChannelListener() {
        let controller = window?.rootViewController as! FlutterViewController
        
        vonageChannel = FlutterMethodChannel(name: "com.vonage",
                                             binaryMessenger: controller.binaryMessenger)
        vonageChannel?.setMethodCallHandler({ [weak self]
            (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
            guard let self = self else { return }
            
            switch(call.method) {
            case "loginUser":
                if let arguments = call.arguments as? [String: String],
                   let token = arguments["token"] {
                    self.loginUser(token: token)
                }
                result("")
            default:
                result(FlutterMethodNotImplemented)
            }
        })
    }
```

The above method "translates" the `Flutter` method calls to methods defined in the `AppDelegate` class (the `loginUser` for now).

And missing the `loginUser` methods inside the same class (we will fill the body soon):

```swift
func loginUser(token: String) {

}
```

Now add `addFlutterChannelListener` method call inside the `application` method:

```swift
override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        addFlutterChannelListener()
        
        GeneratedPluginRegistrant.register(with: self)
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
```

The code is in place - after pressing the `Login As Alice` button the Flutter app will call the `_loginUser` method. Through the `Flutter` platform channel, the method will call the `loginUser` method defined in the `AppDelegate` class.

Run the application from `Xcode` to make sure it is compiling.

Before we will be able to log in user we need to initialize the `Vonage SDK Client`.

### Initialize Client

Open `AppDelegate` class and add the `NexmoClient` import at the top of the file:

```swift
import NexmoClient
```

In the same file add `client` property that will hold a reference to `Vonage Client`.

```swift
@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
    var vonageChannel: FlutterMethodChannel?
    let client = NXMClient.shared

...
```

Now add `initClient` method to initialize the client:

```swift
func initClient() {
        client.setDelegate(self)
    }
```

To call the `initClient` method from the existing `application` method, we're going to need to add the `initClient()` line as shown in the example below:

```swift
override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        initClient()
        addFlutterChannelListener()
        
        GeneratedPluginRegistrant.register(with: self)
        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }
```

Before allowing conversation we need to know that the user has correctly logged in. In the `AppDelegate` file add a delegate to listen for `Vonage Client SDK` connection state changes:

```swift
extension AppDelegate: NXMClientDelegate {
    func client(_ client: NXMClient, didChange status: NXMConnectionStatus, reason: NXMConnectionStatusReason) {
        switch status {
        case .connected:
            notifyFlutter(state: .loggedIn)
        case .disconnected:
            notifyFlutter(state: .loggedOut)
        case .connecting:
            notifyFlutter(state: .wait)
        @unknown default:
            notifyFlutter(state: .error)
        }
    }
}
```

Finally, the `notifyFlutter` method needs to be added to the same class:

```swift
    func client(_ client: NXMClient, didReceiveError error: Error) {
        notifyFlutter(state: .error)
    }
}
```

### Login the User

Modify `loginUser` method body to call `login` on the client instance:

```swift
func loginUser(token: String) {
        self.client.login(withAuthToken: token)
    }
```

This method will allow us to log-in the user (`Alice`) using the `Client SDK` to access the conversation.

### Notify Flutter About Client SDK State Change

To notify `Flutter` of any changes to the state in the `Client SDK`, we'll need to add an `enum` to represents the states of the `Client SDK`. We've already added the equivalent `SdkState` enum in the `main.dart` file. Add the following `SdkState` enum, at the bottom of the `MainActivity.kt` file:

```swift
enum SdkState: String {
        case loggedOut = "LOGGED_OUT"
        case loggedIn = "LOGGED_IN"
        case wait = "WAIT"
        case onCall = "ON_CALL"
        case error = "ERROR"
    }
```

To send these states to `Flutter` (from above delegate) we need to add `notifyFlutter` method in the `AppDelegate` class:

```swift
func notifyFlutter(state: SdkState) {
        vonageChannel?.invokeMethod("updateState", arguments: state.rawValue)
    }
```

Notice that we store the state in the enum, but we are sending it as a string.

### Retrieve SDK State Update By Flutter

To retrieve state updates in `Flutter` we have to listen for method channel updates. Open `main.dart` file and add `_CallWidgetState` constructor with custom handler:

```dart
_CallWidgetState() {
    platformMethodChannel.setMethodCallHandler(methodCallHandler);
  }
```

Inside the same class (`_CallWidgetState`) add the handler method:

```dart
Future<dynamic> methodCallHandler(MethodCall methodCall) async {
    switch (methodCall.method) {
      case 'updateState':
        {
          setState(() {
            var arguments = 'SdkState.${methodCall.arguments}';
            _sdkState = SdkState.values.firstWhere((v) {return v.toString() == arguments;}
            );
          });
        }
        break;
      default:
        throw MissingPluginException('notImplemented');
    }
  }
```

These methods receive the "signal" from Android and convert it to an enum. Now update the contents of the `_updateView` method to support `SdkState.WAIT` and `SdkState.LOGGED_IN` states, as shown in the example below:

```dart
Widget _updateView() {
    if (_sdkState == SdkState.LOGGED_OUT) {
      return ElevatedButton(
          onPressed: () { _loginUser(); },
          child: Text("LOGIN AS ALICE")
      );
    }  else if (_sdkState == SdkState.WAIT) {
      return Center(
        child: CircularProgressIndicator(),
      );
    } else if (_sdkState == SdkState.LOGGED_IN) {
      return ElevatedButton(
          onPressed: () { _makeCall(); },
          child: Text("MAKE PHONE CALL")
      );
    }
  }
```

During `SdkState.WAIT` the progress bar will be displayed. After successful login application will show the `MAKE PHONE CALL` button.

Run the app and click the button labeled `LOGIN AS ALICE`. The `MAKE PHONE CALL` button should appear, which is another state of the `Flutter` app based on the `SdkState` enum`). An example of this is shown in the image below:

![Make a phone call UI state](/content/blog/make-app-to-phone-call-using-ios-and-flutter/makeaphonecall.png)

### Make A Call

We now need to add functionality to make a phone call. Open the `main.dart` file and update the body of `_makeCall` method as shown below:

```dart
Future<void> _makeCall() async {
    try {
      await platformMethodChannel
          .invokeMethod('makeCall');
    } on PlatformException catch (e) {
      print(e);
    }
  }
```

The above method will communicate with `iOS` so we have to update code in the `AppDelegate` class as well. Add `makeCall` clauses to the `switch` statement inside `addFlutterChannelListener` method:

```swift
func addFlutterChannelListener() {
        let controller = window?.rootViewController as! FlutterViewController
        
        vonageChannel = FlutterMethodChannel(name: "com.vonage",
                                             binaryMessenger: controller.binaryMessenger)
        vonageChannel?.setMethodCallHandler({ [weak self]
            (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
            guard let self = self else { return }
            
            switch(call.method) {
            case "loginUser":
                if let arguments = call.arguments as? [String: String],
                   let token = arguments["token"] {
                    self.loginUser(token: token)
                }
                result("")
            case "makeCall":
                self.makeCall()
                result("")
            default:
                result(FlutterMethodNotImplemented)
            }
        })
    }
```

Now in the same file add the `onGoingCall` property, which defines if and when a call is ongoing:

```swift
var onGoingCall: NXMCall?
```

> NOTE: Currently the `Client SDK` does not store ongoing call reference, so we have to store it in the `AppDelegate` class. We will use it later to end the call.

Now in the same class add `makeCall` method:

```swift
func makeCall() {
        client.call("IGNORED_NUMBER", callHandler: .server) { [weak self] (error, call) in
            guard let self = self else { return }
            
            if error != nil {
                self.notifyFlutter(state: .error)
                return
            }
            
            self.onGoingCall = call
            self.notifyFlutter(state: .onCall)
        }
    }
```

The above method sets the state of the `Flutter` app to `SdkState.WAIT` and waits for the `Client SDK` response (error or success). Now we need to add support for both states (`SdkState.ON_CALL` and `SdkState.ERROR`) inside `main.dart` file. Update body of the `_updateView` method to show the same as below:

```dart
Widget _updateView() {
    if (_sdkState == SdkState.LOGGED_OUT) {
      return ElevatedButton(
          onPressed: () { _loginUser(); },
          child: Text("LOGIN AS ALICE")
      );
    } else if (_sdkState == SdkState.WAIT) {
      return Center(
        child: CircularProgressIndicator(),
      );
    } else if (_sdkState == SdkState.LOGGED_IN) {
      return ElevatedButton(
          onPressed: () { _makeCall(); },
          child: Text("MAKE PHONE CALL")
      );
    } else if (_sdkState == SdkState.ON_CALL) {
      return ElevatedButton(
          onPressed: () { _endCall(); },
          child: Text("END CALL")
      );
    } else {
      return Center(
        child: Text("ERROR")
      );
    }
  }
```

Each state change will result in UI modification. Before making a call the application needs specific permissions to use the microphone. In the next step, we're going to add the functionality in the project to request these permissions.

### Request Permissions

The application needs to be able to access the microphone, so we have to request access to the microphone (`Permission.microphone` for `Flutter` ). 

Open `ios/Runner/info.plist` file and add `Privacy - Microphone Usage Description` key with `Make a call` value:

![Setting add microphone permission](/content/blog/make-app-to-phone-call-using-ios-and-flutter/microphone-permission.png)

We already added the [permission_handler](https://pub.dev/packages/permission_handler) package to the `Flutter` project. Now at the top of the `main.dart` file, we'll need to import the `permission_handler` package as shown in the example below:

```dart
import 'package:permission_handler/permission_handler.dart';
```

To trigger the request for certain permissions, we'll need to add the `requestPermissions()` method within the `_CallWidgetState` class inside the `main.dart` file. So add this new method inside the class:

```dart
Future<void> requestPermissions() async {
    await [ Permission.microphone ].request();
  }
```

The above method will request permissions using `permission_handler`.

In the same class, modify the body of the `_makeCall` class to request permissions before calling the method via the method channel:

```dart
Future<void> _makeCall() async {
    try {
      await requestPermissions();
 
      ...
  }
```

Run the app and click `MAKE PHONE CALL` to start a call. The permissions dialogue will appear and, after granting the permissions, the call will start.

> Reminder: we defined the phone number earlier in the `NCCO`

The state of the application will be updated to `SdkState.ON_CALL` and the UI will be updated:

![On call UI](/content/blog/make-app-to-phone-call-using-ios-and-flutter/oncall.png)

### End Call

To end the call we need to trigger the method on the native `iOS` application using `platformMethodChannel`. Inside `main.dart` file, update the body of the `_endCall` method:

```dart
Future<void> _endCall() async {
    try {
      await platformMethodChannel.invokeMethod('endCall');
    } on PlatformException catch (e) {}
  }
```

The above method will communicate with `iOS`, so we have to update code in the `AppDelegate` class as well. Add `endCall` clauses to the `switch` statement inside the `addFlutterChannelListener` method:

```swift
func addFlutterChannelListener() {
        let controller = window?.rootViewController as! FlutterViewController
        
        vonageChannel = FlutterMethodChannel(name: "com.vonage",
                                             binaryMessenger: controller.binaryMessenger)
        vonageChannel?.setMethodCallHandler({ [weak self]
            (call: FlutterMethodCall, result: @escaping FlutterResult) -> Void in
            guard let self = self else { return }
            
            switch(call.method) {
            case "loginUser":
                if let arguments = call.arguments as? [String: String],
                   let token = arguments["token"] {
                    self.loginUser(token: token)
                }
                result("")
            case "makeCall":
                self.makeCall()
                result("")
            case "endCall":
                self.endCall()
                result("")
            default:
                result(FlutterMethodNotImplemented)
            }
        })
    }
```

Now in the same class add the `endCall` method:

```swift
func endCall() {
        onGoingCall?.hangup()
        onGoingCall = nil
        notifyFlutter(state: .loggedIn)
    }
```

The above method sets the state of the `Flutter` app to `SdkState.WAIT` and waits for the response from the `Client SDK`, which can be either error or success. Both UI states are already supported in the `Flutter` application (`_updateView` method).

We have handled ending the call by pressing the `END CALL` button in the `Flutter` application UI. However, the call can also end outside of the `Flutter` app, e.g. the call will be rejected or answered, and later ended by the callee (on the real phone). 

To support these cases we have to add the `NexmoCallEventListener` listener to the call instance and listen for call-specific events. 

In the `AppDelegares.swift` file add `NXMCallDelegate`:

```swift
extension AppDelegate: NXMCallDelegate {
    func call(_ call: NXMCall, didUpdate callMember: NXMCallMember, with status: NXMCallMemberStatus) {
        if (status == .completed || status == .cancelled) {
            onGoingCall = nil
            notifyFlutter(state: .loggedIn)
        }
    }
    
    func call(_ call: NXMCall, didUpdate callMember: NXMCallMember, isMuted muted: Bool) {
        
    }
    
    func call(_ call: NXMCall, didReceive error: Error) {
        notifyFlutter(state: .error)
    }
}
```

To register above listener modify `onSuccess` callback inside `makeCall` method: 

```swift
func makeCall() {
        client.call("IGNORED_NUMBER", callHandler: .server) { [weak self] (error, call) in
            guard let self = self else { return }
            
            if error != nil {
                self.notifyFlutter(state: .error)
                return
            }
            
            self.onGoingCall = call
            self.onGoingCall?.setDelegate(self)
            self.notifyFlutter(state: .onCall)
        }
    }
```

Run the app and make a phone call from the mobile application to a physical phone number.

# Summary

We have successfully built the application. By doing so we have learned how to make a phone call from a mobile application to the phone using Vonage `Client SDK`. For the complete project please check [GitHub](https://github.com/nexmo-community/client-sdk-voice-app-to-phone-flutter). This project additionally contains the Android native code (`android` folder) allowing us to run this app on Android as well.

To familiarize yourself with other functionalities please check [other tutorials](https://developer.vonage.com/client-sdk/tutorials) and [Vonage developer center](https://developer.vonage.com/).

# References

* [Vonage developer center](https://developer.vonage.com/)
* [Write the first Flutter app](https://flutter.dev/docs/get-started/codelab)
* [Flutter Plaftorm chanels](https://flutter.dev/docs/development/platform-integration/platform-channels)