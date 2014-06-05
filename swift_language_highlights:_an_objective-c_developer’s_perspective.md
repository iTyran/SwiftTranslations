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

// ChildhoodAndy
##Containers

You’ve come to know and love NSArray, NSDictionary and their mutable counterparts. Well, now you are going to have to learn about their Swift equivalents. Fortunately, they’re pretty similar. Here is how you declare arrays and dictionaries:

    let array = [1, 2, 3, 4]
    let dictionary = ["dog": 1, "elephant": 2]
    
This should be fairly familiar to you. There’s one slight catch though. In Objective-C, arrays and dictionaries can contain any type you jolly well wish. But in Swift, arrays and dictionaries are typed. And they are typed through the use of our friend from above, generics!

The two variables above can be rewritten with their types expressed (although remember you don’t actually have to do this!) like so:

    let array: Array<Int> = [1, 2, 3, 4]
    let dictionary: Dictionary<String, Int> = ["dog": 1, "elephant": 2]

Notice how generics are used to define what can be stored in the container. There is also a short form for the array, which is slightly more readable, but essentially boils down to the same thing:

    let array: Int[] = [1, 2, 3, 4]

Notice now that you cannot add anything to the array that isn’t of type Int. This may sound like a bad thing, but it’s incredibly useful. No longer does your API have to document what is being stored in the array it returns from a certain method or stored in a property. You can give that information right up to the compiler so that it can be smarter about error checking and optimisation described earlier.

##Mutability

One interesting thing about collections in Swift is their mutability. There are no “mutable” counterparts to Array and Dictionary. Instead, you use the standard let and var. For those who haven’t read the book yet, or delved into Swift at all (and I suggest you do, ASAP!), let is used to declare a variable as constant, and var is used to declare a variable as, well, variable! let is like using const in C/C++/Objective-C.
The way this relates to collections is that collections declared using let cannot change size. That is, they cannot be appended to, or removed from. If you try to, then you get an error like so:

    let array = [1, 2, 3]
    array.append(4)
    // error: immutable value of type 'Array<Int>' only has mutating members named 'append'

The same applies to dictionaries. This fact allows the compiler to reason about such collections and make optimisations as appropriate. If the size cannot change, then the backing store that holds the values never needs to be reallocated to accommodate new values, for example. For this reason it is good practice to always use let for collections that won’t change.

##Strings

Strings in Objective-C are notoriously annoying to deal with. Even simple tasks such as concatenating lots of different values becomes tedious. Take the following example:
    
    Person *person = ...;
        
    NSMutableString *description = [[NSMutableString alloc] init];
    [description appendFormat:@"%@ is %i years old.", person.name, person.age];
    if (person.employer) {
        [description appendFormat:@" They work for %@.", person.employer];
    } else {
        [description appendString:@" They are unemployed."];
    }

This is quite tedious and contains a lot of characters that are nothing to do with the data being manipulated. The same in Swift would look like this:
    
    var description = ""
    description += "\(person.name) is \(person.age) years old."
    if person.employer {
        description += " They work for \(person.employer)."
    } else {
        description += " They are unemployed."
    }
    
Much clearer! Notice the cleaner way of creating a string from a format, and you can now concatenate strings simply by using +=. No more mutable string and immutable string.


Another fantastic addition to Swift is comparison of strings. You’ll be aware that in Objective-C it is not correct to compare strings for equality using ==. Instead you should use the isEqualToString: method. This is because the former is performing pointer equality. Swift removes this level of indirection and instead leaves you being able to directly use == to compare strings. It also means that strings can be used in switch statements. More on that in the next section though.


The final piece of good news is that Swift supports the full Unicode character set natively. You can use any Unicode code-point in your strings, and even function and variable names! You can now have a function called 💩 (pile of poo!) if you want!


Another nugget of good news is there is now a builtin way to calculate the true length of a string. When it comes to the full Unicode range, string length is non-trivial to compute. You can’t just say it’s the number of bytes used to store the string in UTF8, because some characters take more than 1 byte. In Objective-C, NSString does the calculation by counting the number of UTF16, 2-byte pairs are used to store the string. But that’s not technically correct since some Unicode code-points take up two, 2-byte pairs.


Fortunately, Swift has a handy function to calculate the true number of code-points in a string. It uses the top level function called countElements(). You would use it like so:
    
    var poos = "&#x1f4a9;&#x1f4a9;"
    countElements(poos) // 2

It doesn’t quite work for all cases though still. It just counts the number of Unicode code-points. It doesn’t take into account special code-points that alter other characters. For example, you can put an umlaut on the previous character. In that case, countElements() would return 2 for the pair, even though it looks like just 1 character. Like so:
    
    var eUmlaut = "e\u0308" // ë
    countElements(eUmlaut) // 2
    
All that said, I think you’ll agree that strings are pretty awesome in Swift!

// gloryming
##Switch statements

The final thing I want to call out in this brief introduction to Swift is the switch statement. It has been drastically improved in Swift over its Objective-C counterpart. This is an interesting one, because it’s something that couldn’t have been added on to Objective-C without breaking the fundamental truth that Objective-C is a strict superset of C.

The first exciting feature is switching on strings. This is something that you may have wanted to do before, but couldn’t. To “switch” on strings in Objective-C you had to use lots of if-statements with isEqualToString: like so:

    if ([person.name isEqualToString:@"Matt Galloway"]) {
      NSLog(@"Author of an interesting Swift article");
    } else if ([person.name isEqualToString:@"Ray Wenderlich"]) {
      NSLog(@"Has a great website");
    } else if ([person.name isEqualToString:@"Tim Cook"]) {
      NSLog(@"CEO of Apple Inc.");
    } else {
      NSLog(@"Someone else);
    }

This is not particularly readable. It’s also a lot of typing. The same in Swift looks like this:

    switch person.name {
      case "Matt Galloway":
        println("Author of an interesting Swift article")
      case "Ray Wenderlich":
        println("Has a great website")
      case "Tim Cook":
        println("CEO of Apple Inc.")
      default:
        println("Someone else")
    }
    
Aside from the switching on a string, notice something interesting here. There are no breaks in sight. That’s because cases in switches no longer fall through to the next one. No more accidentally falling through bugs!
Now the next switch statement may very well blow your mind, so be prepared!
    
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
    
First up, there is now a break. This is because switches need to be exhaustive, i.e. they need to handle all cases now. In this case, we want the default to do nothing, so a break is added to declare the intentions that nothing should happen.

The next interesting thing is the ... and .. that you see in there. These are new operators and are used to define ranges. The former, defines a range up to and including the right hand number. The latter defines a range up to and excluding the right hand number. These are incredibly useful.

The last thing is the ability to define a case as a calculation of the input. In this case, if the value doesn’t match anything from zero to ten, it prints “Even” if it’s even and “Odd” if it’s odd. Magic!


##Where To Go From Here?
This has hopefully given you a taste of the Swift language and what wonderful gems there are in there. But there’s far more! I encourage you to go and read the Apple book, and other Apple documentation that will help you learn this new language. You’re going to have to do it sooner or later!

We’d love to hear what you think of the Swift language so far, or if there are any cool highlights you’re excited about. Please chime in with your thoughts below!
