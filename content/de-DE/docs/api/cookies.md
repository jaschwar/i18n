## Class: Cookies

> Abfragen und modifizieren von Session Cookies.

Prozess: [Haupt](../glossary.md#main-process)

Auf Instanzen der `Cookies-Klasse` wird über die Cookie-Eigenschaft einer Sitzung zugegriffen.

Ein Beispiel:

```javascript
const { session } = require('electron')

// Durchsuche all Cookies.
session.defaultSession.cookies.get({})
  .then((cookies) => {
    console.log(cookies)
  }).catch((error) => {
    console.log(error)
  })

// Query all cookies associated with a specific url.
session.defaultSession.cookies.get({ url: 'http://www.github.com' })
  .then((cookies) => {
    console.log(cookies)
  }).catch((error) => {
    console.log(error)
  })

// Set a cookie with the given cookie data;
// may overwrite equivalent cookies if they exist.
const cookie = { url: 'http://www.github.com', name: 'dummy_name', value: 'dummy' }
session.defaultSession.cookies.set(cookie)
  .then(() => {
    // success
  }, (error) => {
    console.error(error)
  })
```

### Instanz Events

The following events are available on instances of `Cookies`:

#### Event: 'changed'

* ` Ereignis </ 0>  Ereignis</li>
<li><code>cookie` [Cookie](structures/cookie.md) - The cookie that was changed.
* `cause` String - The cause of the change with one of the following values: 
  * `explicit` - The cookie was changed directly by a consumer's action.
  * `overwrite` - The cookie was automatically removed due to an insert operation that overwrote it.
  * `expired` - The cookie was automatically removed as it expired.
  * `evicted` - The cookie was automatically evicted during garbage collection.
  * `expired-overwrite` - The cookie was overwritten with an already-expired expiration date.
* `removed` Boolean - `true` if the cookie was removed, `false` otherwise.

Emitted when a cookie is changed because it was added, edited, removed, or expired.

### Instanz Methoden

The following methods are available on instances of `Cookies`:

#### `cookies.get(filter)`

* `filter` Object 
  * `url` String (optional) - Retrieves cookies which are associated with `url`. Empty implies retrieving cookies of all urls.
  * `name` String (optional) - Filters cookies by name.
  * `domain` String (optional) - Retrieves cookies whose domains match or are subdomains of `domains`.
  * `path` String (optional) - Retrieves cookies whose path matches `path`.
  * `secure` Boolean (optional) - Filters cookies by their Secure property.
  * `session` Boolean (optional) - Filters out session or persistent cookies.

Returns `Promise<Cookie[]>` - A promise which resolves an array of cookie objects.

Sends a request to get all cookies matching `filter`, and resolves a promise with the response.

#### `cookies.get(filter, callback)`

* `filter` Object 
  * `url` String (optional) - Retrieves cookies which are associated with `url`. Empty implies retrieving cookies of all urls.
  * `name` String (optional) - Filters cookies by name.
  * `domain` String (optional) - Retrieves cookies whose domains match or are subdomains of `domains`.
  * `path` String (optional) - Retrieves cookies whose path matches `path`.
  * `secure` Boolean (optional) - Filters cookies by their Secure property.
  * `session` Boolean (optional) - Filters out session or persistent cookies.
* `callback` Funktion 
  * ` Fehler </ 0> Fehler</li>
<li><code>cookies` [Cookie[]](structures/cookie.md) - an array of cookie objects.

Sends a request to get all cookies matching `filter`, `callback` will be called with `callback(error, cookies)` on complete.

**[Deprecated Soon](modernization/promisification.md)**

#### `cookies.set(details)`

* `details` Object 
  * `url` String - Die Url mit der der Cookie verbunden werden soll. Das Promise wird abgelehnt, wenn die Url ungültig ist.
  * `name` String (optional) - The name of the cookie. Empty by default if omitted.
  * `value` String (optional) - The value of the cookie. Empty by default if omitted.
  * `domain` String (optional) - The domain of the cookie; this will be normalized with a preceding dot so that it's also valid for subdomains. Empty by default if omitted.
  * `path` String (optional) - The path of the cookie. Empty by default if omitted.
  * `secure` Boolean (optional) - Whether the cookie should be marked as Secure. Defaults to false.
  * `httpOnly` Boolean (optional) - Whether the cookie should be marked as HTTP only. Defaults to false.
  * `expirationDate` Double (optional) - The expiration date of the cookie as the number of seconds since the UNIX epoch. If omitted then the cookie becomes a session cookie and will not be retained between sessions.

Returns `Promise<void>` - A promise which resolves when the cookie has been set

Sets a cookie with `details`.

#### `cookies.set(details, callback)`

* `details` Object 
  * `url` String - The url to associate the cookie with.
  * `name` String (optional) - The name of the cookie. Empty by default if omitted.
  * `value` String (optional) - The value of the cookie. Empty by default if omitted.
  * `domain` String (optional) - The domain of the cookie. Empty by default if omitted.
  * `path` String (optional) - The path of the cookie. Empty by default if omitted.
  * `secure` Boolean (optional) - Whether the cookie should be marked as Secure. Defaults to false.
  * `httpOnly` Boolean (optional) - Whether the cookie should be marked as HTTP only. Defaults to false.
  * `expirationDate` Double (optional) - The expiration date of the cookie as the number of seconds since the UNIX epoch. If omitted then the cookie becomes a session cookie and will not be retained between sessions.
* `callback` Funktion 
  * ` Fehler </ 0> Fehler</li>
</ul></li>
</ul>

<p>Sets a cookie with <code>details`, `callback` will be called with `callback(error)` on complete.</p> 
    **[Deprecated Soon](modernization/promisification.md)**
    
    #### `cookies.remove(url, name)`
    
    * `url` String - The URL associated with the cookie.
    * `name` String - The name of cookie to remove.
    
    Returns `Promise<void>` - A promise which resolves when the cookie has been removed
    
    Removes the cookies matching `url` and `name`
    
    #### `cookies.remove(url, name, callback)`
    
    * `url` String - The URL associated with the cookie.
    * `name` String - The name of cookie to remove.
    * `callback` Function
    
    Removes the cookies matching `url` and `name`, `callback` will called with `callback()` on complete.
    
    **[Deprecated Soon](modernization/promisification.md)**
    
    #### `cookies.flushStore()`
    
    Returns `Promise<void>` - A promise which resolves when the cookie store has been flushed
    
    Writes any unwritten cookies data to disk.
    
    #### `cookies.flushStore(callback)`
    
    * `callback` Function
    
    Writes any unwritten cookies data to disk.
    
    **[Deprecated Soon](modernization/promisification.md)**