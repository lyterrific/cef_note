[TOC]

[Updated 2017-11-16](https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage#markdown-header-browser-life-span)

# 1 介绍（Introduction）

[CEF](https://bitbucket.org/chromiumembedded/)(Chromium Embedded Framework)是一个基于Google Chromium 的开源项目。[Google Chromium](http://www.chromium.org/Home)项目和CEF项目有着不同的目标，Google Chromium主要服务于Google Chrome应用，而CEF则是为在第三方应用嵌入浏览器提供支持。CEF屏蔽了底层Chromium和Blink的复杂代码，向使用者提供了一组产品级稳定的API。CEF跟踪Chromium的各版本分支并发布对应的CEF二进制包。CEF为大部分特性提供了默认实现以提供丰富的功能，让使用者做少量的工作即可满足需求。世界上已经有很多公司和机构使用CEF，CEF的安装量超过了100万。 [CEF Wikipedia page](http://en.wikipedia.org/wiki/Chromium_Embedded_Framework#Applications_using_CEF)页面给出了使用CEF的公司和机构的不完全列表。应用CEF的典型场景包括：

* 在已经存在的本地应用嵌入兼容H5的浏览器控件。
* 创建轻量化的壳浏览器，用以托管主要用Web技术开发的应用。
* 有独立的绘制框架的应用可使用CEF对Web内容做离屏渲染。
* 利用CEF做自动化Web测试。

CEF3是基于 [Chromium Content API](http://www.chromium.org/developers/content-module/content-api)多进程构架的下一代CEF，CEF3的多进程框架有以下优势：

* 更好的性能和稳定性（JavaScript和插件在独立的进程内执行）。
* 支持视网膜显示器。
* 支持WebGL和3D CSS的GPU加速。
* 提供新功能，例如WebRTC（网络摄像头支持）和语音输入。
* 通过DevTools远程调试协议和 [ChromeDriver2](https://code.google.com/p/chromedriver/wiki/ChromeDriver2)提供更好的自动化UI测试。
* 更快获得当前以及未来的Web特性和标准的能力。

# 2 开始（Getting Started）

## 2.1 使用二进制包（Using a Binary Distribution）

CEF3的二进制包可以在 [project Downloads page](http://www.magpcss.net/cef_downloads/)下载。其中包含了在特定平台（Windows，Mac OS X 以及
Linux）编译特定版本CEF3所需的全部文件。这些二进制包有着共同的结构：

* cefclient：包含了cefclient示例应用。这个示例中展现了CEF的大部分特性。
* cefsimple ：包含了cefsimple 示例应用。这个示例中利用CEF最少特性生成一个浏览器窗口。
* Debug：包含Debug版下构建的libcef.lib和其他运行库文件。
* include：包含CEF头文件。
* libcef_dll：包含生成libcef_dll_wrapper静态库的源码。所有使用CEF C++ API的应用都要依赖这个静态库。
* Release：包含Release版下构建的libcef.lib和其他运行库文件。
* Resources：包含使用CEF的应用依赖的资源文件（只在windows和Linux下）。包括.pak 文件 (二进制资源文件)和与平台相关的其他文件。

每个二进制包包含一个README.txt文件和一个LICENSE.txt文件，README.txt提供了平台相关的细节，LICENSE.txt包含CEF的BSD版权说明。如果你发布了基于CEF的应用，则应该在应用程序的某个地方包含该版权声明。例如，你可以在”关于”和“授权”页面列出该版权声明，或者用一个单独文档包含该版权明。“关于”和“授权”信息也可以分别在CEF浏览器的”about:license”和”about:credits”页面查看。

基于CEF二进制包的应用程序可以使用各平台上的标准编译工具编译。包括Windows的Visual Studio，Mac OS X的Xcode，以及Linux的gcc/make编译工具链。CEF项目的下载页面包含了特定平台上编译特定版本CEF所需编译工具的版本信息。在Linux上编译CEF时需要特别注意依赖工具链。

## 2.2 从源码编译（Building from Source Code）

CEF可以从源码编译，用户可以使用本地编译工具或者像TeamCity这样的自动化编译工具编译。这需要使用svn或者git下载Chromium和CEF的源码。由于Chromium源码很大，只建议在内存大于6GB的机器上编译。具体细节参
考 [BranchesAndBuilding](https://bitbucket.org/chromiumembedded/cef/wiki/BranchesAndBuilding.md) 。

# 3 示例应用（Sample Application）

cefclient是一个集成CEF的完整应用示例，它的源码包含在CEF每个二进制包中。创建基于CEF的应用的最简单方法是先从cefclient应用程序开始，删除你不
需要的部分。

# 4 重要概念（Important Concepts）

在开发基于CEF3的应用前，需要了解一些重要的基础概念。

## 4.1 C++封装（C++ Wrapper）

libcef 共享库导出的 C API 使得使用者不用关心CEF运行库和基础代码。libcef_dll_wrapper 工程把这些 C API 封装成 C++ API，其源码包含在CEF二进制发布包中。C++应用应链接libcef_dll_wrapper工程生成的libcef_dll_wrapper.lib。C/C++ API的转换层代码是由转换工具自动生成。[UsingTheCAPI](https://bitbucket.org/chromiumembedded/cef/wiki/UsingTheCAPI.md) 页面描述了如何直接使用C API。

## 4.2 进程（Processes）

CEF3使用多进程模型。“browser”进程为主进程，负责窗口管理，界面绘制和网络交互。“browser”进程通常为宿主应用进程，负责实现应用程序的大部分逻辑。Blink的渲染和Js的执行在独立的“render”进程中运行。应用程序的一些逻辑，如Js Binding和对Dom访问，也在“render”进程中执行。默认的进程模型中，会为每个源（scheme + domain）创建新的“render”进程。其他进程按需创建，例如管理插件的“plugin”进程和处理硬件加速的“gpu” 进程等。

默认情况下，主应用程序会被多次启动以生成各自独立的进程。这是向主应用程序通过向CefExecuteProcess函数传递不同的命令行参数实现的。如果主应用程序很大，加载时间比较长，或者不能在非浏览器进程里使用，则宿主程序可使用独立的可执行文件去运行这些进程。这可以通过配置CefSettings.browser_subprocess_path变量做到。

CEF3的进程可以通过IPC通信。browser和render进程中实现的应用逻辑可以通过发送异步消息进行双向通信。render进程还能暴露异步JavaScript API给browser进程，以在browser进程中实现响应。

 [Windows](https://www.chromium.org/developers/how-tos/debugging-on-windows), [Mac OS X](http://www.chromium.org/developers/how-tos/debugging-on-os-x) 和 [Linux](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_debugging.md)给出了平台相关的调试注意事项。

## 4.3 线程（Threads）

CEF3中的每个进程都是多线程的。 [cef_thread_id_t](http://magpcss.org/ceforum/apidocs3/projects/(default)/cef_thread_id_t.html) 给出了完整的线程类型列表。以下是一些主要线程：

* TID_UI 线程是browser进程的主线程。如果应用程序在调用CefInitialize()时，将CefSettings.multi_threaded_message_loop参数设为false，那么这个线程也是应用程序的主线程。
* TID_IO 线程在browser进程中负责处理IPC消息以及网络通信。
* TID_FILE 线程在browser进程中负责与文件系统交互。耗时操作应放在这个线程或者由应用生成的 [CefThread](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefThread.html)中实现。
* TID_RENDERER线程是render进程的主线程。所有的Blink和V8交互在这个线程中实现。

由于CEF采用多线程架构，有必要使用锁和消息传递来保证多线程访问时的数据安全。CefPostTask函数族支持简易的线程间异步消息传递。可以通过CefCurrentlyOn()方法判断当前所在的线程，CEF示例应用（cefclient工程）使用下面的定义来确保函数在期望的线程中执行。这些定义在 [include/wrapper/cef_helpers.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/wrapper/cef_helpers.h?at=master) 中。

```c++
#define CEF_REQUIRE_UI_THREAD()       DCHECK(CefCurrentlyOn(TID_UI));
#define CEF_REQUIRE_IO_THREAD()       DCHECK(CefCurrentlyOn(TID_IO));
#define CEF_REQUIRE_FILE_THREAD()     DCHECK(CefCurrentlyOn(TID_FILE));
#define CEF_REQUIRE_RENDERER_THREAD() DCHECK(CefCurrentlyOn(TID_RENDERER));
```

为支持对代码块的同步访问，CEF在 [include/base/cef_lock.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/base/cef_lock.h?at=master) 中定义了base::Lock 和 base::AutoLock 类型 。 使用示例如下:

```c++
// Include the necessary header.
#include "include/base/cef_lock.h"

// Class declaration.
class MyClass : public CefBase {
 public:
  MyClass() : value_(0) {}
  // Method that may be called on multiple threads.
  void IncrementValue();
 private:
  // Value that may be accessed on multiple theads.
  int value_;
  // Lock used to protect access to |value_|.
  base::Lock lock_;
  IMPLEMENT_REFCOUNTING(MyClass);
};

// Class implementation.
void MyClass::IncrementValue() {
  // Acquire the lock for the scope of this method.
  base::AutoLock lock_scope(lock_);
  // |value_| can now be modified safely.
  value_++;
}
```

## 4.4 引用计数（Reference Counting）

所有框架类都实现CefBase接口，所有实例指针由CefRefPtr智能指针管理，CefRefPtr通过调用AddRef()和Release()方法自动管理引用计数。框架类的实现方式如下：

```c++
class MyClass : public CefBase {
 public:
  // Various class methods here...
 private:
  // Various class members here...
  IMPLEMENT_REFCOUNTING(MyClass);  // Provides atomic refcounting implementation.
};

// References a MyClass instance
CefRefPtr<MyClass> my_class = new MyClass();
```

## 4.5 字符串（Strings）

CEF出于以下原因定义了自己的字符串数据结构：

- libcef库和宿主应用程序可能使用不同的runtimes 管理堆。堆中的所有对象，包括字符

  串，需要在相同的运行时环境中被释放。

- libcef可以编译为支持不同的字符串类型(UTF8，UTF16或wide)的库。默认为
  UTF16。可以通过更改 [cef_string.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/internal/cef_string.h?at=master)文件中的定义并重新编译来修改支持的字符集。当使用宽字节集的时候，切记字符的长度由当前使用的平台决定。

UTF16字符串结构体示例如下：

```c++
typedef struct _cef_string_utf16_t {
  char16* str;  // Pointer to the string
  size_t length;  // String length
  void (*dtor)(char16* str);  // Destructor for freeing the string on the correct heap
} cef_string_utf16_t;
```

通过typedef来设置支持的字符编码：

```c++
typedef char16 cef_char_t;
typedef cef_string_utf16_t cef_string_t;
```

CEF提供了一些操作字符串的C API，例如：

- **cef_string_set** 对字符串变量赋值(支持深拷贝或浅拷贝)。
- **cef_string_clear** 清空字符串。
- **cef_string_cmp** 比较两个字符串。

[cef_string.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/internal/cef_string.h?at=master) 和 [cef_string_types.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/internal/cef_string_types.h?at=master) 中提供了不同编码（ASCII, UTF8, UTF16 and wide）字符串之间相互转换的方法。

在C++中，使用CefString类可以简化对字符串的使用。CefString字符串可以与std::string(UTF8)、std::wstring(wide)类型字符串相互转换。也可以用来包裹一个cef_string_t以对其进行赋值。

与std::string相互转换：

```c++
std::string str = “Some UTF8 string”;

// Equivalent ways of assigning |str| to |cef_str|. Conversion from UTF8 will occur if necessary.
CefString cef_str(str);
cef_str = str;
cef_str.FromString(str);

// Equivalent ways of assigning |cef_str| to |str|. Conversion to UTF8 will occur if necessary.
str = cef_str;
str = cef_str.ToString();
```

与std::wstring相互转换：

```c++
std::wstring str = “Some wide string”;

// Equivalent ways of assigning |str| to |cef_str|. Conversion from wide will occur if necessary.
CefString cef_str(str);
cef_str = str;
cef_str.FromWString(str);

// Equivalent ways of assigning |cef_str| to |str|. Conversion to wide will occur if necessary.
str = cef_str;
str = cef_str.ToWString();
```

使用ASCII字符串赋值：

```c++
const char* cstr = “Some ASCII string”;
CefString cef_str;
cef_str.FromASCII(cstr);
```

CefSettings 等数据结构中含有cef_string_t 类型的数据成员，可利用CefString实现其他类型字符串到cef_string_t 类型的转换：

```c++
CefSettings settings;
const char* path = “/path/to/log.txt”;

// Equivalent assignments.
CefString(&settings.log_file).FromASCII(path);
cef_string_from_ascii(path, strlen(path), &settings.log_file);
```

## 4.6 命令行参数（Command Line Arguments）

在CEF3和Chromium中许多特性可以通过命令行参数配置。这些参数采用“--some-argument[=optional-param]”的 形式，并通过CefExecuteProcess()和CefMainArgs
结构传递给CEF。

* 在传递CefSettings结构给CefInitialize()之前，可将CefSettings.command_line_args_disabled设置为true来禁用对命令行参数的处理。
* 若想在宿主应用程序中指定CEF/Chromium命令行参数，需要实现CefApp::OnBeforeCommandLineProcessing()方法。
* 若想传递应用特定的命令行参数 (non-CEF/Chromium) 给子进程，需要实现CefBrowserProcessHandler::OnBeforeChildProcessLaunch()。

更多关于如何查找已支持的命令行选项的信息，查看[shared/common/client_switches.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/shared/common/client_switches.cc?at=master)文件的注释。

# 5 应用布局（Application Layout）

不同平台上的应用布局有很大不同。比如，在Mac OS X上，你的资源布局必须遵循特定的 app bundles 结构；在Window与Linux上则更灵活，允许自定义CEF库文件与资源文件的位置。为了获取到完整的应用布局示例，可以从[这里](http://opensource.spotify.com/cefbuilds/index.html)下载“Sample Application”。README.txt文件详细说明了哪些文件是可选的，哪些文件是必须的。

## 5.1 Windows

在Windows平台上，默认布局将libcef库文件、相关资源放置在应用可执行文件的同级目
录，文件夹结构大致如下：

```c++
Application/
    cefclient.exe  <= cefclient application executable
    libcef.dll <= main CEF library
    icudtl.dat <= unicode support data
    libEGL.dll, libGLESv2.dll, ... <= accelerated compositing support libraries
    cef.pak, devtools_resources.pak, ... <= non-localized resources and strings
    natives_blob.bin, snapshot_blob.bin <= V8 initial snapshot
    locales/
        en-US.pak, ... <= locale-specific resources and strings
```

使用结构体CefSettings可以自定义CEF库文件、资源文件的位置（查看README.txt文件获取更详细的信息）。虽然在Windows平台上，cefclient项目将资源文件以二进制形式编译进 [cefclient/resources/win/cefclient.rc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefclient/resources/win/cefclient.rc?at=master) 文件，但是改为从本地文件系统加载资源也很容易。

## 5.2 Linux和Mac OS X

参见[Linux](https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage.md#markdown-header-linux)和[Mac OS X](https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage.md#markdown-header-mac-os-x)。

# 6 应用结构（Application Structure）

每个CEF3应用都有相同的结构：

* 提供入口函数，用于初始化CEF、运行子进程执行逻辑或者CEF消息循环。
* 提供CefApp实现，用于处理进程相关的回调。
* 提供CefClient实现，用于处理browser实例相关的回调。
* 调用CefBrowserHost::CreateBrowser()创建一个browser实例，使用CefLifeSpanHandler管理browser对象生命周期。

## 6.1 入口函数（Entry-Point Function）

基于CEF3的应用程序将运行多个进程，这些进程可以使用同一个可执行文件或者可以为子进程指定单独的可执行文件。进程的执行从入口函数开始。[cefclient/cefclient_win.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefclient/cefclient_win.cc?at=master), [cefclient/cefclient_gtk.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefclient/cefclient_gtk.cc?at=master) and [cefclient/cefclient_mac.mm](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefclient/cefclient_mac.mm?at=master)给出了与平台相关的入口函数示例。
当启动子进程时，CEF将使用命令行参数指定的配置信息，这些命令行参数通过
CefMainArgs结构体传入到CefExecuteProcess函数。CefMainArgs的定义与平台相关，在
Linux、Mac OS X平台下，它接收main函数传入的argc和argv参数值。

```cpp
CefMainArgs main_args(argc, argv);
```

在Windows平台下，它接收wWinMain函数传入的实例句柄（HINSTANCE）参数，也可以通过调用GetModuleHandle(NULL)获取这个实例句柄。

```cpp
CefMainArgs main_args(hInstance);
```

### 6.2.1 使用单一可执行文件（Single Executable）

当使用单一可执行文件时，不同类型进程的入口函数有差异。Windows、Linux平台支持使用单一可执行文件，Mac OS X平台则不行。示例：

```cpp
// Program entry-point function.
int main(int argc, char* argv[]) {
  // Structure for passing command-line arguments.
  // The definition of this structure is platform-specific.
  CefMainArgs main_args(argc, argv);

  // Optional implementation of the CefApp interface.
  CefRefPtr<MyApp> app(new MyApp);

  // Execute the sub-process logic, if any. This will either return immediately for the browser
  // process or block until the sub-process should exit.
  int exit_code = CefExecuteProcess(main_args, app.get());
  if (exit_code >= 0) {
    // The sub-process terminated, exit now.
    return exit_code;
  }

  // Populate this structure to customize CEF behavior.
  CefSettings settings;

  // Initialize CEF in the main process.
  CefInitialize(main_args, settings, app.get());

  // Run the CEF message loop. This will block until CefQuitMessageLoop() is called.
  CefRunMessageLoop();

  // Shut down CEF.
  CefShutdown();

  return 0;
}
```

### 6.2.2 子进程使用单独的可执行文件（Separate Sub-Process Executable）

当子进程使用单独的可执行文件时，主进程和子进程需要各自的可执行工程和入口函数。主程序的入口函数示例：

```cpp
// Program entry-point function.
int main(int argc, char* argv[]) {
  // Structure for passing command-line arguments.
  // The definition of this structure is platform-specific.
  CefMainArgs main_args(argc, argv);

  // Optional implementation of the CefApp interface.
  CefRefPtr<MyApp> app(new MyApp);

  // Populate this structure to customize CEF behavior.
  CefSettings settings;

  // Specify the path for the sub-process executable.
  CefString(&settings.browser_subprocess_path).FromASCII(“/path/to/subprocess”);

  // Initialize CEF in the main process.
  CefInitialize(main_args, settings, app.get());

  // Run the CEF message loop. This will block until CefQuitMessageLoop() is called.
  CefRunMessageLoop();

  // Shut down CEF.
  CefShutdown();

  return 0;
}
```

子进程程序的入口函数示例：

```cpp
// Program entry-point function.
int main(int argc, char* argv[]) {
  // Structure for passing command-line arguments.
  // The definition of this structure is platform-specific.
  CefMainArgs main_args(argc, argv);

  // Optional implementation of the CefApp interface.
  CefRefPtr<MyApp> app(new MyApp);

  // Execute the sub-process logic. This will block until the sub-process should exit.
  return CefExecuteProcess(main_args, app.get());
}
```

## 6.3 集成消息循环（Message Loop Integration）

CEF可以与现有应用中消息环境集成，而不用它自己提供的消息循环。有两种方式可以做到：

1. 以周期性执调用CefDoMessageLoopWork()来替代对CefRunMessageLoop()的调用。
  CefDoMessageLoopWork()的每一次调用，都是对CEF消息循环的单次迭代。值得在应用中注意的是，此方法调用次数太少时，CEF消息循环会饿死并降低browser的性能，调用次数太频繁又将降低CPU使用率。
2. 将CefSettings.multi_threaded_message_loop设置为true（仅Windows平台下有效）。
  这将导致CEF用一个单独的线程负责browser UI，而不用主线程负责。使用这个方法时，CefDoMessageLoopWork()和CefRunMessageLoop()都不需要调用。CefInitialze()、CefShutdown()仍在主线程中调用。你需要提供与主线程通信的机制（查看cefclient_win.cpp中提供的消息窗口实例）。在Windows平台下，你可以通过设置命令行参数“--multi-threaded-message-loop ”来测试这一消息模式。

## 6.4 CefSettings

CefSettings数据结构允许定义全局的CEF配置，一些经常的配置项如下：

* **single_process** 设置为true时，browser和renderer共享一个进程。此项也可以通过命令行参数“single-process”配置。
* **browser_subprocess_path** 用于在子进程使用单独的可执行文件时设置子进程可执行文件的路径。
* **multi_threaded_message_loop** 此项为true时，browser进程的消息循环由单独的线程负责。
* **command_line_args_disabled** 此项为true时， 不能使用标准的 CEF/Chromium命令行参数配置browser进程。
* **cache_path** 设置磁盘上用于存放缓存数据的位置。如果此项为空，某些功能使用内存缓存，其他功能使用临时磁盘缓存。只有设置了cache_path，HTML5数据库，如localStorage ，才能跨session缓存。
* **locale** 此设置项将传递给Blink。如果此项为空，将使用默认值“en-US”。在Linux平台下此项被忽略，而使用环境变量中的值，解析的依次顺序为：LANGUAE，LC_ALL，LC_MESSAGES 和LANG。此项也可以通过命令行参数“lang”配置。
* **log_file** 此项用于设置输出debug日志的文件路径。如果为空，默认的日志文件名为debug.log，位于应用程序所在的目录。此项也可以通过命令参数“log-file”配置。
* **log_severity** 此项设置日志级别。只有不低于此等级的日志才会被记录。此项可以通过命令行参数“log-severity”配置，可以设置的值为“verbose”，“info”，“warning”，“error”，“error-report”，“disable”。
* **resources_dir_path** 此项设置资源文件夹绝对路径。如果为空，cef.pak和/或devtools_resourcs.pak必须放置在Windows/Linux平台下的可执行文件同级目录，或Mac OS X下的app bundle Resources directory。此项也可以通过命令行参数“resource-dir-path”配置。
* **locales_dir_path** 此项设置locale文件夹的绝对路径。如果为空，locale文件夹必须是可执行文件所在目录，在Mac OS X平台下此项被忽略，pak文件将从app bundle Resources目录。此项也可以通过命令行参数“locales-dir-path”配置。
* **remote_debugging_port** 此项可以设置1024-65535之间的值，用于在指定端口开启远程调试。例如，如果设置的值为8080，远程调试的URL为http://localhost:8080。CEF或Chrome浏览器能够远程调试CEF。此项也可以通过命令行参数“remote-debugging-port”配置。

## 6.5 CefBrowser与CefFrame

[CefBrowser](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefBrowser.html)和[CefFrame](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefFrame.html)对象用于向被用来发送命令给浏览器和在回调函数里获取状态信息。每个CefBrowser对象包含一个代表页面的顶层frame的主CefFrame对象，同时每个CefBrowser对象可以包含零或多个分别代表不同的子Frame的CefFrame对象。例如，一个浏览器加载了两个iframe，则该CefBrowser对象拥有三个CefFrame对象（顶层frame和两个iframe）。

下面的代码在浏览器的主frame里加载一个URL：

```cpp
browser->GetMainFrame()->LoadURL(some_url);
```

下面的代码执行浏览器的回退操作：

```cpp
browser->GoBack();
```

下面的代码从主frame里获取HTML内容：

```cpp
// Implementation of the CefStringVisitor interface.
class Visitor : public CefStringVisitor {
 public:
  Visitor() {}

  // Called asynchronously when the HTML contents are available.
  virtual void Visit(const CefString& string) OVERRIDE {
    // Do something with |string|...
  }

  IMPLEMENT_REFCOUNTING(Visitor);
};

browser->GetMainFrame()->GetSource(new Visitor());
```

CefBrowser和CefFrame对象在browser进程和render进程都有对应的代理对象。在
browser进程里，可以通过调用CefBrowser::GetHost()控制宿主（Host）行为。例如，浏览器窗口的原生句柄可以通过以下方式获取：

```cpp
// CefWindowHandle is defined as HWND on Windows, NSView* on Mac OS X
// and GtkWidget* on Linux.
CefWindowHandle window_handle = browser->GetHost()->GetWindowHandle();
```

其他方法包括历史导航，加载字符串和请求，发送编辑命令，提取text/html内容等。

## 6.6 CefApp

CefApp接口提供了访问指定进程的回调函数。重要的回调函数如下：

* **OnBeforeCommandLineProcessing** 提供了以编程方式设置命令行参数的机会。
* **OnRegisterCustomSchemes** 提供了注册自定义schemes的机会。
* **GetBrowserProcessHandler** 返回定制browser进程的handler ，该handler 包括了诸如OnContextInitialized的回调。which returns the handler for functionality specific to the browser process including the OnContextInitialized() method.
* **GetRenderProcessHandler** 返回定制Render进程的Handler，该Handler包含JavaScript相关的一些回调以及消息处理的回调。更多细节，请参考[JavaScriptIntegration](https://bitbucket.org/chromiumembedded/cef/wiki/JavaScriptIntegration.md) 。which returns the handler for functionality specific to the render process. This includes JavaScript-related callbacks and process messages. See the [JavaScriptIntegration](https://bitbucket.org/chromiumembedded/cef/wiki/JavaScriptIntegration.md) Wiki page and the “Inter-Process Communication” section for more information.

[cefsimple/simple_app.h](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_app.h?at=master) 和 [cefsimple/simple_app.cc](https://bitbucket.org/chromiumembedded/cef/src/master/tests/cefsimple/simple_app.cc?at=master) 提供了CefApp实现示例。

## 6.7 CefClient

[CefClient](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefClient.html)提供访问浏览器实例相关的回调接口。多个浏览器可以共享一个CefClient实例。几个重要的回调包括：

* 比如处理Browser的生命周期，右键菜单，对话框，通知显示， 拖曳事件，焦点事件，键盘事件的处理接口等等。大部分处理接口是可选的。cef_client.h文件给出了没有实现某个处理接口造成的影响。
* **OnProcessMessageReceived** 当接收到render进程发送过来的IPC消息时被调用。

## 6.8 brower生命周期（Browser Life Span）

browser的生命周期起始于对 CefBrowserHost::CreateBrowser() 或CefBrowserHost::CreateBrowserSync() 的调用。可以在CefBrowserProcessHandler::OnContextInitialized() 回调或者平台相关的消息处理函数（like WM_CREATE on Windows）中调用以上两个函数。

```cpp
// Information about the window that will be created including parenting, size, etc.
// The definition of this structure is platform-specific.
CefWindowInfo info;
// On Windows for example...
info.SetAsChild(parent_hwnd, client_rect);

// Customize this structure to control browser behavior.
CefBrowserSettings settings;

// CefClient implementation.
CefRefPtr<MyClient> client(new MyClient);

// Create the browser asynchronously. Initially loads the Google URL.
CefBrowserHost::CreateBrowser(info, client.get(), “http://www.google.com”, settings, NULL);
```

[CefLifeSpanHandler](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefLifeSpanHandler.html)类提供了管理 browser生命周期必要回调。以下为相关方法和成员：

```cpp
class MyClient : public CefClient,
                 public CefLifeSpanHandler,
                 ... {
  // CefClient methods.
  virtual CefRefPtr<CefLifeSpanHandler> GetLifeSpanHandler() OVERRIDE {
    return this;
  }

  // CefLifeSpanHandler methods.
  void OnAfterCreated(CefRefPtr<CefBrowser> browser) OVERRIDE;
  bool DoClose(CefRefPtr<CefBrowser> browser) OVERRIDE;
  void OnBeforeClose(CefRefPtr<CefBrowser> browser) OVERRIDE;

  // Member accessors.
  CefRefPtr<CefBrowser> GetBrower() { return m_Browser; }
  bool IsClosing() { return m_bIsClosing; }

 private:
  CefRefPtr<CefBrowser> m_Browser;
  int m_BrowserId;
  int m_BrowserCount;
  bool m_bIsClosing;

  IMPLEMENT_REFCOUNTING(MyClient);
};
```

当browser对象创建后OnAfterCreated() 方法立即执行。宿主应用可以用这个方法来维护对主browser对象的引用。

```cpp
void MyClient::OnAfterCreated(CefRefPtr<CefBrowser> browser) {
  // Must be executed on the UI thread.
  REQUIRE_UI_THREAD();

  if (!m_Browser.get())   {
    // Keep a reference to the main browser.
    m_Browser = browser;
    m_BrowserId = browser->GetIdentifier();
  }

  // Keep track of how many browsers currently exist.
  m_BrowserCount++;
}
```

调用CefBrowserHost::CloseBrowser()以销毁browser对象。

```cpp
// Notify the browser window that we would like to close it. This will result in a call to 
// MyHandler::DoClose() if the JavaScript 'onbeforeunload' event handler allows it.
browser->GetHost()->CloseBrowser(false);
```

如果browser对象的关闭事件来源于他父窗口的关闭方法（比如，在父窗口上点击X控钮）。父
窗口需要调用 CloseBrowser(false) 并且等待OS的第二个关闭事件，第二个关闭事件的到来意味着browser允许被关闭。如果在JavaScript ‘onbeforeunload’事件处理中或者 DoClose()回调中取消了关闭操作，则OS不会发送第二个关闭事件。注意一下面示例中对IsCloseing()的判断——它在第一个关闭事件中返回false，在第二个关闭事件中返回true(当 DoCloase 被调用后)。

Windows平台下，在父窗口的WndProc里处理WM_ClOSE消息：

```cpp
case WM_CLOSE:
  if (g_handler.get() && !g_handler->IsClosing()) {
    CefRefPtr<CefBrowser> browser = g_handler->GetBrowser();
    if (browser.get()) {
      // Notify the browser window that we would like to close it. This will result in a call to 
      // MyHandler::DoClose() if the JavaScript 'onbeforeunload' event handler allows it.
      browser->GetHost()->CloseBrowser(false);

      // Cancel the close.
      return 0;
    }
  }

  // Allow the close.
  break;

case WM_DESTROY:
  // Quitting CEF is handled in MyHandler::OnBeforeClose().
  return 0;
}
```

Linux和OS X平台下的示例请参考[这里](https://bitbucket.org/chromiumembedded/cef/wiki/GeneralUsage#markdown-header-browser-life-span)。

DoClose()方法设置m_blsClosing 标志位为true，并返回false以发送第二次的OS关闭事件。

```cpp
bool MyClient::DoClose(CefRefPtr<CefBrowser> browser) {
  // Must be executed on the UI thread.
  REQUIRE_UI_THREAD();

  // Closing the main window requires special handling. See the DoClose()
  // documentation in the CEF header for a detailed description of this
  // process.
  if (m_BrowserId == browser->GetIdentifier()) {
    // Set a flag to indicate that the window close should be allowed.
    m_bIsClosing = true;
  }

  // Allow the close. For windowed browsers this will result in the OS close
  // event being sent.
  return false;
}
```

当操作系统捕捉到第二次关闭事件，它才会允许父窗口真正关闭。该动作会先触发OnBeforeClose() 回调，请在该回调里释放所有对浏览器对象的引用。

```cpp
void MyHandler::OnBeforeClose(CefRefPtr<CefBrowser> browser) {
  // Must be executed on the UI thread.
  REQUIRE_UI_THREAD();

  if (m_BrowserId == browser->GetIdentifier()) {
    // Free the browser pointer so that the browser can be destroyed.
    m_Browser = NULL;
  }

  if (--m_BrowserCount == 0) {
    // All browser windows have closed. Quit the application message loop.
    CefQuitMessageLoop();
  }
}
```

请参考cefclient示例里给出的在不同平台下的完整处理流程。

## 6.9 离屏渲染（Off-Screen Rendering）

在离屏渲染模式下，CEF不会创建原生浏览器窗口。CEF为宿主应用提供无效区和像素缓存
区，而宿主应用负责将鼠标键盘以及焦点事件通知到CEF。离屏渲染目前不支持硬件加速，所以性能上可能逊色于非离屏渲染。离屏浏览器将收到和窗口浏览器同样的事件通知，例如生命周期事件。下面介绍如何使用离屏渲染：

* 实现 [CefRenderHandler](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefRenderHandler.html) 接口。除特别说明外，所有的方法都需要重写。
* 在将CefWindowInfo数据结构传递给CefBrowserHost::CreateBrowser()之前调用CefWindowInfo::SetAsWindowless()，如果没有父窗口被传递给SetAsWindowless，则有些功能将不可用，如右键菜单。
* CefRenderHandler::GetViewRect()方法将被调用以获得需要的可视区域
* CefRenderHandler::OnPaint() 方法将被调用以提供无效区和更新过的像素缓存。cefclient程序里使用OpenGL绘制，但你可以使用任何别的绘制技术。
* 通过调用CefBrowserHost::WasResized()可改变浏览器大小。这将导致调用GetViewRect()获取新的浏览器大小，之后调用OnPaint()重新绘制。
* 调用CefBrowserHost::SendXXX()方法通知浏览器鼠标、键盘和焦点事件。
* 调用CefBrowserHost::CloseBrowser()销毁浏览器。

使用命令行参数--off-screen-rendering-enabled 运行cefclient，可以测试离屏渲染的效果。

# 7 投递任务（Posting Tasks）

一个进程内，可以通过CefPostTask族的函数（see the [include/cef_task.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/cef_task.h?at=master)）在线程之间投递任务。任务在被投递到目标线程的消息循环里后异步执行。

CEF 提供base::Bind 和base::Callback 模板回调类，用于将绑定的方法，对象和参数传递给 CefPostTask。[include/base/cef_callback.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/base/cef_callback.h?at=master) 给出了完整的base::Bind 和 base::Callback使用方法.  [include/wrapper/cef_closure_task.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/wrapper/cef_closure_task.h?at=master) 给出了 base::Closure 向 CefTask转换的帮助。例如:

```cpp
// Include the necessary headers.
#include “include/base/cef_bind.h”
#include “include/wrapper/cef_closure_task.h”

// To execute a bound function:

// Define a function.
void MyFunc(int arg) { /* do something with |arg| on the UI thread */ }

// Post a task that will execute MyFunc on the UI thread and pass an |arg|
// value of 5.
CefPostTask(TID_UI, base::Bind(&MyFunc, 5));

// To execute a bound method:

// Define a class.
class MyClass : public CefBase {
 public:
  MyClass() {}
  void MyMethod(int arg) { /* do something with |arg| on the UI thread */ }
 private:
  IMPLEMENT_REFCOUNTING(MyClass);
};

// Create an instance of MyClass.
CefRefPtr<MyClass> instance = new MyClass();

// Post a task that will execute MyClass::MyMethod on the UI thread and pass
// an |arg| value of 5. |instance| will be kept alive until after the task
// completes.
CefPostTask(TID_UI, base::Bind(&MyClass::MyMethod, instance, 5));
```

如果宿主程序需要保留一个引用用于循环操作，则可以使用CefTaskRunner类。例如，获取UI线程task runner的代码如下：

```cpp
CefRefPtr<CefTaskRunner> task_runner = CefTaskRunner::GetForThread(TID_UI);
```

# 8. 进程间通信(Inter-Process Communication (IPC))

由于CEF3采用多进程模型，所以需要提供进程间通信机制。CefBrowser和CefFrame对象在borwser和render进程里都有代理对象。每个CefBrowser和CefFrame对象都有自己的唯一ID，便于在borwser和render进程中定位匹配的代理对象。

## 8.1 处理启动消息(Process Startup Messages)

为了给所有render进程提供一样的启动信息，需要实现browser进程的CefBrowserProcessHander::OnRenderProcessThreadCreated()方法。这将会给render进程的CefRenderProcessHandler::OnRenderThreadCreated()方法传入信息。

## 8.2 处理运行时消息(Process Runtime Messages)

在进程生命周期的任何时候，都可以通过CefProcessMessage类传递进程间消息。这些信息和特定的CefBrowser实例相关。可以通过CefBrowser::SendProcessMessage()方法发送消息。用户可以通过CefProcessMessage::GetArgumentList()在进程间消息中包含任意的状态信息。

```cpp
// Create the message object.
CefRefPtr<CefProcessMessage> msg= CefProcessMessage::Create(“my_message”);

// Retrieve the argument list object.
CefRefPtr<CefListValue> args = msg->GetArgumentList();

// Populate the argument values.
args->SetString(0, “my string”);
args->SetInt(0, 10);

// Send the process message to the render process.
// Use PID_BROWSER instead when sending a message to the browser process.
browser->SendProcessMessage(PID_RENDERER, msg);
```

从browser进程发送到render进程的消息将会在CefRenderProcessHandler::OnProcessMessageReceived()中接收。从render进程发送到browser进程的消息将会在CefClient::OnProcessMessageReceived()中接收。

```cpp
bool MyHandler::OnProcessMessageReceived(
    CefRefPtr<CefBrowser> browser,
    CefProcessId source_process,
    CefRefPtr<CefProcessMessage> message) {
  // Check the message name.
  const std::string& message_name = message->GetName();
  if (message_name == “my_message”) {
    // Handle the message here...
    return true;
  }
  return false;
}
```

通过在消息参数里包含frame ID（调用CefFrame::GerIdentifier()获取），可以将消息与特性的CefFrame绑定。消息接收进程可以CefBrowser::GetFrame()找到对应的CefFrame。

```cpp
// Helper macros for splitting and combining the int64 frame ID value.
#define MAKE_INT64(int_low, int_high) \
    ((int64) (((int) (int_low)) | ((int64) ((int) (int_high))) << 32))
#define LOW_INT(int64_val) ((int) (int64_val))
#define HIGH_INT(int64_val) ((int) (((int64) (int64_val) >> 32) & 0xFFFFFFFFL))

// Sending the frame ID.
const int64 frame_id = frame->GetIdentifier();
args->SetInt(0, LOW_INT(frame_id));
args->SetInt(1, HIGH_INT(frame_id));

// Receiving the frame ID.
const int64 frame_id = MAKE_INT64(args->GetInt(0), args->GetInt(1));
CefRefPtr<CefFrame> frame = browser->GetFrame(frame_id);
```

## 8.3 异步JavaScript绑定(Asynchronous JavaScript Bindings)

[JavaScriptIntegration](https://bitbucket.org/chromiumembedded/cef/wiki/JavaScriptIntegration.md)被集成在render进程，但是经常需要和browser进程交互。 JavaScript API应该被设计成可使用 [closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures) 和 [promises](http://www.html5rocks.com/en/tutorials/es6/promises/)异步执行。

### 8.3.1 通用消息转发(Generic Message Router)

CEF 提供了在renderer进程的JavaScript 运行时（ JavaScript running）与browser 进程的C++运行时（C++ running）之间转发异步消息的转发器的通用实现。应用通过标准CEF C++回调 (OnBeforeBrowse, OnProcessMessageRecieved, OnContextCreated等)与转发器交互。render侧的转发器支持通用的
JavaScript回调函数注册机制，同时browser侧的转发器则支持应用程序注册特定的Handler进行逻辑处理。CefMessageRouter 的使用示例详见 [message_router example](https://bitbucket.org/chromiumembedded/cef-project/src/master/examples/message_router/?at=master) 。 [include/wrapper/cef_message_router.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/wrapper/cef_message_router.h?at=master) 给出了完整的使用方式。

### 8.3.2 自定义实现(Custom Implementation)

CEF应用程序也可以给出自己的异步JavaScript绑定。简单的示例如下：

1. 在render进程的JavaScript中注册回调函数。

```JavaScript
// In JavaScript register the callback function.
app.setMessageCallback('binding_test', function(name, args) {
  document.getElementById('result').value = "Response: "+args[0];
});
```

2. render维护一个对这个回调函数的引用。

```cpp
// Map of message callbacks.
typedef std::map<std::pair<std::string, int>,
                 std::pair<CefRefPtr<CefV8Context>, CefRefPtr<CefV8Value> > >
                 CallbackMap;
CallbackMap callback_map_;

// In the CefV8Handler::Execute implementation for “setMessageCallback”.
if (arguments.size() == 2 && arguments[0]->IsString() &&
    arguments[1]->IsFunction()) {
  std::string message_name = arguments[0]->GetStringValue();
  CefRefPtr<CefV8Context> context = CefV8Context::GetCurrentContext();
  int browser_id = context->GetBrowser()->GetIdentifier();
  callback_map_.insert(
      std::make_pair(std::make_pair(message_name, browser_id),
                     std::make_pair(context, arguments[1])));
}
```

3. render进程发送异步IPC消息到browser进程。
4. browser进程接收到IPC消息并处理。
5. browser进程处理完毕后，发送一个异步IPC消息给Render进程以返回处理结果。
6. render进程接收到IPC消息后调用在JavaScript注册的对应回调函数处理返回的结果。

```cpp
// Execute the registered JavaScript callback if any.
if (!callback_map_.empty()) {
  const CefString& message_name = message->GetName();
  CallbackMap::const_iterator it = callback_map_.find(
      std::make_pair(message_name.ToString(),
                     browser->GetIdentifier()));
  if (it != callback_map_.end()) {
    // Keep a local reference to the objects. The callback may remove itself
    // from the callback map.
    CefRefPtr<CefV8Context> context = it->second.first;
    CefRefPtr<CefV8Value> callback = it->second.second;

    // Enter the context.
    context->Enter();

    CefV8ValueList arguments;

    // First argument is the message name.
    arguments.push_back(CefV8Value::CreateString(message_name));

    // Second argument is the list of message arguments.
    CefRefPtr<CefListValue> list = message->GetArgumentList();
    CefRefPtr<CefV8Value> args = CefV8Value::CreateArray(list->GetSize());
    SetList(list, args);  // Helper function to convert CefListValue to CefV8Value.
    arguments.push_back(args);

    // Execute the callback.
    CefRefPtr<CefV8Value> retval = callback->ExecuteFunction(NULL, arguments);
    if (retval.get()) {
      if (retval->IsBool())
        handled = retval->GetBoolValue();
    }

    // Exit the context.
    context->Exit();
  }
}
```

7. 在CefRenderProcessHandler::OnContextReleased()里释放相关的V8资源。

```cpp
void MyHandler::OnContextReleased(CefRefPtr<CefBrowser> browser,
                                  CefRefPtr<CefFrame> frame,
                                  CefRefPtr<CefV8Context> context) {
  // Remove any JavaScript callbacks registered for the context that has been released.
  if (!callback_map_.empty()) {
    CallbackMap::iterator it = callback_map_.begin();
    for (; it != callback_map_.end();) {
      if (it->second.first->IsSame(context))
        callback_map_.erase(it++);
      else
        ++it;
    }
  }
}
```

## 8.4 同步请求(Synchronous Requests)

需要在browser进程和render进程间实现同步通信的场景很少。应该被尽量避免发生同步通信，因为这会对render进程的性能造成负面影响。但是，当你一定要做进程间同步通信时，可以考虑使用 [synchronous XMLHttpRequests](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Synchronous_and_Asynchronous_Requests)，XMLHttpRequest在等待browser进程网络响应的时候阻塞render进程。browser进程可以通过自定义scheme Handler或者网络交互处理XMLHttpRequest。
等待。Browser进程可以通过自定义scheme Handler或者网络交互处理XMLHttpRequest。

# 9. 网络层(Network Layer)

默认情况下，CEF3的网络请求会在宿主应用中被透明处理。然而CEF3也暴露了一系列网络相关的函
数用以处理网络请求。

网络相关的回调函数可在不同线程被调用，因此要注意相关文档的说明并保证数据安全。

## 9.1 自定义请求(Custom Requests)

在浏览器frame中加载URL的最简单方法试调用CefFrame::LoadURL():

```cpp
browser->GetMainFrame()->LoadURL(some_url);
```

如果应用希望发送包含自定义请求头部或上传数据的复杂请求，则可以调用CefFrame::LoadRequest()方法。该方法接受单一的 [CefRequest](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefRequest.html)对象参数。

```cpp
// Create a CefRequest object.
CefRefPtr<CefRequest> request = CefRequest::Create();

// Set the request URL.
request->SetURL(some_url);

// Set the request method. Supported methods include GET, POST, HEAD, DELETE and PUT.
request->SetMethod(“POST”);

// Optionally specify custom headers.
CefRequest::HeaderMap headerMap;
headerMap.insert(
    std::make_pair("X-My-Header", "My Header Value"));
request->SetHeaderMap(headerMap);

// Optionally specify upload content.
// The default “Content-Type” header value is "application/x-www-form-urlencoded".
// Set “Content-Type” via the HeaderMap if a different value is desired.
const std::string& upload_data = “arg1=val1&arg2=val2”;
CefRefPtr<CefPostData> postData = CefPostData::Create();
CefRefPtr<CefPostDataElement> element = CefPostDataElement::Create();
element->SetToBytes(upload_data.size(), upload_data.c_str());
postData->AddElement(element);
request->SetPostData(postData);
```

### 9.1.1 浏览器无关的请求(Browser-Independent Requests)

应用程序可以通过CefURLRequest类发送和浏览器无关的网络请求。实现 [CefURLRequestClient](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefURLRequestClient.html) 接口以处理请求结果。CefURLRequest可以在browser和render进程使用。

```cpp
class MyRequestClient : public CefURLRequestClient {
 public:
  MyRequestClient()
    : upload_total_(0),
      download_total_(0) {}

  virtual void OnRequestComplete(CefRefPtr<CefURLRequest> request) OVERRIDE {
    CefURLRequest::Status status = request->GetRequestStatus();
    CefURLRequest::ErrorCode error_code = request->GetRequestError();
    CefRefPtr<CefResponse> response = request->GetResponse();

    // Do something with the response...
  }

  virtual void OnUploadProgress(CefRefPtr<CefURLRequest> request,
                                uint64 current,
                                uint64 total) OVERRIDE {
    upload_total_ = total;
  }

  virtual void OnDownloadProgress(CefRefPtr<CefURLRequest> request,
                                  uint64 current,
                                  uint64 total) OVERRIDE {
    download_total_ = total;
  }

  virtual void OnDownloadData(CefRefPtr<CefURLRequest> request,
                              const void* data,
                              size_t data_length) OVERRIDE {
    download_data_ += std::string(static_cast<const char*>(data), data_length);
  }

 private:
  uint64 upload_total_;
  uint64 download_total_;
  std::string download_data_;

 private:
  IMPLEMENT_REFCOUNTING(MyRequestClient);
};
```

发送请求：

```cpp
// Set up the CefRequest object.
CefRefPtr<CefRequest> request = CefRequest::Create();
// Populate |request| as shown above...

// Create the client instance.
CefRefPtr<MyRequestClient> client = new MyRequestClient();

// Start the request. MyRequestClient callbacks will be executed asynchronously.
CefRefPtr<CefURLRequest> url_request = CefURLRequest::Create(request, client.get());
// To cancel the request: url_request->Cancel();
```

可以通过CefRequest::SetFlags()自定义请求的行为，这些标志位包括：

* **UR_FLAG_SKIP_CACHE** 如果设置了该标志位，则处理请求响应时，缓存将被跳过。
* **UR_FLAG_ALLOW_CACHED_CREDENTIALS** 如果设置了该标志位，则可能会在请求中包含cookie并在收到响应时保存cookie。同时UR_FLAG_ALLOW_CACHED_CREDENTIALS标志位也必须被设置。
* **UR_FLAG_REPORT_UPLOAD_PROGRESS** 如果设置了该标志位，则当请求拥有请求体时，上载进度事件将会被触发。
* **UR_FLAG_NO_DOWNLOAD_DATA** 如果设置了该标志位，则CefURLRequestClient::OnDownloadData方法不会被调用。
* **UR_FLAG_NO_RETRY_ON_5XX** 如果设置了该标志位，则5xx重定向错误会被交给observer 去处理，而不自动重试。这个功能目前只能在browser进程发起的请求中使用。

例如，为了跳过缓存并不报告下载数据：

```cpp
request->SetFlags(UR_FLAG_SKIP_CACHE | UR_FLAG_NO_DOWNLOAD_DATA);
```

## 9.2 处理请求(Request Handling)

CEF3 支持两种方式处理网络请求。一种是实现scheme Handler，这种方式允许为特定的域(sheme+domain)请求注册对应的请求响应。另一种是请求拦截，允许处理任意的网络请求。

**使用HTTP scheme而非自定义scheme可以避免一系列潜在的问题。**

如果选择使用自定义scheme（有别于HTTP , HTTPS 等）则需要在CEF中注册，以让CEF按希望的方式处理请求。如果你希望自定义的shceme与HTTP行为(support POST requests and enforce [HTTP access control (CORS)](https://developer.mozilla.org/en-US/docs/HTTP/Access_control_CORS) restrictions)一样，则应该注册一个standard 的scheme。如果你的自定义shceme可被跨域执行或者通过XMLHttpRequest 向你的scheme handler发送POST请求 ，则应该考虑使用使用HTTP scheme代替自定义scheme以避免潜在问题。若使用自定义scheme，则需要在所有进程中实现CefApp::OnRegisterCustomSchemes()回调以注册scheme的属性。

```cpp
void MyApp::OnRegisterCustomSchemes(CefRefPtr<CefSchemeRegistrar> registrar) {
  // Register "client" as a standard scheme.
  registrar->AddCustomScheme("client", true, ...);
}
```

### 9.2.1 通用资源管理器(Generic Resource Manager)

CEF 提供了用来管理资源请求的资源管理器的通用实现。这些请求可以来自来自若干个数据源。用户为每个数据源注册处理请求的handler，例如磁盘目录、压缩文档或者其他自定义实现。 应用通过标准CEF C++回调(OnBeforeResourceLoad, GetResourceHandler)与转发器交互。 示例与使用方法见 [resource_manager example](https://bitbucket.org/chromiumembedded/cef-project/src/master/examples/resource_manager/?at=master) 和 [include/wrapper/cef_resource_manager.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/wrapper/cef_resource_manager.h?at=master) 。

### 9.2.2 Scheme响应(Scheme Handler)

调用CefRegisterSchemeHandlerFactory()可以注册一个scheme响应，最好在CefBrowserProcessHandler::OnContextInitialized()方法里调用。例如，为“client://myapp/”请求注册响应：

```cpp
CefRegisterSchemeHandlerFactory("client", “myapp”, new MySchemeHandlerFactory());
```

响应可以被用于内置scheme(HTTP，HTTPS等），也可以被用于自定义scheme。当使用内置scheme时，需要为你的应用程序设置一个唯一域名。实现[CefSchemeHandlerFactory](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefSchemeHandlerFactory.html)和 [CefResourceHandler](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefResourceHandler.html)类去处理请求并返回响应数据。如果使用自定义scheme，必须实现 CefApp::OnRegisterCustomSchemes 。示例和使用方法参见[scheme_handler example](https://bitbucket.org/chromiumembedded/cef-project/src/master/examples/scheme_handler/?at=master)和[include/cef_scheme.h](https://bitbucket.org/chromiumembedded/cef/src/master/include/cef_scheme.h?at=master)。

如果响应数据类型是已知的，则CefStreamResourceHandler类提供了CefResourceHandler类的默认实现。

```cpp
// CefStreamResourceHandler is part of the libcef_dll_wrapper project.
#include “include/wrapper/cef_stream_resource_handler.h”

const std::string& html_content = “<html><body>Hello!</body></html>”;

// Create a stream reader for |html_content|.
CefRefPtr<CefStreamReader> stream =
    CefStreamReader::CreateForData(
        static_cast<void*>(const_cast<char*>(html_content.c_str())),
        html_content.size());

// Constructor for HTTP status code 200 and no custom response headers.
// There’s also a version of the constructor for custom status code and response headers.
return new CefStreamResourceHandler("text/html", stream);
```

### 9.2.3 请求拦截 ( Request Interception)

CefRequestHandler::GetResourceHandler()方法支持拦截任意请求。它和scheme handler 方法使用相同的CefResourceHandler 类。如果使用自定义scheme，必须实现 CefApp::OnRegisterCustomSchemes 。

```cpp
CefRefPtr<CefResourceHandler> MyHandler::GetResourceHandler(
      CefRefPtr<CefBrowser> browser,
      CefRefPtr<CefFrame> frame,
      CefRefPtr<CefRequest> request) {
  // Evaluate |request| to determine proper handling...
  if (...)
    return new MyResourceHandler();

  // Return NULL for default handling of the request.
  return NULL;
}
```

### 9.2.4 过滤请求（Response Filtering）

CefRequestHandler::GetResourceResponseFilter() 方法支持对请求相应数据的过滤。

## 9.3 其他回调（Other Callbacks）

 [CefRequestHandler](http://magpcss.org/ceforum/apidocs3/projects/(default)/CefRequestHandler.html)接口还提供了其他回调函数以处理其他网络相关事件。包括授权、cookie处理、外部协议处理、证书错误等。

## 9.4 代理（Proxy Resolution）

CEF3使用与Google Chrome一样的命令行参数设置代理。

```
--proxy-server=host:port
      Specify the HTTP/SOCKS4/SOCKS5 proxy server to use for requests. An individual proxy
      server is specified using the format:

        [<proxy-scheme>://]<proxy-host>[:<proxy-port>]

      Where <proxy-scheme> is the protocol of the proxy server, and is one of:

        "http", "socks", "socks4", "socks5".

      If the <proxy-scheme> is omitted, it defaults to "http". Also note that "socks" is equivalent to
      "socks5".

      Examples:

        --proxy-server="foopy:99"
            Use the HTTP proxy "foopy:99" to load all URLs.

        --proxy-server="socks://foobar:1080"
            Use the SOCKS v5 proxy "foobar:1080" to load all URLs.

        --proxy-server="sock4://foobar:1080"
            Use the SOCKS v4 proxy "foobar:1080" to load all URLs.

        --proxy-server="socks5://foobar:66"
            Use the SOCKS v5 proxy "foobar:66" to load all URLs.

      It is also possible to specify a separate proxy server for different URL types, by prefixing
      the proxy server specifier with a URL specifier:

      Example:

        --proxy-server="https=proxy1:80;http=socks4://baz:1080"
            Load https://* URLs using the HTTP proxy "proxy1:80". And load http://*
            URLs using the SOCKS v4 proxy "baz:1080".

--no-proxy-server
      Disables the proxy server.

--proxy-auto-detect
      Autodetect  proxy  configuration.

--proxy-pac-url=URL
      Specify proxy autoconfiguration URL.
```

如果代理请求授权，CefRequestHandler::GetAuthCredentials()回调会被调用。将isProxy参数为true以获取用户名和密码。

```cpp
bool MyHandler::GetAuthCredentials(
    CefRefPtr<CefBrowser> browser,
    CefRefPtr<CefFrame> frame,
    bool isProxy,
    const CefString& host,
    int port,
    const CefString& realm,
    const CefString& scheme,
    CefRefPtr<CefAuthCallback> callback) {
  if (isProxy) {
    // Provide credentials for the proxy server connection.
    callback->Continue("myuser", "mypass");
    return true;
  }
  return false;
}
```

网络内容加载可能会因为代理而有延迟。为了更好的用户体验，可以考虑让你的应用程序先显示一个静态页面，再通过[meta refresh](http://en.wikipedia.org/wiki/Meta_refresh)重定向到真实的网站。重定向将被阻塞直到代理解析完成。可以指定--no-proxy-server 禁用代理并做相关测试。代理延迟现象可以通过用命令行“chrome -url=... ”启动chrome浏览器复现。