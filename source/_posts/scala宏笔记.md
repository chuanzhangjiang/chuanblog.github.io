
---
title: scala宏笔记
date: 2021-01-29 14:59:25
tags:
- scala
- 编程
- 宏
---

## 前言
这是备忘录，除非你时间很多，否则不建议你阅读。
## 宏代码和测试代码不能在同一个编译环境中
[sbt宏项目](https://www.scala-sbt.org/1.x/docs/Macro-Projects.html)
## 定义宏函数
``` scala
import scala.language.experimental.macros //使得函数体可以使用macro做前缀
import scala.reflect.macros.whitebox.Context //宏所需的环境

object MacroTest {

  def macroTest(input: String): Unit = macro macroTestImpl

  def macroTestImpl(c: Context)(input: c.Tree): c.Tree = {
    import c.universe._
    q"println($input)"
  }
}
```
宏函数`macroTest`调用了他的实现`macroTestImpl`,`macroTestImpl`格式需要注意两点，参数`c: Context`必须作为地一个参数传入，独占一个括号;第二个括号中的参数必须和宏函数`macroTest`的第一个括号中的参数一一对应，对应原则：参数名必须相同，参数类型统一替换为`c.Tree`(c.Expr[_]的情况暂不讨论)。

下面是几种定义样例
``` scala
object MacroTest {

  def macroTest[T](input: String, input2: Any): T = macro macroTestImpl[T]

  def macroTestImpl[T: c.WeakTypeTag](
      c: Context
  )(input: c.Tree, input2: c.Tree): c.Tree = {
    import c.universe._
    ???
  }
}
```

``` scala
object MacroTest {

  def macroTest(input: String*): Unit = macro macroTestImpl

  def macroTestImpl(
      c: Context
  )(input: c.Tree*): c.Tree = {
    import c.universe._
    ???
  }
}
```

``` scala
object MacroTest {

  def macroTest(input: String)(input2: String): Unit = macro macroTestImpl

  def macroTestImpl(
      c: Context
  )(input: c.Tree)(input2: c.Tree): c.Tree = {
    import c.universe._
    ???
  }
}
```

## 关于`Tree`
### 方法调用
`a.b`相当于对应`Select（a, b）`
### 参数传递
`a(b)`相当于`Apply(a, List(b))`
### 常量表示
`Literal(Constant("hello"))`
### 变量表示
`TermName("a")`
### 其他
可以使用`showRaw`方法打赢`q`插值器的内容查看各种对应关系。

## 写宏速查
### 获取传入类型参数的类型
``` scala
val t: c.WeakTypeTag[T] = implicitly[c.WeakTypeTag[T]]
```
### 判断一个类是否是case类型
使用`ClassSymbol.isCaseClass`,比如从`t: c.WeakTypeTag[T]`中获取该类型是否是caseClass：
``` scala
t.tpe.typeSymbol.asClass.isCaseClass
```
### 获取某个类的伴生对象对应的symbol
使用`ClassSymbol.companion`,比如从`t: c.WeakTypeTag[T]`中获取该类对应的伴生对象:
``` scala
t.tpe.typeSymbol.asClass.companion
```
### 获取某个类的主构造其symbol
使用`ClassSymbol.primaryConstructor`,比如从`t: c.WeakTypeTag[T]`中获取该类对应的主构造器:
``` scala
t.tpe.typeSymbol.asClass.primaryConstructor
```
### 获取方法的参数
使用`MethodSymbol.paramLists`

### 获取方法中的参数是否存在默认值
使用`TermSymbol.isParamWithDefault`,如将`t: c.WeakTypeTag[T]`的主构造器中存在默认值的参数的名字存入List中返回回来：
``` scala
object MacroTest {

  def macroTest[T]: List[String] = macro macroTestImpl[T]

  def macroTestImpl[T: c.WeakTypeTag](c: Context): c.Tree = {
    import c.universe._
    val t = implicitly[c.WeakTypeTag[T]]
    val defaultParamNameList =
      t.tpe.typeSymbol.asClass.primaryConstructor.asMethod.paramLists.flatten
        .filter(_.asTerm.isParamWithDefault)
        .map(_.asTerm.name.toString())
    q"""
    List(..$defaultParamNameList)
    """
  }
}
```
使用
``` scala
object Main {
  case class A(normal1: String, default: String = "a", default2: Int = 2)
  def main(args: Array[String]): Unit = {
    println(MacroTest.macroTest[A])
  }
}
```
输出结果
```
List(default, default2)
```

### 获取某一类的所有成员
`Type.members`,如获取`t: c.WeakTypeTag[T]`中的所有成员:
``` scala
t.tpe.members
```

### 获取方法中参数的默认值
获取默认参数值的方法命名规则：
```
//类中可以拿到
<方法名>$default$<参数下标>
```
对于case类中，主构造函数中的默认参数方法命名规则：
```
// 需要在伴生对象中拿到
$lessinit$greater$default$<参数下标>
```

### 使用`q`构造AST时，需要传入参数类型
可以直接使用`t: c.WeakTypeTag[T]`:
```
q"def foo[$t]:$t = bar"
```
也可以使用`Type`类型
```
q"def foo[${t.tpe}]: ${t.tpe} = bar"
```

### 构造一个类
直接使用伴生对象:
```
q"$companion(..$args)"
```
或者使用老朋友`t: c.WeakTypeTag[T]`：
```
q"new $t(..$args)"
```

### 获取前缀`c.prefix`
比如存在：
```
class Foo[T] {
  def bar = macro ???
}
```
那么：
```
new Foo[String].bar
```
的`c.prefix.tree`就是`new Foo[String]`,就相当于点的左边。

### 定义字符串插值器
``` scala
implicit class TestStringContext(sc: StringContext) {
    def test(args: Any*): String = ???
}
```
使用
``` scala
val a = ???
test"hello $a"
// 相当与
val a = ???
new TestStringContext(new StringContext("hello", "")).test(a)
```

### 主动提示异常
``` scala
c.error(c.enclosingPosition, "异常提示")
EmptyTree
```

### 最后一点
别害怕，print大法好，哪里有疑问print哪里:
``` scala
def defaultParamValues[T]: Map[String, _] = macro defaultParamValuesImpl[T]

def defaultParamValuesImpl[T: c.WeakTypeTag](c: Context): c.Tree = {
    import c.universe._
    val t = implicitly[c.WeakTypeTag[T]]
    q"""
    println(${showRaw(t)})
    ???
    """
  }
```
`showRaw`太看不清楚就加个`toString`
``` scala
println(${showRaw(t.toString)})
```
`toString`不准确就去掉它。