# a-beginner-friendly-tour-through-functional-programming-in-scala

<!--
Author: zhange
Email: zhange.zzg@gmail.com
create time: 2017-02-26 15:54:09
tags: [-]
description: -
-->

**注意：**本文内容来源于书籍、博客、官方文档等的翻译、整理、摘抄，仅供个人学习交流禁止转载，如有侵权请联系删除！

[TOC]

[翻译自：A Beginner-Friendly Tour through Functional Programming in Scala](http://degoes.net/articles/easy-monads?utm_content=buffer827ff&utm_medium=social&utm_source=twitter.com&utm_campaign=buffer)

函数式编程的基本核心十分简单：通过组合函数(function)来构建程序。

这里，“function”并非指“计算机科学”中的函数，它指一段机器代码，而是一个“*数理函数(mathematical  function)*”：

1. **Totality**：一个函数必须为所有可能的输入生成一个值；
2. **Determinism**：一个函数必须为相同的输入返回相同的值；
3. **Purity**：函数唯一的副作用必须是计算它的返回值。

所有这些属性，给你一种前所未有的能力来解释你的代码：调用函数并传入任何输入，你总会得到一个有效的值，而且相同的输入总是会得到相同的结果，同时函数不会再做其他任何事，比如发射核导弹....

这种微小的想法对于大型软件工程有着深刻的简化作用，因为这意味着，你的大脑只需要追踪更少的东西就能理解程序的行为。事实上，你可以通过理解程序的个别部分来理解程序的整个行为 - 而无需一次在脑子中掌握一切！

目前，函数式编程并不总是拥有一个“简单”的声誉，但我认为是由于以下几个因素：

1. **Unfamiliarity**(不太常见)：函数式编程与大多数专业人员使用的编程类型有着很大的不同。因为它不太常见，看起来很难的样子。最好是将函数式编程与第一次学习编程的经验相比较(而不是学习一种你已经知道的新的编程语言的经验)。
2. **Jargon**(术语太多)：函数式编程中有太多术语，比如：“immutability”, “recursion”, “induction”, “hylomorphism”, “transducer”, “functor”等等一大堆。这些概念并不一定很难，但拥有可怕的冠冕堂皇的名字，命令式方式的大量编程经验也不能帮助你理解任何新的术语。
3. **Motivation**(学习动机)：从我的经验来看，人们对于学习那些可以清晰的看到如何达成个人目标的东西会感觉更容易。然而，随着越来越多的人开始学习函数式编程(并与他人分享知识)，情况也在改善，但是对于函数式编程的核心概念并没有太大积极性。

有时函数式编程通常被关联为“*高等静态类型函数式编程(advanced statically-typed functional programming)*”，这时，学习“函数式编程”的人其实在一次学习两种东西：函数式编程和一个高级类型系统(多数命令式语言拥有相对简单的类型系统)。

然而，有些人可以用弱类型语言进行函数式编程，或者使用高级类型系统进行命令式编程，因此这二者并无关联。

很多函数式开发者喜欢展示我们为何对函数式编程如此兴奋。然而，我所写的大部分内容并非对函数式的好奇，甚至不是面向新手函数式开发者。相反，我写的都是我感兴趣的东西，这通常是高级函数编程的中间形式，对于那些多年来一直在进行函数式编程的人比较有用。

所以我在做一个实验：我将展示一些函数式编程的思想以帮助我们构建一些实际程序。但是我会以一种有助于前面提到的那些因素的方式进行。

这是一篇为非函数式开发者准备的文章，或者那些了解一点但想知道更多的开发者。

希望你发现它会有用 - 但相比有用，我更希望你发现它鼓舞人心。激发足够的投入和必要的努力，来推动你的函数式编程知识，尽管看起来似乎会有点难。

或者也不一定，看完这篇文章之后一切看起来都会变得很容易！

## 不切实际的 FP

展示函数式编程能力的一个典型例子就是泛型排序函数：

```scala
def sort[A: Orderig](as:List[A]):List[A] = as match{
  case Nil => Nil
  case a :: as =>
    val (before, after) = as partition (_ < a)
    sort(before) ++ (a :: sort(after))
}
```

这个例子很漂亮，因为它展示了函数式编程如何以如此简化的效果来表达一个程序。

看完代码，你自己可能会确认下面这几点：

- 空列表排序仍会得到空列表；
- 一个首元素为 a 后很 as 的列表，经过排序后由 小于 a 的列表、a、大于 a 的列表依次构成。

该函数的每个部分都可以独立推理。如果相信一段是正确的，那么就可以相信整体的正确性。

此外，这个`sort`函数，因为它是数学意义上的函数，更容易测试和复用。我们可以传入任何列表并预期得到一个排序后的列表。因为我们知道对于同一个输入，函数总是会返回相同的结果。

因此我们的测试表示起来也会非常简单：

```scala
assert(sort[Int](Nil) == Nil)
assert(sort[Int](3 :: 2 :: 1 :: Nil) == 1 :: 2 :: 3 :: Nil)
```

(事实上，可以以函数式编程的方式来更加有力的表示，不过这就是另外一篇文章的主题了。)

虽然在这个例子中解释的函数式编程的好处看起来很简单，但这是一个很大的延伸，想象如果或如何对这些好处进行扩展以超越我们的玩具样例。

事实上，函数式编程的头号反对者就声称它只适合这些玩具例子，在”现实世界“编程中完全失败。

让我们找一个我们想到的现实世界中最简单的例子：一个函数失败时并不返回一个值。

## 完整性

我以完整性和正确性定义了一个例子，如果丢掉完整性要求会发生什么呢？

好吧，这个函数不需要返回任何东西。更实际的说，函数会一直运行，或者通过宿主语言支持的其他方式来转义这个返回值的需求 - 通常是抛出一个异常。

一个永不返回的函数是因为没有跳出而一直运行(脱离循环的边缘条件，或类似其他原因)，但异常又是什么呢？

在异常出现之前，程序员使用一个返回值来表示函数的成功或失败。在 C 代码中，比如：

```c
int parse_config(config *cfg){
  FileHandle handle;
  char *bytes;
  
  handle = new Handle();
  handle = file_open("config.cfg");
  if(handle == NULL) return -1;
  
  char *bytes = file_read(handle);
  if(bytes == null) return -2;
  .....
  return 0;
}
```

在这种世界里，引入异常处理似乎不可思议。程序员可以避免混乱的错误应用逻辑处理问题，从异常的短路行为中获益。

异常的主要问题是，在一个支持它的语言中，你无法保证他们可能会发生的地方。这意味着他们可能在任何地方触发。这表示，如果时间够长，他们可以无处不在(甚至在 Java 中，未检异常可以随处抛出，其中经常还包含受检异常！)。

同时由于 null 的存在，导致激增了大量防御性、攻击性的异常处理，尽管有些并没有什么意义，但导致了更多错误的发生和怪异的边缘问题。

幸运的是，我们可以轻松的同时实现完整性和整洁代码，但比老旧语言需要更多设施的支持。

我了实现这个想法，我们定义一个函数来返回列表的第一个元素：

```scala
def head[A](as: List[A]): A = as.head
```

这个函数并不完整，取决于你传入的是一个什么列表，可能会返回可能也不会返回列表的首元素。如果你传入一个空列表，函数永远都不会返回，而不是抛出一个异常。

如果想要该函数完整，仅需要引入一个数据结构来建模*optionality*的概念 - 一个东西可能有也可能没有。

让我们称之为`Maybe`：

```scala
sealed trait Maybe[A]
final case class There[A](value: A) extends Maybe[A]
final case class NotThere[A]() extends Maybe[A]
```

通过这个数据结构，我们可以把这个”伪函数“`head`转换为真的函数：

```scala
def head[A](as:List[A]):Maybe[A] = as match {
  case Nil => NotThere[A]()
  case a :: _ => There[A](a)
}
```

现在当我们考虑使用该函数的代码时，不再需要考虑该函数没有返回的可能性。因为该函数总是能够返回。由于不需考虑这种可能性，使用`head`函数的代码表述起来也更为简单，包含更少需要分享的场景。

`Maybe`数据结构并没有提供跟异常一样的能力。有一条，它不会包含任何对于`head`函数来说意味着 OK 的错误信息(因为我们知道错误是什么-空列表)，但对于其他函数来说可能并不有效。

为了解决这个问题，我们可以引入一个新的数据结构，称为`Resullt`，对*exceptionality*进行建模：

```scala
sealed trait Result[E, A]
final case class Error[E, A](error: E) extends Result[E, A]
final case class Success[E, A](value: A) extends Result[E, A]
```

这种类型支持我们创建一个`file_open`这样的完整性函数：

```scala
def file_open(path:String):Result[FileError, FileHandle] = ???
```

现在我们可以拥有跟异常一行的信息。然而，如果我们需要很多操作需要执行，同时每个操作都返回一个`Result`，这看起来我们会拥有相当多的模板代码，不免让人回忆起异常之间的日子：

```scala
def parse_config:Result[FileError, Config] = {
  file_open("config.cfg") match {
    case Error(e) => Error(e)
    case Success(handle) =>
      file_read(handle) match {
        case Error(e) => Error(e)
        case Success(bytes) => ???
      }
  }
}
```

我们已经创建了`file_open`、`file_read`函数，等等，简化了我们的表述，同时也引入了不少模板代码，使代码难以阅读。

为了夺回之前异常的优势，我们需要识别上面代码的模式。如果你研究几分钟，则会发现下面的模式：

```scala
doX match {
  case Error(e) => Error(e)
  case Value(x) => doY(x) match {
    case Error(e) => Error(e)
    case Success(y) => doZ(y) match {
      case Error(e) => Error(e)
      case Success(z) => doW(w) match {
        ...
      }
    }
  }
}
```

你会发现，`doY`、`doZ`、`doW`都会从上一个操作产生的`Result[E, A]`那接收一个 A，然后生成一个新的`Result[E, A]`。

这暗示我们可以通过一个`chain`方法分解重复的代码：

```scala
def chain[E, A, B](result:Result[E, A])(f: A => Result[E, B]): Result[E, B] = 
  result match{
    case Error(e) => Error(e)
    case Success(a) => f(a)
  }
```

现在，可以使用`chain`方法来重新实现原来的`parse_config`：

```scala
def parse_config: Result[FileError, Config] = {
  chain(file_open("config.cfg")) {handle => 
    chain(file_read(handle)){ bytes =>
      ???
    }
  }
}
```

这样通过在`Result[E,A]`上调用`chain`来减少模板代码。通过这种方式，我们既可以拥有异常的短路优势，错误处理逻辑又由`chain`方法拆分，应用逻辑就无需再关注这些。

如果你使用`Result`这样的结构来建模异常场景，通常你会发现需要一个下面这样的工具方法：

```scala
// change the A in a Result[E, A] into a B by using the provided function f
def change[E, A, B](result:Result[E, A])(f: A => B):Result[E, B] = result match{
  case Error(e) => Error(e)
  case Success(a) => Success(f(a))
}
```

这是一个类 map 的函数(将列表中的元素映射为其他类型)。如果你喜欢 OO 风格，可以把`chain`、`change`方法包括在`Result`类之中：

```scala
sealed trait Result[E, A] {
  def change[B](f: A => B):Result[E, B] = this match {
    case Error(e) => Error(e)
    case Success(a) => Success(f(a))
  }
  
  def chain[B](f: A => Result[E, B]):Result[E, B]= this.match{
    case Error(e) => Error(e)
    case Success(a) => f(a)
  }
}

final case class Error[E, A](error:E) extends Result[E, A]
final case c;ass Success[E, A](value:A) extends Result[E, A]
```

这样一来代码就会更可读：

```scala
def parse_config:Result[FileError, Config] = {
  file_open("config.cfg").chain{ chanle =>
    file_read(handle).chain{bytes =>
      ???
    }
  }
}
```

更进一步，如果你把`change`、`chain`方法称作`map`、`flatMap`，Scala 则会提供更加灵巧的方式来进一步简化代码格式：

```scala
def parse_config:Result[FileError, Config] = {
  for{
    handle <- file_open("config.cfg")
    bytes <- file_read(handle)
  } yield ???
}
```

最终，我们首先了函数的完整性，同时也拥有异常的短路特性和关注点分离。

我们所需要的也就是`chain`函数(即 flatMap)和`Result`数据结构。其余的则自然引入。

注意，`chain`函数接收一个函数作为参数。这个参数作为一个匿名函数(在其上下文中捕获任意引用)提供，这论证了为什么这种技术永远不会出现在 C 代码中。如果 C 拥有一类函数、垃圾回收或者引用计数，很可能这种模式会自行出现而无需函数式编程社区的任何输入。

函数的其他要求是**确定性**和**纯洁性**，下面的章节会讲到。

## 确定性 & 纯洁性

一个函数，如果拥有不确定性，或者所做的并非仅仅是计算一个返回值，那它就会变得非常难以推理。

比如我用过的一个库，会在构造器中执行 IO：

```java
class Logging{
  private OutputStream ostream;
  public logging(File file){
    ostream = new FileOutputStream(file);
  }
}
```

该构造器可能会抛出异常，并且难以预料！

另一个例子，当你把 Java 中的`URL`类放入一个数据结构时，它会执行一个网络连接。原因是 equals 和 hash 编码方法会触发地址的识别。

除了不纯函数的意味性质外(谁知道他们什么时候会干些什么！)，非确定性(non-determinism)导致函数的测试和表述变得尤其困难。你可能会被迫对影响函数行为的那些状态的表述代码进行 mock。

纯函数，确定性函数，在现场之外不会做任何不正当的事。你可以期望他们每次都会根据相同的输入返回相同的结果，这意味着代码容易测试且易于理解。

但如果你尝试使所有的函数都是完整的、确定的，并且是纯的，当你执行一些输入输出或副作用操作时则会很快撞到墙上 - 一堵说明了太多函数式编程与真实软件不切实际的墙。

事实上，这墙早就被粉碎了，而不是沿着这条路走下去，让我们看一下一个 IO 的例子，看能不能推荐一种方案。

### Console IO

假如我们正开发一个控制台程序，改程序需要从控制台读取输入，然后再会写数据到控制台上。这样的程序能解决很多有用的问题，如果我们能想出如何以确定性、纯函数的方式来构建一个控制台程序，那就能推广到其他类型的程序了。

如果你针对该问题思考了一会，可能会得出下面的想法：可以定义一些描述控制台副作用的数据结构来构建程序，而不是调用那些不确定或不纯的函数。

假如这个`ConsoleIo`是我们这个程序的说明书，首先，我们需要一种方式来描述”写入输入到控制台“这种副作用。

一种方式看起来可能会是这样：

```scala
sealed trait ConsoleIO
final case class WriteLine(line:String) extends ConsoleIO
```

这种方式未免也太简单了，因为这样我们只能将文本的一行写入到控制台：

```scala
def helloWorld: ConsoleIO = WriteLine("Hello World!")
```

这是一个完整的、确定的纯函数 -- 但是并没有什么卵用。它顶多能描述一个程序将文本的一行写入到控制台。文本可能会变，当然，甚至是函数的一个参数，但最终，程序只会对控制台有一个副作用。

幸运的是，将该结构扩展为支持多个顺序的副作用也很简单：

```scala
sealed trait ConsoleIO
final case class WriteLine(line:String, then:ConsoleIO) extends ConsoleIO
```

通过这种方式，我们可以描述更加复杂的”副作用化“程序，比如：

```scala
def infiniteHelloWorld:ConsoleIO = WriteLine("Hello World", infiniteHelloWorld)
```

如果我们引入一个”thunk“以避免在构造中压栈(blow stack)，该结构就能够描述一个向控制台写入无限个”Hello World“的程序。像这种无限制的程序并没有什么用，不过我们可以给`ConsoleIO`添加另一项，让他可以支持终止：

```scala
sealed trait ConsoleIO
final case class WriteLine(line:String, then:ConsoleIO) extends ConsoleIO
final case object End extends ConsoleIO
```

现在我们可以描述一个将文本行打印指定次数到控制台的程序：

```scala
def printTextNTimes(text:String, n:Int):ConsoleIO = {
  if(n <= 0) End
  else WriteLine(text, printTextNTimes(text, n -1))
}
```

该函数也是一个完整的、确定的纯函数。当然，他实际上不会打印任何东西到控制台，但我们很快就能做到。

目前，我们只能描述一个写入文本到控制台的程序。因此扩展一个从控制台读取输入的程序也相当简单：

```scala
sealed trait ConsoleIO
final case class WriteLine(line:String, then:ConsoleIO) extends ConsoleIO
final case class ReadLine(process:String => ConsoleIO) extends ConsoleIO
final case class object End extends ConsoleIO
```

注意构造一个`ReadLine`的值时我们需要提供一个函数，传入从控制台读到的行，返回另一个`ConsoleIO`，代表该副作用程序的剩余部分。

我们向`ReadLine`传入一个函数，它作为一个保证，将来的某个时间某人会通过控制台给我们一行输入，然后我们再将其返回给程序的”剩余“部分。

该结构有足够的能力来描述我们的交互程序。比如，下面的程序会问你的名字并向你 Say hello：

```scala
def socialProgram:ConsoleIO = WriteLine(
  "hello, what is your name?",
  ReadLine(name =>
    WriteLine("Hello, " + name + "!", End)
  )
)
```

记得这是一个完整的、确定的纯函数。人们想象使用这个结构来描述非常复杂的副作用程序。事实上，任何仅需要控制台 IO 的程序都可以使用该结构来描述。

注意任何使用`ConsoleIO`描述的程序都会在某个点终止。这些程序不能返回一个值。

如果我们需要这样的”控制台式程序“：该程序生成的东西在其他程序又能够使用。这样我们需要泛型化`End`来接收一些类型为`A`的值，这迫使我们需要给`ConsoleIO`添加一个新的类型参数`A`，并贯穿于其他项。

最终的结果看起来稍微有点复杂：

```scala
sealed trait ConsoleIO[A]
final case class WriteLine[A](line:String, then:ConsoleIO[A]) extends ConsoleIO[A]
final case class ReadLine[A](process: String => ConsoleIO[A]) extends ConsoleIO[A]
final case class EndWith[A](value: A) extends ConsoleIO[A]
```

现在，`ConsoleIO[A]`能够描述一个读写控制台并被一个类型为`A`的值终止的程序。这支持我们构建生成值的程序，然后这些值再被其他程序消费。

我们能够使用该结构创建之前的”Hello, !“程序，但这次，我们能从程序中返回用户的名字：

```scala
val userNameProgram:ConsoleIO[String] = WriteLine(
  "Hello, what is your name?",
  ReadLine(name =>
    WriteLine("Hello, " + name + "!", EndWith(name))  
  )
)
```

`ConsoleIO`中我们唯一丧失的是修改返回值类型的能力。比如，你想构建另一个程序并不是返回用户名，而是名字的长度，如何复用`userNameProgram`呢？

当前这种方式是不可能的。我们需要一些更强大的东西来实现这个打算。如果我们拥有一个 List 则可以使用 map 来改变结果类型。而这正是`ConsoleIO`所需要的。

我们可以直接给他添加一个 map 函数：

```scala
sealed trait ConsoleIO[A]{
  def map[B](f: A=>B):ConsoleIO[B] = Map(this, f)
}
final case class WriteLine[A](line: String, then: ConsoleIO[A]) extends ConsoleIO[A]
final case class ReadLine[A](process: String => ConsoleIO[A]) extends ConsoleIO[A]
final case class EndWith[A](value: A) extends ConsoleIO[A]
final case class Map[A0, A](v: ConsoleIO[A0], f: A0 => A) extends ConsoleIO[A]
```

现在给出任何一个`ConsoleIO[A]`，我们都可以通过函数`A => B`将其转换为`ConsoleIO[B]`。因此现在我们可以编写一个新的`userNameLenProgram`，计算用户名字的字符长度：

```scala
def userNameLenProgram:ConsoleIO[Int] = userNameProgram.map(_.length)
```

随着`map`的加入，

未完成....