---
layout: post
title: "soot tutorial"
subtitle: "from smart leki"
date: 2022-09-27 21:12:00
author: "HapppyTsing"
catalog: false
header-style: text
tags:
  - soot
---

# IR

soot 包含四种 IR 形式：

- Baf：基于栈的 bytecode
- Jimple：**soot 的核心**，三地址码。Jimple 适用于绝大多数的不需要精确 control flow 或者 SSA 的静态分析。
- Simple：Static Single Assignment 版的 Jimple，保证了每个局部变量都有一个静态定义，不知道有啥用。
- Grimp：和 Jimple 类似，多了允许树形表达和 new 指令。相比于 Jimple，更贴近 Java code，所以更适合人来读。

```java
public static void main(String[] args) {
```

## Jimple

Soot 能直接创建 Jimple 码，也可由 Java source code、bytecode or class files 创建。

相对于 bytecode 的 200 多种指令，Jimple 只有 15 条，分别是：

- 核心指令：NopStmt、IdentityStmt、AssignStmt
- 函数内控制流指令：IfStmt、GotoStmt\TableSwitchStmt、LookUpSwitchStmt
- 函数间控制流：InvoeStmt、ReturnStmt、ReturnVoidStmt
- 监视器指令：EnterMonitorStmt、ExitMonitorStmt
- 处理异常：ThrowStmt
- 退出：RetStmt

要对 Jimple 操作，首先需要实例化一个 Jimple 的环境对象 scene :

```java
val scence = Scene.v()
```

然后，再对 scene 进行分析和操作

```java
package com.wang;

public class Foo {
    public static void main(String[] args) {
        Foo f = new Foo();
        int a = 7;
        int b = 14;
        int x = (f.bar(21) + a) * b ;
        System.out.println(x);
    }
    public int bar(int n) { return n + 42; }
}
```

上述 Java 代码转化为 Jimple 后如下：

```java
public class com.wang.Foo extends java.lang.Object
{

    public void <init>()
    {
        com.wang.Foo r0;

        r0 := @this: com.wang.Foo;

        specialinvoke r0.<java.lang.Object: void <init>()>();

        return;
    }

    public static void main(java.lang.String[])
    {
        java.io.PrintStream $r1;
        com.wang.Foo $r0;
        int $i0, $i1, i2;
        java.lang.String[] r2;
        r2 := @parameter0: java.lang.String[];
        $r0 = new com.wang.Foo;
		// InvokeStmt, specialinvode 调用构造函数. virtualinvode 大多数调用都是这个。
        specialinvoke $r0.<com.wang.Foo: void <init>()>();
        $i0 = virtualinvoke $r0.<com.wang.Foo: int bar(int)>(21);     // 也是 AssignStmt
        $i1 = $i0 + 7;
        i2 = $i1 * 14;
        $r1 = <java.lang.System: java.io.PrintStream out>;
        virtualinvoke $r1.<java.io.PrintStream: void println(int)>(i2);
        return;
    }

    public int bar(int)
    {
        int i0, $i1;
        com.wang.Foo r0;
        r0 := @this: com.wang.Foo;   // IdentityStmt
        i0 := @parameter0: int;      // IdentityStmt
        $i1 = i0 + 42;               // AssignStmt
        return $i1;                  // ReturnStmt
    }
}
```

在 Soot 中，既有与 Java 对应的概念：

- 类 Class
- 字段 Filed
- 方法 Method

还有 Soot 自定义的概念：

- Unit：在 Jimple 中的实现是 stmt，在 Grimp 中的实现是 Inst
- Box
- Value
- CallGraph
- ...

### Scene

scene 中可以直接获取 Java Source 中的所有 Class，每一个 Class 都会生成一个 Jimple 文件与之对应。如果是内部类，名称前加其所在类的名称并用`$`符号连接。

> # Tips
>
> 使用 javac 编译一个名为`outClass`的类，其内部存在一个内部类`inClass`，使用`javac outclass`后，会生成两个文件：
>
> - outClass.class
> - outClass$inclass.class
>
> 故而，scene 本质上应该就是做了编译工作？此处存疑！

通过 Scene，在开启了`whole program mode`的情况下，还可以用于生成 `Call Graph`。

### Class

class 代表一个 Java Class 类，相应的通过 Jimple 可以获取 class 中包含的：

- 字段 Field
- 方法 Method
- class 的父类方法 superClass
- 继承的接口 Interface 等。

根据 Java 的定义，每一个类会包含有构造器，因此 class 的 methods 中一定会有一个构造方法，方法名为`<init>`，参数与构造方法的参数相同。

同时，在主类 mainClass，即包含有 main 入口方法的类中，如果定义有全局变量，虽然这也属于 Field，但是这些变量的初始化操作在一个新的方法中执行，方法名为`<clinit>`。

```java
public class com.wang.Foo extends java.lang.Object
```

### Field

Class 中的字段即 Field，字段被引用或者赋值时，是通过 FieldRef 实现的。FieldRef 中包含了 declaringClass，name 等基本信息。

在 class 中通过`getFieldByName(String name)`获取字段时，如果没有该名字的字段，Jimple 会自动生成一个。

```java
java.lang.String[] r2;
```

### Method

方法 Method 的组成有参数 ParameterList，返回类型 ReturnType，方法体 Body。

Method 的 Body 是分析和修改 Method 的基础，包含了 Method 中所有的语句单元 Unit。

Method 也可以被调用，调用的方法为 MethodRef。

同样，在通过`getMethodByName(String name)`时，如果名字、参数、返回类型有一个不同，Jimple 会自动生成一个，并且该方法体中有 Error。

```java
com.wang.Foo: int bar(int)
```

### Body -> Unit

Body 代表了一个 Method 的方法体，每个 Body 里面有三个主链，分别是 Units 链、Locals 链、Traps 链：

- Units：方法的语句，在 Jimple 中，Unit 的实现就是 Stmt；而 Grimp 中，Unit 的实现是 Inst。
- Locals：方法的局部变量
- Traps：方法的异常处理

soot 中共有四种 Body：`BafBody`, `JimpleBody`,`ShimpleBody` and `GrimpBody`，在具体实现中，都继承了抽象类 Body。

执行语句 Unit 的类型 Stmt 常见的有:

```java
r0 := @this: com.wang.Foo;                            // IdentityStmt ThisRef
i0 := @parameter0: int;      						// IdentityStmt ParameterRef
$i1 = i0 + 42;              					    // AssignStmt
return $i1;                  						// ReturnStmt
specialinvoke $r0.<com.wang.Foo: void <init>()>();    // invokeStmt
```

### Value

Value 用于表示数据，Value 共有如下几类：

- `Local`
- `Constant`
- `Expr`
- `Ref`，如`ParameterRef`, `CaughtExceptionRef` and `ThisRef`

其中 Expr 是最有趣的，它可以细分为多种实现，例如：`NewExpr`和`AddExpr`。

一般来说，一个 Expr 可以对若干个 Value 进行一些操作并且返回另一个 Value。

```java
y = x + 1;    // AssignStmt
```

以上述的 AssignStmt 为例，其 leftOp 是 `y`，rightOp 是`x + 1`，此时`AddExpr`将其作为操作数，返回新的 Value。其中 `y` 是 `Local` 类型的 Value，而 `1`是 `Constant`类型的 Value。

### box

box 是一个指针，在 soot 中有两类 box：

- UnitBox
- ValueBox

#### UnitBox

每个 Unit 都会提供 getUnitBoxes() 方法，该方法大多数情况下返回空集，但代码中存在：

- GotoStmt：返回单个元素的列表，如下示例
- SwitchStmt：返回多个元素的列表

```java
    x = 5;
    goto l2;
    y = 3;
l2: z = 9;
```

此时对 `goto l2`这个 Unit 调用方法 `Unit.getUnitBoxes() `，会返回一个 UnitBox，指向 l2。

#### ValueBox

与 UnitBox 类似，是指向 Value 的指针。

1. 对于这个 AssignStmt 来说，需要首先获取他的 Boxes，这些 Boxes 里面包含了 Value 的指针。
2. 遍历 Boxes，cast 为 ValueBox，然后获取他的 Value，如果是 AddExpr 的话，那就是我们想优化的。
3. 对于 AddExpr 来说，获取他的左值和右值，也就是两个 value。
4. 如果都是常量 IntConstant 的话，那么就把他们的 value 加在一起。
5. 重新给 AssignStmt 的 box 赋值，让他指向 sum 和的常量。

```java
public void foldAdds(Unit u)
{
    Iterator ubIt = u.getUseBoxes().iterator();
    while (ubIt.hasNext())
    {
        ValueBox vb = (ValueBox) ubIt.next();
        Value v = vb.getValue();
        if (v instanceof AddExpr)
        {
            AddExpr ae = (AddExpr) v;
            Value lo = ae.getOp1(), ro = ae.getOp2();
            if (lo instanceof IntConstant && ro instanceof IntConstant)
            {
                IntConstant l = (IntConstant) lo,
                      r = (IntConstant) ro;
                int sum = l.value + r.value;
                vb.setValue(IntConstant.v(sum));
            }
        }
    }
}
```

# Phases

soot 的执行分为若干个`phase`，每个`phase`的实现称为`pack`。此外，每个`phase`又被细分为若干个`subphase`。

> # Tips
>
> 可以把 soot 理解为 maven，它们的执行都分为若干步骤，且可以通过参数选择开启或关闭特定的步骤！

执行的第一个`phase`，名为`jb`，该阶段解析 Class、Jimple、Source 文件，将解析结果 `Jimple Body`，输入不同此后的的 `phase`中。

此外，`jb`细分为`jb.a`等若干个`subphase`。

> # Tips
>
> Body 是方法体，程序内分析（intra-procedure analysis），其实就是分析单个方法内部的代码，如果该方法内部调用了其它方法，不会进行深入的分析。而程序间分析（inter-procedure analysis）则是在前者的基础上，对方法调用进行更为深入的分析。

如图所示，存在名为`jtp`的若干种`pack`，其命名规则为：

- 第一个字母，表示接受哪种形式的 IR 作为输入： s for Shimple, j for Jimple, b for Baf and g for Grimp.
- 第二个字母，表示`pack`的角色：b for body creation, t for user-defined transformations, o for optimizations and a for attribute generation (annotation).
- 最后一个字母 `p`，代表着 `pack`：例如 `jap`的含义是 `Jimple Annotation Pack`，包含了所有在 `intra-procedural analysis`中构建的内容。

值得特别关注的是：

- jtp(Jimple transformation pack)和 stp(Shimple transformation pack)。因为任何用户定义的 `BodyTransfer`(如从分析中得出对信息的标签) 可以被插入(inject)到这两个 `pack` 中，作为 Soot 执行过程的一部分。
- jap(Jimple annotation pack)：存放优化的结果，也可以使用`BodyTransfer`，作为 Soot 执行过程的一部分。
- jop，默认关闭，使用`-o`开启。

> ## Question
>
> 这是不是插桩？

![intra-procedure analysis](https://happytsing-figure-bed.oss-cn-hangzhou.aliyuncs.com/soot/image-20220919205001688.png)

上述及上图表示的是 intra-procedural analysis 的执行流程，但 inter-procedure analysis 略有不同。

首先，需要将 soot 置于 `Whole-program`模式，可以通过设置 option 为 `-w`来实现。

在该模式下，soot 首先为所有的方法执行 Jimple Body（jd），然后执行新增的四种全程序 `pack`：

- cg：call graph generation
- wjtp：whole Jimple transformation pack
- wjop, the whole-jimple optimization pack (通过 `-W`开启)
- wjap：whole Jimple annotation pack

这四种`pack`是可以改变的，为其添加 `SceneTransformers`进行一个全程序的分析。

![inter-procedure analysis](https://happytsing-figure-bed.oss-cn-hangzhou.aliyuncs.com/soot/image-20220919204941135.png)

使用下述命令获取 soot 支持的所有`pack`

```shell
java -cp $SOOT_HOME soot.Main -pl
# $PACH_NAME=wjtp
java -cp $SOOT_HOME soot.Main -ph $PACK_NAME
```

所有的 phases and subphases 都接受 option `enabled`，想要让该 phase 运行，`enabled`必须为`true`。为了方便书写，可以用：

- on，代替 enable:true，例如想开启 Spark，`-p cg.spark on`
- off，代替 enable:false

# The Data-Flow Framework

**前置知识**

Data Flow Analysis，可以理解为：How Data Flows on CFG?

每次语句的执行，如 Jimple 中的 stmt 执行，都会将输入状态转换为新的输出状态。数据流分析可以分为 forward Analysis 和 backward Analysis 两类。

在 intra-procedural Analysis 中，将不对方法调用进行细致分析。

在 CFG 中，结点有通常两种：

- 单独的三地址码，例如 Jimple 就是一种三地址码
- 基本块 Basic Blocks，是三地址码的集合，存在单进单出的性质等

此外，CFG 中分支 meet 时，处理方法有两种：

- may analysis，此时采用并集 ∪，即可以误报，但不许漏报。
- must analysis，此时采用交集 ∩ ，即可以漏报，但不许误报。

**soot 实现**

在 Soot 作者的文档中，其对做一个 flow analysis 分为四个步骤：

- Decide what the nature of the analysis is. 即考虑 forward Analysis or backward Analysis？是否需要考虑分支信息？
- Decide what is the intended approximation. 即考虑 may analysis or must analysis？这决定了在结点 meet 时，选择 ∪ or ∩
- performing the actual flow. 本质上是为中间表示的每个语句编写等式，例如，如何处理赋值语句？
- entry node and inner nodes 的初始状态. 通常初始状态为空集或全集。例如 available exporessions analysis 中，entry node 初始化为全集(1...1)，inner nodes 初始化为空集(0...0)。

## Step1: nature of the analysis

Soot 提供了三种实现：

- ForwardFlowAnalysis,
- BackwardFlowAnalysis
- ForwardBranchedFlowAnalysis.

前两种的结果是两个 Map：

- from nodes to IN sets
- from nodes to OUT sets

最后一种还添加了分支信息，因此多了一个 Map：

- from nodes to IN sets,
- from nodes to fall-through OUT sets
- from nodes to branch OUT sets.

上述三种都是通过 fixed-point mechanism using a worklist algorithm 来实现的，如果你想自己实现，可以通过继承如下三个类：AbstractFlowAnalysis (the top one), FlowAnalysis or BranchedFlowAnalysis.

举个例子：very-busy expressions，该分析是为了找到必定使用的表达式，例如：`if(x>y) then a=1;b=2; else a=1;`,此时`a=1`就是 very-busy expression。

为了进行 very-busy expressions analysis，需要使用 backward analysis，因此我们用于分析的类必须继承 BackwardFlowAnalysis。

然后为了利用 soot 提供的功能，需要提供一个实现如下功能的构造函数：

- call the super’s constructor
- invoke the fixed-point mechanism

具体实现如下：

```java
class VeryBusyExpressionAnalysis extends BackwardFlowAnalysis{
    public VeryBusyExpressionAnalysis(DirectedGraph g) {
        super(g);
        doAnalysis();
	}
}
```

## Step2: Approximation level

该步骤用于决定是 may or must analysis，其实就是决定在结点交汇时使用 ∪ or ∩。

仍旧以 very-busy expressions analysis 为例，它使用 must analysis，也就是在结点交汇（meet）时使用交集 ∩。

在 soot 的实现中，通过重写 `merge()` 方法，来确定 meet 的处理方法：

```java
class VeryBusyExpressionAnalysis extends BackwardFlowAnalysis{
    @Override
    protected void merge(Object in1, Object in2, Object out) {
        FlowSet inSet1 = (FlowSet)in1,
        inSet2 = (FlowSet)in2,
        outSet = (FlowSet)out;
        inSet1.intersection(inSet2, outSet);
    }
    @Override
    protected void copy(Object source, Object dest) {
        FlowSet srcSet = (FlowSet)source,
        destSet = (FlowSet)dest;
        srcSet.copy(destSet);
    }
}
```

在上述的例子中可以看到，soot 抽象了 lattice element 的表示方式，本例中使用`FlowSet`。因为这种抽象，因此需要重写 `copy()`方法，用于将 lattice element 的内容复制给另一个 lattice element。

## Step3: Performing flow

此处是数据流分析中的 `transfer function`，是核心部分。

该阶段的处理分为两个步骤：

- kill()：当流从 IN 经过 NODE 到 OUT 的时候，需要 kill 掉一些已经无效的数据。
- gen()：需要向 OUT 中加入 NODE 生成的数据。

也就是说，最终的 OUT[NODE] = gen<sub>NODE</sub> ∪ ( IN[NODE] - kill<sub>NODE</sub> )

```java
class VeryBusyExpressionAnalysis extends BackwardFlowAnalysis{
    @Override
    protected void flowThrough(Object in, Object node, Object out) {
        FlowSet inSet = (FlowSet)source,
        outSet = (FlowSet)dest;
        Unit u = (Unit)node;
        kill(inSet, u, outSet);
        gen(outSet, u);
	}
    private void kill(FlowSet inSet, Unit u, FlowSet outSet){ /* 用户自定义实现 */ }
    private void gen(FlowSet outSet, Unit u) { /* 用户自定义实现 */ }
}
```

其中 `kill()`和`gen()`方法需要用户根据实际的情况实现，可以参考[示例源代码](https://github.com/soot-oss/soot/blob/develop/tutorial/guide/examples/analysis_framework/src/dk/brics/soot/analyses/SimpleVeryBusyExpressions.java)。

## Step4: Initial state

该步骤用于对 entry node 以及其 inner node 进行初始化，分别由`entryInitialFlow` 和 `newInitialFlow`两个方法实现：

```java
class VeryBusyExpressionAnalysis extends BackwardFlowAnalysis{
    @Override
    protected Object entryInitialFlow() {
		return new ValueArraySparseSet();
	}
    @Override
	protected Object newInitialFlow() {
		return new ValueArraySparseSet();
	}
}

```

在 very-busy expressions analysis 中，entry node 是最后一句语句，并用空集来初始化它，其余的 inner node 也全部都初始化为空集。

## FlowSet

FlowSet 代表着与控制流图 CFG 中 结点 node 相关联的数据集。对于 very-busy expressions analysis，a node’s flow set 代表 a set of expressions busy at that node.

有两种 FlowSet：

- bounded FlowSet：A bounded set is one that knows its universe of possible values，实现接口`BoundedFlowSet<T>`
- unbounded FlowSet：与 bounded 相反，实现接口 `FlowSet<T>`

继承了`FlowSet<T>`的接口需要实现：

- clone()
- clear()
- isEmpty()
- copy(FlowSet dest) // deep copy of this into dest
- union(FlowSet other, FlowSet dest) // dest <- this ∪ other
- intersection(FlowSet other, FlowSet dest) // dest <- this ∩ other
- difference(FlowSet other, FlowSet dest) // dest <- this - other

In addition, when implementing `BoundedFlowSet<T>`, it needs to provide methods for producing the set's complement and its topped set (i.e., a lattice element containing all the possible values).

soot 实现了四种 FlowSet，分别是：ArraySparseSet、ArrayPackedSet、ToppedSet 和 DavaFlowSet

- **ArraySparseSet** is an unbounded flow set. The set is represented as an array of references

- **ArrayPackedSet** is a bounded flow set. Requires that the programmer provides a FlowUniverse object.

  A FlowUniverse object is simply a wrapper for some sort of collection or array, and it should contain all the possible values that might be put into the set.

- **ToppedSet** wraps another flow set (bounded or not) adding information regarding whether it is the top set (⊤) of the lattice.

In our very-busy expressions example, we need to have flow sets containing expressions and as such we want them to be compared for equivalence — i.e., two different occurrences of `a + b` will be different instantiations of some class implementing BinopExpr; thus they will never compare equal. To remedy this, we use a modified version of ArraySparseSet, where we have changed the implementation of the contains method as such:

```java
public boolean contains(Object obj) {
    for (int i = 0; i < numElements; i++)
        if (elements[i] instanceof EquivTo && ((EquivTo) elements[i]).equivTo(obj))
        	return true;
        else if (elements[i].equals(obj))
        	return true;
    return false;
}
```

## CFG

soot 在 package `soot.toolkits.graph`中提供了若干种 CFG( Control Flow Graphs )，都是基于接口 `DirectedGraph<N>`，该接口定义了若干个 getter 方法：

- the entry and exit points to the graph,
- the successors and predecessors of a given node,
- an iterator to iterate over the graph in some undefined orde and the graphs size (number of nodes).

此处讨论的 CFG 的 node 是 Soot Units，在 Jimple 中，Units 的实现是 Stmt。此外，仅讨论 intra-procedural flow

Abstract class：`UnitGraph`，提供了构建 CFG 的工具，此外，soot 还提供了三种具体实现：BriefUnitGraph, ExceptionalUnitGraph and TrapUnitGraph.

- **BriefUnitGraph**：最简单的实现，it doesn’t have edges representing control flow due to exceptions being thrown
- **ExceptionalUnitGraph**：最常用的实现，it includes edges from throw clauses to their handler(catch block, referred to in Soot as Trap), that is if the trap is local to the method body.
- **TrapUnitGraph**：类似 ExceptionalUnitGraph，但会考虑可能引发的异常。

在 soot 中，可以为 body 生成 CFG，例如：

```java
UnitGraph g = new ExceptionalUnitGraph(body);
```

## Wrapping the results of the analysis

> todo

# Call Graph

**前置知识**

Call Graph 是过程间分析的必要数据结构，当前有数种方法来构建 Call Graph，例如：

- Class Hierarchy Analysis( CHA) ：最快，但精度差。
- Points-To Analysis：指针分析，慢，但精度高。

Call Graph 是调用关系图，其每一个 Node 表示方法，而 Edge 表示调用。

**延申知识**

引入过程间分析后，我们向 CFG 中**添加 Call Graph 相应的元素**，得到过程间的控制流图（ICFG）

**soot 实现**

当使用 `-w` option 来开启 `whole program mode`时，可以通过 `getCallGraph()`方法来从环境类 `Scene`中访问`Call Graph`。

当什么都不设置时，默认采用 `CHA`生成 Call Graph。

调用图中的每条边都包含四个元素：

- source method
- source statement (if applicable)
- target method
- the kind of edge
  - static invocation
  - virtual invocation
  - interface invocation

![image-20220922220854158](https://happytsing-figure-bed.oss-cn-hangzhou.aliyuncs.com/soot/image-20220922220854158.png)

Call Graph 提供了三种方法：

- edgesInto(method)：查询进入方法的 edge
- edgesOutOf(method)：查询来自方法的 edge
- edgesOutOf(statement)查询来自特定 statement 的 edge，

Each of these methods return an Iterator over Edge constructs. soot 提供了 three so-called adapters 来遍历边的不同元素：

- Sources：遍历 source method
- Units：source statement
- Targets：target method

示例如下：

```java
public void printPossibleCallers(SootMethod target) {
    	CallGraph cg = Scene.v().getCallGraph();
        Iterator sources = new Sources(cg.edgesInto(target));
        while (sources.hasNext()) {
        SootMethod src = (SootMethod)sources.next();
        System.out.println(target + " might be called by " + src);
    }
}
```

# Points-To Analysis

指针分析，用于生成更精确的 CFG。可以在 soot 中通过两种框架来实现指针分析：

- SPARK
- Paddle

Soot provides three implementations of the points-to interface: CHA (a dumb version), SPARK and Paddle. The dumb version simply assumes that every variable might point to every other variable which is conservatively sound but not terribly accurate.

```shell
$ java -cp $SOOT_HOME soot.Main -pl | grep cg
cg                            Call graph constructor
    cg.cha                       Builds call graph using Class Hierarchy Analysis
    cg.spark                     Spark points-to analysis framework
    cg.paddle                    Paddle points-to analysis framework
```

可以看到，cg 是一个 phase，它有三个 subphase，默认情况下，执行`cg.cha`，如果想使用 spark，则需要启用 `cg.spark`。

## spark

通过`-p cg.spark on`来启用 spark，此外还有一些你可能感兴趣的 options：

- verbose. (设定 SPARK 在分析过程中，打印多种信息【提示信息】)
- propagator SPARK. (包含两个传播算法，原生迭代算法，基于 worklist 的算法)
- simple-edges-bidirectional. (如果设置为真，则所有的边都为双向的)
- on-fly-cg.（通过设置此选项，进行更加精确的 points-to 分析，得到精确的 call graph）
- set-impl. (描述 points-to 集合的实现。可能的值为 hash,bit,hybrid,array,double)
- double-set-old 以及 double-set-new.

# Usage

First of all, [install](https://repo1.maven.org/maven2/org/soot-oss/soot/4.3.0/soot-4.3.0-jar-with-dependencies.jar) soot.

在 soot 中，区分了三种类：

- argument classes：用户在命令行的 classname 中显式指定的类，也可以是 -process-dir 指定的目录中扫描得到的类。
- application classes：需要 soot 分析，并被转换为输出的类。所有 argument classes 必须式 application classes。
- library classes：application classes 引用的类，但不是 application classes。它们会被用于分析和转换，但自己不会被转换或输出。

存在两种模式会影响类的分类方式：

- application mode：该模式下，argument 引用的类将成为 application classes
- non-application mode：在该模式下，argument 引用的类将成为 library classes

使用`--app`开启 application mode。

```
java [javaOptions] soot.Main --app classname
```

## Shell

使用命令行操作 soot，整体命令如下：

```shell
java [javaOptions] soot.Main [sootOptions] classname
```

- `javaOptions`：java 可以接收的选项，例如通过 `-cp $SOOT_HOME` 来指定 soot 的 jar 包目录.
- `sootOptions`：soot 可以接收的选项，通过 `java [javaOptions] soot.Main -h` 获取选项列表
- `classname`：soot 要分析的类

### options

#### general options

| general options | desc                                                                 |
| --------------- | -------------------------------------------------------------------- |
| `-h`            | 显示帮助信息                                                         |
| `-pl`           | 打印支持的 phases                                                    |
| `-ph <phase> `  | 打印指定 phase 的详细信息                                            |
| `-w`            | Run in whole-program mode, 启用过程间分析, enable wjpp, cg, and wjap |

#### input options

| input options                                   | desc                                                       |
| ----------------------------------------------- | ---------------------------------------------------------- |
| `-cp <path>`                                    | 需要 soot 分析的类的地址                                   |
| `-pp`                                           | Prepend the given soot classpath to the default classpath. |
| `-allow-phantom-refs`                           | Allow unresolved classes; may cause errors                 |
| `-no-bodies-for-excluded`                       | Do not load bodies for excluded classes                    |
| `-process-path <dir>`<br />`-porcess-dir <dir>` | 处理 dir 中所有的类,，该 dir 也可以是 jar 包               |

**`-pp`的必要性**

soot 有自己的 classpath，该 classpath 下是 soot 需要分析的文件，如`A.class`。

但是当我们使用 `java soot.Main -cp $SOOT_CP A.class`，仍然会报错：java.lang.RuntimeException: None of the basic classes could be loaded! Check your Soot class path!

其原因是 soot 需要用到 java 的自带的各种类，如 java.lang.Object，但当前的 classpath 下肯定不存在包含该类的 jar 包，因此报错。

此时推荐的解决办法有两种：

- 添加`jce.jar`, `rt.jar` 到 soot 的 classpath。其中`jce.jar`是使用了`-w`参数进入过程间分析才需要用到。
- 采用 `-pp` options，前提是正确设置 `$JAVA_HOME`，会自动加载`jce.jar`和 `rt.jar`。

此外还有一种不推荐的方法：

- `-allow-phantom-refs`：soot 将为无法解析的类创建一个 phantom class。本质是告诉 soot，无法提供你需要的类，你自己看着办把！

该选项很危险，许多请情况下无法获取你想要的结果。

> ###### Important
>
> 当存在多个 classpath 时：
>
> - linux：冒号`:`分隔
> - windows：分号`;`分隔
>
> 此外，PATH 的斜杠：
>
> - linux：反斜杠 `/`
> - windows：正斜杠 `\`

#### output options

| output options | desc                                           |
| -------------- | ---------------------------------------------- |
| `-f <format>`  | soot 的输出格式，常用`J`表示输出`Jimple`格式。 |
| `-d <dir>`     | soot 的结果输出到目录 dir 中                   |

#### processing options

| processing options | desc                                                                |
| ------------------ | ------------------------------------------------------------------- |
| `–W`               | Perform whole program optimizations, enable wjop and wsop           |
| `–O`               | Perform intraprocedural optimizations, enable bop, gop, jop and sop |

#### phase options

已知 soot 由若干个 phase 构成，每个 phase 又细分为若干个 subphase，这些 phase 的实现称为 pack。

而 phase options 的作用就是，让你改变每个 phase 的行为！

使用 `-p <puase>.<subphase>`开始，此后接设置，格式为 `OPT:VAL`，多个设置用逗号 `,`分隔。

如下示例：对`cg.spark`这个 subphase 进行自定义，对其开启 verbose 和 on-fly-cg：

- verbose：false(default), When this option is set to true, Spark prints detailed information about its execution.
- on-fly-cg：true(defaulte), When this option is set to true, the call graph is computed on-the-fly as points-to information is computed. Otherwise, an initial CHA approximation to the call graph is used.

```
java [javaOptions] soot.Main classname -p cg.spark verbose:true,on-fly-cg:false
```

已知 phase 的实现是 pack，pack 是 transformer 的集合，每一个 transformer 对应于一个 subphase。当 pack 调用时，它将按顺序执行每一个 transformer。

每一个 transformer 是`extend`了 `BodyTransformer` or `SceneTransformer` 的类的实例，且必须重写其中的`internalTransform()`方法，该方法 providing an implementation which carries out some transformation on the code being analyzed.

也就是说，添加一个 transformer 就是新增了一个 `subphase`。

具体代码将在 java api 中讲解。

> For more [options](https://soot-build.cs.uni-paderborn.de/public/origin/develop/soot/soot-develop/options/soot_options.htm).

### Example

Prepare:

```shell
wget -P soot https://repo1.maven.org/maven2/org/soot-oss/soot/4.3.0/soot-4.3.0-jar-with-dependencies.jar
```

下述代码将处理 `HelloWorld.java`，并最终在`sootOutput/com.wang.HelloWorld.jimple`输出结果。

```shell
export SOOT_HOME = shell_soot/soot-4.3.0-jar-with-dependencies.jar
export POJO_HOME = src/test/java
# verify install
java -cp shell_soot/soot-4.3.0-jar-with-dependencies.jar  soot.Main  -h

# compile
javac src/test/java/com/wang/HelloWorld.java

# use soot output .jimple
java -cp shell_soot/soot-4.3.0-jar-with-dependencies.jar soot.Main -cp src\test\java -pp -f J com.wang.HelloWorld
```

- `-cp $SOOT_HOME` 指定 soot 的 jar 包目录
- `-cp $POJO_HOME` : 指定所要分析 `.class` 文件的目目录
- `-pp`: 指定 soot 去自动搜索 java 的 path， 主要是 rt.jar 和 jce.jar， soot 会去$JAVA_HOME 下寻找
- `-f J`: 指定输出文件类型， J 就是 jimple
- `com.wang.HelloWorld`: 你要分析的 class 的名字

## Java API

> https://github.com/HappyTsing/soot-tutorial
>
> https://github.com/HappyTsing/cflow_analysis

> ### Reference
>
> - [soot 介绍 - 简要 未细看](https://fynch3r.github.io/Soot/)
> - [soot survivors guide - 微过时 浅述应用 入门看](https://www.brics.dk/SootGuide/sootsurvivorsguide.pdf)
> - [soot survivors guide - 前四章翻译 建议直接看 EN](https://zhuanlan.zhihu.com/p/79801764)
> - [Fundamental-Soot-objects - soot 自定义的概念](https://github.com/soot-oss/soot/wiki/Fundamental-Soot-objects)
> - [Jimple 基本概念 - Foundamental-Soot-objects 的中文版](https://colinxiong.github.io/program/2015/08/19/Jimple-introduction)
> - [soot 安装与 shell/java 生成 Jimple 文件 - 参考安装](https://blog.csdn.net/qq_45401577/article/details/123958021)
> - [Packs-and-phases-in-Soot - Transfer 代码示例](https://github.com/soot-oss/soot/wiki/Packs-and-phases-in-Soot)
> - [soot 学习笔记 - 微门槛 用处不大](https://blog.csdn.net/beswkwangbo/category_2710855.html)
> - [soot options - 必看官网文档 使用查阅](https://soot-build.cs.uni-paderborn.de/public/origin/develop/soot/soot-develop/options/soot_options.htm)
> - [Soot-as-a-command-line-tool - 更多运行示例 出错查询](https://github.com/soot-oss/soot/wiki/Introduction:-Soot-as-a-command-line-tool)
> - [soot java api](https://soot-build.cs.uni-paderborn.de/public/origin/develop/soot/soot-develop/jdoc/)
> - [简单入门代码](https://xz.aliyun.com/t/11643#toc-2)
