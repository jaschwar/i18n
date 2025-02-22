# Membangun petunjuk (Windows)

Ikuti panduan di bawah ini untuk membangun Elektron di Linux.

## Prasyarat

* Windows 10 / Server 2012 R2 atau lebih tinggi
* Visual Studio 2017 15.7.2 or higher - [download VS 2019 Community Edition for free](https://www.visualstudio.com/vs/) 
  * See [the Chromium build documentation](https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md#visual-studio) for more details on which Visual Studio components are required.
  * If your Visual Studio is installed in a directory other than the default, you'll need to set a few environment variables to point the toolchains to your installation path. 
    * `vs2019_install = DRIVE:\path\to\Microsoft Visual Studio\2019\Community` (replace `2019` and `Community` with your installed versions)
    * `WINDOWSSDKDIR = DRIVE:\path\to\Windows Kits\10`
* [Python 2.7.10 or higher](http://www.python.org/download/releases/2.7/) 
  * Contrary to the `depot_tools` setup instructions linked below, you will need to use your locally installed Python with at least version 2.7.10 (with support for TLS 1.2). To do so, make sure that in **PATH**, your locally installed Python comes before the `depot_tools` folder. Right now `depot_tools` still comes with Python 2.7.6, which will cause the `gclient` command to fail (see https://crbug.com/868864).
  * [Python for Windows (pywin32) Extensions](https://pypi.org/project/pywin32/#files) is also needed in order to run the build process.
* [Node.js](https://nodejs.org/download/)
* [Git](http://git-scm.com)
* Debugging Tools for Windows of Windows SDK 10.0.15063.468 if you plan on creating a full distribution since `symstore.exe` is used for creating a symbol store from `.pdb` files. 
  * Different versions of the SDK can be installed side by side. To install the SDK, open Visual Studio Installer, select `Change` → `Individual Components`, scroll down and select the appropriate Windows SDK to install. Another option would be to look at the [Windows SDK and emulator archive](https://developer.microsoft.com/en-us/windows/downloads/sdk-archive) and download the standalone version of the SDK respectively.
  * The SDK Debugging Tools must also be installed. If the Windows 10 SDK was installed via the Visual Studio installer, then they can be installed by going to: `Control Panel` → `Programs` → `Programs and Features` → Select the "Windows Software Development Kit" → `Change` → `Change` → Check "Debugging Tools For Windows" → `Change`. Or, you can download the standalone SDK installer and use it to install the Debugging Tools.

Jika saat ini Anda tidak memiliki instalasi Windows, [dev.microsoftedge.com](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/) memiliki versi Windows yang timebombed yang dapat anda gunakan untuk membangun Electron.

Bangunan Electron dilakukan sepenuhnya dengan script baris perintah dan tidak dapat dilakukan dengan Visual Studio. Anda bisa mengembangkan electron dengan editor tapi dukungan untuk bangunan dengan Visual Studio akan datang di masa depan.

**Catatan:** Walaupun Visual Studio tidak digunakan untuk membangun, hal ini masih **diperlukan** karena kita perlu membangun toolchains yang menyediakan.

## Bangunan

See [Build Instructions: GN](build-instructions-gn.md)

## Membangun 32bit

To build for the 32bit target, you need to pass `target_cpu = "x86"` as a GN arg. You can build the 32bit target alongside the 64bit target by using a different output directory for GN, e.g. `out/Release-x86`, with different arguments.

```powershell
$ gn gen out/Release-x86 --args="import(\"//electron/build/args/release.gn\") target_cpu=\"x86\""
```

Langkah lain membangun tentu sama persis.

## Proyek Visual Studio

To generate a Visual Studio project, you can pass the `--ide=vs2017` parameter to `gn gen`:

```powershell
$ gn gen out/Debug --ide=vs2017
```

## Penyelesaian masalah

### Perintah xxxx tidak ditemukan

Jika Anda mengalami kesalahan seperti `Command xxxx tidak ditemukan`, Anda dapat mencoba menggunakannya `VS2015 Command Prompt` konsol untuk menjalankan bangunan skrip.

### Kesalahan fatal kompilator internal: C1001

Pastikan Anda menginstal pembaruan Visual Studio terbaru.

### LNK1181: tidak dapat membuka file masukan 'kernel32.lib'

Try reinstalling 32bit Node.js.

### Error: ENOENT, stat 'C:\Users\USERNAME\AppData\Roaming\npm'

Creating that directory [should fix the problem](https://stackoverflow.com/a/25095327/102704):

```powershell
$ mkdir ~\AppData\Roaming\npm
```

### node-gyp tidak dikenali sebagai perintah internal atau eksternal

Anda mungkin mendapatkan kesalahan ini jika Anda menggunakan Git Bash untuk bangunan, sebaiknya gunakan PowerShell atau VS2015 Command Prompt sebagai gantinya.

### cannot create directory at '...': Filename too long

node.js has some [extremely long pathnames](https://github.com/electron/node/tree/electron/deps/npm/node_modules/libnpx/node_modules/yargs/node_modules/read-pkg-up/node_modules/read-pkg/node_modules/load-json-file/node_modules/parse-json/node_modules/error-ex/node_modules/is-arrayish), and by default git on windows doesn't handle long pathnames correctly (even though windows supports them). This should fix it:

```sh
$ git config --system core.longpaths true
```

### error: use of undeclared identifier 'DefaultDelegateCheckMode'

This can happen during build, when Debugging Tools for Windows has been installed with Windows Driver Kit. Uninstall Windows Driver Kit and install Debugging Tools with steps described above.

### ImportError: No module named win32file

Make sure you have installed `pywin32` with `pip install pywin32`.

### Build Scripts Hang Until Keypress

This bug is a "feature" of Windows' command prompt. It happens when clicking inside the prompt window with `QuickEdit` enabled and is intended to allow selecting and copying output text easily. Since each accidental click will pause the build process, you might want to disable this feature in the command prompt properties.