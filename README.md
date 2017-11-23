# libcef-build-instructions
libcef build instruction, based on branch 2623(Support WinXP)

## General
	 	 CEF Version：2623
   	Chromium Version：49.0.2623.95
	        Platform：Window XP SP3+

## Windows Build Requirement
	* Win7+，64-bit, RAM: 8G+, Disk: 40GB+
	* Visual Studio VS2014 update4
	* Windows 10 SDK
	* VPN
	* Python
	* Chromium deploy tool: depot_tools and gclient
	* Cygwin Terminal

## Install depot_tools
	* Download Package
		wget https://storage.googleapis.com/chrome-infra/depot_tools.zip
	* Extract depot_tools.zip to a specified directory(Like：D:\depot_tools)
	* Add 'D:\depot_tools' to computer environment PATH

4. 准备Chromium源码
	4.1 下载源码
		wget https://gsdview.appspot.com/chromium-browser-official/chromium-49.0.2623.95.tar.xz
	4.2 解压源码
		mkdir -p chromium/src
		tar zxvf chromium-49.0.2623.95.tar.gz -C chromium/src

5. 准备CEF源码
	5.1 下载源码
		cd chromium/src
		git clone https://bitbucket.org/chromiumembedded/cef.git
		git checkout -t origin/2623

6. 准备Chromium依赖
	6.1 下载Bison
		cd chromium/src/third_party
		git clone https://chromium.googlesource.com/chromium/deps/bison
	6.2 下载gperf
		cd chromium/src/third_party
		git clone https://chromium.googlesource.com/chromium/deps/gperf
	6.3 下载yasm.exe，放到chromium\src\third_party\yasm\binaries\win目录下
	6.4 下载d3dcompiler_47.dll文件，放到%VS_ROOT%\Redis\d3d\x86目录下
	6.5 添加缺失文件（chromium\src\chrome\test\data\webui\i18n_process_css_test.html）

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

7. 修改third_party\WebKit\Source\web\WebViewImpl.h，实现setUseExternalPopupMenusThisInstance函数
	
	void setUseExternalPopupMenusThisInstance(bool useExternalPopupMenus)
    {
        m_shouldUseExternalPopupMenus = useExternalPopupMenus;
    }

8. 去掉编译选项'/WX'，禁止把警告当做错误处理
	注释掉chromium\src\tools\gyp\pylib\gyp\msvs_emulation.py关于'/WX'的内容:
	"""cl('WarnAsError', map={'true': '/WX'})"""

9. 准备工程配置脚本
	9.1 创建一个批处理文件（create_projects.bat)
	9.2 将以下内容放入批处理文件中

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

10. 配置CEF工程
	./create_projects.bat

11. 编译工程
	cd chromium\src
	ninja -C out\Release
