# `<webview>`タグ

## 警告

Electron の `webview` タグは [Chromium の `webview`](https://developer.chrome.com/apps/tags/webview) に基づきつつ、劇的に変更されています。 これはレンダリング、ナビゲーション、イベントルーティングを含む `webview` の安定性に影響しています。 私たちは、`webview` タグを使用せずに、`iframe` や Electron の `BrowserView` 、埋め込みコンテンツを完全に避けるアーキテクチャといった代替案を検討することを推奨しています。

## 有効にする

既定では `webview` タグは Electron 5 以降では無効化されています。 タグを有効にするには、`BrowserWindow` を構築するときに `webviewTag` webPreferences オプションを設定します。 詳しい情報については、[BrowserWindow コンストラクタ](browser-window.md) を参照してください。

## 概要

> 分離したフレームとプロセスに外部ウェブコンテンツを表示します。

プロセス: [Renderer](../glossary.md#renderer-process)

`webview`タグを使用して、Electron アプリに 'ゲスト' コンテンツ (ウェブページなど) を埋め込むことができます。ゲストコンテンツは `webview` コンテナに含まれています。 アプリ内の埋め込みページは、ゲストコンテンツのレイアウトとレンダリングの方法を制御します。

`iframe` とは異なり、`webview` はアプリとは別のプロセスで実行されます。 それはウェブページと同じ権限を持っておらず、アプリと組み込みコンテンツの間のすべてのやりとりは非同期になります。 これにより、埋め込みコンテンツからアプリが保護されます。 **注釈:** ホストページから webview 上で呼び出されるほとんどのメソッドは、メインプロセスへの同期呼び出しを必要とします。

## サンプル

アプリにウェブページを埋め込むには、アプリの埋め込みページ (これはゲストコンテンツを表示するアプリページ) へ `webview` タグを追加します。 最もシンプルな形式では、`webview` タグには、ウェブページの `src` と、`webview` コンテナの見た目を制御する CSS スタイルが含まれます。

```html
<webview id="foo" src="https://www.github.com/" style="display:inline-flex; width:640px; height:480px"></webview>
```

ゲストコンテンツを制御したい場合は、`webview` のイベントを傍受し、`webview` のメソッドを使用してそれらのイベントに応答する JavaScript を記述することでできます。 ここでは、2つのイベントリスナーを持つサンプルコードを示します。1つはウェブページのロード開始を、もう1つはウェブページのロード停止を傍受し、ロード時に "ロード中..." というメッセージを表示します。

```html
<script>
  onload = () => {
    const webview = document.querySelector('webview')
    const indicator = document.querySelector('.indicator')

    const loadstart = () => {
      indicator.innerText = 'ロード中...'
    }

    const loadstop = () => {
      indicator.innerText = ''
    }

    webview.addEventListener('did-start-loading', loadstart)
    webview.addEventListener('did-stop-loading', loadstop)
  }
</script>
```

## 内部実装

内部では `webview` は [Out-of-Process iframe (OOPIF) ](https://www.chromium.org/developers/design-documents/oop-iframes) で実装されています。 `webview` タグは本質的には、見えない DOM を用いてその内側に `iframe` 要素をラップしたカスタム要素です。

なので `webview` の動作はクロスドメイン `iframe` ととても似ています。例として、

* `webview`をクリックしたとき、ページフォーカスが埋め込みフレームから `webview` に移動します。
* `webview` にキーボード、マウス、スクロールイベントリスナを追加することはできません。
* 埋め込みフレームと `webview` 間のすべての反応は非同期です。

## CSS スタイルの注意事項

`webview` タグのスタイルでは、`webview` コンテナでの子の `iframe` 要素の高さと幅を完全に埋めるのに内部的に `display:flex;` を使用しているので、古典的かつフレックスボックスなレイアウトを使用するときは注意して下さい。 `display:inline-flex;` をインラインレイアウトに指定しない限り、デフォルトの`display:flex;` CSS プロパティを上書きしないでください。

## タグの属性

`webview` タグには以下の属性があります。

### `src`

```html
<webview src="https://www.github.com/"></webview>
```

見える URL を返します。 この属性に書き込むと、トップレベルのナビゲーションが開始されます。

`src` に独自の値を代入すると、現在のページがリロードされます。

`src` 属性は、`data:text/plain,Hello, world!` などのデータ URL を受け取ることもできます。

### `nodeintegration`

```html
<webview src="http://www.google.com/" nodeintegration></webview>
```

この属性が存在する場合、`webview` のゲストページは Node Integration を持ち、低レベルのシステムリソースにアクセスするのに、`require` や `process` のような Node API が使用できます。 デフォルトでは、ゲストページ内の Node Integration は無効化されています。

### `nodeintegrationinsubframes`

```html
<webview src="http://www.google.com/" nodeintegrationinsubframes></webview>
```

NodeJS サポートを有効にする実験的な機能です。これは `webview` 内の iframe のようなサブフレーム内でサポートされます。 すべてのプリロードは iframe 毎にロードされます。メインフレーム内かそうでないか判断するには `process.isMainFrame` が使用できます。 デフォルトではゲストページ内のこのオプションは無効化されています。

### `enableremotemodule`

```html
<webview src="http://www.google.com/" enableremotemodule="false"></webview>
```

この属性が `false` の場合、`webview` 内のゲストページは [`remote`](remote.md) モジュールにアクセスできません。 remote モジュールはデフォルトで利用可能です。

### `plugins`

```html
<webview src="https://www.github.com/" plugins></webview>
```

この属性が存在する場合、`webview` 内のゲストページはブラウザのプラグインを使用することができます。プラグインはデフォルトでは無効です。

### `preload`

```html
<webview src="https://www.github.com/" preload="./test.js"></webview>
```

ゲストのページで他のスクリプトを実行する前に読み込まれるスクリプトを指定します。 スクリプトの URL のプロトコルは、`file:` または `asar:` のいずれかでなければなりません。これは、ゲストページ内で `require` によってロードされるためです。

ゲストページに Node Integration がない場合、このスクリプトはすべての Node APIにアクセスできますが、Node によって挿入されたグローバルオブジェクトはこのスクリプトの実行が終了した後に削除されます。

**注釈:** このオプションは、`will-attach-webview` イベントに指定された `webPreferences` に `preloadURL` (`preload` ではない) として表示されます。

### `httpreferrer`

```html
<webview src="https://www.github.com/" httpreferrer="http://cheng.guru"></webview>
```

ゲストページの参照先 URL を設定します。

### `useragent`

```html
<webview src="https://www.github.com/" useragent="Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; AS; rv:11.0) like Gecko"></webview>
```

ページがナビゲートされる前にゲストページ用のユーザーエージェントを設定します。 一度ページがロードされた場合は、`setUserAgent` メソッドを使用してユーザーエージェントを変更します。

### `disablewebsecurity`

```html
<webview src="https://www.github.com/" disablewebsecurity></webview>
```

この属性が存在すると、ゲストページでウェブセキュリティが無効になります。ウェブセキュリティはデフォルトで有効になっています。

### `partition`

```html
<webview src="https://github.com" partition="persist:github"></webview>
<webview src="https://electronjs.org" partition="electron"></webview>
```

ページで使用されるセッションを設定します。 `partition` が `persist:` 始まりの場合、ページはアプリの全ページで利用可能な永続的なセッションを同じ `partition` で使用します。 `persist:` プレフィックスがない場合、ページは、インメモリセッションを使用します。 同じ `partition` を割り当てることによって、複数のページが同じセッションを共有できます。 `partition` が設定されていない場合は、アプリのデフォルトのセッションが使用されます。

アクティブなレンダラープロセスのセッションは変更できないため、この値は最初のナビゲーションの前にのみ変更できます。 その後の値の変更は、DOM 例外によって失敗します。

### `allowpopups`

```html
<webview src="https://www.github.com/" allowpopups></webview>
```

この属性が存在すると、ゲストページは新しいウィンドウを開くことが許可されます。 ポップアップはデフォルトで無効になっています。

### `webpreferences`

```html
<webview src="https://github.com" webpreferences="allowRunningInsecureContent, javascript=no"></webview>
```

webview で設定するウェブ環境設定を指定する `,` 区切りの文字列リスト。 サポートされている設定の文字列の完全なリストは、[BrowserWindow](browser-window.md#new-browserwindowoptions) にあります。

この文字列は、`window.open` の features 文字列と同じ形式に従います。 名前自体には `true` のブール値が与えられます。 設定は、`=` とそれに続く値を含めることによって別の値に設定できます。 特殊な値として、`yes` と `1` は `true` として解釈され、`no` と `0` は `false` として解釈されます。

### `enableblinkfeatures`

```html
<webview src="https://www.github.com/" enableblinkfeatures="PreciseMemoryInfo, CSSVariables"></webview>
```

有効にする Blink 機能を指定する `,` 区切りの文字列リスト。 サポートされている機能の文字列の完全なリストは、[RuntimeEnabledFeatures.json5](https://cs.chromium.org/chromium/src/third_party/blink/renderer/platform/runtime_enabled_features.json5?l=70) ファイルにあります。

### `disableblinkfeatures`

```html
<webview src="https://www.github.com/" disableblinkfeatures="PreciseMemoryInfo, CSSVariables"></webview>
```

無効にする Blink 機能を指定する `,` 区切りの文字列リスト。 サポートされている機能の文字列の完全なリストは、[RuntimeEnabledFeatures.json5](https://cs.chromium.org/chromium/src/third_party/blink/renderer/platform/runtime_enabled_features.json5?l=70) ファイルにあります。

## メソッド

`webview` タグには以下のメソッドがあります。

**注釈:** メソッドを使用する前に webview 要素をロードする必要があります。

**サンプル**

```javascript
const webview = document.querySelector('webview')
webview.addEventListener('dom-ready', () => {
  webview.openDevTools()
})
```

### `<webview>.loadURL(url[, options])`

* `url` URL
* `options` Object (任意) 
  * `httpReferrer` (String | [Referrer](structures/referrer.md)) (任意) - HTTPリファラのURL。
  * `userAgent` String (任意) - リクエスト元のユーザーエージェント。
  * `extraHeaders` String (任意) - "\n" で区切られた追加のヘッダー
  * `postData` ([UploadRawData[]](structures/upload-raw-data.md) | [UploadFile[]](structures/upload-file.md) | [UploadBlob[]](structures/upload-blob.md)) (任意)
  * `baseURLForDataURL` String (任意) - データURLによってロードされたファイルの (最後のパス区切り文字を含む) ベースURL。 これは指定された `url` がデータURLで、他のファイルをロードする必要がある場合のみ必要です。

戻り値 `Promise<void>` - この promise は、ページ読み込みが完了した時 ([`did-finish-load`](webview-tag.md#event-did-finish-load) を参照) に解決され、ページの読み込みに失敗した時 ([`did-fail-load`](webview-tag.md#event-did-fail-load) を参照) に拒否されます。

`url` を webview にロードします。`url` には、`http://` または `file://` のような、プロトコルのプレフィックスを含みます。

### `<webview>.downloadURL(url)`

* `url` String

ナビゲーションなしで `url` のリソースのダウンロードを初期化します。

### `<webview>.getURL()`

戻り値 `String` - ゲストページの URL。

### `<webview>.getTitle()`

戻り値 `String` - ゲストページのタイトル。

### `<webview>.isLoading()`

戻り値 `Boolean` - ゲストページがまだリソースを読み込んでいるかどうか。

### `<webview>.isLoadingMainFrame()`

戻り値 `Boolean` - メインフレーム (iframe やフレーム内のフレームだけではない) がまだ読み込んでいるかどうか。

### `<webview>.isWaitingForResponse()`

戻り値 `Boolean` - ゲストページが、ページのメインリソースからの最初の応答を待機しているかどうか。

### `<webview>.stop()`

保留中のナビゲーションを停止します。

### `<webview>.reload()`

ゲストページを再読み込みします。

### `<webview>.reloadIgnoringCache()`

ゲストページを、キャッシュを無視して再読み込みします。

### `<webview>.canGoBack()`

戻り値 `Boolean` - ゲストページが前に戻れるかどうか。

### `<webview>.canGoForward()`

戻り値 `Boolean` - ゲストページが次に進めるかどうか。

### `<webview>.canGoToOffset(offset)`

* `offset` Integer

戻り値 `Boolean` - `offset` 番目のゲストページへ行けるかどうか。

### `<webview>.clearHistory()`

ナビゲーション履歴を消去します。

### `<webview>.goBack()`

ゲストページを前に戻します。

### `<webview>.goForward()`

ゲストページを次に進めます。

### `<webview>.goToIndex(index)`

* `index` Integer

指定した絶対インデックスへナビゲーションします。

### `<webview>.goToOffset(offset)`

* `offset` Integer

現在のエントリから指定したオフセットへナビゲーションします。

### `<webview>.isCrashed()`

戻り値 `Boolean` - レンダラープロセスがクラッシュしたかどうか。

### `<webview>.setUserAgent(userAgent)`

* `userAgent` String

ゲストページページのユーザエージェントをオーバーライドします。

### `<webview>.getUserAgent()`

戻り値 `String` - ゲストページのユーザエージェント。

### `<webview>.insertCSS(css)`

* `css` String

ゲストページへ CSS を注入します。

### `<webview>.executeJavaScript(code[, userGesture, callback])`

* `code` String
* `userGesture` Boolean (任意) - 省略値は `false`。
* `callback` Function (任意) - スクリプトが実行されたあとに呼ばれる。 
  * `result` Any

戻り値 `Promise<any>` - 実行されたコードの結果で resolve する Promise。コードの結果が reject な Promise である場合は reject な Promise。

ページ内の `code` を評価します。 `userGesture` が設定されている場合、ページのユーザジェスチャコンテキストが作成されます。 `requestFullScreen` のようなユーザの操作を必要とする HTML API は、このオプションを自動化に利用できます。

**[非推奨予定](modernization/promisification.md)**

### `<webview>.executeJavaScript(code[, userGesture])`

* `code` String
* `userGesture` Boolean (任意) - 省略値は `false`。

戻り値 `Promise<any>` - 実行されたコードの結果で resolve する Promise。コードの結果が reject な Promise である場合は reject な Promise。

ページ内の `code` を評価します。 `userGesture` が設定されている場合、ページのユーザジェスチャコンテキストが作成されます。 `requestFullScreen` のようなユーザの操作を必要とする HTML API は、このオプションを自動化に利用できます。

### `<webview>.openDevTools()`

ゲストページの開発者向けツールウインドウを開きます。

### `<webview>.closeDevTools()`

ゲストページの開発者向けツールウインドウを閉じます。

### `<webview>.isDevToolsOpened()`

戻り値 `Boolean` - ゲストページに開発者向けツールウインドウが適用されているかどうか。

### `<webview>.isDevToolsFocused()`

戻り値 `Boolean` - ゲストページの開発者向けツールウインドウがフォーカスされているかどうか。

### `<webview>.inspectElement(x, y)`

* `x` Integer
* `y` Integer

ゲストページの (`x`, `y`) の位置の要素の検査を開始します。

### `<webview>.inspectSharedWorker()`

ゲストページに表示されている共有ワーカーコンテキストの開発者向けツールを開きます。

### `<webview>.inspectServiceWorker()`

ゲストページに表示されているサービスワーカコンテキストの開発者向けツールを開きます。

### `<webview>.setAudioMuted(muted)`

* `muted` Boolean

ゲストページをミュートに設定します。

### `<webview>.isAudioMuted()`

戻り値 `Boolean` - ゲストページがミュートされているかどうか。

### `<webview>.isCurrentlyAudible()`

戻り値 `Boolean` - 音声が現在再生中かどうか。

### `<webview>.undo()`

ページの `undo` 編集コマンドを実行します。

### `<webview>.redo()`

ページの `redo` 編集コマンドを実行します。

### `<webview>.cut()`

ページの `cut` 編集コマンドを実行します。

### `<webview>.copy()`

ページの `copy` 編集コマンドを実行します。

### `<webview>.paste()`

ページの `paste` 編集コマンドを実行します。

### `<webview>.pasteAndMatchStyle()`

ページの `pasteAndMatchStyle` 編集コマンドを実行します。

### `<webview>.delete()`

ページの `delete` 編集コマンドを実行します。

### `<webview>.selectAll()`

ページの `selectAll` 編集コマンドを実行します。

### `<webview>.unselect()`

ページの `unselect` 編集コマンドを実行します。

### `<webview>.replace(text)`

* `text` String

ページの `replace` 編集コマンドを実行します。

### `<webview>.replaceMisspelling(text)`

* `text` String

ページの `replaceMisspelling` 編集コマンドを実行します。

### `<webview>.insertText(text)`

* `text` String

フォーカスされた要素に `text` を挿入します。

### `<webview>.findInPage(text[, options])`

* `text` String - 検索するコンテンツ。空にしてはいけません。
* `options` Object (任意) 
  * `forward` Boolean (任意) - 前方または後方を検索するかどうか。省略値は `true`。
  * `findNext` Boolean (任意) - 操作が最初のリクエストなのか、辿っているのかどうか。省略値は `false`。
  * `matchCase` Boolean (任意) - 大文字と小文字を区別する検索かどうか。省略値は `false`。
  * `wordStart` Boolean (任意) - 単語の始めだけを見るかどうか。省略値は `false`。
  * `medialCapitalAsWordStart` Boolean (任意) - `wordStart` と組み合わせたとき、マッチの途中が大文字で始まり、小文字や記号が続く場合に、それを受け入れるかどうか。 他のいくつかの単語内一致を受け入れる。省略値は `false`。

戻り値 `Integer` - リクエストに使われたリクエスト ID。

ウェブページ内の `text` のすべてのマッチを探すリクエストを開始します。 リクエストの結果は [`found-in-page`](webview-tag.md#event-found-in-page) イベントを読むことで取得できます。

### `<webview>.stopFindInPage(action)`

* `action` String - [`<webview>.findInPage`](#webviewfindinpagetext-options) リクエストの終了時に実行されるアクションを指定します。 
  * `clearSelection` - 選択を消去する。
  * `keepSelection` - その選択を通常の選択に変換する。
  * `activateSelection` - 選択ノードをフォーカスして、クリックする。

指定された `action` で、`webview` の `findInPage` リクエストを停止します。

### `<webview>.print([options])`

* `options` Object (任意) 
  * `silent` Boolean (任意) - プリンタの設定をユーザに尋ねないかどうか。省略値は `false`。
  * `printBackground` Boolean (任意) - ウェブページの背景色と画像も印刷するかどうか。省略値は `false`。
  * `deviceName` String (任意) - 使用するプリンタデバイスの名前。省略値は `''`。

`webview` のウェブページを印刷します。`webContents.print([options])` と同じです。

### `<webview>.printToPDF(options, callback)`

* `options` Object 
  * `marginsType` Integer (任意) - 使用するマージンの種類を指定する。デフォルトマージンには 0 を、マージン無しには 1 を、最小マージンには 2 を使用する。
  * `pageSize` String | Size (任意) - 生成する PDF のページサイズを指定します。 `A3`、`A4`、`A5`、`Legal`、`Letter`、`Tabloid`、またはミクロン単位の `width` と `height` を含む Object にできる。
  * `printBackground` Boolean (任意) - CSS 背景を印刷するかどうか。
  * `printSelectionOnly` Boolean (任意) - 選択部分だけを印刷するかどうか。
  * `landscape` Boolean (任意) - `true` で横向き、`false` で縦向き。
* `callback` Function 
  * `error` Error
  * `data` Buffer

`webview` のウェブページを PDF として印刷します。`webContents.printToPDF(options, callback)` と同じです。

**[非推奨予定](modernization/promisification.md)**

### `<webview>.printToPDF(options)`

* `options` Object 
  * `marginsType` Integer (任意) - 使用するマージンの種類を指定する。デフォルトマージンには 0 を、マージン無しには 1 を、最小マージンには 2 を使用する。
  * `pageSize` String | Size (任意) - 生成する PDF のページサイズを指定します。 `A3`、`A4`、`A5`、`Legal`、`Letter`、`Tabloid`、またはミクロン単位の `width` と `height` を含む Object にできる。
  * `printBackground` Boolean (任意) - CSS 背景を印刷するかどうか。
  * `printSelectionOnly` Boolean (任意) - 選択部分だけを印刷するかどうか。
  * `landscape` Boolean (任意) - `true` で横向き、`false` で縦向き。

戻り値 `Promise<Buffer>` - 生成された PDF データで実行されます。

`webview` のウェブページを PDF として印刷します。`webContents.printToPDF(options)` と同じです。

### `<webview>.capturePage([rect, ]callback)`

* `rect` [Rectangle](structures/rectangle.md) (任意) - キャプチャする範囲
* `callback` Function 
  * `image` [NativeImage](native-image.md)

`rect` 内のページのスナップショットをキャプチャします。 完了時に、`callback` が `callback(image)` で呼ばれます。 `image` はスナップショットのデータを格納する [NativeImage](native-image.md) のインスタンスです。 `rect` を省略すると、表示されているページ全体をキャプチャします。

**[非推奨予定](modernization/promisification.md)**

### `<webview>.capturePage([rect])`

* `rect` [Rectangle](structures/rectangle.md) (任意) - キャプチャするページ内の領域。

戻り値 `Promise<NativeImage>` - [NativeImage](native-image.md) を解決します

`rect` 範囲内のページのスナップショットを撮ります。`rect` を省略すると、表示されているページ全体をキャプチャします。

### `<webview>.send(channel[, arg1][, arg2][, ...])`

* `channel` String
* `...args` any[]

`channel` を介してレンダラープロセスに非同期メッセージを送信します。任意の引数を送ることもできます。 レンダラープロセスは [`ipcRenderer`](ipc-renderer.md) モジュールで `channel` イベントをリッスンしてメッセージを処理できます。

サンプルについては [webContents.send](web-contents.md#contentssendchannel-arg1-arg2-) を参照して下さい。

### `<webview>.sendInputEvent(event)`

* `event` Object

入力 `event` をページに送ります。

`event` オブジェクトの詳細については、[webContents.sendInputEvent](web-contents.md#contentssendinputeventevent) を参照してください。

### `<webview>.setZoomFactor(factor)`

* `factor` Number - 拡大率。

指定の倍率に拡大率を変更します。拡大率は百分率なので、300% = 3.0 です。

### `<webview>.setZoomLevel(level)`

* `level` Number - 拡大レベル。

指定レベルに拡大レベルを変更します。 原寸は 0 で、各増減分はそれぞれ 20% ずつの拡大または縮小を表し、デフォルトで元のサイズの 300% から 50% までに制限されています。 この式は `scale := 1.2 ^ level` です。

### `<webview>.getZoomFactor()`

戻り値 `Number` - 現在の拡大率。

### `<webview>.getZoomLevel()`

戻り値 `Number` - 現在の拡大レベル。

### `<webview>.setVisualZoomLevelLimits(minimumLevel, maximumLevel)`

* `minimumLevel` Number
* `maximumLevel` Number

ピンチによる拡大レベルの最大値と最小値を設定します。

### `<webview>.setLayoutZoomLevelLimits(minimumLevel, maximumLevel)`

* `minimumLevel` Number
* `maximumLevel` Number

レイアウトベースな (つまり Visual ではない) 拡大レベルの最大値と最小値を設定します。

### `<webview>.showDefinitionForSelection()` *macOS*

ページ上の選択された単語を検索するポップアップ辞書を表示します。

### `<webview>.getWebContents()`

戻り値 [`WebContents`](web-contents.md) - この `webview` に関連付けられた webContents。

これは [`remote`](remote.md) モジュールに依存しています。したがって、このモジュールが無効になっていると利用できません。

### `<webview>.getWebContentsId()`

戻り値 `Number` - この `webview` の WebContents ID。

## DOM イベント

`webview` タグでは、以下の DOM イベントを使用できます。

### イベント: 'load-commit'

戻り値:

* `url` String
* `isMainFrame` Boolean

ロードがコミットされたときに発生します。これには、現在のドキュメント内のナビゲーションとサブフレームのドキュメントレベルの読み込みが含まれますが、非同期のリソース読み込みは含まれません。

### イベント: 'did-finish-load'

ナビゲーションが終了した時、すなわち、タブのくるくるが止まったときや、`onload` イベントが送られた後に、発行されます。

### イベント: 'did-fail-load'

戻り値:

* `errorCode` Integer
* `errorDescription` String
* `validatedURL` String
* `isMainFrame` Boolean

このイベントは `did-finish-load` のようですが、ロードが失敗した、キャンセルされた、`window.stop()` が呼び出されたなどで発生します。

### イベント: 'did-frame-finish-load'

戻り値:

* `isMainFrame` Boolean

フレームのナビゲーションが終了したときに発行されます。

### イベント: 'did-start-loading'

タブのくるくるが始まるタイミングに対応しています。

### イベント: 'did-stop-loading'

タブのくるくるが止まるタイミングに対応しています。

### イベント: 'dom-ready'

指定のフレームの document が読み込まれたときに発行されます。

### イベント: 'page-title-updated'

戻り値:

* `title` String
* `explicitSet` Boolean

ナビゲーション中にページタイトルが設定されたときに発生します。 `explicitSet` は、タイトルがファイル URL から合成されている場合に false になります。

### イベント: 'page-favicon-updated'

戻り値:

* `favicons` String[] - URLの配列。

ページがファビコンの URL を受け取ると発行されます。

### イベント: 'enter-html-full-screen'

HTML API にトリガーされてページがフルスクリーンになるときに発生します。

### イベント: 'leave-html-full-screen'

HTML API にトリガーされてページがフルスクリーンから抜けるときに発生します。

### Event: 'console-message'

戻り値:

* `level` Integer
* `message` String
* `line` Integer
* `sourceId` String

ゲストウィンドウがコンソールメッセージをロギングすると発行されます。

以下のサンプルコードは、ログレベルやその他のプロパティに関係なく、すべてのログメッセージを埋め込みのコンソールに転送します。

```javascript
const webview = document.querySelector('webview')
webview.addEventListener('console-message', (e) => {
  console.log('ゲストページのメッセージログ:', e.message)
})
```

### イベント: 'found-in-page'

戻り値:

* `result` Object 
  * `requestId` Integer
  * `activeMatchOrdinal` Integer - アクティブなマッチの位置。
  * `matches` Integer - マッチの個数。
  * `selectionArea` Object - 最初のマッチ領域の座標。
  * `finalUpdate` Boolean

[`webview.findInPage`](#webviewfindinpagetext-options) リクエストの結果が有効なときに発行されます。

```javascript
const webview = document.querySelector('webview')
webview.addEventListener('found-in-page', (e) => {
  webview.stopFindInPage('keepSelection')
})

const requestId = webview.findInPage('test')
console.log(requestId)
```

### イベント: 'new-window'

戻り値:

* `url` String
* `frameName` String
* `disposition` String - `default`、`foreground-tab`、`background-tab`、`new-window`、`save-to-disk`、`other` にできる。
* `options` Object - 新しい [`BrowserWindow`](browser-window.md) を作成するのに使われるオプション。

ゲストページが新しいブラウザウィンドウを開くときに発生します。

以下のサンプルコードは、システムのデフォルトブラウザで新しい URL を開きます。

```javascript
const { shell } = require('electron')
const webview = document.querySelector('webview')

webview.addEventListener('new-window', async (e) => {
  const protocol = require('url').parse(e.url).protocol
  if (protocol === 'http:' || protocol === 'https:') {
    await shell.openExternal(e.url)
  }
})
```

### イベント: 'will-navigate'

戻り値:

* `url` String

ユーザまたはページがナビゲーションを開始したいときに発行されます。 `window.location` オブジェクトが変更されるか、ユーザがページ内のリンクをクリックしたときに発生します。

このイベントは、 `<webview>.loadURL` や `<webview>.back` のような、API によってプログラム上から開始されるナビゲーションのときには発行されません。

アンカーリンクのクリックや `window.location.hash` の更新のような、ページ内ナビゲーションでも発行されません。これを意図する場合は `did-navigate-in-page` を使用して下さい。

`event.preventDefault()` を呼んでも効果は **ありません**。

### イベント: 'did-navigate'

戻り値:

* `url` String

ナビゲーションが完了したときに発行されます。

このイベントは、アンカーリンクのクリックや `window.location.hash` の更新のような、ページ内ナビゲーションでは発行されません。これを意図する場合は `did-navigate-in-page` を使用して下さい。

### イベント: 'did-navigate-in-page'

戻り値:

* `isMainFrame` Boolean
* `url` String

ページ内ナビゲーションが発生したときに発行されます。

ページ内ナビゲーションが行われるとき、ページのURLは変更されますがページ外でのナビゲーションは発生しません。 これが発生する例は、アンカーリンクがクリックされたときや、DOM の `hashchange` イベントがトリガーされたときです。

### イベント: 'close'

ゲストのページ自身が閉じようとしたときに発生します。

以下のサンプルコードは、ゲストが自身を閉じるときに `webview` を `about:blank` にナビゲートします。

```javascript
const webview = document.querySelector('webview')
webview.addEventListener('close', () => {
  webview.src = 'about:blank'
})
```

### イベント: 'ipc-message'

戻り値:

* `channel` String
* `args` Array

ゲストページが埋め込みページに非同期メッセージを送信したときに発生します。

`sendToHost` メソッドと `ipc-message` イベントを使用すると、ゲストページと埋め込みページの間で通信できます。

```javascript
// 埋め込みページ。
const webview = document.querySelector('webview')
webview.addEventListener('ipc-message', (event) => {
  console.log(event.channel)
  // "pong" と出力される
})
webview.send('ping')
```

```javascript
// ゲストページ。
const { ipcRenderer } = require('electron')
ipcRenderer.on('ping', () => {
  ipcRenderer.sendToHost('pong')
})
```

### イベント: 'crashed'

レンダラープロセスがクラッシュしたときに発生します。

### イベント: 'plugin-crashed'

戻り値:

* `name` String
* `version` String

プラグインプロセスがクラッシュしたときに発行されます。

### イベント: 'destroyed'

webContents が破棄されたときに発生します。

### イベント: 'media-started-playing'

メディアの再生を開始するときに発行されます。

### イベント: 'media-paused'

メディアが一時停止、または再生が終了したときに発行されます。

### イベント: 'did-change-theme-color'

戻り値:

* `themeColor` String

ページのテーマカラーが変更されたときに発行されます。これはよく、このような meta タグによって発生します。

```html
<meta name='theme-color' content='#ff0000'>
```

### イベント: 'update-target-url'

戻り値:

* `url` String

マウスをリンクにマウスオーバーしたり、キーボードでリンクにフォーカスしたときに発行されます。

### イベント: 'devtools-opened'

開発者向けツールが開かれたときに発行されます。

### イベント: 'devtools-closed'

開発者向けツールが閉じられたときに発行されます。

### イベント: 'devtools-focused'

開発者向けツールがフォーカスされた / 開かれたときに発行されます。