# clipboard

> Perform copy and paste operations on the system clipboard.

Process: [Main](../glossary.md#main-process), [Renderer](../glossary.md#renderer-process)

The following example shows how to write a string to the clipboard:

```javascript
const { clipboard } = require('electron')
clipboard.writeText('Example String')
```

On Linux, there is also a `selection` clipboard. To manipulate it you need to pass `selection` to each method:

```javascript
const { clipboard } = require('electron')
clipboard.writeText('Example String', 'selection')
console.log(clipboard.readText('selection'))
```

## Методи

The `clipboard` module has the following methods:

**Note:** Experimental APIs are marked as such and could be removed in future.

### `clipboard.readText([type])`

* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Returns `String` - The content in the clipboard as plain text.

### `clipboard.writeText(text[, type])`

* `text` String
* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Writes the `text` into the clipboard as plain text.

### `clipboard.readHTML([type])`

* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Returns `String` - The content in the clipboard as markup.

### `clipboard.writeHTML(markup[, type])`

* `markup` String
* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Writes `markup` to the clipboard.

### `clipboard.readImage([type])`

* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Returns [`NativeImage`](native-image.md) - The image content in the clipboard.

### `clipboard.writeImage(image[, type])`

* `image` [NativeImage](native-image.md)
* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Writes `image` to the clipboard.

### `clipboard.readRTF([type])`

* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Returns `String` - The content in the clipboard as RTF.

### `clipboard.writeRTF(text[, type])`

* `text` String
* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Writes the `text` into the clipboard in RTF.

### `clipboard.readBookmark()` *macOS* *Windows*

Връща `Object`:

* `title` String
* `url` String

Returns an Object containing `title` and `url` keys representing the bookmark in the clipboard. The `title` and `url` values will be empty strings when the bookmark is unavailable.

### `clipboard.writeBookmark(title, url[, type])` *macOS* *Windows*

* `title` String
* `url` String
* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Writes the `title` and `url` into the clipboard as a bookmark.

**Note:** Most apps on Windows don't support pasting bookmarks into them so you can use `clipboard.write` to write both a bookmark and fallback text to the clipboard.

```js
clipboard.write({
  text: 'https://electronjs.org',
  bookmark: 'Electron Homepage'
})
```

### `clipboard.readFindText()` *macOS*

Returns `String` - The text on the find pasteboard. This method uses synchronous IPC when called from the renderer process. The cached value is reread from the find pasteboard whenever the application is activated.

### `clipboard.writeFindText(text)` *macOS*

* `text` String

Writes the `text` into the find pasteboard as plain text. This method uses synchronous IPC when called from the renderer process.

### `clipboard.clear([type])`

* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Clears the clipboard content.

### `clipboard.availableFormats([type])`

* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Returns `String[]` - An array of supported formats for the clipboard `type`.

### `clipboard.has(format[, type])` *Experimental*

* `format` String
* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Returns `Boolean` - Whether the clipboard supports the specified `format`.

```javascript
const { clipboard } = require('electron')
console.log(clipboard.has('<p>selection</p>'))
```

### `clipboard.read(format)` *Experimental*

* `format` String

Returns `String` - Reads `format` type from the clipboard.

### `clipboard.readBuffer(format)` *Experimental*

* `format` String

Returns `Buffer` - Reads `format` type from the clipboard.

### `clipboard.writeBuffer(format, buffer[, type])` *Experimental*

* `format` String
* `buffer` Buffer
* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

Writes the `buffer` into the clipboard as `format`.

### `clipboard.write(data[, type])`

* `data` Object 
  * `text` String (optional)
  * `html` String (optional)
  * `image` [NativeImage](native-image.md) (optional)
  * `rtf` String (optional)
  * `bookmark` String (optional) - The title of the url at `text`.
* `type` String (optional) - Can be `selection` or `clipboard`. `selection` is only available on Linux.

```javascript
const { clipboard } = require('electron')
clipboard.write({ text: 'test', html: '<b>test</b>' })
```

Writes `data` to the clipboard.