# Swift语言亮点：从一个Objective-C开发人员的角度

只要你喜欢，你可以坐享keynote乐趣,兴奋的开始尝试所有最新的API。然后你可以竖起耳朵听新语言:Swift!它不是object-c的扩展，它是一门新的语言。你兴奋么？你开心么？也许你自己也不知道想什么。

Swift无疑将在未来改变我们编写的IOS和mac应用的方式。在本文中，我简要概述一些Swift语言的亮点，并于object-c里对应的作对比。

注意：本文不是一篇Swift入门教程。apple已经发布了关于一本不错的[Swift手册](https://itunes.apple.com/us/book/swift-programming-language/id881256329?mt=11&ign-mpt=uo%3D8)，并且我强烈建议你阅读。相反，本篇将讨论一些特别酷的要点并且玩转它！

## 类型

第一重点是Swift提供的[类型推导](http://en.wikipedia.org/wiki/Type_inference)。一种语言如果提供类型推导，那么程序员不需要注释变量的类型。编译器会自动根据变量去推断是什么类型并且设置这个变量。例如，编译器可以自动设置这个变量为String型：

    // automatically inferred
    var name1 = "Matt"
    // explicit typing (optional in this case)
    var name2:String = "Matt"
    
类型推导给Swift语言提供了[类型安全](http://en.wikipedia.org/wiki/Type_safety)。编译器（基本上所有类型除了少数特殊案例）知道所有对象的类型。这使他做一些如何编译代码的决策，因为它掌握更多信息。

事实上这与object-c强壮动态的性质形成鲜明的对比。在object-c中，就是没有编译时候，其类型是知道的。在运行时，这是因为你可以在已存在的类中添加方法，添加全新的类和在实例中改变类型

让我们看更多地细节。参考下面的object-c：

    Person *matt = [[Person alloc] initWithName:@"Matt Galloway"];
    [matt sayHello];
    
编译器调用sayHello时，它会去头文件检测声明的方法叫sayHello。如果没有就会报错，这就是它所要做的事情。它通常会在第一行跑出bugs信息给你。它可以捕捉如拼写错误，但是由于动态的特性，在运行时编辑器不知道sayHello的改变或者必要的存在。在协议中它是一个可选方法，例如（记住所有respindsToSelector: checks?）

由于缺乏强转类型，没有编译器可以在objective-c调用方法时做优化。处理动态调度的方法叫obj_msgSend.我很确信的说你已经看到很多向后追踪！在这个方法中，选择器实现了向上然后跳。你不得不同意这增加开销和复杂性。

现在我们看下Swift里相似的代码： 

    var matt = Person(name:"Matt Galloway")
    matt.sayHello()
    
在Swift中，在任何方法调用，编译器对相关类型了如指掌。它很快就定位sayHello()在哪里定义。正是基于此，它跳过动态调度而立即跳向实现达到充分利用优化的效果。此外，在objective-c它使用vtable风格调度，开销远低于动态调度。这种调度在c++中使用虚函数。
使用Swift编辑器更加有助于我们的开发效率。它将帮助我们进入代码编辑中更好的处理与类型相关的bugs。通过开启智能优化它也将使你的代码运行更加快速。

## 泛型

Swift另一个特性是泛型。如果你对c++很熟悉，你可以联想他像模板。Swift关于类型是严格的，你必须声明一个函数参数作为特定的类型。但是你也可以定义有多个有不同的类型的参数的方法。

一个经常用的到结构体例子。你想同时存值，在Swift里你需要像下面实现的方式做：

    struct IntPair {
        let a: Int!
        let b: Int!
     
        init(a: Int, b: Int) {
            self.a = a
            self.b = b
        }
     
        func equal() -> Bool {
            return a == b
        }
    }
        
    let intPair = IntPair(a: 5, b: 10)
    intPair.a // 5
    intPair.b // 10
    intPair.equal() // false
    
这非常有用！你可能不是很清楚为什么是这样的特性,但请相信我:机会永远是无穷无尽的。很快你就会在你的代码中随意应用。

## 容器

你熟悉并喜爱着 NSArray，NSDictionary 以及它们的可变副本。好了，现在你要学习Swift中等价的容器了。幸运的是，它们非常相似。下面是如何声明一个数组和字典：

    let array = [1, 2, 3, 4]
    let dictionary = ["dog": 1, "elephant": 2]

你应该对他们很熟悉。但有一个细微差别。在 Objective-C 中，数组和字典可以包含任何你希望的类型。但是在 Swift 中，数组和字典有类型限制，并且通过上面提到的“泛型”的使用来限定类型。

两个以上的变量可以用它们的表达类型来重写（但记住你并不需要真的这么做），如下：

    let array: Array<Int> = [1, 2, 3, 4]
    let dictionary: Dictionary<String, Int> = ["dog": 1, "elephant": 2]

注意泛型是如何用来定义容器的存储的。还有一个数组的缩写形式，这个更具有可读性，但本质上是一样的。

    let array: Int[] = [1, 2, 3, 4]

注意现在你不能往数组里面添加非`Int`型的元素。这听起来挺糟糕，但它非常有用。再也不需要用API来记录数组里存储了哪些从某个方法返回或者以属性存储的元素。你可以告诉编译器这些信息，编译器在错误检查方面会更加智能，并且可以提早做出优化。

## 可变性

在 Swift 中关于集合的一个有趣点就是它们的可变性。数组和字典并没有可变副本。相反，你要使用标准的`let`和`var`。对于那些没有读过Swfit书籍或者钻研Swift（我建议你尽快）的人来说，`let`用来声明一个常量，`var`用来声明一个可变的变量。`let`就像我们在C/C++/Objective-C中使用的`const`一样。

我们用`let`来声明一个容量不可变的集合。也就是说，它们不能添加元素或者删除元素。如果你尝试那么做，那么你会得到这么一个错误：

    let array = [1, 2, 3]
    array.append(4)
    // error: immutable value of type 'Array<Int>' only has mutating members named 'append'
    
这同样适用于字典。这样编译器会分析这些集合并适当作出优化。如果大小不能改变，那么内存就不用重新分配来容纳新的值。例如，出于这个原因，总使用`let`来声明一个大小不会改变的集合是个很好的做法。

## 字符串

Objective-C的字符串处理起来让人恼火。即使是简单的任务如将很多字符串串联起来也会变得乏味。看下面这个例子：
    
    Person *person = ...;
        
    NSMutableString *description = [[NSMutableString alloc] init];
    [description appendFormat:@"%@ is %i years old.", person.name, person.age];
    if (person.employer) {
        [description appendFormat:@" They work for %@.", person.employer];
    } else {
        [description appendString:@" They are unemployed."];
    }
    
这是相当繁琐的，并且包含了很多与操作数据无关的字符。同样的功能在 Swift 中是这样的：    
    
    var description = ""
    description += "\(person.name) is \(person.age) years old."
    if person.employer {
        description += " They work for \(person.employer)."
    } else {
        description += " They are unemployed."
    }
    

更加清晰了！注意这个创建格式化字符串的简洁方式，现在你可以简单通过`+=`来连接字符串了。没有什么可变和不可变的字符串之说了。

Swift 还增加了一个奇妙的特性就是字符串的比较。你会意识到，在 Objective-C中用`==`来比较两个字符串是否相等是不正确的，相反，你应该使用`isEqualToString:`方法。因为前者比较用的是指针。Swift 消除了这个间接的方法，而是让你直接能够用`==`来比较字符串。这也就意味着，可以在 switch 语句中使用字符串。更多信息见下一节 switch 部分。

最后一个好消息是，Swift原生支持完整的Unicode字符集。你可以在你的字符串甚至是函数和变量名中使用任何Unicode字符。现在，如果你愿意的话，可以创建一个名为💩（一堆便便！）的函数！

另外一个重磅消息就是现在有一个内置的方法来计算一个字符串的真实长度。当涉及到完整的Unicode范围时，字符串的长度计算起来并不简单。你不能认为就是以UTF8存储的字符串的字节数，因为一些字符超过1个字节。在 Objective-C中，`NSString`通过计算UTF16的数量来计算长度，使用2-byte对用来存储字符串。但是，由于一些Unicode字符占用两个2-byte对，所以这样从技术上说是不对的。

幸运的是，Swift有一个便捷的的函数来计算字符串字符的真实数量。它使用一个顶层函数叫做`countElements()`。你可以像这样使用它：
    
    var poos = "&#x1f4a9;&#x1f4a9;"
    countElements(poos) // 2
    
它虽然仍然可以工作，但并不完全适用于所有情况。它只计算Unicode字符的数量。它并没有考虑到能够改变其他字符的特殊字符。例如，你可以把元音变音放在前一个字符处。在这种情况下，`countElements()`将返回2代表一对，尽管它看起来像是1个字符。像这样：
    
    var eUmlaut = "e\u0308" // ë
    countElements(eUmlaut) // 2
    
说了这么多，我想你会同意，Swift处理字符串确实很棒！

## Switch语句

在这个Swift语言的简介中我想讲的是最后一件是Switch语句。与Objective-C相比，它在Swift中被大幅度地改进了。这是件有趣的事，因为在Objective-C中你无法添加一些东西，除非你打破Obejctive-C是严格的意义上C语言的超集这个事实。

第一个令人兴奋的特征是在字符串上做Switch判断。这可能是你之前就想做的事情，但是一直无法这样做。在Objective-C中如果你想在字符串上做Switch判断，你必须使用大量的if语句配合isEqualToString来达到目的，例如这样：

    if ([person.name isEqualToString:@"Matt Galloway"]) {
      NSLog(@"Author of an interesting Swift article");
    } else if ([person.name isEqualToString:@"tairan.com"]) {
      NSLog(@"Has a great website");
    } else if ([person.name isEqualToString:@"Tim Cook"]) {
      NSLog(@"CEO of Apple Inc.");
    } else {
      NSLog(@"Someone else);
    }

这样的实现方式可读性不是非常的好。这种实现有非常多的类型。同样的实现在Swift中是这样的：

    switch person.name {
      case "Matt Galloway":
        println("Author of an interesting Swift article")
      case "tairan.com":
        println("Has a great website")
      case "Tim Cook":
        println("CEO of Apple Inc.")
      default:
        println("Someone else")
    }

除了在字符串上使用Switch判断，注意这里还有些有趣的事情。这里没有break关键字。这是因为Switch判断语句里面的每种情况不再传递到下一种。不再有因为意外的传递而产生的bug！

现在，下一个Switch语句会按照你的想法执行，所以准备好吧！
    
    switch i {
    case 0, 1, 2:
        println("Small")
    case 3...7:
        println("Medium")
    case 8..10:
        println("Large")
    case let ii where i % 2 == 0:
        println("Even")
    case let ii where i % 2 == 1:
        println("Odd")
    default:
        break
    }

首先，现在有一个break关键字。这是因为Switch语句需要穷举，例如现在需要处理所有的情况。这种情况下，我们希望默认是什么都不做，所以添加一个break关键字来声明表示我们希望什么都不做。

下一个有趣的事是这里的“...”和“..”操作符。他们是新的操作符用来定义范围的。原来与之对应的是，定义一个范围并且包括右边的数字。然后定义一个范围并不包括右边的数字。这非常有用。

最后一件事是能够像计算输出一样定义一种情况。这种情况下，如果值不能和0~10中的数匹配，并且如果是偶数就会打印“偶数”，如果是奇数则打印“奇数”。奇特吧！


##何去何从?

希望以上简介能够给你品尝到Swift语言是多么美妙的一种语言。但是它的美妙并不仅仅如此！我建议你去读一下Apple Book和一些其他苹果文档，这会对你学习这门新语言有帮助。

你迟早会用它去做一些事！

我们很原因听到你对Swift语言目前的想法，或者你一些你特别感兴趣的点。请在下面的评论中告诉我们你的想法！
