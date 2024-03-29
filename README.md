# WeexDemo
# swift项目中weex集成使用

对于IOS中：

1、创建swift项目并集成好pod

2、按官方的例子，在profile中添加：

    pod 'WeexSDK'
    pod 'SDWebImage'
    pod 'WXDevtool'
    pod 'SocketRocket'
    
3、在appDelegate中:


 ```
 import WeexSDK

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        self.window = UIWindow.init(frame: UIScreen.main.bounds)
        self.initailWeex() //对weexSDK进行配置
        self.window?.rootViewController = UINavigationController.init(rootViewController: ViewController())
        self.window?.makeKeyAndVisible()
        return true
    }
    
    
     func initailWeex() {
        //始始化sdk
        WXSDKEngine.initSDKEnvironment()
        //如果需要Native交互事件
        
        //WXSDKEngine.registerModule("event", with: NSClassFromString("WXEventModule")!)
        
        //weex的一些输入配置
        //WXLog.setLogLevel(.WXLogLevelAll)
        // 导航栏的一操作配置，如果需要，则在本地创建WXNavigationDefaultImpl文件，对协议进行修改     WXSDKEngine.registerHandler(WXNavigationDefaultImpl(), with: WXNavigationProtocol.self)
    }
```
    
WXNavigationDefaultImpl定义的内容： 

```
import UIKit
import WeexSDK
class WXNavigationDefaultImpl: NSObject, WXNavigationProtocol {
    func navigationController(ofContainer container: UIViewController) -> Any {
        return self;
    }
    
    func setNavigationBarHidden(_ hidden: Bool, animated: Bool, withContainer container: UIViewController) {
        
    }
    
    func setNavigationBackgroundColor(_ backgroundColor: UIColor, withContainer container: UIViewController) {
        
    }
    
    func setNavigationItemWithParam(_ param: [AnyHashable : Any], position: WXNavigationItemPosition, completion block: WXNavigationResultBlock?, withContainer container: UIViewController) {
        
    }
    
    func clearNavigationItem(withParam param: [AnyHashable : Any], position: WXNavigationItemPosition, completion block: WXNavigationResultBlock?, withContainer container: UIViewController) {
        
    }
    
    ///导航栏push的操作
    func pushViewController(withParam param: [AnyHashable : Any], completion block: WXNavigationResultBlock?, withContainer container: UIViewController) {
        guard let tempDict = param as? NSDictionary,
                let urlStr = tempDict["url"] as? String else {
            return
        }
        guard let url = URL.init(string: urlStr) else{
            return
        }
        let vc = WXBaseViewController.init(sourceURL: url)
        vc.hidesBottomBarWhenPushed = true
        container.navigationController?.pushViewController(vc, animated: true)
    }
    
    func popViewController(withParam param: [AnyHashable : Any], completion block: WXNavigationResultBlock?, withContainer container: UIViewController) {
        
    }
}
```

4、在ViewController中：

```
    var instance: WXSDKInstance!
    
    func showWeexView(){
        self.instance = WXSDKInstance()
        self.instance.viewController = self
        self.instance.frame = self.view.frame
        
        //开始加载weex页面回调
        self.instance.onCreate = { aView in
            self.view.addSubview(aView)
        }
        
        //加载失败回调
        self.instance.onFailed = { error in
        
        }
        
        //加载完成回调
        self.instance.renderFinish = { aView in
                    
        }
         
        //注：
        // 1、如果加载的是本地的路径， 用Bundle.main.url
        // 2、Bundle.main.path加载的话，file:///...
        guard let url = Bundle.main.url(forResource: "home.js", withExtension: "") else {
            return
        }
        
        self.instance.render(with: url, options: ["bundleUrl": url.absoluteString], data: nil)
    
    }    
```

二、根据网上的安装好weex-cli工具

1、新建weexDemo项目：

```
 weex create weexDemo
 
```
创建完后，会要求选择是使用npm源还是yarn源，完成后直接npm start 或者yarn start 运行起来，可以看到如下，说明成功了，在目录下可以找到生成的dist文件夹：
![image](https://github.com/hwq992689548/WeexDemo/blob/main/%E6%88%AA%E5%B1%8F2023-05-13%2000.06.31.png?raw=true)


2、weex是单页面打包模式，如果需要进行多个页面一起打包，则需要对webpack.common.conf 文件进行修改配置（后面再说）

3、yarn build 进行打包，可以看到dist文件夹生成了js文件
(1)、本地测试：
把dist文件夹拖进到xcode工程中（可以把weexDemo项目放置在xcode项目目录下，点击xcode添加文件夹引入，这样yarn build后，xcode中引入的js就是最新的，直接再次运行xcode即可）
（2）、可以在mac中启动apacha进行测试

    ```
    开启apache:  sudo apachectl start
    重启apache:  sudo apachectl restart
    关闭apache:  sudo apachectl stop
    
    找到/etc/apache2 有个config文件，记录了apache的启动目录，如：/Library/WebServer/Documents，将整个dist或单个js文件拖入Documents目录下，用http://localhost/dist/index.js查看是否成功看到js内容 
    
    ```
    
 注：1、127.0.0.1貌似没用起作用
    2、webpack.common.conf文件中，有提示
  
  ![image](https://github.com/hwq992689548/WeexDemo/blob/main/%E6%88%AA%E5%B1%8F2023-05-13%2000.51.29.png?raw=true)

还真信了邪，顺手就在vue页面中粘贴了：
import weex from 'weex-vue-render';
结果一运行到模拟器或手机上就报window的错


三、weex多页面打包配置
1、在src下创建pages文件夹以及装对应js入口的文件夹如：package

 ![image](https://github.com/hwq992689548/WeexDemo/blob/main/%E6%88%AA%E5%B1%8F2023-05-13%2000.38.34.png?raw=true)
 
2、pages里面创建页面vue文件，package文件夹中，则创建好对应的js, copy entry.js中的内容，修改一下引入路径以及router.push即可，如：

```
import { router } from "../router";
const App = require("@/pages/home/home.vue"); //对应的文件路径
new Vue(Vue.util.extend({ el: "#root", router }, App));
router.push("/home/");

```

3、在configs文件夹下，创建一个package.js文件（自已定义名字），用于在webpack.common.conf中用到

```
const helper = require("./helper");
const package = {
  home: helper.root("package/home.js"),
  login: helper.root("package/login.js"),
  launch: helper.root("package/launch.js"),
};

module.exports = {
  package,
};

```

4、修改webpack.common.conf文件

（1）、在第11行左右找到
const weexEntry = {
  index: helper.root("entry.js"),
};

将const改为 let,并添加合并自定义的package路径

 ```
 let weexEntry = {
  index: helper.root("entry.js"),
};

// 自己定义的 package 多页面
const package = require("./package");
//合并weexEntry 和 自己定义的package
weexEntry = Object.assign(weexEntry, package.package);
 
 ```
 
 （2）、找到getEntryFile的方法进行修改：
 
 ```
    
const getEntryFile = () => {
  const routerFile = path.join(vueWebTemp, config.routerFilePath);
  fs.outputFileSync(
    routerFile,
    getRouterFileContent(helper.root(config.routerFilePath))
  );
  let weexEntryKeys = Object.keys(weexEntry);
  let weexEntryValues = Object.values(weexEntry);
  let back = {};
  for (let key in weexEntryKeys) {
    var fileName = weexEntryKeys[key] + ".js";
    
    //这里注路径，web上路径不对会提示找不到./router
    if (fileName != "index.js") {
      fileName = "src/" + fileName;
    }
    let entryFile = path.join(vueWebTemp, fileName);
    let helperRoot = weexEntryValues[key];
    fs.outputFileSync(entryFile, getEntryFileContent(helperRoot, routerFile));
    back[weexEntryKeys[key]] = entryFile;
  }

  let backObject = JSON.parse(JSON.stringify(back));
  return backObject;
};

 ```
 
# 四、 weex与Native交互
 
 1、Native调用Weex

swift中：

直接调用：self.instance.fireGlobalEvent("方法名", param: [:])
```

 self.instance.fireGlobalEvent("callJS", params: ["id":"4333"])

```

weex中，监听方法：

```
created(){
        var globalEvent = weex.requireModule('globalEvent');
        globalEvent.addEventListener('callJS', (ret) => {
            console.log(ret["id"])
        })
    },
    
```



2、Weex调用Native

（1）Weex中声名的wx_export_method是oc方法，所以要新建一个oc类WXEventModule.h与WXEventModule.m, 继承 WXModuleProtocol协议
```
WXEventModule.h 内容

#import <Foundation/Foundation.h>
#import <WeexSDK/WeexSDK.h>

NS_ASSUME_NONNULL_BEGIN

@interface WXEventModule : NSObject<WXModuleProtocol>

@end

NS_ASSUME_NONNULL_END

```

WXEventModule.m 文件：

```
#import "WXEventModule.h"
@implementation WXEventModule
@synthesize weexInstance;

// 用 WX_EXPORT_METHOD 声名js调用Native的方法
WX_EXPORT_METHOD(@selector(openURL: callback:))
WX_EXPORT_METHOD(@selector(myPrint))


- (void)openURL:(NSString *)url callback:(WXModuleCallback)callBack {
//    NSLog(@"%@", url);
}

- (void)myPrint{
//    NSLog(@"2344");
}
@end

```

因为是swift工程，在 xxxx-Bridging-Header.h中，引入oc类的方法
#import "WXEventModule.h"

（2）、再建立一个swift类，也可以同名：WXEventModule.swift，拓展WXEventModule中方法即可在swift中实现。

```
extension WXEventModule {
    @objc func openURL(_ url: String, callback: @escaping WXModuleCallback) {
         
    }

   @objc func myPrint(){
        print("myPrint")
    }
}

```


（3）、在weex中，

```

click1(){
    weex.requireModule("WXEventModule").myPrint()
},

click2(){
    weex.requireModule("WXEventModule").openURL('https://www.baidu.com', (ret)=>{
          console.log(ret["id"])
    })
}

           
           
           

```


 




