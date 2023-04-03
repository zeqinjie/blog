---
title: FlutterEngineGroup 引擎启动流程（iOS）
date: 2023-04-02 16:08:42
categories: "Flutter"
tags:
	- Flutter
---

## 流程图

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3132df83ca814d1b8b074b50a0f994c2~tplv-k3u1fbpfcp-zoom-1.image)

## 创建 FlutterEngineGroup

### Flutter 引擎组的作用

- 1、主要用于管理多个 Flutter 引擎，从 FlutterEngineGroup 生成的 FlutterEngine 具有常用共享资源（例如 GPU 上下文、字体度量和隔离线程的快照）的性能优势，从而加快首次渲染的速度、降低延迟并降低内存占用。
- 2、官方说：采用 Group 管理多引擎在内存上，除了第一个 Engine 对象之外，后续每个 Engine 对象在 Android 和 iOS 上仅占用 180kB 。“实际上是否，待验证” 。
- 3、例子 let engines = FlutterEngineGroup(name: "multiple-flutters", project: nil)

### FlutterEngineGroup 源码

- 从源码上看 FlutterEngineGroup 类主要有一个 engines 数组，去维护管理这些引擎

```
@interface FlutterEngineGroup ()
@property(nonatomic, copy) NSString* name;
@property(nonatomic, retain) NSMutableArray<NSValue*>* engines;
// 对象来指定入口文件和 Assets
@property(nonatomic, retain) FlutterDartProject* project;
@end

@implementation FlutterEngineGroup {
  int _enginesCreatedCount;
}

/// 创建 FlutterEngineGroup ,内部初始化数组
- (instancetype)initWithName:(NSString*)name project:(nullable FlutterDartProject*)project {
  self = [super init];
  if (self) {
    _name = [name copy];
    _engines = [[NSMutableArray<NSValue*> alloc] init];
    _project = [project retain];
  }
  return self;
}
...
```

## 创建 FlutterEngine

### 1、makeEngineWithOptions

- 1、引擎创建逻辑，当引擎池没有引擎时，通过`makeEngine`创建一个新引擎。

- 2、如果引擎池里有引擎，则返回当前引擎，并且通过 `spawnWithEntrypoint` 方法设置相同的入口点、dart 库路径、路由、入口参数。
    - a、FlutterEngineGroup 是怎么做到部分的引擎资源共享呢？重点在于 `spawnWithEntrypoint` 内部实现，后面我们再说~

- 3、例子： let newEngine = appDelegate.engines.makeEngine(with: options)

- 4、makeEngineWithOptions 源码如下：

```
/// 创建并返回引擎
- (FlutterEngine*)makeEngineWithOptions:(nullable FlutterEngineGroupOptions*)options {
  NSString* entrypoint = options.entrypoint;
  NSString* libraryURI = options.libraryURI;
  NSString* initialRoute = options.initialRoute;
  NSArray<NSString*>* entrypointArgs = options.entrypointArgs;

  FlutterEngine* engine;
  // 当引擎没有创建过，那么就创建一个新的引擎
  if (self.engines.count <= 0) {
    engine = [self makeEngine];
    [engine runWithEntrypoint:entrypoint
                   libraryURI:libraryURI
                 initialRoute:initialRoute
               entrypointArgs:entrypointArgs];
  } else {
    // 已经存在引擎，则获取当前引擎，并通过 spawnWithEntrypoint 方法做一系列操作
    FlutterEngine* spawner = (FlutterEngine*)[self.engines[0] pointerValue];
    engine = [spawner spawnWithEntrypoint:entrypoint
                               libraryURI:libraryURI
                             initialRoute:initialRoute
                           entrypointArgs:entrypointArgs];
  }
  [_engines addObject:[NSValue valueWithPointer:engine]];

  NSNotificationCenter* center = [NSNotificationCenter defaultCenter];
  [center addObserver:self
             selector:@selector(onEngineWillBeDealloced:)
                 name:kFlutterEngineWillDealloc
               object:engine];

  return engine;
}

```

### 2、makeEngine

-   makeEngine 的源码

```
// @property(nonatomic, readonly) NSMutableDictionary* pluginPublications;
// @property(nonatomic, readonly) NSMutableDictionary<NSString*, FlutterEngineRegistrar*>* registrars;


/// 创建新的引擎
- (FlutterEngine*)makeEngine {
  NSString* engineName = [NSString stringWithFormat:@"%@.%d", self.name, ++_enginesCreatedCount];
  FlutterEngine* result = [[FlutterEngine alloc] initWithName:engineName project:self.project];
  return [result autorelease];
}

/// 实现逻辑
- (instancetype)initWithName:(NSString*)labelPrefix
                     project:(FlutterDartProject*)project
      allowHeadlessExecution:(BOOL)allowHeadlessExecution
          restorationEnabled:(BOOL)restorationEnabled {
  self = [super init];
  NSAssert(self, @"Super init cannot be nil");
  NSAssert(labelPrefix, @"labelPrefix is required");

  _restorationEnabled = restorationEnabled;
  _allowHeadlessExecution = allowHeadlessExecution;
  _labelPrefix = [labelPrefix copy];

  _weakFactory = std::make_unique<fml::WeakPtrFactory<FlutterEngine>>(self);

  if (project == nil) {
    _dartProject.reset([[FlutterDartProject alloc] init]);
  } else {
    _dartProject.reset([project retain]);
  }

  if (!EnableTracingIfNecessary([_dartProject.get() settings])) {
    NSLog(
        @"Cannot create a FlutterEngine instance in debug mode without Flutter tooling or "
        @"Xcode.\n\nTo launch in debug mode in iOS 14+, run flutter run from Flutter tools, run "
        @"from an IDE with a Flutter IDE plugin or run the iOS project from Xcode.\nAlternatively "
        @"profile and release mode apps can be launched from the home screen.");
    [self release];
    return nil;
  }

  _pluginPublications = [[NSMutableDictionary alloc] init];
  _registrars = [[NSMutableDictionary alloc] init];
  [self recreatePlatformViewController];

  _binaryMessenger = [[FlutterBinaryMessengerRelay alloc] initWithParent:self];
  _textureRegistry = [[FlutterTextureRegistryRelay alloc] initWithParent:self];
  _connections.reset(new flutter::ConnectionCollection());

  NSNotificationCenter* center = [NSNotificationCenter defaultCenter];
  [center addObserver:self
             selector:@selector(onMemoryWarning:)
                 name:UIApplicationDidReceiveMemoryWarningNotification
               object:nil];

  [center addObserver:self
             selector:@selector(applicationWillEnterForeground:)
                 name:UIApplicationWillEnterForegroundNotification
               object:nil];

  [center addObserver:self
             selector:@selector(applicationDidEnterBackground:)
                 name:UIApplicationDidEnterBackgroundNotification
               object:nil];

  [center addObserver:self
             selector:@selector(onLocaleUpdated:)
                 name:NSCurrentLocaleDidChangeNotification
               object:nil];

  return self;
}
```

#### a、初始化 FlutterDartProject 对象

- 1、用于 Flutter 引擎中运行 Dart 代码。Flutter 引擎通过 FlutterDartProject 与 Dart 代码进行交互，比如启动 Dart VM，加载 Dart 库，执行 Dart 函数等等。在 Flutter 应用程序中，通常会创建一个 FlutterDartProject 实例用于管理 Dart 代码的执行。
- 2、初始化 _settings ，从指定的 bundle 中读取 FlutterSettings 配置信息，并返回一个 FlutterSettings 对象。这个对象包含了应用的各种配置信息，例如引擎的版本、是否启用 Dart 开发者工具、是否支持热重载等。

```
struct Settings {
    Settings();
    ...
    // Enable the Impeller renderer on supported platforms. Ignored if Impeller is
    // not supported on the platform.
    bool enable_impeller = false;
}

- (instancetype)init {
  return [self initWithPrecompiledDartBundle:nil];
}

- (instancetype)initWithPrecompiledDartBundle:(nullable NSBundle*)bundle {
  self = [super init];

  if (self) {
    _settings = FLTDefaultSettingsForBundle(bundle);
  }

  return self;
}

flutter::Settings FLTDefaultSettingsForBundle(NSBundle* bundle) {
    auto command_line = flutter::CommandLineFromNSProcessInfo();

    // Precedence:
    // 1. Settings from the specified NSBundle.
    // 2. Settings passed explicitly via command-line arguments.
    // 3. Settings from the NSBundle with the default bundle ID.
    // 4. Settings from the main NSBundle and default values.

    NSBundle* mainBundle = [NSBundle mainBundle];
    NSBundle* engineBundle = [NSBundle bundleForClass:[FlutterViewController class]];

    bool hasExplicitBundle = bundle != nil;
    if (bundle == nil) {
        bundle = [NSBundle bundleWithIdentifier:[FlutterDartProject defaultBundleIdentifier]];
    }
    if (bundle == nil) {
        bundle = mainBundle;
    }

    auto settings = flutter::SettingsFromCommandLine(command_line);

    settings.task_observer_add = [](intptr_t key, const fml::closure& callback) {
        fml::MessageLoop::GetCurrent().AddTaskObserver(key, callback);
    };

    settings.task_observer_remove = [](intptr_t key) {
        fml::MessageLoop::GetCurrent().RemoveTaskObserver(key);
    };

    settings.log_message_callback = [](const std::string& tag, const std::string& message) {
        // TODO(cbracken): replace this with os_log-based approach.
        // https://github.com/flutter/flutter/issues/44030
        std::stringstream stream;
        if (!tag.empty()) {
            stream << tag << ": ";
        }
        stream << message;
        std::string log = stream.str();
        syslog(LOG_ALERT, "%.*s", (int)log.size(), log.c_str());
    };

    // The command line arguments may not always be complete. If they aren't, attempt to fill in
    // defaults.

    // Flutter ships the ICU data file in the bundle of the engine. Look for it there.
    if (settings.icu_data_path.empty()) {
        NSString* icuDataPath = [engineBundle pathForResource:@"icudtl" ofType:@"dat"];
        if (icuDataPath.length > 0) {
            settings.icu_data_path = icuDataPath.UTF8String;
        }
    }

    if (flutter::DartVM::IsRunningPrecompiledCode()) {
        if (hasExplicitBundle) {
            NSString* executablePath = bundle.executablePath;
            if ([[NSFileManager defaultManager] fileExistsAtPath:executablePath]) {
                settings.application_library_path.push_back(executablePath.UTF8String);
            }
        }

        // No application bundle specified.  Try a known location from the main bundle's Info.plist.
        if (settings.application_library_path.empty()) {
            NSString* libraryName = [mainBundle objectForInfoDictionaryKey:@"FLTLibraryPath"];
            NSString* libraryPath = [mainBundle pathForResource:libraryName ofType:@""];
            if (libraryPath.length > 0) {
                NSString* executablePath = [NSBundle bundleWithPath:libraryPath].executablePath;
                if (executablePath.length > 0) {
                    settings.application_library_path.push_back(executablePath.UTF8String);
                }
            }
        }

        // In case the application bundle is still not specified, look for the App.framework in the
        // Frameworks directory.
        if (settings.application_library_path.empty()) {
            NSString* applicationFrameworkPath = [mainBundle pathForResource:@"Frameworks/App.framework"
                                      ofType:@""];
            if (applicationFrameworkPath.length > 0) {
                NSString* executablePath =
                    [NSBundle bundleWithPath:applicationFrameworkPath].executablePath;
                if (executablePath.length > 0) {
                    settings.application_library_path.push_back(executablePath.UTF8String);
                }
            }
        }
    }

    // Checks to see if the flutter assets directory is already present.
    if (settings.assets_path.empty()) {
        NSString* assetsName = [FlutterDartProject flutterAssetsName:bundle];
        NSString* assetsPath = [bundle pathForResource:assetsName ofType:@""];

        if (assetsPath.length == 0) {
            assetsPath = [mainBundle pathForResource:assetsName ofType:@""];
        }

        if (assetsPath.length == 0) {
            NSLog(@"Failed to find assets path for "%@"", assetsName);
        } else {
            settings.assets_path = assetsPath.UTF8String;

            // Check if there is an application kernel snapshot in the assets directory we could
            // potentially use.  Looking for the snapshot makes sense only if we have a VM that can use
            // it.
            if (!flutter::DartVM::IsRunningPrecompiledCode()) {
                NSURL* applicationKernelSnapshotURL =
                    [NSURL URLWithString:@(kApplicationKernelSnapshotFileName)
 relativeToURL:[NSURL fileURLWithPath:assetsPath]];
                if ([[NSFileManager defaultManager] fileExistsAtPath:applicationKernelSnapshotURL.path]) {
                    settings.application_kernel_asset = applicationKernelSnapshotURL.path.UTF8String;
                } else {
                    NSLog(@"Failed to find snapshot: %@", applicationKernelSnapshotURL.path);
                }
            }
        }
    }

    // Domain network configuration
    // Disabled in https://github.com/flutter/flutter/issues/72723.
    // Re-enable in https://github.com/flutter/flutter/issues/54448.
    settings.may_insecurely_connect_to_all_domains = true;
    settings.domain_network_policy = "";

    // Whether to enable Impeller.
    NSNumber* nsEnableWideGamut = [mainBundle objectForInfoDictionaryKey:@"FLTEnableWideGamut"];
    // TODO(gaaclarke): Make this value `on` by default (pending memory audit).
    BOOL enableWideGamut = nsEnableWideGamut ? nsEnableWideGamut.boolValue : NO;
    settings.enable_wide_gamut = enableWideGamut;

    // Whether to enable Impeller.
    NSNumber* enableImpeller = [mainBundle objectForInfoDictionaryKey:@"FLTEnableImpeller"];
    // Change the default only if the option is present.
    if (enableImpeller != nil) {
        settings.enable_impeller = enableImpeller.boolValue;
    }

    NSNumber* enableTraceSystrace = [mainBundle objectForInfoDictionaryKey:@"FLTTraceSystrace"];
    // Change the default only if the option is present.
    if (enableTraceSystrace != nil) {
        settings.trace_systrace = enableTraceSystrace.boolValue;
    }

    NSNumber* enableDartProfiling = [mainBundle objectForInfoDictionaryKey:@"FLTEnableDartProfiling"];
    // Change the default only if the option is present.
    if (enableDartProfiling != nil) {
        settings.enable_dart_profiling = enableDartProfiling.boolValue;
    }

    // Leak Dart VM settings, set whether leave or clean up the VM after the last shell shuts down.
    NSNumber* leakDartVM = [mainBundle objectForInfoDictionaryKey:@"FLTLeakDartVM"];
    // It will change the default leak_vm value in settings only if the key exists.
    if (leakDartVM != nil) {
        settings.leak_vm = leakDartVM.boolValue;
    }

    #if FLUTTER_RUNTIME_MODE == FLUTTER_RUNTIME_MODE_DEBUG
    // There are no ownership concerns here as all mappings are owned by the
    // embedder and not the engine.
    auto make_mapping_callback = [](const uint8_t* mapping, size_t size) {
        return [mapping, size]() { return std::make_unique<fml::NonOwnedMapping>(mapping, size); };
    };

    settings.dart_library_sources_kernel =
        make_mapping_callback(kPlatformStrongDill, kPlatformStrongDillSize);
    #endif  // FLUTTER_RUNTIME_MODE == FLUTTER_RUNTIME_MODE_DEBUG

    // If we even support setting this e.g. from the command line or the plist,
    // we should let the user override it.
    // Otherwise, we want to set this to a value that will avoid having the OS
    // kill us. On most iOS devices, that happens somewhere near half
    // the available memory.
    // The VM expects this value to be in megabytes.
    if (settings.old_gen_heap_size <= 0) {
        settings.old_gen_heap_size = std::round([NSProcessInfo processInfo].physicalMemory * .48 /
                                        flutter::kMegaByteSizeInBytes);
    }

    // This is the formula Android uses.
    // https://android.googlesource.com/platform/frameworks/base/+/39ae5bac216757bc201490f4c7b8c0f63006c6cd/libs/hwui/renderthread/CacheManager.cpp#45
    CGFloat scale = [UIScreen mainScreen].scale;
    CGFloat screenWidth = [UIScreen mainScreen].bounds.size.width * scale;
    CGFloat screenHeight = [UIScreen mainScreen].bounds.size.height * scale;
    settings.resource_cache_max_bytes_threshold = screenWidth * screenHeight * 12 * 4;

    return settings;
}

```

##### Flutter 3.9.0 版才默认渲染引擎 skia or impeller ? 我可以修改关闭默认 skia 渲染模式吗? 以及其他默认配置调整吗？我们看看 flutter::Settings 默认设置的源码。

- 从源码读到发现 mainBundle key FLTEnableImpeller 是设置 impeller 渲染模式。截图默认设置 YES，所以我们也可以关闭 impeller 采用 skia 渲染引擎~

![iShot_2023-04-01_21.56.15.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9bc6ce2414174d4a836306ebacf0cbbb~tplv-k3u1fbpfcp-zoom-1.image)

- 1、Flutter 3.9.0 版本中发现，默认Flutter Engine中资源缓存所占用的最大内存大小其实是按屏幕大小动态调整的~很神奇吧~~
- 2、因此我们阅读源码去发现 Flutter 引擎一些默认配置~

#### b、初始化插件表 _pluginPublications 与 _registrars

- 1、_pluginPublications：每当一个插件在 FlutterEngine 中注册时， key 对应的 value 会被设置 [NSNull null]，并将一个 Flutter 插件注册到 registrar 中，发布到 Flutter 插件库中。
- 2、_registrars: 管理插件的注册表，存储的是 FlutterPluginRegistrar 插件实例对象。
- 3、源码可以查看注册插件的逻辑如下

```
- (void)publish:(NSObject*)value {
  _flutterEngine.pluginPublications[_pluginKey] = value;
}
...   
- (NSObject<FlutterPluginRegistrar>*)registrarForPlugin:(NSString*)pluginKey {
  NSAssert(self.pluginPublications[pluginKey] == nil, @"Duplicate plugin key: %@", pluginKey);
  self.pluginPublications[pluginKey] = [NSNull null];
  FlutterEngineRegistrar* result = [[FlutterEngineRegistrar alloc] initWithPlugin:pluginKey
                                                                    flutterEngine:self];
  self.registrars[pluginKey] = result;
  return [result autorelease];
}
```

- 例如注册 dart-native 插件 ，registry 是当前引擎 Engine 调用上述的 registrarForPlugin 注册 DartNativePlugin 插件。

```
+ (void)registerWithRegistry:(NSObject<FlutterPluginRegistry>*)registry {
  [DartNativePlugin registerWithRegistrar:[registry registrarForPlugin:@"DartNativePlugin"]];
}
```

#### c、recreatePlatformViewController

- 1、设置渲染 API 为 软件渲染模式
- 2、重新创建并配置与 Flutter 引擎相关的平台视图控制器 FlutterPlatformViewsController

```
- (void)recreatePlatformViewController {
  _renderingApi = flutter::GetRenderingAPIForProcess(FlutterView.forceSoftwareRendering);
  _platformViewsController.reset(new flutter::FlutterPlatformViewsController());
}
```

#### d、初始化 Flutter 与原生宿主通信对象 _binaryMessenger && _textureRegistry

- 1、FlutterBinaryMessengerRelay 是 Flutter 引擎内部通信的消息传递机制，它允许 Flutter 插件和宿主应用之间相互通信。
- 2、FlutterTextureRegistryRelay 则是负责管理 Flutter 引擎中所有纹理的注册表。Flutter 引擎中的纹理用于在 Flutter 中绘制图像，比如从原生平台传递到 Flutter 中的图像或视频等资源。
- 3、这两行代码是在初始化 FlutterEngine 对象时创建了这两个关键的对象，确保了后续 Flutter 插件和 Flutter 引擎之间的通信和纹理管理能够正常进行。
- 4、_connections.reset(new flutter::ConnectionCollection()) 初始化连接集合，以便在 Flutter Engine 中创建和管理通信通道。

```
_binaryMessenger = [[FlutterBinaryMessengerRelay alloc] initWithParent:self];
_textureRegistry = [[FlutterTextureRegistryRelay alloc] initWithParent:self];
_connections.reset(new flutter::ConnectionCollection());
```

##### Flutter 引擎实现原生与 Flutter 通信协议 FlutterBinaryMessenger

- 原生通过注册的 FlutterMethodChannel 的 _messenger （FlutterBinaryMessenger）对象给 Flutter 端传消息调用 sendOnChannel:message 方法

```
/// 给 Flutter 发消息
- (void)invokeMethod:(NSString*)method arguments:(id)arguments {
    FlutterMethodCall* methodCall = [FlutterMethodCall methodCallWithMethodName:method
                                                       arguments:arguments];
    NSData* message = [_codec encodeMethodCall:methodCall];
    [_messenger sendOnChannel:_name message:message];
}

/// 接受 Flutter 端的消息
- (void)setMethodCallHandler:(FlutterMethodCallHandler)handler {
    if (!handler) {
        if (_connection > 0) {
            [_messenger cleanUpConnection:_connection];
            _connection = 0;
        } else {
            [_messenger setMessageHandlerOnChannel:_name binaryMessageHandler:nil];
        }
        return;
    }
    // Make sure the block captures the codec, not self.
    // `self` might be released before the block, so the block needs to retain the codec to
    // make sure it is not released with `self`
    NSObject<FlutterMethodCodec>* codec = _codec;
    FlutterBinaryMessageHandler messageHandler = ^(NSData* message, FlutterBinaryReply callback) {
        FlutterMethodCall* call = [codec decodeMethodCall:message];
        handler(call, ^(id result) {
            if (result == FlutterMethodNotImplemented) {
                callback(nil);
            } else if ([result isKindOfClass:[FlutterError class]]) {
                callback([codec encodeErrorEnvelope:(FlutterError*)result]);
            } else {
                callback([codec encodeSuccessEnvelope:result]);
            }
        });
    };
    _connection = SetMessageHandler(_messenger, _name, messageHandler, _taskQueue);
}
```

#### f、注册 App 生命相关通知

```
  NSNotificationCenter* center = [NSNotificationCenter defaultCenter];
  [center addObserver:self
             selector:@selector(onMemoryWarning:)
                 name:UIApplicationDidReceiveMemoryWarningNotification
               object:nil];

  [center addObserver:self
             selector:@selector(applicationWillEnterForeground:)
                 name:UIApplicationWillEnterForegroundNotification
               object:nil];

  [center addObserver:self
             selector:@selector(applicationDidEnterBackground:)
                 name:UIApplicationDidEnterBackgroundNotification
               object:nil];

  [center addObserver:self
             selector:@selector(onLocaleUpdated:)
                 name:NSCurrentLocaleDidChangeNotification
               object:nil];
```

### 3、runWithEntrypoint:

- 1、这段代码实现了创建 Flutter Shell 的功能。Shell 是 Flutter 的核心部分之一，用于启动 Dart 代码和 Flutter 应用程序，从而展示 Flutter UI。Shell 是一个 C++ 应用程序，它与 Dart 代码之间使用 Embedder API 进行通信。在 iOS 平台上，Flutter 的 Shell 是由 Objective-C 代码创建的，与宿主应用程序交互。FlutterEngine 类是与 Shell 交互的主要接口。

- 2、Shell 的作用？

    -   a、Shell 是 Flutter 的底层框架，它提供了一个跨平台的渲染引擎、视图系统和一套基础库。Shell 是构建 Flutter 应用程序的基础，它将应用程序逻辑和 Flutter 引擎的交互封装在一起。
    -   b、Flutter 应用程序通常包含一个或多个 Shell，每个 Shell 包含一个渲染线程和一个 Dart 执行上下文。Shell 接收来自 Dart 代码的指令，并将其翻译成图形、动画和其他视觉效果，以及响应用户的输入事件。Shell 还提供了对 Flutter 引擎的访问，使开发者可以配置引擎和访问它的功能。

```
/// 创建 Flutter Shell 并启动引擎
- (BOOL)runWithEntrypoint:(NSString*)entrypoint
               libraryURI:(NSString*)libraryURI
             initialRoute:(NSString*)initialRoute
           entrypointArgs:(NSArray<NSString*>*)entrypointArgs {
  if ([self createShell:entrypoint libraryURI:libraryURI initialRoute:initialRoute]) {
    [self launchEngine:entrypoint libraryURI:libraryURI entrypointArgs:entrypointArgs];
  }
  return _shell != nullptr;
}
```

#### a、createShell

- 这段代码实现了创建 Flutter Shell 的功能。Shell 是 Flutter 的核心部分之一，用于启动 Dart 代码和 Flutter 应用程序，从而展示 Flutter UI。Shell 是一个 C++ 应用程序，它与 Dart 代码之间使用 Embedder API 进行通信。在 iOS 平台上，Flutter 的 Shell 是由 Objective-C 代码创建的，与宿主应用程序交互。FlutterEngine 类是与 Shell 交互的主要接口。
-   具体步骤如下：

 - 1、首先，该函数检查 Shell 是否已经创建。如果已经创建，函数直接返回。
 - 2、接下来，该函数将传入的 entrypoint、libraryURI 和 initialRoute 参数保存到对象属性中。entrypoint 表示 Dart 代码的入口点，libraryURI 表示应用程序的 Dart 库文件路径，initialRoute 表示应用程序的初始路由。如果 initialRoute 参数为空，该函数尝试从 Dart 配置中获取默认的路由。
 - 3、然后，该函数设置了 FlutterView.forceSoftwareRendering 属性。该属性用于控制是否启用软件渲染，根据 Dart 配置中的 enable_software_rendering 参数来决定是否启用。
 - 4、接着，该函数创建了一个 Flutter 的 PlatformData 对象。PlatformData 包含了许多平台相关的信息，如窗口大小、文本方向、缩放比例等。在 iOS 平台上，PlatformData 包含了平台视图的大小和位置信息。
 - 5、然后，该函数设置了入口点信息，并生成一个线程标签用于标识该 FlutterEngine 的线程。接着，函数创建了一个 ThreadHost 对象，该对象表示线程的主机，并使用 makeThreadHost 函数创建了三个线程：UI 线程、IO 线程和 Raster 线程。这些线程将在后续的 Shell 创建过程中使用。
 - 6、然后，该函数创建了两个回调函数 on_create_platform_view 和 on_create_rasterizer。on_create_platform_view 回调函数用于创建平台视图，而 on_create_rasterizer 回调函数用于创建光栅化器。这些回调函数将在后续的 Shell 创建过程中使用。
 - 7、接着，该函数创建了一个 TaskRunners 对象，该对象封装了线程标签、平台任务运行器、光栅化任务运行器、UI 任务运行器和 IO 任务运行器。这些任务运行器将在后续的 Shell 创建过程中使用。
 - 8、接下来，该函数检查应用程序是否处于后台状态。如果应用程序处于后台状态，则禁用 GPU，否则启用 GPU。禁用 GPU 可以减少电量消耗和热量，从而延长电池寿命。
 - 9、然后，该函数调用 Shell::Create 函数创建 Shell。Shell::Create 函数是一个同步函数，它会阻塞当前线程，直到 Shell 创建完成
 - 10、最后，调用 setupShell 设置 Shell , 并且判断如果是 isProfilerEnabled 模式下，开启性能分析

```
// 创建 Flutter Shell
- (BOOL)createShell:(NSString*)entrypoint
         libraryURI:(NSString*)libraryURI
       initialRoute:(NSString*)initialRoute {
  if (_shell != nullptr) {
    FML_LOG(WARNING) << "This FlutterEngine was already invoked.";
    return NO;
  }

  self.initialRoute = initialRoute;

  auto settings = [_dartProject.get() settings];
  if (initialRoute != nil) {
    self.initialRoute = initialRoute;
  } else if (settings.route.empty() == false) {
    self.initialRoute = [NSString stringWithCString:settings.route.c_str()
                                           encoding:NSUTF8StringEncoding];
  }

  FlutterView.forceSoftwareRendering = settings.enable_software_rendering;

  auto platformData = [_dartProject.get() defaultPlatformData];

  SetEntryPoint(&settings, entrypoint, libraryURI);

  NSString* threadLabel = [FlutterEngine generateThreadLabel:_labelPrefix];
  _threadHost = std::make_shared<flutter::ThreadHost>();
  *_threadHost = [FlutterEngine makeThreadHost:threadLabel];

  // Lambda captures by pointers to ObjC objects are fine here because the
  // create call is synchronous.
  flutter::Shell::CreateCallback<flutter::PlatformView> on_create_platform_view =
      [self](flutter::Shell& shell) {
        [self recreatePlatformViewController];
        return std::make_unique<flutter::PlatformViewIOS>(
            shell, self->_renderingApi, self->_platformViewsController, shell.GetTaskRunners());
      };

  flutter::Shell::CreateCallback<flutter::Rasterizer> on_create_rasterizer =
      [](flutter::Shell& shell) { return std::make_unique<flutter::Rasterizer>(shell); };

  flutter::TaskRunners task_runners(threadLabel.UTF8String,                          // label
                                    fml::MessageLoop::GetCurrent().GetTaskRunner(),  // platform
                                    _threadHost->raster_thread->GetTaskRunner(),     // raster
                                    _threadHost->ui_thread->GetTaskRunner(),         // ui
                                    _threadHost->io_thread->GetTaskRunner()          // io
  );

  _isGpuDisabled =
      [UIApplication sharedApplication].applicationState == UIApplicationStateBackground;
  // Create the shell. This is a blocking operation.
  std::unique_ptr<flutter::Shell> shell = flutter::Shell::Create(
      /*platform_data=*/platformData,
      /*task_runners=*/task_runners,
      /*settings=*/settings,
      /*on_create_platform_view=*/on_create_platform_view,
      /*on_create_rasterizer=*/on_create_rasterizer,
      /*is_gpu_disabled=*/_isGpuDisabled);

  if (shell == nullptr) {
    FML_LOG(ERROR) << "Could not start a shell FlutterEngine with entrypoint: "
                   << entrypoint.UTF8String;
  } else {
    [self setupShell:std::move(shell)
        withVMServicePublication:settings.enable_vm_service_publication];
    if ([FlutterEngine isProfilerEnabled]) {
      [self startProfiler];
    }
  }

  return _shell != nullptr;
}

```

#### b、launchEngine

-   配置启动 Dart 应用程序引擎的一些列配置包括

    -  1、代码库路径
    -  2、入口点（entrypoint）
    -  3、包含可执行文件的路径
    -  4、环境变量
    -  5、启动参数等...

```
// 启动引擎
- (void)launchEngine:(NSString*)entrypoint
libraryURI:(NSString*)libraryOrNil
entrypointArgs:(NSArray<NSString*>*)entrypointArgs {
  // Launch the Dart application with the inferred run configuration.
  self.shell.RunEngine([_dartProject.get() runConfigurationForEntrypoint:entrypoint
                                        libraryOrNil:libraryOrNil
                                        entrypointArgs:entrypointArgs]);
}

- (flutter::RunConfiguration)runConfigurationForEntrypoint:(nullable NSString*)entrypointOrNil
libraryOrNil:(nullable NSString*)dartLibraryOrNil
entrypointArgs:
(nullable NSArray<NSString*>*)entrypointArgs {
    auto config = flutter::RunConfiguration::InferFromSettings(_settings);
    if (dartLibraryOrNil && entrypointOrNil) {
        config.SetEntrypointAndLibrary(std::string([entrypointOrNil UTF8String]),
                               std::string([dartLibraryOrNil UTF8String]));

    } else if (entrypointOrNil) {
        config.SetEntrypoint(std::string([entrypointOrNil UTF8String]));
    }

    if (entrypointArgs.count) {
        std::vector<std::string> cppEntrypointArgs;
        for (NSString* arg in entrypointArgs) {
            cppEntrypointArgs.push_back(std::string([arg UTF8String]));
        }
        config.SetEntrypointArgs(std::move(cppEntrypointArgs));
    }

    return config;
}
```

### 4、spawnWithEntrypoint

-   📢 当引擎组里 self.engines.count > 0 的时候，则通过下面源码创建一个引擎，同时它会共享第一个引擎的部分资源~
-   该方法用于创建一个新的 FlutterEngine 实例，该实例包含一个 FlutterShell 实例和一个 FlutterDartProject 实例，并且会通过调用 FlutterShell 实例的 Spawn 方法来生成一个新的 Shell 实例，这个新的 Shell 实例可以用于控制新的 Flutter 引擎实例。
-   具体步骤如下：

 - 1.根据 _dartProject 获取运行配置 configuration。
 - 2.从当前 FlutterShell 实例 _shell 中获取平台视图，并通过 on_create_platform_view 方法创建一个新的 PlatformViewIOS 实例，同时通过 on_create_rasterizer 方法创建一个新的 Rasterizer 实例。
 - 3.调用 _shell 的 Spawn 方法创建一个新的 Shell 实例，并将步骤2中创建的 PlatformViewIOS 和 Rasterizer 实例传递给 Spawn 方法。
 - 4.创建一个新的 FlutterEngine 实例 result，将 *threadHost、* profiler、_profiler_metrics 和 _isGpuDisabled 属性复制到 result 中。
 - 5.调用 result 的 setupShell:withVMServicePublication: 方法，将步骤3中创建的 Shell 实例作为参数传递给 setupShell 方法，同时传递一个 NO 值，表示不启用 VM service publication。
 - 6.返回 result。

总之，这段代码是用于创建一个新的 Flutter 引擎实例的，并通过调用 FlutterShell 实例的 Spawn 方法来生成一个新的 Shell 实例，这个新的 Shell 实例可以用于控制新的 Flutter 引擎实例。

```
- (FlutterEngine*)spawnWithEntrypoint:(/*nullable*/ NSString*)entrypoint
                           libraryURI:(/*nullable*/ NSString*)libraryURI
                         initialRoute:(/*nullable*/ NSString*)initialRoute
                       entrypointArgs:(/*nullable*/ NSArray<NSString*>*)entrypointArgs {
  NSAssert(_shell, @"Spawning from an engine without a shell (possibly not run).");
  FlutterEngine* result = [[FlutterEngine alloc] initWithName:_labelPrefix
                                                      project:_dartProject.get()
                                       allowHeadlessExecution:_allowHeadlessExecution];
  flutter::RunConfiguration configuration =
      [_dartProject.get() runConfigurationForEntrypoint:entrypoint
                                           libraryOrNil:libraryURI
                                         entrypointArgs:entrypointArgs];

  fml::WeakPtr<flutter::PlatformView> platform_view = _shell->GetPlatformView();
  FML_DCHECK(platform_view);
  // Static-cast safe since this class always creates PlatformViewIOS instances.
  flutter::PlatformViewIOS* ios_platform_view =
      static_cast<flutter::PlatformViewIOS*>(platform_view.get());
  std::shared_ptr<flutter::IOSContext> context = ios_platform_view->GetIosContext();
  FML_DCHECK(context);

  // Lambda captures by pointers to ObjC objects are fine here because the
  // create call is synchronous.
  flutter::Shell::CreateCallback<flutter::PlatformView> on_create_platform_view =
      [result, context](flutter::Shell& shell) {
        [result recreatePlatformViewController];
        return std::make_unique<flutter::PlatformViewIOS>(
            shell, context, result->_platformViewsController, shell.GetTaskRunners());
      };

  flutter::Shell::CreateCallback<flutter::Rasterizer> on_create_rasterizer =
      [](flutter::Shell& shell) { return std::make_unique<flutter::Rasterizer>(shell); };

  std::string cppInitialRoute;
  if (initialRoute) {
    cppInitialRoute = [initialRoute UTF8String];
  }

  std::unique_ptr<flutter::Shell> shell = _shell->Spawn(
      std::move(configuration), cppInitialRoute, on_create_platform_view, on_create_rasterizer);

  result->_threadHost = _threadHost;
  result->_profiler = _profiler;
  result->_profiler_metrics = _profiler_metrics;
  result->_isGpuDisabled = _isGpuDisabled;
  [result setupShell:std::move(shell) withVMServicePublication:NO];
  return [result autorelease];
}
```

-   通过 spawnWithEntrypoint 源码我们知道了采用 Group 管理多引擎，为什么能大大降低内存原因：因为多个引擎的公用的是同一个 _shell 部分资源~



## 题外话

-   Flutter 网络请求是在哪个线程中执行的吗？会阻塞引发 App 卡顿吗？

## ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/839563def4764ac28b54e828c3f0ffb1~tplv-k3u1fbpfcp-zoom-1.image)

## 参考

-   【官方文档】[在 iOS 应用中添加 Flutter 页面](https://flutter.cn/docs/development/add-to-app/ios/add-flutter-screen?tab=engine-uikit-objc-tab)
-   【官方文档】[FlutterEngine](<https://api.flutter.dev/objcdoc/Classes/FlutterEngine.html>)
