Cross platform test automation with Appium, WebDriver and Cucumber-JVM
=================================================================================

OpenCredoの記事の通りにAppiumのサンプルを動かしてみた(*´艸｀*)：  
http://www.opencredo.com/2013/09/19/cross-platform-test-automation-with-appium-webdriver-and-cucumber-jvm/

## 事前準備

1. [Appium](http://appium.io/)をダウンロードしてインストールする
2. [Android SDK](http://developer.android.com/sdk/index.html)をダウンロードして解凍しておく
3. [Nodejs](http://nodejs.org/)をダウンロードしてインストールする（AppiumはNodeJsで書かれているため実行環境が必要）
4. Android SDKをダウンロードした後、sdkのディレクトリを環境変数PATHに指定する
    - OSXの例)
        1. Terminalを起動する
        2. `$ cd ~`
        3. `$ touch .bash_profile`
        4. .bash_profileを好みのエディタで開く
        5. `$ export ANDROID_HOME=<path_to_androidSDK_directory>/sdk/`  
	         `$ export PATH=${PATH}:$ANDROID_HOME/tools:$ANDROID_HOME/platform_tools`
    - Windowsの例）
      - 環境変数にANDROID_HOMEとPATHを追加
5. [Maven](http://maven.apache.org/download.cgi)をダウンロードしてインストールする。  
    Mavenが既にインストールされているか確認したい場合は、`$ mvn -version` をターミナルから実行すると良い。  
    もし`unknown command`メッセージが表示された場合は、Mavenをダウンロードしてインストールすること。
6. [ここのサンプルプロジェクト](https://github.com/kuronekomichael/TestingWithAppium)一式をダウンロードする。  
    プロジェクトのAddContactWebsiteフォルダにはHTMLページ、
    ContactManagerSampleフォルダにはネイティブアプリであるContactMangaer.apkをテストするファイルがそれぞれ格納されている。

```
<sample project>
│
├── AddContactWebsite
│   ├── formPage.html
│   └── index.html
│
├── ContactManager.apk <==== テスト対象になるAndroidアプリ
│
└── ContactManagerSample
    ├── ContactManagerSample.iml
    ├── pom.xml                       <=== Mavenビルド設定ファイル
    └── src
        └── test
            ├── java
            │   ├── androidTest                          <== 非Cucumber(JUnit)テストファイル
            │   │   └── TestAndroidWithoutCucumber.java
            │   └── cucumber                             <== Cucumberテストファイル
            │       └── test
            │           ├── AddContactStepDefs.java
            │           ├── RunTest.java
            │           └── config
            │               └── Settings.java
            └── resources                                 <== Cucumber用のテストリソース
                ├── cucumber
                │   └── test
                │       └── Contacts.feature
                └── environment.properties
```


## テストの実行

準備ができたら、実際にテストを実行する。

1. Androidエミュレータを設定するために、Android SDKディレクトリ内のEclipseを起動する
    1. AVD Managerを起動する
        1. Eclipseでは、Window > Android Virtual Device Managerを選択する
        2. Android Virtual Deviceパネルのリストが空の場合は、Newボタンから新たなAVDを作成する
    2. AVDの詳細を入力する
	      1. AVDの名前をつけて、デバイスをリストから選択する。ターゲットになるプラットフォーム(Androidのバージョン)も指定する
	      2. OKをクリックしてAVDを作成する
    3. AVDの準備ができたら、AVD Managerは閉じて、エミュレータをStartする
2. AVD ManagerなしにAVDを起動させたい場合は、terminalから `<ANDROID_SDK_Directory>/sdk/tools` ディレクトリへ移動し、`$ ./emulator -avd <emulator_name>` を実行する
3. `<ANDROID_SDK_Directory>/sdk/platform-tools`から `$./adb devices` を実行することでエミュレータのリストが取得できる。  
もしadb devicesが何も結果を返さなかったら、
```
$ adb kill-server
$ adb start-server
```
    を実行してadbを再起動し、再度`$./adb devices`を試す。
4. Appiumを開いて、IPアドレスに127.0.0.1、ポート番号4723(デフォルト設定)を指定してLaunchボタンをクリックする。  
    `bash: /usr/local/bin/node: No such file or directory`エラーが発生した場合は、node.jsのダウンロードとインストールが必要。  
    これでAppium Serverが起動した状態になる。
5. テストを実行するために、アプリをemulatorにインストールする必要がある。  
    [サンプルプロジェクト](https://github.com/kuronekomichael/TestingWithAppium)のコンパイル済みのapkファイルのパスを
    TestAndroidWithoutCucumber.java と AddContactStepDefs.java に書き加える必要がある。  
    もしAndroidプロジェクトのセットアップ方法についてもっと知りたければ、[Appiumのドキュメント](https://github.com/appium/appium/blob/master/docs/running-tests.md#preparing-your-app-for-test-android)を参照のこと。
### TestAndroidWithoutCucumber.javaの修正例
<サンプルプロジェクト>/ContactManagerSample/src/test/java/androidTest/TestAndroidWithoutCucumber.java：
    ```
  - File app = new File("<path_to_folder_containing_apkFile>", "ContactManager.apk");
  + File app = new File("/Users/kuronekomichael/Build/github/AppiumTest/TestingWithAppium/", "ContactManager.apk");
    ```
### AddContactStepDefs.javaの修正例
<サンプルプロジェクト>/ContactManagerSample/src/test/java/cucumber/test/AddContactStepDefs.java：
```
  - capabilities.setCapability("app", "<path>/ContactManager.apk");
  + capabilities.setCapability("app", "/Users/kuronekomichael/Build/github/AppiumTest/TestingWithAppium/ContactManager.apk");
```
6. これでテストの準備は完了。  
    通常のJUnitのテストのサンプルと、BDDフレームワークのCucumberを使うサンプルが用意してある。
    terminalからpom.xmlのあるディレクトリへ移動して、以下のcommandを入力することでAppiumのテストが実行できる。

### 通常のJUnitのテストとしてAppiumを実行する場合

```
  $ mvn clean install -Dtest=TestAndroidWithoutCucumber
```

### Cucumberを使ってAppiumのテストを実行する場合
Cucumberを使う場合は、AndroidアプリをテストするケースとWebアプリをテストするケースの２通りのサンプルが用意されている。

1. Cucumberを使って、Androidアプリのテストを実行
```
$ mvn clean install -Dtest=RunTest -Ddevice=android
```
2. Cucumberを使って、Webアプリのテストを実行  
サンプルプロジェクトのAddContactWebsiteフォルダ以下に、index.htmlが配置されている。  
terminalからそのパスを入力することで、テストを実行することができる。
```
$ mvn clean install -Dtest=RunTest -Ddevice=firefox -DapplicationUrl=/Users/kuronekomichael/Build/github/AppiumTest/TestingWithAppium/AddContactWebsite/index.html
```

これらの"device"や"applicationUrl"パラメタは、プロジェクトのpom.xmlの中で設定として使われる。  
ターミナルから設定した値は、これらの値を上書きして実行される。  
テストを実行する前に、pom.xmlに正しく指定しておけばcommandでパラメタを指定する必要はない。  
例えば下記の編集例の通りにpom.xml内に正しく"applicationUrlを"指定した場合は、
`$ mvn clean install -Dtest=RunTest -Ddevice=firefox`だけでテストが実行できる。

#### pom.xmlの編集例
```
- <applicationUrl>file:///Users/oanarusu/Documents/TestingWithAppium/AddContactWebsite/index.html</applicationUrl>
+ <applicationUrl>file:///Users/kuronekomichael/Build/github/AppiumTest/TestingWithAppium/AddContactWebsite/index.html</applicationUrl>
```

## FAQ

テスト中に下記のようなエラーが発生してしまった場合：

	A session is either terminated or not started (Original error:Could not find adb; do you have the Android SDK installed and the tools + platform-tools folders added to your PATH?)(WARNING: The server did not provide any stacktrace information)

- sdkフォルダへのパスを通っていない可能性がある。
`~/.bash_profile`にANDROID_HOMEの設定やPATHの指定が正しく追記されているか確認する。
- apkへのパスが正しく指定されていない可能性もある。再度確認する。

That should do it!
