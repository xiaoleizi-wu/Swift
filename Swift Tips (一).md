## Swift Tips (一)
> @noescape: 闭包不会跳出函数的生命周期，函数调用完后，闭包的生命周期结束。
> rethrows: 若是一个函数本身不会抛出异常，但是若参数闭包抛出异常，那么就会把异常继续抛出；
> 
> throw: 这个函数可能会抛出异常，无论参数闭包是不是异常
     
     
### delegate
```
@objc protocol MyClassProxy {
    func method()
}

class MyClass: NSObject {
    public func countNumnber(number: Int) -> Int {
        return number + 1
    }
}
class ViewController: UIViewController, MyClassProxy {
	var instance: MyClassDelegate!
	override func viewDidLoad() {
        super.viewDidLoad()
        instance = MyClassDelegate()
   		 instance.delegate = self
	}	
	
}
```

### Singleton
```
private let sharedInstance = Singleton()

class Singleton: NSObject {
    class var shared: Singleton {
        return sharedInstance
    }
    public func doSomeAction() {
        print("========single do some action===========")
    }
    
}

let single: Singleton = Singleton.shared
single.doSomeAction()

```
```
class MyClass: NSObject {
    public func countNumnber(number: Int) -> Int {
        return number + 1
    }
}
let myClass = MyClass.init()
let result = myClass.countNumnber(number: 3)
print("==============\(result)")
        
let f = myClass.countNumnber(number: 4)
print("==============\(f)")
```

### optional
```
推荐可选项解包，而非强制解包，若是有nil强制解包出现crash
func optionalAction() {
	let cities: Dictionary<String, Int> = ["Paris": 2241, "Madrid": 3165, "Berlin": 3562]
   // 或者这种方式
   // let cities: [String: Int] = ["Paris": 2241, "Madrid": 3165, "Berlin": 3562]
   // 此种方式不需要强制解包
   if let parisPopulation = cities["Paris"] {
       print("the population of paris is \(parisPopulation)")
   } else {
       print("unknown city population")
   }
 }
```

### Associated Objct
```
private var key: Void?
extension ViewController {
    var name: String? {
        get {
            return objc_getAssociatedObject(self, &key) as? String
        }
        set {
            objc_setAssociatedObject(self, &key, newValue, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
        }
    }
}
```

### Lock
```
@synchronized 幕后调用的是objc_sync中的objc_sync_enter和objc_sync_exit
func myMethod(another: AnyObject) {
    objc_sync_enter(another)
    // 在enter和exit之间不会被其它线程改变
    objc_sync_exit(another)
}

或者用closure分装起来
func synchronized(lock: AnyObject, closure: ()->()) {
    objc_sync_enter(lock)
    closure()
    objc_sync_exit(lock)
}
```

### Toll_Free Bridging
> Toll_Free原理是：Cocoa框架中的大部分NS开头的类其实是CF中对应类从在，即NS是CF的更改封装
> 例如NSstring对应的是CFStringRef,应为OC中ARC只对NSObject自动引用计数，所以CF对象是无法
> 进行内存管理需要加上__bridge表示内存管理权不变, 但是Swift是可以直接转换
> 

```
func BridgeAction() {
    let fileURL = NSURL.init(string: "SomeURL")
    var theSoundID: SystemSoundID = 0
    AudioServicesCreateSystemSoundID(fileURL!, &theSoundID)
}
```

### @encode
> 在OC中传入一个类型，我们就可以获取这个类型代表的字符串
> 
> 例如：char *typeChar1 = @encode(int332_t) typechar1 = "i"
// char *typeChar2 = @encode(NSArray) typechar2 = "{NSArray=#}"

> Swift使用了自己的Metatype来处理类型，并且在运行时保留这些信息，Swift没有这个关键字
> 但是Cocoa为我们提供了NSValue和objcType属性来获取对于的类型指针
> NSNumber是NSValue的子类

```
func MetaTypeAction() {
    let int: Int = 0
    let float: Float = 0.0
    let double: Double = 0.0
    
    let intNumber: NSNumber = NSNumber.init(value: int)
    let floatNumber: NSNumber = NSNumber.init(value: float)
    let doubleNumber: NSNumber = NSNumber.init(value: double)
    
    let n1 = String.init(utf8String: intNumber.objCType)
    let n2 = String.init(utf8String: floatNumber.objCType)
    let n3 = String.init(utf8String: doubleNumber.objCType)
    
    print(n1)
    print(n2)
    print(n3)
//    Optional("q")
//    Optional("f")
//    Optional("d")
}

```
### 数组
```
func enumerateArray() {
    let arr: NSArray = [1, 2, 3, 4]
    var result: Int = 0
    arr.enumerateObjects { (num, index, stop) in
        result += num as! Int
//        if index == 2 {
//
//        }
    }
    
    // 或者
    for (index, num) in [1, 2, 3, 4].enumerated() {
        result += num
        if index == 2 {
            break
        }
    }
}
```

### 类族
```
class Drinking {
    typealias LiquidColor = UIColor
    var color: LiquidColor {
        return LiquidColor.clear
    }
    class func drinking(name: String) -> Drinking {
        var drinking: Drinking
        switch name {
        case "Coke":
            drinking = Coke.init()
        case "Beer":
            drinking = Beer.init()
        default:
            drinking = Drinking.init()
        }
        return drinking
    }
}

class Coke: Drinking {
    override var color: LiquidColor {
        return LiquidColor.black
    }
}

class Beer: Drinking {
    override var color: LiquidColor {
        return LiquidColor.yellow
    }
}
```


