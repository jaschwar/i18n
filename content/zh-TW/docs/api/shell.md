# shell

> 使用預設應用程式管理檔案及 URL。

處理序: [主處理序](../glossary.md#main-process), [畫面轉譯器](../glossary.md#renderer-process)

The `shell` module provides functions related to desktop integration.

An example of opening a URL in the user's default browser:

```javascript
const { shell } = require('electron')

shell.openExternal('https://github.com')
```

## 方法

The `shell` module has the following methods:

### `shell.showItemInFolder(fullPath)`

* `fullPath` String

Show the given file in a file manager. If possible, select the file.

### `shell.openItem(fullPath)`

* `fullPath` String

Returns `Boolean` - Whether the item was successfully opened.

Open the given file in the desktop's default manner.

### `shell.openExternalSync(url[, options])`

* `url` String - Max 2081 characters on Windows, or the function returns false.
* `options` 物件 (選用) 
  * `activate` Boolean (optional) - `true` to bring the opened application to the foreground. The default is `true`. *macOS*
  * `workingDirectory` String (optional) - The working directory. *Windows*

Returns `Boolean` - Whether an application was available to open the URL.

Open the given external protocol URL in the desktop's default manner. (For example, mailto: URLs in the user's default mail agent).

**已被取代**

### `shell.openExternal(url[, options])`

* `url` String - Max 2081 characters on windows.
* `options` Object (選用) 
  * `activate` Boolean (optional) - `true` to bring the opened application to the foreground. The default is `true`. *macOS*
  * `workingDirectory` String (optional) - The working directory. *Windows*

Returns `Promise<void>`

Open the given external protocol URL in the desktop's default manner. (For example, mailto: URLs in the user's default mail agent).

### `shell.moveItemToTrash(fullPath)`

* `fullPath` String

Returns `Boolean` - Whether the item was successfully moved to the trash.

Move the given file to trash and returns a boolean status for the operation.

### `shell.beep()`

Play the beep sound.

### `shell.writeShortcutLink(shortcutPath[, operation], options)` *Windows*

* `shortcutPath` String
* `operation` String (選用) - 預設值為 `create`, can be one of following: 
  * `create` - Creates a new shortcut, overwriting if necessary.
  * `update` - Updates specified properties only on an existing shortcut.
  * `replace` - Overwrites an existing shortcut, fails if the shortcut doesn't exist.
* `options` [ShortcutDetails](structures/shortcut-details.md)

Returns `Boolean` - Whether the shortcut was created successfully.

Creates or updates a shortcut link at `shortcutPath`.

### `shell.readShortcutLink(shortcutPath)` *Windows*

* `shortcutPath` String

回傳 [`ShortcutDetails`](structures/shortcut-details.md)

Resolves the shortcut link at `shortcutPath`.

An exception will be thrown when any error happens.