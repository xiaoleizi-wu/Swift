## Swift Tips(二)

#### 哈希

	Swift中NSObject对象中使用用‘=’, 子类没有重载该操作符，会转为'-isEqual:'方法
	Swift提供了一个HasTable的协议，实现该协议为Class提供哈希支持
#### 哈希问题：
1. 哈希定义是单向的，对于相同的值，期望有相同的hash，但是若是不同的值可能有相同的hash
2. hash随着系统环境和时间会发生变化

```
protocol Hashable: Equatable {
    var hashValue: Int { get }
}

extension Int: Hashable {
    
}

let num = 19
print(num.hashValue)
```
#### 判等

> 在OC中通常使用isEqualTo来判断字符串是否相等，但是在Swift中是没有的，通过‘==’操作符来判断
> 
> 在OC中'=='使用意思是判断两个对象是否指向同一块地址，而不是相同内容
> 在Swift中实现协议需要定义合适的‘==’操作符， 若是相等返回true,否则返回false
 
```
let str1: String = "111"
let str2: String = "222"
str1 == str2 // false


protocol Equatable {
    static func == (lhs: Self, rhs: Self) -> Bool
}

class TodoItem {
    let uuid: String
    let title: String
    
    init(uuid: String, title: String) {
        self.uuid = uuid
        self.title = title
    }
}

extension TodoItem: Equatable {
    
}

func == (lhs: TodoItem, rhs: TodoItem) ->Bool {
    return lhs.uuid == rhs.uuid
}
```
#### 操作符
```
struct Vector2D {
    var x = 0.0
    var y = 0.0
}

// 若是多个，操作很麻烦
let v1 = Vector2D.init(x: 2.0, y: 3.0)
let v2 = Vector2D.init(x: 1.0, y: 4.0)
let v3 = Vector2D.init(x: v1.x + v2.x, y: v1.y + v2.y)

// 定义一个操作符号
func +(left: Vector2D, right: Vector2D) -> Vector2D {
    return Vector2D.init(x: left.x + right.x, y: left.y + right.y)
}
let v4 = v1 + v2
```

#### 局部scope
```
// Swift使用匿名闭包，写OC中的语法糖
let titleLabel: UILabel = {
    let label: UILabel = UILabel.init(frame: CGRect.init(x: 0, y: 0, width: 100, height: 20))
    label.textColor = UIColor.red
    label.text = "呵呵呵"
    return label
}()
```

#### KeyPath
> Swift使用KVO需要依赖dynammic和@objc进行修饰，需要消耗性能
> 对于非NSObject类，不提供KeyPath方法

```
class MyClass: NSObject {
    @objc dynamic var date = NSDate.init()
}
class AnotherClass: NSObject {
    var myObject: MyClass!
    var observation: NSKeyValueObservation?
    override init() {
        super.init()
        myObject = MyClass.init()
        observation = myObject.observe(\MyClass.date, options: [.new], changeHandler: { (_, change) in
            if let newDate = change.newValue {
                print("AnotherClass date change \(newDate)")
            }
        })
    }
    
}

```
#### 类型判断

##### OC中使用判断类型
	[objc1 isKindofClass: [ClassA class]]; 判断objc1是ClassA或者其子类
	[objc2 isMemberofClass: [ClassB class]]; 当且仅当classB 成立
 
##### SWift isKind, isMember
 
##### is: 类似于OC中的isKindof, swift不仅可以用于class，还可以作用于struct, enum
 
``` 
class ClassA: NSObject {}
class ClassB: ClassA {}

let obj1: NSObject = ClassB.init()
let obj2: NSObject = ClassB.init()
let obj: AnyObject = ClassB.init()

obj1.isKind(of: ClassA.self) // true
obj2.isMember(of: ClassA.self) // false

if obj is ClassA {
    print("属于 ClassA ")
}

if obj is ClassB {
    print("属于 ClassB")
}
```


#### GCD 和延迟

```
typealias Task = (_ cancel: Bool) -> Void

func delay(_ time: TimeInterval, task: @escaping () -> ()) -> Task {
    
    func dispath_later(block: @escaping ()->()) {
        let t = DispatchTime.now() + time
        DispatchQueue.main.asyncAfter(deadline: t, execute: block)
    }
    
    var closure: (() -> Void)? = task
    var result: Task?
    
    let delayClosure: Task = {
        cancel in
        if let internalClosure = closure {
            if cancel == false {
                DispatchQueue.main.async(execute: {
                    internalClosure()
                })
            }
        }
        closure = nil
        result = nil
    }
    
    result = delayClosure
    
    dispath_later {
        if let delayClosure = result {
            delayClosure(false)
        }
    }
    
    return result!
}
```

#### 值类型和引用类型

> Swift分为值类型和引用类型，值类型在传递和赋值在复制，引用类型对象的一个指向；
> 
> Swift为enum和struct为值类型， class为引用类型
> Swift中所有的内建类都是值类型，不仅仅包括Int, Bool甚至是String， Array, Dictonary都是值类型
 
    值类型的好处是减少内存的分配和回收次数。
    值类型被复制的时机是值类型的内容发生改变的时候
 


#### String 和 NSString
>  Swift中所有的Api都接受String的参数和返回的String类型, 没必要再去转换
> 
>  String类型是Struct，相比NSObject的NSString类来说，符合不变的特性
> 
>  string实现了Collection协议，只有String才可以使用，而NSString是无法使用的

```
let levels = "ABCD"
let nsRange = NSRange.init(location: 1, length: 4)
// 当range没有repacingCharacters方法，需要转换
(levels as NSString).replacingCharacters(in: nsRange, with: "AAAA")
```

#### @autoreleasepool
> 在app中整个主线程跑在有个自动释放池中，没个主的runloop结束时候都进行drain操作
> 
> 当加入一个自动释放池，这样我们就可以在循环某个特定的时候进行内存释放，这样保证了不会因为内存
##### 不足出现crash
```
func loadBigData() {
    if let path = Bundle.main.path(forResource: "big", ofType: "png") {
        for i in 1...10000 {
            autoreleasepool {
                let data = NSData.init(contentsOfFile: path)
                Thread.sleep(forTimeInterval: 0.5)
            }
        }
    }
}
```
##### weak unowned

```
class A: NSObject { }
// 通过weak向编译器说明不需要持有a, 当obj指向nil的时候，整个环境就没有对A的这个实例持有了
class B: NSObject {
    weak var a: A? = nil
    override init() {
        
    }
    deinit {
        print("B deinit")
    }
}
```


