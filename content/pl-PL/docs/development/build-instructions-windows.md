# Instrukcje Budowania (Windows)

Postępuj zgodnie z wytycznymi poniżej do zbudowania Electrona dla Windowsa.

## Wymagania

* Windows 10 / Server 2012 R2 lub nowszy
* Visual Studio 2017 15.7.2 or higher - [download VS 2019 Community Edition for free](https://www.visualstudio.com/vs/) 
  * See [the Chromium build documentation](https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md#visual-studio) for more details on which Visual Studio components are required.
  * If your Visual Studio is installed in a directory other than the default, you'll need to set a few environment variables to point the toolchains to your installation path. 
    * `vs2019_install = DRIVE:\path\to\Microsoft Visual Studio\2019\Community` (replace `2019` and `Community` with your installed versions)
    * `WINDOWSSDKDIR = DRIVE:\path\to\Windows Kits\10`
* [Python 2.7.10 lub wyższa wersja](http://www.python.org/download/releases/2.7/) 
  * Contrary to the `depot_tools` setup instructions linked below, you will need to use your locally installed Python with at least version 2.7.10 (with support for TLS 1.2). To do so, make sure that in **PATH**, your locally installed Python comes before the `depot_tools` folder. Right now `depot_tools` still comes with Python 2.7.6, which will cause the `gclient` command to fail (see https://crbug.com/868864).
  * [Python for Windows (pywin32) Extensions](https://pypi.org/project/pywin32/#files) is also needed in order to run the build process.
* [Node.js](https://nodejs.org/download/)
* [Git](http://git-scm.com)
* Debugging Tools for Windows of Windows SDK 10.0.15063.468 if you plan on creating a full distribution since `symstore.exe` jest używany do tworzenia magazynu symboli z `.pdb` pliki. 
  * Obok siebie mogą być instalowane różne wersje SDK. To install the SDK, open Visual Studio Installer, select `Change` → `Individual Components`, scroll down and select the appropriate Windows SDK to install. Another option would be to look at the [Windows SDK and emulator archive](https://developer.microsoft.com/en-us/windows/downloads/sdk-archive) and download the standalone version of the SDK respectively.
  * Narzędzia debugowania SDK również muszą być zainstalowane. If the Windows 10 SDK was installed via the Visual Studio installer, then they can be installed by going to: `Control Panel` → `Programs` → `Programs and Features` → Select the "Windows Software Development Kit" → `Change` → `Change` → Check "Debugging Tools For Windows" → `Change`. Możesz też pobrać samodzielny instalator pakietów SDK i użyć go do zainstalowania narzędzi do debugowania (Debugging Tools).

Jeżeli nie posiadasz aktualnie instalacji Windowsa [dev.microsoftedge.com](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/) posiada wersje, które możesz użyć aby zbudować Electron.

Budowanie Electrona odbywa się w pełni przy użyciu skryptów wiersza poleceń i nie może zostać wykonane w Visual Studio. Możesz rozwijać Electron z każdym edytorem, ale wsparcie dla budowy z Visual Studio powstanie w przyszłości.

**Uwaga:** Mimo tego, że Visual Studio nie jest używane do budowania, jest ono **potrzebne** ponieważ potrzebujemy narzędzi budowania, które dostarcza.

## Kompilowanie

See [Instrukcje Budowania (Ogólne)](build-instructions-gn.md)

## Budowanie 32bit

To build for the 32bit target, you need to pass `target_cpu = "x86"` as a GN arg. You can build the 32bit target alongside the 64bit target by using a different output directory for GN, e.g. `out/Release-x86`, with different arguments.

```powershell
$ gn gen out/Release-x86 --args="import(\"//electron/build/args/release.gn\") target_cpu=\"x86\""
```

Inne kroki budowania są dokładnie takie same.

## Projekt Visual Studio

To generate a Visual Studio project, you can pass the `--ide=vs2017` parameter to `gn gen`:

```powershell
$ gn gen out/Debug --ide=vs2017
```

## Rozwiązywanie problemów

### Komenda xxxx nie znaleziona

Jeśli wystąpił bład taki jak `Komenda xxxx nie znaleziona`, możesz spróbować użyć kompilacji skryptów za pomocą `Wiersza polecenia VS2015`.

### Błąd krytyczny wewnętrznego kompilatora: C1001

Upewnij się, że masz zainstalowaną najnowszą aktualizację programu Visual Studio.

### LNK1181: cannot open input file 'kernel32.lib'

Try reinstalling 32bit Node.js.

### Error: ENOENT, stat 'C:\Users\USERNAME\AppData\Roaming\npm'

Creating that directory [should fix the problem](https://stackoverflow.com/a/25095327/102704):

```powershell
$ mkdir ~\AppData\Roaming\npm
```

### node-gyp is not recognized as an internal or external command

Ten błąd może wystąpić, jeśli używasz Git Bash do budynku, zamiast tego należy użyć środowiska PowerShell lub wiersz polecenia VS2015.

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