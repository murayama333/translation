# Quick Start

Electronは、ネイティブAPIランタイムを提供することで、JavaScriptによるデスクトップアプリ開発を実現します。Webサーバの代わりにデスクトップ・アプリケーションとしてNode.jsランタイムを使います。

ElectronはJavaScriptのGUIライブラリではありません。ElectronはGUIにWebページを使います。そしてミニマルなChromiumブラウザをJavaScriptでコントロールします。


### メインプロセス

Electronには、package.jsonのメインスクリプトによって起動するプロセスがあります。このプロセスをメインプロセスと呼びます。メインプロセスで実行されるスクリプトは、Webページを作ることによって、ディスプレイを描画します。

### レンダラプロセス

ElectronはWebページの表示にChromiumを使うので、Chromiumのマルチプロセスアーキテクチャの影響を受けます。Electronの中の個々のWebページは独自のプロセスで処理されます。これらのプロセスはレンダラプロセスと呼ばれます。

通常のブラウザはサンドボックスな環境で動作します。つまり、ネイティブリソースへのアクセスは禁止されています。一方ElectronはWebページの中からNode.jsのAPIを使います。これによってOSとの低レベルなやりとりを実現します。


### メインプロセスとレンダラプロセスの違い

メインプロセスはBrowserWindowインスタンスを作成することでWebページを作ります。個々のBrowserWindowインスタンスは、独自のレンダラプロセスでWebページを実行します。BrowserWindowインスタンスが終了すると、関連するレンダラプロセスも終了します。

メインプロセスはすべてのWebページと関連するレンダラプロセスを管理します。個々のレンダラプロセスは独立しているので、そのWebページの処理だけに集中できます。

Webページにおいて、ネイティブGUIに関連するAPI呼び出しは禁止されています。Webページ内でのネイティブGUIリソースの管理、リソースをリークしやすくとても危険だからです。WebページにおいてGUIオペレーションを必要とする場合は、Webページのレンダラプロセスはメインプロセスに問い合わせる必要があります。メインプロセスはそれらのオペレーションに対処するようにします。

Electronは、メインプロセスとレンダラプロセスがやりとりする方法を用意しています。たとえばメッセージをやりとりするipcRenderer、ipcMainモジュールなどがありますし、remoteモジュールはRPCスタイルのコミュニケーションを実現します。これらの詳細についてはhow to share data between web pagesを参照してください。



## はじめてのElectronアプリ

通常、Electronアプリは次のような構造になっています。

```
your-app/
├── package.json
├── main.js
└── index.html
```

package.jsonのフォーマットはNodeモジュールの定義と同じです。アプリの起動スクリプトをmainフィールドで定義します。package.jsonは次のようになります。

```
{
  "name"    : "your-app",
  "version" : "0.1.0",
  "main"    : "main.js"
}
```

*Note:* mainフィールドが省略された場合、Electronはindex.jsを呼び出します。

main.jsはウィンドウの作成やシステムイベントのハンドリングを定義します。典型的なものだと次のようになるでしょう。

```javascript
'use strict';

const electron = require('electron');
const app = electron.app;  // Module to control application life.
const BrowserWindow = electron.BrowserWindow;  // Module to create native browser window.

// Keep a global reference of the window object, if you don't, the window will
// be closed automatically when the JavaScript object is garbage collected.
var mainWindow = null;

// Quit when all windows are closed.
app.on('window-all-closed', function() {
  // On OS X it is common for applications and their menu bar
  // to stay active until the user quits explicitly with Cmd + Q
  if (process.platform != 'darwin') {
    app.quit();
  }
});

// This method will be called when Electron has finished
// initialization and is ready to create browser windows.
app.on('ready', function() {
  // Create the browser window.
  mainWindow = new BrowserWindow({width: 800, height: 600});

  // and load the index.html of the app.
  mainWindow.loadURL('file://' + __dirname + '/index.html');

  // Open the DevTools.
  mainWindow.webContents.openDevTools();

  // Emitted when the window is closed.
  mainWindow.on('closed', function() {
    // Dereference the window object, usually you would store windows
    // in an array if your app supports multi windows, this is the time
    // when you should delete the corresponding element.
    mainWindow = null;
  });
});
```

最後にWebページを表示するためにindex.htmlを作成します。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using node <script>document.write(process.versions.node)</script>,
    Chrome <script>document.write(process.versions.chrome)</script>,
    and Electron <script>document.write(process.versions.electron)</script>.
  </body>
</html>
```

## アプリケーションの実行

main.js、index.html、package.jsonの準備ができたので、ローカルで動かして、期待通りに動作するか確認してみましょう。

### electron-prebuiltの使う場合

npmによってelectron-prebuiltをグローバルにインストールしている場合、アプリのソースディレクトリで次のように実行するだけです。
If you’ve installed electron-prebuilt globally with npm, then you will only need to run the following in your app’s source directory:

```
electron .
```

グローバルではなくローカルにインストールしている場合は、次のようにします。

```
./node_modules/.bin/electron .
```

### Electonバイナリを手動でインストールした場合

手動でElectronをダウンロードした場合、実行バイナリを利用します。

#### Windows

```
$ .\electron\electron.exe your-app\
```

#### Linux

```
$ ./electron/electron your-app/
```

#### OS X

```
$ ./Electron.app/Contents/MacOS/Electron your-app/
```

Electron.appはElectronのリリースパッケージの一部です。[こちら](https://github.com/atom/electron/releases)からダウンロードできます。


### ディストリビューションとして実行する場合

アプリを作成したら、[Application Distribution guide](http://electron.atom.io/docs/v0.37.2/tutorial/application-distribution)に従ってアプリケーションを配布できます。そうすることで、パッケージされたアプリとして実行できるようになります。


### Try this Example

atom/electron-quick-startリポジトリをクローンして実行できます。

*Note:* Exampleの実行にはGitとNode.jsが必要です。

```
# Clone the repository
$ git clone https://github.com/atom/electron-quick-start
# Go into the repository
$ cd electron-quick-start
# Install dependencies and run the app
$ npm install && npm start
 See something that needs fixing? Propose a change on the source.
 Need a different version of the docs? See the available versions.
```
