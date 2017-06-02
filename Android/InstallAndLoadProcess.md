#插件安装和加载流程
基于插件框架V1.2.4.0带升级
##安装流程
###初始化插件框架
使用插件之前，必然是先要初始化插件框架，让我们先看一下初始化方法：

```
public void init(@NonNull Context hostAppContext, UpgradeInterface upgradeImpl, WidgetManager.HostVersionProvider versionProvider) {
        ...
        //省略一些初始化和回调的设置
        ...
        this.initAndInstallAllPlugins(hostAppContext, (WidgetManager.HostVersionProvider)versionProvider);
    }
```

所有重载的`init`方法最终都是调用到这里，方法中其他的赋值并不是重点，主要的方法就是

```
initAndInstallAllPlugins(hostAppContext, (WidgetManager.HostVersionProvider)versionProvider)
```

接着我们看一下这个方法

```
private void initAndInstallAllPlugins(@NonNull Context hostAppContext, WidgetManager.HostVersionProvider versionProvider) {
        mLogger.i("Init plugins, plugin framework\'s version is: 1.2.4.0");
        if(this.shouldInitFramework()) {
            this.initFramework(hostAppContext);
            int lastHostVersion = SharedPreferencesManager.getInt("KEY_CURRENT_HOSTAPP_VERSION_CODE", -1);
            int currentHostVersion = versionProvider.getHostVersion();
            if(lastHostVersion != -1 && currentHostVersion == lastHostVersion) {
                this.installUpgradedWidgets();
            } else {
                this.mCurrentHostAppVersionCode = currentHostVersion;
                this.installAssetsWidgets();
            }

        }
    }
```
首先，判断是否应该初始化。如果`WidgetManagerState`的状态不是`INVALID_STATE`，则不需要再次执行初始化，并且如果已经是`INITIALIZED`状态，则通知外部插件框架已经准备完毕。代码如下：

```
private boolean shouldInitFramework() {
        if(this.mState != WidgetManager.WidgetManagerState.INVALID_STATE) {
            mLogger.e("WidgetsManager Duplicate initialization!!!");
            if(this.mState == WidgetManager.WidgetManagerState.INITIALIZED) {
                this.notifyWidgetManagerPrepared();
            }

            return false;
        } else {
            return true;
        }
    }
```

接着初始化插件框架`initFramework(hostAppContext)`

```
private void initFramework(Context hostContext) {
        this.mState = WidgetManager.WidgetManagerState.INITIALIZING;
        this.notifyFrameworkListeners(new FrameworkEvent(1));
        ContextProvider.init(hostContext);
        SharedPreferencesManager.createInstance(hostContext);
        this.mHostAppContext = hostContext;
        this.sActivityThread = ActivityThread.currentActivityThread();
        this.mWidgetInstaller = new WidgetInstaller(hostContext);
        this.mInstalledWidgets.clear();
    }
```

1. 首先设置状态为初始化中，并通知所有listener
2. 为`ContextProvider`设置宿主的context
3. 为宿主创建一个名为letv_plugin_info的SharedPreference
4. 赋值`mHostAppContext`，`sActivityThread`
5. 创建`WidgetInstaller`
6. 清空已安装的插件

让我们在回到`initAndInstallAllPlugins`方法，当框架的初始化执行完毕之后，我们需要比对宿主应用当前的版本号和我们之前存储在`SharedPreference`里面的是否一致，如果一致，则执行`installUpgradedWidgets`，否则执行`installAssetsWidgets`。


###安装插件
框架初始化完成，接着就要安装插件了，从上面我们可以看到，如果版本更新了，则会安装宿主应用自带的最新插件，否则则安装本地插件。两个加载过程都是异步的。

####installAssetsWidgets
安装宿主Assets中自带的插件版本，首先会创建一个新的`Thread`，执行如下代码  

```
WidgetManager.this.mInstalledWidgets = WidgetManager.this.mWidgetInstaller.installAssetsWidgets(WidgetManager.this.mWidgetsToLoad);  

SharedPreferencesManager.putInt("KEY_CURRENT_HOSTAPP_VERSION_CODE", WidgetManager.this.mCurrentHostAppVersionCode);  

WidgetManager.this.afterWidgetInstalled();
```

在上面我们初始化了一个`WidgetInstaller`，现在调用它的`installAssetsWidgets`方法。

```
List<Widget> installAssetsWidgets(List<Integer> widgetsToLoad) {
        this.mLogger.i("installAssetsWidgets start");
        this.ensureInstallFolder();
        this.copyAssetsWidgets();
        List installedWidgets = this.loadInstalledWidgets(widgetsToLoad);
        this.mLogger.i("installAssetsWidgets end");
        return installedWidgets;
    }
```

`ensureInstallFolder`最终方法会在/data/data/packagename/files/目录下创建一个plugins文件夹  
`copyAssetsWidgets`方法遍历宿主Assets目录中plugins文件夹下的所有插件，

```
if(matcher.match(fileName)) {
   InputStream is = null;

   try {
        is = assetManager.open("plugins" + File.separator + fileName);
        String e1 = this.suffixFilenameWithTimeStamp(fileName);

        try {
             String e2 = this.getWidgetInstallDir() + e1;
             File installedFile = new File(e2);
             this.backupWidgetIfExist(installedFile);
             this.copy(is, installedFile);
             this.mLogger.d("Copy plugin in asset: " + e1);
             filesCopied.add(e2);
         } catch (IOException var18) {
             var18.printStackTrace();
         }
   } catch (IOException var19) {
       var19.printStackTrace();
   } finally {
        IOUtils.closeStream(is);
   }
}
```

如果当前要安装的插件已经存在了，`backupWidgetIfExist`方法会将其备份，然后将待安装的插件拷贝到指定的目录。完成了插件的拷贝，最后执行到`loadInstalledWidgets(widgetsToLoad)`方法。  
在该方法中，会验证之前拷贝的文件是否是apk，如果不是则删除，如果是，就会通过`WidgetParser`来为其创建一个`Widget`。

```
Widget widget = this.widgetParser.parseWidget(filePath);
if(widget == null) {
   this.mLogger.e("Widget file parse failure, will delete path:" + filePath);
   IOUtils.deleteFileOrDir(childFile);
} else if(widgetsToLoad == null || widgetsToLoad.contains(Integer.valueOf(widget.getWidgetId()))) {
  ((WidgetImpl)widget).setState(5);//INSTALLED
  this.createOutputDir(widget);
  this.installSoLibInWidgetApk(widget);
  installedWidgets.add(widget);
}
```
Widget创建成功的话，最终会将插件的状态设置为已安装。`createOutputDir`会创建插件的Dex和lib的目录(/data/data/packagename/files/plugins_data/widgetId/)。还会拷贝.so的文件，最终`installedWidgets`会返回给`WidgetManager`中的`mInstalledWidgets`。

让我们看一下`parseWidget(filePath)`方法，

```
Widget parseWidget(@Nullable String widgetPath) {
        WidgetImpl widget = this.parsePackageInfo(widgetPath);
        if(widget == null && !TextUtils.isEmpty(widgetPath)) {
            IOUtils.deleteFileOrDir(new File(widgetPath));
        }

        return widget;
    }
```
这里主要是通过`parsePackageInfo(widgetPath)`获取一个插件的实例。那么接下来我们看一下这个方法。代码比较长，只列出部分。

```
PackageParser parser = PackageParser.getPackageParser();
Package p = parser.parsePackage(filePath, this.hostContext);
PackageInfo widgetPackageInfo = p.packageInfo;
```
首先获取`PackageParser`，这里会根据系统的版本做处理。

```
if(VERSION.SDK_INT >= 21) {
   parser = new PackageParser(new ParsePackageMethodApi21AndAbove(sPackageParserClass), new GeneratePackageInfoMethodApi17AndAbove(sPackageParserClass));
} else if(VERSION.SDK_INT >= 17) {
   parser = new PackageParser(new ParsePackageMethodApi20AndBelow(sPackageParserClass), new GeneratePackageInfoMethodApi17AndAbove(sPackageParserClass));
} else if(VERSION.SDK_INT == 16) {
   parser = new PackageParser(new ParsePackageMethodApi20AndBelow(sPackageParserClass), new GeneratePackageInfoMethodApi16(sPackageParserClass));
} else {
   parser = new PackageParser(new ParsePackageMethodApi20AndBelow(sPackageParserClass), new GeneratePackageInfoMethodApi15AndBelow(sPackageParserClass));
}
```
然后通过`PackageParser`去解析当前插件的`PackageInfo`和`Activity`的`IntentFilters`生成一个Package对象。接下来就是创建一个`WidgetImpl`对象，并利用我们获取到的数据为其属性赋值。

让我们回到`installAssetsWidgets`方法，在获取到了已安装插件列表后，还需要做一些最后的处理`afterWidgetInstalled`。

```
private void afterWidgetInstalled() {
        this.verifyInitialLoadedWidgets();
        UpgradeProvider.getUpgrader().onUpgradeCompleted();
        this.verifyInstalledPluginsFileExecutable();
        this.initFrameworkLastStep();
        mLogger.i("init plugin framework has done");
}
```

`verifyInitialLoadedWidgets`方法中会检查当前所有已安装的插件是否有冲突(widgetId相同)，冲突的插件只保留最新版本，并删除旧版本。  
`verifyInstalledPluginsFileExecutable`方法会验证插件是否是损坏的(Dex文件)。  

最后执行`initFrameworkLastStep`方法，调用了`HookInstaller`的`installHook`方法创建了`PackageManager`和`Toast`的动态代理。

```
private void initFrameworkLastStep() {
        try {
            HookInstaller.getInstance().installHook(this.mHostAppContext, (ClassLoader)null);
            HookInstaller.getInstance().setHookEnable(true);
        } catch (Throwable var2) {
            var2.printStackTrace();
        }

        PendingPluginActivityManager.getInstance().startPendingActivities(new StartPendingActivityListener() {
            public void onAllActivitiesStarted() {
                WidgetManager.this.notifyWidgetManagerPrepared();
            }
        });
    }
```

至此，插件框架的初始化完成，插件也安装完毕。并且通知外界WidgetManager的状态为`INITIALIZED`。

####installUpgradedWidgets
```
WidgetManager.this.mInstalledWidgets = WidgetManager.this.mWidgetInstaller.installUpgradeWidgets(WidgetManager.this.mWidgetsToLoad);
WidgetManager.this.afterWidgetInstalled();
```

仍然是调用之前初始化的`WidgetInstaller`的方法，`installUpgradeWidgets`。

```
List<Widget> installUpgradeWidgets(List<Integer> widgetsToLoad) {
        this.mLogger.i("installUpgradeWidgets start");
        this.copyUpgradedWidgets();
        List installedWidgets = this.loadInstalledWidgets(widgetsToLoad);
        this.mLogger.i("installUpgradeWidgets end");
        return installedWidgets;
    }
```
首先`copyUpgradedWidgets`方法，将下载下来的升级文件全部拷贝到安装目录，

```
private void copyUpgradedWidgets() {
        List downloadedPackagesPath = UpgradeProvider.getUpgrader().getDownloadedFiles();
        if(downloadedPackagesPath != null) {
            this.mLogger.d("Plugin\'s upgrade apks which are already downloaded, to widgetInstall: " + downloadedPackagesPath);
            Iterator var2 = downloadedPackagesPath.iterator();

            while(var2.hasNext()) {
                String pkgFilePath = (String)var2.next();
                this.copyToInstallDir(pkgFilePath);
            }

        }
    }
```

`copyToInstallDir`和`copyAssetsWidgets`有些类似，都是备份了原来的插件，然后拷贝文件。然后又来到了`loadInstalledWidgets`方法就和前面的分析一致了。

##加载插件
我们在自己的项目监听FrameworkEvent

```
if (event.getEventType() == FrameworkEvent.EVENT_FRAMEWORK_INITIALIZED) {
    if (WidgetManager.getInstance().getWidget(getWidgetId()) == null) {
        downloadPlugin();
    } else if (!WidgetManager.getInstance().getWidget(getWidgetId()).isLoaded()) {
        loadWidget();
    }
}
```
前面已经说了，插件框架初始化完成后，会通知外界，那么就来到了我们这段代码。如果这时我们获取不到插件，则需要下载，如果有，因为我们在之前安装插件的过程已经把插件的状态设置为了`INSTALLED`，则插件没有处于加载状态，需要加载插件。

###下载插件
最终下载插件我们调用了 
 
```
WidgetManager.getInstance().installWidgetsFromNetwork(插件name的list, 
this, 插件id的list);
```
this：当前class实现了`WidgetDownloadListener`。

```
public void installWidgetsFromNetwork(List<String> widgetsName, Object downloadListener, List<String> upgradeTasksId) {
        int size = widgetsName.size();
        ArrayList upgradeEntities = new ArrayList(size);

        for(int i = 0; i < size; ++i) {
            String widgetName = (String)widgetsName.get(i);
            if(this.isWidgetInstalled(widgetName)) {
                mLogger.i("Plugin \'" + widgetName + "\' already installed, do not download it");
            } else {
                String upgradeId = (String)upgradeTasksId.get(i);
                upgradeEntities.add(new UpgradeEntity(widgetName, "0", upgradeId));
                this.mWidgetUpgradeTaskRecord.addTask(upgradeId, -1);
            }
        }

        if(!upgradeEntities.isEmpty()) {
            UpgradeProvider.getUpgrader().checkUpgradeAndDownload(upgradeEntities, downloadListener);
        }

    }
```
接下来，我们获取到所有需要下载的插件列表，然后交给升级模块去处理。

`UpgradeInterfaceImpl`是`UpgradeInterface`的实现类，我们实际调用的就是这个类的方法。这个方法内没有太多内容，最终是调用了`WidgetUpgrade`的`checkAndDownload`方法。


```
public void checkAndDownload(List<UpgradeRequestEntity> upgradeRequestEntities, final WidgetDownloadListener widgetDownloadListener) {
        String externalApkPath = ContextProvider.getApplicationContext().getExternalFilesDir("apk") + File.separator + "upgrade_";
        for (final UpgradeRequestEntity upgradeRequestEntity : upgradeRequestEntities) {
            final String taskId = upgradeRequestEntity.getId();
            final Upgrade upgrade = new Upgrade(taskId, mDeviceParameters, mUpgradeDomain, externalApkPath + taskId + ".papk");
            mUpgradeMap.put(taskId, upgrade);
            upgrade.check(upgradeRequestEntity.getApplicationName(), upgradeRequestEntity.getVersionCode(), new CheckingUpgradeCallback() {
                @Override
                public void onUpgradeResult(int upgradeStatus, @Nullable UpgradeInfo upgradeInfo) {
                    // 如果是推荐或强制升级则直接下载升级包
                    if (upgradeStatus == Upgrade.UPGRADE_STATUS_FORCE_UPGRADE
                            || upgradeStatus == Upgrade.UPGRADE_STATUS_RECOMMEND_UPGRADE) {
                        WidgetUpgrade.this.mLogger.i("start download widget" + upgradeRequestEntity.getId());
                        upgrade.download(new DownloadListener() {
                            @Override
                            public void onProgressChanged(long downloadedSize, long totalSize) {
                                widgetDownloadListener.onProgressChanged(taskId, downloadedSize, totalSize);

                            }

                            @Override
                            public void onDownloadSuccess(UpgradeInfo upgradeInfo) {
                                widgetDownloadListener.onDownloadSuccess(taskId, upgradeInfo);
                            }

                            @Override
                            public void onDownloadError(int errorCode, UpgradeInfo upgradeInfo) {
                                widgetDownloadListener.onDownloadError(taskId, errorCode, upgradeInfo);
                            }

                            @Override
                            public void onURLChanged(String url) {
                                widgetDownloadListener.onURLChanged(taskId, url);
                            }
                        });
                    }
                }
            });
        }
    }
```
其实这个方法中没有什么内容，就是开启下载，然后调用回调。最终插件下载成功，我们也会通过之前实现的listener得到回调。

###加载插件
在我们自己的项目中调用如下方法加载指定插件。
```
WidgetManager.getInstance().widgetLoad(
		WidgetManager.getInstance().getWidget(getWidgetId()), this);
```

this：当前class实现了`WidgetEventListener`，插件事件的回调。

实际方法中核心的部分如下，我们创建了一个`WidgetLoader`来异步加载插件。

```
WidgetLoader loader = new WidgetLoader((WidgetImpl)widget);
            loader.doAsyncLoad(this.mHostAppContext, new WidgetLoadListener() {
                public void onLoadSuccess(WidgetImpl widget, WidgetContext widgetContext) {
                    WidgetImpl.notifyWidgetEvent(listener, 22, widget);
                }

                public void onLoadFailure(WidgetImpl widget, int errorReason) {
                    WidgetImpl.notifyWidgetEvent(listener, 21, errorReason, widget);
                }
            });
```
在`doAsyncLoad`方法中会先判断如果不是在主线程触发的加载，则会直接通知加载失败，前面提到的插件框架`onFrameworkEvent`的回调可能是在异步线程。最终会执行如下方法。

```
(new WidgetLoader.LoadAsyncTask(this.mWidget, hostContext, 
loadListener)).executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR, new Void[0])
```

`LoadAsyncTask`是一个`AsyncTask`，此时将插件的Sate设置为LOADING。实际负责执行插件加载的方法是

```
private WidgetContext loadWidget(WidgetImpl widget, Context hostContext) {
        WidgetContext widgetContext = WidgetContextMaker.makeWidgetContext(widget, hostContext);
        if(widgetContext == null) {
            mLogger.i("makeWidgetContext retrun null!!! on [loadWidget]");
            return null;
        } else {
            this.parseWidgetManifest(widget, widgetContext);
            return widgetContext;
        }
    }
```
首先我们需要先创建出`WidgetContext`，然后去解析插件的Manifest。

####创建插件Context
```
Context clonedAppContext = null;
try {
     clonedAppContext = hostAppContext.createPackageContext(hostAppContext.getApplicationInfo().packageName, 3);
} catch (NameNotFoundException var9) {
     var9.printStackTrace();
     return null;
}
```

`createPackageContext`就是`Context`的方法了，会为给定的packagename(也就是我们的宿主应用的包名)创建一个`Context`对象。每次调用此方法都会返回新的`Context`对象，但是会共享Resources和ClassLoader。

然后接下来我们要创建`WidgetClassLoader`。

```
ClassLoaderMaker.make(srcLocation, widget.getDexDir(), widget.getLibDir(), 
hostAppContext.getClassLoader(), widget.getDependentPackages());
```

这个方法会先确保我们传过去的dex文件路径是一个文件夹并且可读可写。然后new了一个`WidgetClassLoader`。

拿到了`WidgetClassLoader`，接下来就是为插件设置`Resources`和`LoadedApk`。

```
Resources superResource = hostAppContext.getResources();
LoadedApk widgetLoadedApk = createPluginLoadedApk(widget, clonedAppContext, superResource, cl);
if(widgetLoadedApk == null) {
   mLogger.w("create plugin LoadedApk fail when makeWidgetContext");
   return null;
} else {
   setFieldOfLoadedApk("mResources", (Object)null, widgetLoadedApk);
   Resources resource = widgetLoadedApk.getResources(WidgetManager.getInstance().currentActivityThread());
   if(!modifyApplicationContext(clonedAppContext, resource, widgetLoadedApk)) {
      mLogger.w("Modify clonedAppContext fail when makeWidgetContext");
      return null;
   } else {
     return new WidgetContextWrapper(widget.getWidgetId(), clonedAppContext);
   }
}
```

先获取宿主应用的`Resources`，用于创建给插件使用的`LoadedApk`。让我们看一下创建的过程。

```
private static LoadedApk createPluginLoadedApk(Widget widget, Context context, Resources res, DexClassLoader cl) {
        Class loadedApkClass = null;

        try {
            loadedApkClass = Class.forName("android.app.LoadedApk");
        } catch (ClassNotFoundException var13) {
            var13.printStackTrace();
        }

        LoadedApk loadedWidgetApk = createLoadedApkBeforeAndroid5(widget, context, res, loadedApkClass);
        if(loadedWidgetApk == null) {
            loadedWidgetApk = createLoadedApkAndroid5(widget, context, res, cl, loadedApkClass);
            if(loadedWidgetApk == null) {
                loadedWidgetApk = createLoadedApkHisense(widget, context, res, cl, loadedApkClass);
            }

            if(loadedWidgetApk == null) {
                return null;
            }
        }

        setFieldOfLoadedApk("mClassLoader", cl, loadedWidgetApk);
        setFieldOfLoadedApk("mAppDir", widget.getLocation(), loadedWidgetApk);
        setFieldOfLoadedApk("mResDir", widget.getLocation(), loadedWidgetApk);
        setFieldOfLoadedApk("mDataDir", widget.getDataDir(), loadedWidgetApk);
        File dataDirFile = new File(widget.getDataDir());
        setFieldOfLoadedApk("mDataDirFile", dataDirFile, loadedWidgetApk);
        if(VERSION.SDK_INT >= 24) {
            setFieldOfLoadedApk("mDeviceProtectedDataDirFile", dataDirFile, loadedWidgetApk);
            setFieldOfLoadedApk("mCredentialProtectedDataDirFile", dataDirFile, loadedWidgetApk);
        }

        try {
            Class e = Class.forName("android.app.ContextImpl");
            Field packageInfoField = e.getDeclaredField("mPackageInfo");
            packageInfoField.setAccessible(true);
            Object loadedApkInfoToCopyFrom = packageInfoField.get(context);
            replaceOtherContentInLoadedApk(loadedApkClass, loadedApkInfoToCopyFrom, loadedWidgetApk);
        } catch (ClassNotFoundException var10) {
            mLogger.e("Error when replace other fields of LoadedApk, exception: " + var10);
        } catch (NoSuchFieldException var11) {
            mLogger.e("Error when replace other fields of LoadedApk, exception: " + var11);
        } catch (IllegalAccessException var12) {
            mLogger.e("Error when replace other fields of LoadedApk, exception: " + var12);
        }

        return loadedWidgetApk;
    }
```
首先通过反射创建`LoadedApk`的一个实例，涉及的方法都是构造函数的参数有区别，但是最终拿到`LoadedApk`之后的处理是一样的，利用反射，将一系列的资源都替换成插件的，字段太多，可以自行查看代码。

接着我们先将`LoadedApk`的mResources通过反射置为null，然后在调用`getResources(ActivityThread mainThread)`方法，让其使用我们插件的资源生成新的`Resources`。

最后将获得的resource和widgetLoadedApk都通过反射的方式赋给`clonedAppContext`，这样`WidgetContext`就算创建完成。我们返回一个`WidgetContextWrapper`。

####parseWidgetManifest
```
private void parseWidgetManifest(WidgetImpl widget, WidgetContext widgetContext) {
        if(!widget.hasLightManifest()) {
            WidgetLightManifest manifest = ManifestParser.parse(widgetContext);
            widget.setLightManifest(manifest);
        }
    }
```
这一步主要是解析我们插件中的`AndroidManifest.xml`。解析其中的`Service`。

至此插件加载完成，回到`LoadAsyncTask`的`onPostExecute`方法，将插件状态设置为LOADED，然后创建LifeCallback，并调用其`onLoad`方法，LifeCallback是之前我们在`parseWidget`方法中从meta获取到的。最后回调`onLoadSuccess`。

```
this.widget.setWidgetContext(this.widgetContext);
this.widget.setState(15);
this.createLifeCallback(this.widget);
this.widget.widgetOnLoad();
if(this.loadListener != null) {
   this.loadListener.onLoadSuccess(this.widget, this.widgetContext);
}
```

##END
以上就是插件的安装和加载全过程。


