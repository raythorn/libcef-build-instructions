# libcef-build-instructions
libcef build instruction, based on branch 2623(Support WinXP)

## General
```bash
     CEF Version：2623
Chromium Version：49.0.2623.95
        Platform：Window XP SP3+
```

## Windows Build Requirement
* Win7+，64-bit, RAM: 8G+, Disk: 40GB+
* Visual Studio VS2014 update4
* Windows 10 SDK
* VPN
* Python
* Chromium deploy tool: depot_tools and gclient
* Cygwin Terminal

## depot_tools
```bash
wget https://storage.googleapis.com/chrome-infra/depot_tools.zip
unzip depot_tools.zip -d D:\
```
Add 'D:\depot_tools' to computer environment PATH

## Chromuim
```bash
wget https://gsdview.appspot.com/chromium-browser-official/chromium-49.0.2623.110.tar.xz
mkdir -p chromium/src
tar Jxvf chromium-49.0.2623.110.tar.xz -C chromium/src
```

## CEF
```bash
cd chromium/src
git clone https://bitbucket.org/chromiumembedded/cef.git
git checkout -t origin/2623
```

## Dependency
#### bison
```bash
cd chromium/src/third_party
git clone https://chromium.googlesource.com/chromium/deps/bison
```

#### gperf
```bash
cd chromium/src/third_party
git clone https://chromium.googlesource.com/chromium/deps/gperf
```

#### yasm
  Download yasm.exe, and put into chromium\src\third_party\yasm\binaries\win

#### d3dcompiler_47.dll
  Download d3dcompiler_47.dll, and put into %VS_ROOT%\Redist\d3d\x86

#### Add missing file(chromium\src\chrome\test\data\webui\i18n_process_css_test.html）
```html
<!doctype html>
<style>
<include src="../../../../ui/webui/resources/css/i18n_process.css">
</style>
<h1 i18n-content="buy"></h1>
<span i18n-values=".innerHTML:link"></span>
<script>
function testI18nProcess_NbspPlaceholder() {
  var h1 = document.querySelector('h1');
  var span = document.querySelector('span');
  assertFalse(document.documentElement.hasAttribute('i18n-processed'));
  assertEquals('', h1.textContent);
  assertEquals('', span.textContent);
  /* We can't check that the non-breaking space hack actually works because it
   * uses :psuedo-elements that are inaccessible to the DOM. Let's just check
   * that they're not auto-collapsed. */
  assertNotEqual(0, h1.offsetHeight);
  assertNotEqual(0, span.offsetHeight);
  h1.removeAttribute('i18n-content');
  assertEquals(0, h1.offsetHeight);
  span.removeAttribute('i18n-values');
  assertEquals(0, span.offsetHeight);
}
</script>
```

## Delete compile flag '/WX' to forbid treat warning as error
  Comment follow content in file chromium\src\tools\gyp\pylib\gyp\msvs_emulation.py
  """cl('WarnAsError', map={'true': '/WX'})"""

## Create gen_projects.bat
```bash
set CEF_VCVARS=none
set GYP_MSVS_OVERRIDE_PATH=%VS2013%
set CEF_USE_GN=0
set DEPOT_TOOLS_WIN_TOOLCHAIN=0
set GYP_DEFINES=buildtype=Official branding=Chromium windows_sdk_path="C:\Program Files (x86)\Microsoft Visual Studio 12.0"
set GYP_GENERATORS=ninja
set GYP_MSVS_VERSION=2013
set PATH=%WINSDK10%\bin\10.0.16299.0\x86;%VS2013%\VC\bin;%PATH%
set LIB=%WINSDK10%\Lib\10.0.16299.0\um\x86;%WINSDK10%\Lib\10.0.16299.0\ucrt\x86;%VS2013%\VC\lib;%VS2013%\VC\atlmfc\lib;%LIB%
set INCLUDE=%WINSDK10%\Include\10.0.16299.0\um;%WINSDK10%\Include\10.0.16299.0\ucrt;%WINSDK10%\Include\10.0.16299.0\shared;%WINSDK10%\Include\10.0.16299.0\winrt;%VS2013%\VC\include;%VS2013%\VC\atlmfc\include;%INCLUDE%
call cef_create_projects.bat
```

## Generate CEF build project
```bash
./create_projects.bat
```

## Update libcef_dll_wrapper.ninja
  Change cflags in 'chromium/src/out/Release/obj/cef/libcef_dll_wrapper.ninja' 'MT' to '/MD'.
  Change cflags in 'chromium/src/out/Debug/obj/cef/libcef_dll_wrapper.ninja' 'MTd' to '/MDd'.

## Compile
```bash
cd chromium\src
ninja -C out\Release
```

## Congratulations
Get your coffee, and enjoy your building!