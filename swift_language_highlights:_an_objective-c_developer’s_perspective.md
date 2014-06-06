#Swift Language Highlights: An Objective-C Developer’s Perspective
// 紫夜行者

If you were like me this Monday, you were sitting back enjoying the keynote, excited to start trying out all the new lovely APIs. And then your ears pricked up as you listened to words about a new language: Swift! It suddenly hit you that this is not an extension to Objective-C, but a completely brand new language. Maybe you were excited? Maybe you were happy? Maybe you didn’t know what to think.
Swift has surely changed the way we’re going to write iOS and Mac applications in the future. In this post, I outline some of the highlights of the Swift language, contrasting them to their counterparts in Objective-C.
Note this post is not designed to be a Swift get started guide. Apple have released a fantastic book about this, and I strongly suggest you read it. Instead, this is a discussion of some particularly cool areas to notice and play around with!

##Types

The first huge thing that Swift provides is type inference. In a language that uses type inference, the programmer doesn’t need to annotate variables with type information. The compiler infers it from what value is being set to the variable. For example, the compiler can automatically set this variable to a String:

    // automatically inferred
    var name1 = "Matt"
    // explicit typing (optional in this case)
    var name2:String = "Matt"
    
    
Along with type inference, Swift brings type safety. In Swift, the compiler (in all but a few special cases) knows the full type of an object. This allows it to make some decisions about how to compile code, because it has more information at hand.
This is in stark contrast to Objective-C which is extremely dynamic in nature. In Objective-C, no type is truly known at compile time. This is in part because you can add methods to existing classes, add entirely new classes and even change the type of an instance, all at runtime.
Let’s take a look at that in some more detail. Consider the following Objective-C:

    Person *matt = [[Person alloc] initWithName:@"Matt Galloway"];
    [matt sayHello];
    
    
When the compiler sees the call to sayHello, it can check to see if there’s a method declared in the headers it can see on the type Person called sayHello. It can error if there isn’t one, but that’s about all it can do. This is often enough to catch the first line of bugs that you might introduce. It will catch things like typos. But because of the dynamic nature, the compiler doesn’t know if the sayHello is going to change at runtime or even necessarily even exist. It could be an optional method on a protocol, for example. (Remember all those respondsToSelector: checks?).
Because of this lack of strong typing, there is very little the compiler can do to make optimisations when calling methods in Objective-C. The method that handles dynamic dispatch is called objc_msgSend. I’m sure you’ve seen this in many a backtrace! In this function, the implementation of the selector is looked up and then jumped to. You cannot argue this doesn’t add overhead and complexity.
Now look at the same code in Swift:
    
    var matt = Person(name:"Matt Galloway")
    matt.sayHello()
    
In Swift, the compiler knows much more about the types in play in any method call. It knows exactly where sayHello() is defined. Because of this, it can optimise certain call sites by jumping directly to the implementation rather than having to go through dynamic dispatch. In other cases, it can use vtable style dispatch, which is far less overhead than dynamic dispatch in Objective-C. This is the kind of dispatch that C++ uses for virtual functions.
The compiler is much more helpful in Swift. It will help stop subtle type related bugs from entering your codebase. It will also make your code run faster by enabling smart optimisations.


##Generics

Another huge feature of Swift is generics. If you’re familiar with C++, then you can think of these as being like templates. Because Swift is strict about types, you must declare a function to take parameters of certain types. But sometimes you have some functionality that is the same for multiple different types.
An example of this would be the often useful structure of a pair. You want a pair of values to be stored together. You could implement this in Swift for integers like so:
    
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
    
Pretty useful! It might seem unclear why you’d want this sort of feature at this time, but trust me: the opportunities are endless. You’ll soon start to see where you can apply these in your own code.

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

// gloryming
##Switch语句

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
