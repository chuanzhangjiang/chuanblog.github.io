---
title: scala宏学习文档
date: 2021-02-01 21:55:29
tags:
- scala 
- 宏
---

- [前言](#前言)
- [创建测试项目](#创建测试项目)
- [定义一个宏函数](#定义一个宏函数)
- [抽象语法树(AST)和q插值器](#抽象语法树ast和q插值器)
- [定义具有类型参数的宏方法](#定义具有类型参数的宏方法)
- [使用macro bundle的方式定义宏](#使用macro-bundle的方式定义宏)
- [定义隐式宏](#定义隐式宏)
- [提取器宏](#提取器宏)
- [](#)

## 前言
此文适用于scala 2.13.x版本。

## 创建测试项目
由于宏和测试用代码不能存在于同一个编译环境下，故使用sbt创建一个主项目，然后在主项目中创建子项目，用来存放宏代码,下面开始干。

首先，创建项目目录,进入项目目录：
```
$ mkdir macro-doc-sample
$ cd macro-doc-sample
```
新建build.sbt文件：
```
$ touch build.sbt
```
打开build.sbt输入如下配置：
``` scala
lazy val scalav = "2.13.3"

// 所有项目的公用配置
lazy val commonSettings = Seq(
  scalaVersion := scalav,
  organization := "me.zjc"
)

//scala宏在scala反射库中，需要手动引入
lazy val scalaReflect = "org.scala-lang" % "scala-reflect" % scalav

// 根目录项目
lazy val root = (project in file("."))
  .dependsOn(macros)
  .settings(
    name := "macro-doc-sample",
    commonSettings
  )

// 宏项目
lazy val macros = (project in file("macros"))
  .settings(
    name := "macros",
    commonSettings,
    libraryDependencies += scalaReflect
  )
```
使用sbt加载项目：
```
$ sbt
```
加载完毕，在两个项目中分别创建源码文件夹：
```
$ mkdir -p src/main/scala/me/zjc/macrodocsample
$ mkdir -p macros/src/main/scala/me/zjc/macros
```
OK，可以开始写宏了！

## 定义一个宏函数
在macro项目源码目录新建文件HelloMacro.scala，输入如下代码:
``` scala
package me.zjc.macros

import scala.reflect.macros.whitebox.Context
import scala.language.experimental.macros

object HelloMacro {
  def hello(msg: String): Unit = macro helloImpl

  def helloImpl(c: Context)(msg: c.Expr[String]): c.Expr[Unit] = {
    import c.universe._
    val result = q"""println("hello " + $msg)"""
    c.Expr(result)
  }
}
```
首先来看开头导入的`scala.reflect.macros.whitebox.Context`和`scala.language.experimental.macros`，导入前者是为了可以使用宏环境(Context),宏环境可以理解为宏在运行时所以依赖的环境，编写宏需要从这个环境中获取各种各样的信息，宏还可以通过这个环境中反馈一些信息给用户，环境分为白盒环境(whitebox.Context)和黑盒环境(blackbox.Context),具体区别以后在说，目前不做讨论。导入后者是为了能正常使用`macro`关键字。

接下来看两个方法`Hello`和`HelloImpl`,我们写好宏之后，我们只调用`hello`方法，并不会去直接使用`HelloImpl`方法,使用`hello`方法就像调用普通的方法一样：
``` scala
HelloMacro.hello("macro")
```
在编译器编译之后，会对上面的方法进行宏展开，所以在我们眼里代码是长上面这个样子的，但是在JVM眼中，代码长下面这个样子：
``` scala
println("hello " + "macro")
```
OK，现在我们来观察`helloImpl`,顾名思义，这是`hello`宏方法的具体实现，定义`helloImpl`方法是讲究格式的，第一个参数必须是`Context`宏环境，第二个参数必须和`hello`方法的第一个参数同名，如果`hello`有多个参数依次类推，这些参数的类型需要为`c.Expr[hello方法参数对应类型]`，返回类型`c.Expr[hello方法返回值类型]`,当然，如果你想图方便可以将参数类型和返回类型统一设置为`c.Tree`。

## 抽象语法树(AST)和q插值器
开始写宏的具体实现之前我们需要对scala的抽象语法树和q插值器进行介绍,抽象语法树即代码树，是代码的树型表示法，打个比方，一个简单的代码片段：
```
AObjct.bFunc()
```
转换为抽象语法树之后将会变为如下：
```
Apply(Select(Ident(TermName("AObjct")), TermName("bFunc")), List())
```
本质是，scala2版本的宏就是对着上面这玩意儿编程，我们在把目光放到`helloImpl`方法上,它接收一个参数`msg: c.Expr[String]`,其实他本质上接收的就是一个抽象语法树，比如当我们给`hello`方法传入`"macro"`时，`helloImpl`方法接受到的参数长这样:
```
Literal(Constant("macro"))
```
聪明的你可能已经注意到，`helloImpl`方法的返回值依然是一个`c.Expr[Unit]`,所以当我们想要让`hello`方法宏展开之后变为`println("hello macro")`的话，我们就需要将`println("hello macro")`变成一颗抽象语法树返回回来，它的语法树长这样:
``` scala
Apply(Ident(TermName("println")), List(Literal(Constant("hello macro"))))
```
所以`helloImpl`方法应该长这样:
``` scala
def helloImpl(c: Context)(msg: c.Expr[String]): c.Expr[Unit] = {
  import c.universe._
  val tree =
    Apply(Ident(TermName("println")), List(Literal(Constant("hello macro"))))
  c.Expr(tree)
}
```
好的，我们新建一个Main(src/main/scala/me/zjc/macrodocsample/Main.scala)对象测试一下：
``` scala
package me.zjc.macrodocsample

import me.zjc.macros.HelloMacro

object Main extends App {
  HelloMacro.hello("")
}
```
使用sbt运行项目:
```
$ sbt
$ run
```
不出意外可以成功打印"hello macro",上面的例子并没有运用上传入参数msg，就已经让人头疼了，更何况以后更复杂的情况。所以q插值器出现了，上面的例子如果使用q插值器，将会变成你所熟悉的样子：
``` scala
def helloImpl(c: Context)(msg: c.Expr[String]): c.Expr[Unit] = {
  import c.universe._
  val tree = q"""println("hello macro")"""
  c.Expr(tree)
}
```
再次运行项目，得到同样的结果，此时我们还可以将msg参数代入到抽象语法树中:
``` scala
def helloImpl(c: Context)(msg: c.Expr[String]): c.Expr[Unit] = {
  import c.universe._
  val tree = q"""println("hello " + $msg)"""
  c.Expr(tree)
}
```
q插值器可以将我们输入的字符串转换成抽象语法树，就是这么神奇。具体的q插值器的使用还可以参考翔一样的[官方文档。](https://docs.scala-lang.org/overviews/quasiquotes/intro.html)

## 定义具有类型参数的宏方法
我们在macro子项目中新建文件`MacroWithType.scala`:
``` scala
package me.zjc.macros

import scala.language.experimental.macros
import scala.reflect.macros.whitebox.Context

object MacroWithType {
  def getNonParamObj[T]: T = macro getNonParamObjImpl[T]
  def getNonParamObjImpl[T: c.WeakTypeTag](c: Context): c.Expr[T] = {
    import c.universe._
    val t = implicitly[c.WeakTypeTag[T]]
    val result = q"new $t()"
    c.Expr(result)
  }
}
```
定义impl方法时候加入bundle context参数`c.WeakTypeTag`即可，使用上面的例子可以用来实例化所有构造器可以不需要传入参数的类，我们可以来测试一下，下面我们用scalaTest来测试我们的代码,加入scalaTest依赖:
``` scala
// 宏项目
lazy val macros = (project in file("macros"))
  .settings(
    name := "macros",
    commonSettings,
    libraryDependencies ++= Seq(
      scalaReflect,
      "org.scalatest" %% "scalatest" % "3.2.2" % "test"
    )
  )
```
创建测试目录`macros/src/test/scala/me/zjc/macros/`,新建文件`MacroWithTypeTest.scala`:
``` scala
package me.zjc.macros

import org.scalatest.funsuite.AnyFunSuite

class MacroWithTypeTest extends AnyFunSuite {
  private class A()
  test("generate a no param constructor class A") {
    val a = MacroWithType.getNonParamObj[A]
    assert(a.isInstanceOf[A])
  }
}
```
运行测试:
```
$ sbt
$ project macros
$ test
```
不出意外将会看见all tests passed。

## 使用macro bundle的方式定义宏
因为比较好理解，直接show code，创建`MacroWithBundle.scala`:
``` scala
package me.zjc.macros

import scala.language.experimental.macros
import scala.reflect.macros.whitebox.Context

object MacroWithBundle {

  def macroWithBundleFunc: String = macro Macros.impl

  // 这就是macro bundle
  private class Macros(val /*这个val不能漏掉*/ c: Context) {
    import c.universe._
    def impl: c.Expr[String] = {
      val resultStr = "hello macro"
      c.Expr(q"""$resultStr""")
    }
  }
}
```
测试代码:
``` scala
package me.zjc.macros

import org.scalatest.funsuite.AnyFunSuite

class MacroWithBundleTest extends AnyFunSuite {
  test("""invok macro method will get a String value "hello macro" """) {
    assert(MacroWithBundle.macroWithBundleFunc == "hello macro")
  }
}
```

## 定义隐式宏
新建一个文件`ImplicitMacro.scala`：
``` scala
package me.zjc.macros

object ImplicitMacro {

  trait PrettyShowUtil[T] {
    def show(x: T): String
  }

  def show[T](x: T)(implicit prettyShowUtil: PrettyShowUtil[T]): String =
    prettyShowUtil.show(x)
}
```
这里定义了一个trait:`PrettyShowUtil`,这个trait里有一个`show`方法用于将某一类型的对象以某种字符串的形式返回回来。我们还在trait外面定应了一个`show`方法，用于简化`PrettyShowUtil`的`show`方法的调用，作用域内只要存在一个适当类型的`PrettyShowUtil`就能方便的调用`show`方法，这里创建一个测试文件`ImplicitMacro.scala`：
``` scala
package me.zjc.macros

import org.scalatest.funsuite.AnyFunSuite

class ImplicitMacroTest extends AnyFunSuite {
  import ImplicitMacro._
  test("""class A will return a string "A(someParam)" """) {
    //重点代码开始
    class A(val value: String)
    implicit val aPrettyShowUtil = new PrettyShowUtil[A] {
      override def show(x: A): String = s"A(${x.value})"
    }
    //重点代码结束
    assert(show(new A("hello")) == "A(hello)")
  }
}
```
此时当我们出现一个类B的时候，又得在定义一个`PrettyShowUtil[A]`：
``` scala
class A(val value: String)
implicit val aPrettyShowUtil = new PrettyShowUtil[A] {
  override def show(x: A): String = s"A(${x.value})"
}

class B(val value: String)
implicit val bPrettyShowUtil = new PrettyShowUtil[B] {
  override def show(x: B): String = s"B(${x.value})"
}
```
此时我们可以使用隐式宏来解此问题，我们继续完善我们的`ImplicitMacro.scala`：
``` scala
package me.zjc.macros

import scala.language.experimental.macros
import scala.reflect.macros.whitebox.Context

object ImplicitMacro {

  trait PrettyShowUtil[T] {
    def show(x: T): String
  }

  def show[T](x: T)(implicit prettyShowUtil: PrettyShowUtil[T]): String =
    prettyShowUtil.show(x)

  //新增代码
  implicit def materialize[T]: PrettyShowUtil[T] =
    macro materializeImpl[T]

  //新增代码
  def materializeImpl[T: c.WeakTypeTag](
      c: Context
  ): c.Tree = {
    import c.universe._
    ???
  }
}
```
我们把主要精力集中在`materializeImpl`方法上:
``` scala
def materializeImpl[T: c.WeakTypeTag](
      c: Context
  ): c.Tree = {
    import c.universe._
    val t = implicitly[c.WeakTypeTag[T]]
    //如对于类class A(val v1: String, val v2: Int),将转换成List(x.v1, x.v2)这种格式
    val paramValues = t.tpe.typeSymbol.asClass.primaryConstructor.asMethod
      .paramLists(0)
      // .map(param => q"x.$param")这种方式无法访问到param，因为这里是private类型的
      .map(param => q"x.${TermName(param.name.toString())}")
    q"""
    new PrettyShowUtil[$t] {
      def show(x: $t): String = {
        val paramsTemp = $paramValues
        paramsTemp.mkString(${t.tpe.toString()} + "(", ", ", ")")
      }
    }
    """
  }
```
在测试文件中添加测试方法:
``` scala
test(" pretty show class by macro ") {
  class A(val value: String, val value2: Int)
  val value01 = "yo,~"
  val value02 = 2
  assert(show(new A(value01, value02)) == s"A($value01, $value02)")
}
```
## 提取器宏
这部分没学好,求助各位网友给点资料学习，暂时没看出来比普通提取器强大在哪里，直接附上[官方文档地址](https://docs.scala-lang.org/overviews/macros/extractors.html)吧，留个坑以后填。

## 