#参考文档
<a>https://github.com/Microsoft/react-native-code-push</a>
# CodePush安装

安装CodePush

```bash
 npm install --save react-native-code-push
```
  
## 插件安装 Android
通过下面两步将CodePush集成到你的Android工程

1. 修改`android/settings.gradle`，添加下面的代码。 `注：这个操作类似于vs的一个解决方案多个子项目`
 
```bash
  include ':app', ':react-native-code-push'
  project(':react-native-code-push').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-code-push/android/app')
```
2. 修改`android/app/build.gradle`,把 `:react-native-code-push`项目加入到编译依赖
  
```bash
  ...
  dependencies {
      ...
      compile project(':react-native-code-push')
  }
```
3. (Only needed in v1.8.0+ of the plugin) In your android/app/build.gradle file, add the codepush.gradle file as an additional build task definition underneath react.gradle:
```bash
  ...
  apply from: "react.gradle"
  apply from: "../../node_modules/react-native-code-push/android/codepush.gradle"
  ...
```

## 插件配置 Android
经过上面的步骤就完成了CodePush插件的安装，接下来需要配置CodePush以实现重新定位`index.android.bundle`

1. 修改`MainAcivity.java`让CodePush起作用
  
```java
...
// 1. Import the plugin class (if you used RNPM to install the plugin, this
// should already be done for you automatically so you can skip this step).
import com.microsoft.codepush.react.CodePush;

public class MainActivity extends ReactActivity {
    // 2. Override the getJSBundleFile method in order to let
    // the CodePush runtime determine where to get the JS
    // bundle location from on each app start
    @Override
    protected String getJSBundleFile() {
        return CodePush.getBundleUrl();
    }

    @Override
    protected List<ReactPackage> getPackages() {
        // 3. Instantiate an instance of the CodePush runtime and add it to the list of
        // existing packages, specifying the right deployment key. If you don't already 
        // have it, you can run "code-push deployment ls <appName> -k" to retrieve your key.
        return Arrays.<ReactPackage>asList(
            new MainReactPackage(),
            // new CodePush() <-- remove this generated line if you used RNPM for plugin installation.
            new CodePush("deployment-key-here", this, BuildConfig.DEBUG)
        );
    }

    ...
}

```
2. 修改`android/app/build.gradle` versionName `1.0`修改为`1.0.0`

```
android {
...
defaultConfig {
    ...
    versionName "1.0.0"
    ...
}
...
}
```

# codepush命令行支持
1. 全局安装 code-push-cli
  
```bash
npm install -g code-push-cli
```
2. 创建Codepush帐号(弹出注册窗口，注册后会生成一串码)
（可以用code-push login 登录，然后生成一个文件.注意网络问题会告知不成功，多操作几次）

```
code-push register
```
3. 添加一个应用,将staging key设置到info.plist的CodePushDeploymentKey

```
  code-push app add appName
```
4. 打包bundle

```sh
#!/usr/bin/env bash

# bundle.sh 需要给执行权限 chmod +x bundle.sh

# android
react-native bundle \
--platform android \
--entry-file index.android.js \
--bundle-output ./release/index.android.jsbundle \
--assets-dest ./release

# ios
react-native bundle \
--platform ios \
--entry-file index.ios.js \
--bundle-output ./release/main.jsbundle \
--assets-dest ./release

# 发布到codepush服务器
#
code-push release wealth  ./release 1.0.0
```
5. 发布到codepush

`特别提醒:` 最后的版本号一定要与 android工程及ios工程中设置的版本号保持一致,因为codepush不会跨软件版本号更新
```
code-push release MyApp android/app/src/main/assets/index.android.jsbundle 1.0.1
```

# 插件安装(IOS)
1. 用xcode打开你的ios工程
2. 找到`CodePush.xcodeproj`,具体路径`node_modules/react-native-code-push`,将其拖拽到`Libraries`下
3. 选中`Build Phases`
4. 将`Libraries.CodePush.xcodeproj/Products`目录下的`libCodePush.a`拖拽到`Link Binary With Libraries`
5. 展开`Link Binary With Libraries` ,点击`+`按钮,选中`libz.tbd`后点击`Add`
6. 项目配置中的`Build Setting`标签,找到`Head Search Paths`编辑它的值,新加一个值`$(SRCROOT)/../node_modules/react-native-code-push`,从下拉里选择`recursive`

# 插件配置(IOS)
1. 编辑 `AppDelegate.m`,导入`CodePush.h`

    ```
    #import "CodePush.h"
    ```
2. 找到下面的代码断

    ```
    jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
    ```
3. 替换为

    ```js
    jsCodeLocation = [CodePush bundleURL];
    ```
通过以上配置使你的apps总是加载最新版的`bundle`,第一次启动应用时,
