title: Scala-Language
date: 2015-09-12 11:21:10
categories: 编程语言
tags: [Scala]
---

#### 基本数据结构
- 列表List `val numbers = List(1, 2, 3, 4)`
- 集合Set `Set(1, 1, 2)`
- 元组Tuple `val hostPort = ("localhost", 80)`
  - 元组的取值 `hostPort._1`
  - 元组的特殊语法创建 `1 -> 2`
- 映射Map `Map(1 -> 2)` `Map(1 -> "one", 2 -> "two")`
- 选项Option: 表示一个有可能包含值的容器
  - Some
  - None
  - List.get返回的是Option类型, Option本身是泛型的

<!-- more -->

#### 函数组合子
- map: 对列表中的每个元素应用一个函数, 返回应用后的元素所组成的列表
  - `numbers.map((i: Int) => i * 2)`
  - 传入一个部分应用函数 `numbers.map(timesTwo _)`

- foreach: 和map类型, 但是不返回值, 尝试返回的话, 返回类型是void

- filter: 移除任何对传入函数计算结果为false的元素
  - `numbers.filter(_ % 2 != 0)`

- zip: 将两个列表的内容集合到一个对偶列表中
  - `List(1,2,3,4).zip(List(5,6,7,8))` 返回 List[(Int, String)]

- partition: 将使用给定的谓词函数做分割列表
  - `numbers.partition(_ % 2 == 0)` 返回 (List[Int], List[Int])

- find: 返回集合中第一个匹配谓词函数的元素
  - `numbers.find((i: Int) => i > 5)` 返回 Option[Int] = Some(6)

- drop & dropWhile:
  - drop: 将删除前i个元素:
    - `numbers.drop(5)` 返回List(6,7,8,9,10)
  - dropWhile: 将删除元素直到找到第一个匹配谓词函数的元素
    - `numbers.dropWhile(_ % 2 != 0)` 返回List(2,3,4,5,6,7,8,9)

- foldLeft: 其实类型于python的中reduce功能
  - `numbers.foldLeft(0)((m: Int, n: Int) => m + n)` 0为初始值, m为累加器
  - `numbers.foldLeft(0) { (m: Int, n: Int) => println("m: " + m + " n: " + n); m + n }` 直接观察运行过程

- foldRight: 和foldLeft相反
  - `numbers.foldRight(0) { (m: Int, n: Int) => println("m: " + m + " n: " + n); m + n }`

- flatten: 将嵌套结构扁平化为一个层次的集合
  - `List(List(1, 2), List(3, 4)).flatten` 返回List(1,2,3,4)

- flatMap: 一个常用的组合子, 结合映射map和flatten. 需要一个处理嵌套列表的函数, 将结果串联起来, 可以看做是先扁平化在映射的快捷操作
  - `val nestedNumbers = List(List(1, 2), List(3, 4))`
  - `nestedNumbers.flatMap(x => x.map(_ * 2))`
  - 或者先映射在扁平化处理 `nestedNumbers.map((x: List[Int]) => x.map(_ * 2)).flatten`

#### 扩展函数组合子


```scala
def ourMap(numbers: List[Int], fn: Int => Int): List[Int] = {
  numbers.foldRight(List[Int]()) { (x: Int, xs:List[Int]) =>
    fn(x) :: xs
  }
}
```


```scala
  def timesTwo(x: Int) = x * 2
```


```scala
  ourMap(numbers, timesTwo(_))
  ourMap(numbers, timesTwo)
```

#### Map?
  - 所有展示的函数组合子都可以在Map上使用. Map可以被看作是一个二元组的列表, 所以你写的函数要处理一个键和值的二元组.
  - `val extensions = Map("steve" -> 100, "bob" -> 101, "joe" -> 201)`
  - `extensions.filter((namePhone: (String, Int)) => namePhone._2 < 200)`
  - 使用模式匹配 `extensions.filter({case (name, extension) => extension < 200})`

#### 函数组合
- compose 组合其他函数形成一个新的函数f(g(x))
  - `val fComposeG = f _ compose g _`
  - fComposeG("panqd") 返回 f(g(panqd))

- andThen:
  - 和compose很像, 但是调用顺序是先调用第一个函数, 在调用第二个函数
  - `val fAndThenG = f _ andThen g _ `
  - fAndThenG = g(f(panqd))

#### 柯里化 VS 偏应用
- case语句是一个名为PartialFunction的函数的子类
- 多个case语句的集合是共同组合在一起的多个PartialFunction(偏函数)

- 理解偏函数
  - 对于给定的输入类型, 函数可接受该类型的任何值. 一个Int => String的函数可以接受任意的int的值, 并返回一个字符串.
  - 对于给定的输入类型, 偏函数只能接受该类型的某些特定的值. 一个Int => String的偏函数可能不能接受所有的int的值为输入.
  - 偏函数和部分函数应用无关
  - `val one: PartialFunction[Int, String] = { case 1 => "one" }`
  - `one.isDefinedAt(1) 返回true`
  - `one.isDefinedAt(2) 返回false`
  - 偏函数可以使用orElse组成新的函数, 得到的偏函数反应了是否对给定的参数进行了定义
    - `val wildcard: PartialFunction[Int, String] = { case _ => "something else" }`
    - `val p = one orElse two orElse three orElse wildcard` `p(5)`

#### Scala中的类型
- 参数化多态性(简单的说就是泛型编程)
  - `2 :: 1 :: "bar" :: "foo" :: Nil` 创建一个List
  - `head` 返回Any类型, 即已经无法得知原来的类型
  - `def drop1[A](l: List[A]) = l.tail`
- 局部类型推断
  - `def id[T](x: T) = x`
  - `val x = id(322)`
  - `val x = id("hey")`
  - `val x = id(Array(1,2,3,4))`
  - 协变, 逆变, 不变, 边界, 量化
- 存在量化
- 视窗

#### 高级集合
- List 标准的链表
- Set 集合没有重复
- Seq 有一个给定的序列
- Map 键值对的映射容器

#### 层次结构, 下面介绍的都是特质, 在可变和不可变包中都有实现
- Traversable: 均可被遍历, 这个特质定义了标准的函数组合子, 根据foreach来写
- Iterable: iterator()方法返回一个Iterator来迭代元素
- Seq:
- Set:
- Map:

#### 方法:
- Traversable:
  - def head : A 获取第一个元素
  - def tail : Traversable[A] 获取除第一个元素外的集合
  - def map [B] (f: (A) => B) : CC[B] 返回每个元素都被f转化的集合
  - def foreach [X] (f: Elem => X): Unit 在每个元素上执行f
  - def find (p: (A) => Boolean) : Option[A] 返回匹配的第一个元素
  - def filter (p: (A) => Boolean) : Traversable[A] 过滤出符合谓词函数的元素
  - def partition (p: (A) ⇒ Boolean) : (Traversable[A], Traversable[A]) 根据谓词函数划分集合
  - def groupBy [K] (f: (A) => K) : Map[K, Traversable[A]] 转换

    ```scala
    def toArray : Array[A]
    def toArray [B >: A] (implicit arg0: ClassManifest[B]) : Array[B]
    def toBuffer [B >: A] : Buffer[B]
    def toIndexedSeq [B >: A] : IndexedSeq[B]
    def toIterable : Iterable[A]
    def toIterator : Iterator[A]
    def toList : List[A]
    def toMap [T, U] (implicit ev: <:<[A, (T, U)]) : Map[T, U]
    def toSeq : Seq[A]
    def toSet [B >: A] : Set[B]
    def toStream : Stream[A]
    def toString () : String
    def toTraversable : Traversable[A]
    ```

- Iterable: 添加一个迭代器的访问 `def iterator: Iterator[A]`
  - def hasNext(): Boolean
  - def next: A

- Set:
  - def contains(key: A): Boolean
  - def +(elem: A): Set[A]
  - def -(elem: A): Set[A]

- Map:
  - 构建方式 `Map.empty ++ List(("a", 1), ("b", 2), ("c", 3))`

- 常用的子类:
  - HashSet
  - HashMap
  - TreeMap
  - Vector: `IndexedSeq(1, 2, 3)`
  - Range: `for (i <- 1 to 3) { println(i) }`
    - `(1 to 3).map { i => i }`

- 默认实现:
  - `Iterable(1, 2)` `res0: Iterable[Int] = List(1, 2)`
  - `Seq(1, 2)` 同上
  - `Set(1, 2)` `res31: scala.collection.immutable.Set[Int] = Set(1, 2)`

### 可变集合
- collection.mutable.Map
  - getOrElseUpdate
  - +=
- ListBuffer和ArrayBuffer
- LinkedList and DoubleLinkedList
- PriorityQueue
- Stack 和 ArrayStack
- StringBuilder 有趣的是，StringBuilder的是一个集合

#### 与java的生活
- 通过**JavaConverters package**轻松地在Java和Scala的集合类型之间转换

  ```scala
  import scala.collection.JavaConverters._
    val sl = new scala.collection.mutable.ListBuffer[Int]
    val jl : java.util.List[Int] = sl.asJava
    val sl2 : scala.collection.mutable.Buffer[Int] = jl.asScala
    assert(sl eq sl2)
  ```

- 双向转换

  ```scala
  scala.collection.Iterable <=> java.lang.Iterable
  scala.collection.Iterable <=> java.util.Collection
  scala.collection.Iterator <=> java.util.{ Iterator, Enumeration }
  scala.collection.mutable.Buffer <=> java.util.List
  scala.collection.mutable.Set <=> java.util.Set
  scala.collection.mutable.Map <=> java.util.{ Map, Dictionary }
  scala.collection.mutable.ConcurrentMap <=> java.util.concurrent.ConcurrentMap
  ```

- 单向转换

  ```scala
  scala.collection.Seq => java.util.List
  scala.collection.mutable.Seq => java.util.List
  scala.collection.Set => java.util.Set
  scala.collection.Map => java.util.Map
  ```