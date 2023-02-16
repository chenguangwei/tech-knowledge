## Arthas ByteKit 深度解读(2)：本地变量及参数绑定

### 前言

本文通过分析ByteKit的本地变量绑定（LocalVarsBinding）处理代码，结合Java Opcode手册、asm代码、javap反汇编字节码等工具，深入讲解每个指令的用法及在本场景的实际作用。结合上下文线索，从字节码的角度去理解ByteKit 本地变量绑定的实现过程。

相关文章：
[Arthas ByteKit 深度解读(1)：基本原理介绍](https://github.com/alibaba/arthas/issues/1310)

### 简介

[Arthas ByteKit ](https://github.com/alibaba/arthas/tree/master/bytekit)为新开发的字节码工具库，基于ASM提供更高层的字节码处理能力，面向诊断/APM领域，不是通用的字节码库。ByteKit期望能提供一套简洁的API，让开发人员可以比较轻松的完成字节码增强。

本文分析的是本地变量绑定`@Binding.LocalVars Object[] vars`的实现逻辑。本地变量绑定的作用是通过注解@Binding.LocalVars 将目标代码拦截位置的本地变量映射到Object[] vars数组，在增强逻辑中可以直接通过vars数组进行访问。

一般是要结合本地变量名称绑定`@Binding.LocalVarNames String[] varNames`获取本地变量的名称和值。本地变量名称绑定（LocalVarNamesBinding）的实现逻辑与本地变量值绑定（LocalVarsBinding）类似，本文不再赘述。

### 代码概览

为了方便讲解，对LocalVarsBinding 的代码加上注释：

```
public class LocalVarsBinding extends Binding{

    @Override
    public void pushOntoStack(InsnList instructions, BindingContext bindingContext) {

        // 获取当前绑定位置的第一条指令
        AbstractInsnNode currentInsnNode = bindingContext.getLocation().getInsnNode();
        // 获取当前指令位置上有效的本地变量列表
        List<LocalVariableNode> results = AsmOpUtils
                .validVariables(bindingContext.getMethodProcessor().getMethodNode().localVariables, currentInsnNode);

        // 创建对象数组： new Object[ result.size() ]
        AsmOpUtils.push(instructions, results.size());
        AsmOpUtils.newArray(instructions, AsmOpUtils.OBJECT_TYPE);

        // 遍历本地变量列表，将每个变量添加到上面的Object数组中
        for (int i = 0; i < results.size(); ++i) {
            // dup : 复制栈顶最后一条指令，即上面的 object array ref
            AsmOpUtils.dup(instructions);
            
            // 写入的数组 index
            AsmOpUtils.push(instructions, i);

            // 读取本地变量，压入栈顶
            LocalVariableNode variableNode = results.get(i);
            AsmOpUtils.loadVar(instructions, Type.getType(variableNode.desc), variableNode.index);
            // 将primitive 类型转换为box对象
            AsmOpUtils.box(instructions, Type.getType(variableNode.desc));
            
            // 保存栈顶的变量到数组
            AsmOpUtils.arrayStore(instructions, AsmOpUtils.OBJECT_TYPE);
        }

    }

    @Override
    public Type getType(BindingContext bindingContext) {
        return AsmOpUtils.OBJECT_TYPE;
    }

}
```

`currentInsnNode` 为此绑定类开始处理的第一条指令，是在处理`@AtEnter`,`@AtLine`,`@AtExit` 等注解时进行匹配获得的，与本文主题无太大关系，这里不做展开。

下面顺着代码依次展开，追本溯源，力图从JVM字节码、ASM两个层面讲明白。

### 读取有效的本地变量列表

首先看一下AsmOpUtils.validVariables()方法，到底是怎么获取到当前指令位置可以访问的本地变量列表。

```
public static List<LocalVariableNode> validVariables(List<LocalVariableNode> localVariables,
        AbstractInsnNode currentInsnNode) {
    List<LocalVariableNode> results = new ArrayList<LocalVariableNode>();

    // find out current valid local variables
    for (LocalVariableNode localVariableNode : localVariables) {
        // 判断[start - end) 指令链表是否包含currentInsnNode 指令
        for (AbstractInsnNode iter = localVariableNode.start; iter != null
                && (!iter.equals(localVariableNode.end)); iter = iter.getNext()) {
            if (iter.equals(currentInsnNode)) {
                results.add(localVariableNode);
                break;
            }
        }
    }
    return results;
}
```

查看ASM LocalVariableNode 类的属性，可以看到 start 为第一条可以访问此变量的指令（包含），end 为最后一条指令（不包含，即end指令不能访问此变量）：

```
public class LocalVariableNode {

  /** The name of a local variable. */
  public String name;

  /** The first instruction corresponding to the scope of this local variable (inclusive). */
  public LabelNode start;

  /** The last instruction corresponding to the scope of this local variable (exclusive). */
  public LabelNode end;

  /** The local variable's index. */
  public int index;
```

不着急往下看，我们回顾一下JVM字节码中的LocalVariableTable，分析下面样例代码的LocalVariableTable：Start 为当前Method字节码偏移位置，Length 为变量有效的范围，Slot 为变量槽， Name为变量名，Signature为变量类型签名。

```
public void test1(int a, long b, double c, String d) {
    int var1 = a;
    long var2 = b;
    String var3 = d;
}
  public void test1(int, long, double, java.lang.String);
    descriptor: (IJDLjava/lang/String;)V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=11, args_size=5
         0: iload_1
         1: istore        7
         3: lload_2
         4: lstore        8
         6: aload         6
         8: astore        10
        10: return
      LineNumberTable:
        line 10: 0
        line 11: 3
        line 12: 6
        line 14: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/taobao/arthas/bytekit/asm/interceptor/LocalVarsTest;
            0      11     1     a   I
            0      11     2     b   J
            0      11     4     c   D
            0      11     6     d   Ljava/lang/String;
            3       8     7  var1   I
            6       5     8  var2   J
           10       1    10  var3   Ljava/lang/String;
```

借用一句话来描述：本地变量的访问范围刚好符合栈后进先出的特点，前面定义的变量的作用范围比后面的要大，后面定义的变量范围比前面的小。当指令的偏移位置满足本地变量的[Start, Start + Length) 时，可以访问此本地变量。

到这里疑问就来了：**为什么ASM的LocalVariableNode 的start, end是指令引用，而不是指令偏移位置？**

从指令解析的角度来说，直接使用指令偏移位置是最简单的，但不要忘记了，asm的主要目标是提供更轻量级的字节码修改功能，一旦字节码被修改，原来的指令偏移位置就失去意义。我的理解是与其每次修改指令都修改本地变量表的偏移位置，还不如使用指令的引用更加容易维护这个变量作用范围。 asm在生成最终字节码时，会自动重建 LocalVariableTable。

讲完本地变量表及asm的封装后，就很容易想到，LocalVariableNode的[start - end)的指令链表包含的所有指令都可以访问这个变量。所以要判断某个指令S是否可以访问本地变量V，只需要判断指令S是否包含在在本地变量V的[start - end)的指令链表中。

### 创建Object数组

本节分析的代码片段：

```
  // 创建对象数组： new Object[ result.size() ]
  AsmOpUtils.push(instructions, results.size());
  AsmOpUtils.newArray(instructions, AsmOpUtils.OBJECT_TYPE);
```

AsmOpUtils.newArray() 的代码：

```
public static void newArray(final InsnList insnList, final Type type) {
	insnList.add(new TypeInsnNode(Opcodes.ANEWARRAY, type.getInternalName()));
}
```

从[JVM Opcode Reference](http://homepages.inf.ed.ac.uk/kwxm/JVM/codeByFn.html) 查询anewarray指令：

> Description: allocate a new array of objects
> Stack
>
> | Before | After     |
> | ------ | --------- |
> | size   | array ref |
>
> Example
> Java---------------------
> String x = new String [3]
> Jasm-------------------
> bipush 3 ; set size
> anewarray java/lang/String ; make a new String array
> astore_1 ; and stick it in variable 1

这个指令很好理解，将栈顶的 size取出，作为新建数组的长度，然后将创建的数组对象引用压入栈顶。就是说要先将数组长度压入栈，然后调用anewarray指令（参数为数组元素类型）。本节分析的两行代码相当于 `new Object[ result.size() ]`。

### 数组元素赋值

本节分析的代码片段：

```
    // dup : 复制栈顶最后一条指令，即上面的 object array ref
    AsmOpUtils.dup(instructions);
    
    // 写入的数组 index
    AsmOpUtils.push(instructions, i);

    // 读取本地变量，压入栈顶
    	...
    
    // 保存栈顶的变量到数组
    AsmOpUtils.arrayStore(instructions, AsmOpUtils.OBJECT_TYPE);
```

1）AsmOpUtils.dup(instructions)
dup 指令的文档

> Description: duplicate top of stack
> **Stack**
>
> | Before   | After    |
> | -------- | -------- |
> | whatever | whatever |
> |          | whatever |

这个指令将栈顶的数据项复制一条，且将复制的数据项压入栈顶。上面的文档中栈变化可能有点不清晰，实际上是栈顶增长了，表达为下面的方式肯更加恰当：

> | Before   |
> | -------- |
> | whatever |
> | others   |
>
> | After    |
> | -------- |
> | whatever |
> | whatever |
> | others   |

大家了解文档中栈比较表格的意思后也不影响理解，栈底部空白就是其它数据项，后面文档中的栈变化不再单独列举。本节第一行`AsmOpUtils.dup(instructions)` 复制当前栈顶的数据项，其实就是刚刚创建的Object数组引用。

当前栈内容为：

| Stack                      |
| -------------------------- |
| **object array ref (dup)** |
| object array ref           |

2）AsmOpUtils.push(instructions, i)

```
    public static void push(InsnList insnList, final int value) {
        if (value >= -1 && value <= 5) {
        	// 优先使用常量指令压栈，不需要参数( iconst_0, iconst_1, ..., iconst_5 )
            insnList.add(new InsnNode(Opcodes.ICONST_0 + value));
        } else if (value >= Byte.MIN_VALUE && value <= Byte.MAX_VALUE) {
        	// byte 范围，bipush 压入
            insnList.add(new IntInsnNode(Opcodes.BIPUSH, value));
        } else if (value >= Short.MIN_VALUE && value <= Short.MAX_VALUE) {
        	// short范围，sipush 压入
            insnList.add(new IntInsnNode(Opcodes.SIPUSH, value));
        } else {
        	// 超出short范围，使用加载常量指令ldc
            insnList.add(new LdcInsnNode(value));
        }
    }
```

看起来代码很多，其实就是将value压入栈顶，这个方法封装了不同的场景处理逻辑，简化上层调用。
当前栈内容为：

| Stack                      |
| -------------------------- |
| index                      |
| **object array ref (dup)** |
| object array ref           |

3）AsmOpUtils.arrayStore(instructions, AsmOpUtils.OBJECT_TYPE)
这里的数组是Object[]，对应的是aastore 指令：

> Description: store value in array[index]
> **Stack**
>
> | Before | After |
> | ------ | ----- |
> | value  |       |
> | index  |       |
> | array  |       |

将value保存到array[index]， 注意栈的数据顺序，栈顶为value，接着是index，接着是array引用。 本节的场景中，遍历读取本地变量放到栈上，然后保存到数组指定位置中。
每次循环都是相同的处理过程：

- 用dup指令复制栈顶数据项（ object array ) ，栈顶 +1 数据项
- push 数组索引index，栈顶 +1 数据项
- load var，栈顶 +1 数据项
- aastore 指令保存 var到数组，消耗栈顶3条数据项，恢复到循环前的栈状态

### 加载本地变量

本节分析的代码片段：

```
// 读取本地变量，压入栈顶
LocalVariableNode variableNode = results.get(i);
AsmOpUtils.loadVar(instructions, Type.getType(variableNode.desc),  variableNode.index);
```

AsmOpUtils.loadVar() 方法的代码如下：

```
public static void loadVar(final InsnList instructions, Type type, final int index) {
    instructions.add(new VarInsnNode(type.getOpcode(Opcodes.ILOAD), index));
}
```

com.alibaba.arthas.deps.org.objectweb.asm.Type#getOpcode() 代码如下：

```
/**
* Returns a JVM instruction opcode adapted to this {@link Type}. This method must not be used for
* method types.
*
* @param opcode a JVM instruction opcode. This opcode must be one of ILOAD, ISTORE, IALOAD,
*     IASTORE, IADD, ISUB, IMUL, IDIV, IREM, INEG, ISHL, ISHR, IUSHR, IAND, IOR, IXOR and
*     IRETURN.
* @return an opcode that is similar to the given opcode, but adapted to this {@link Type}. For
*     example, if this type is {@code float} and {@code opcode} is IRETURN, this method returns
*     FRETURN.
*/
public int getOpcode(final int opcode) {
...
    switch (sort) {
    case VOID:
      if (opcode != Opcodes.IRETURN) {
        throw new UnsupportedOperationException();
      }
      return Opcodes.RETURN;
    case BOOLEAN:
    case BYTE:
    case CHAR:
    case SHORT:
    case INT:
      return opcode;
    case FLOAT:
      return opcode + (Opcodes.FRETURN - Opcodes.IRETURN);
    case LONG:
      return opcode + (Opcodes.LRETURN - Opcodes.IRETURN);
    case DOUBLE:
      return opcode + (Opcodes.DRETURN - Opcodes.IRETURN);
    case ARRAY:
    case OBJECT:
    case INTERNAL:
      if (opcode != Opcodes.ILOAD && opcode != Opcodes.ISTORE && opcode != Opcodes.IRETURN) {
        throw new UnsupportedOperationException();
      }
      return opcode + (Opcodes.ARETURN - Opcodes.IRETURN);
    case METHOD:
      throw new UnsupportedOperationException();
    default:
      throw new AssertionError();
  }
...
}
```

上面代码作用是获取type实例对应的opcode，用iload举例说明，下面是不同类型对应的指令：

| type   | inst  | stack size |
| ------ | ----- | ---------- |
| int    | iload | 1          |
| float  | fload | 1          |
| object | aload | 1          |
| double | dload | 2          |
| long   | lload | 2          |

stack size 是占用的栈大小，2表示占用两个位置，我们看一下 iload 和 lload的文档。
iload的文档:

> Description: push integer from the local variable
> **Stack**
>
> | Before | After   |
> | ------ | ------- |
> |        | integer |

lload的文档:

> Description: push long from local variable n
> **Stack**
>
> | Before | After      |
> | ------ | ---------- |
> |        | long word1 |
> |        | long word2 |

分析了部分JVM字节码指令，发现涉及到long/double的指令的数据大都是2个word，其它的int, float, object指令的数据占1个word，不清楚的话查看Opcode手册。
有上面的知识基础后，可以知道加载局部变量的数据是不定长度的，可能为1个word或者2个word。其中2个word的数据占用2个数据项，不满足上一小节aastore指令的循环要求，当然从语法上也是错误的，Object [] 不能直接赋值int/long，需要转换为box包装类型。

当前栈内容，如果local var type为 byte/short/int/float/object等 ：

| Stack            |
| ---------------- |
| local var        |
| index            |
| object array ref |
| object array ref |

如果local var type为 long/double ( type size 为2）：

| Stack            |
| ---------------- |
| local var word1  |
| local var word2  |
| index            |
| object array ref |
| object array ref |

### 将primitive 类型转换为box对象

本节分析的代码片段：

```
// 将primitive 类型转换为box对象
AsmOpUtils.box(instructions, Type.getType(variableNode.desc));
```

AsmOpUtils.box() 代码如下：

```
public static void box(final InsnList instructions, Type type) {
	// 本身为对象类型，不需要包装
	if (type.getSort() == Type.OBJECT || type.getSort() == Type.ARRAY) {
		return;
	}

	if (type == Type.VOID_TYPE) {
		// push null
		instructions.add(new InsnNode(Opcodes.ACONST_NULL));
	} else {
		// 获取primitive type的包装类型
		Type boxed = getBoxedType(type);
		// new instance. 创建包装类型对象
		newInstance(instructions, boxed);
		if (type.getSize() == 2) {
			// Pp -> Ppo -> oPpo -> ooPpo -> ooPp -> o
			// dupX2
			dupX2(instructions);
			// dupX2
			dupX2(instructions);
			// pop
			pop(instructions);
		} else {
			// p -> po -> opo -> oop -> o
			// dupX1
			dupX1(instructions);
			// swap
			swap(instructions);
		}
		invokeConstructor(instructions, boxed, new Method("<init>", Type.VOID_TYPE, new Type[] { type }));
	}
}
```

这是本文最为神奇的一段代码，其中 dupX2 - dupX2 - pop以及它的注释着实让人摸不着头脑，后面会深入解读。我们先分析下面两行代码：

```
// 获取primitive type的包装类型
Type boxed = getBoxedType(type);
// new instance. 创建包装类型对象
newInstance(instructions, boxed);
```

AsmOpUtils.getBoxedType() 及 newInstance() 代码如下：

```
// 返回type对应的包装类型
public static Type getBoxedType(final Type type) {
	switch (type.getSort()) {
	case Type.BYTE:
		return BYTE_TYPE;
	case Type.BOOLEAN:
		return BOOLEAN_TYPE;
	case Type.SHORT:
		return SHORT_TYPE;
	case Type.CHAR:
		return CHARACTER_TYPE;
	case Type.INT:
		return INTEGER_TYPE;
	case Type.FLOAT:
		return FLOAT_TYPE;
	case Type.LONG:
		return LONG_TYPE;
	case Type.DOUBLE:
		return DOUBLE_TYPE;
	}
	return type;
}

// 使用new指令创建对象，将对象的引用压入栈顶
public static void newInstance(final InsnList instructions, final Type type) {
	instructions.add(new TypeInsnNode(Opcodes.NEW, type.getInternalName()));
}
```

new 指令文档：

> Description: create a new object
> **Stack**
>
> | Before | After            |
> | ------ | ---------------- |
> |        | object reference |

#### 分支1：local var为1个word

接下来我们先看 `type.getSize() == 1` 分支，即local var为1个word，记住当前栈的内容：

| Stack            |
| ---------------- |
| box object ref   |
| local var        |
| index            |
| object array ref |
| object array ref |

再看 `type.getSize() == 1` 分支的处理代码：

```
// p -> po -> opo -> oop -> o
// dupX1
dupX1(instructions);
// swap
swap(instructions);
```

1. dup_x1指令文档：

> Description: duplicate top word of stack and put duplicate beneath second word of stack
> **Stack**
>
> | Before    | After     |
> | --------- | --------- |
> | whatever1 | whatever1 |
> | whatever2 | whatever2 |
> |           | whatever1 |

这个指令的作用是复制栈顶第1个word，插入到第2个word下面。结合本小节的stack内容，执行dup_x1指令后，栈内容变为：

| Stack                            |
| -------------------------------- |
| **box object ref**               |
| local var                        |
| **box object ref (dup_x1 插入)** |
| index                            |
| object array ref                 |
| object array ref                 |

1. 接着执行`swap(instructions)`：

```
public static void swap(final InsnList insnList) {
	insnList.add(new InsnNode(Opcodes.SWAP));
}
```

即`swap`指令，其文档如下：

> Description: swap top two words on stack
> **Stack**
>
> | Before | After |
> | ------ | ----- |
> | one    | two   |
> | two    | one   |

栈内容变为：

| Stack                            |
| -------------------------------- |
| **local var (swap 交换)**        |
| **box object ref (swap 交换)**   |
| **box object ref (dup_x1 插入)** |
| index                            |
| object array ref                 |
| object array ref                 |

1. 接着执行invokeConstructor()语句：

```
invokeConstructor(instructions, boxed, new Method("<init>", Type.VOID_TYPE, new Type[] { type }))
```

invokeConstructor()方法的代码：

```
public static void invokeConstructor(final InsnList instructions, final Type type, final Method method) {
	String owner = type.getSort() == Type.ARRAY ? type.getDescriptor() : type.getInternalName();
	instructions.add(new MethodInsnNode(Opcodes.INVOKESPECIAL, 
		owner, method.getName(), method.getDescriptor(), false));
}
```

即插入`invokespecial` 指令，其文档为：

> Description: calls a special class method.
> n is the number of arguments to the method.
> the long method name is really a path name, the name of the class,
> the parenthesized argument list of the method called, and the return type.
> Primitive types are represented by their capitalized first letter, ie I for an integer.
> Constructors are path followed by `<init>()V`
>
> **Stack**
>
> | Before           | After          |
> | ---------------- | -------------- |
> | arg n            | returned value |
> | ...              |                |
> | arg 1            |                |
> | object reference |                |
>
> **Example**
> Jasm-------------------
> invoke packages/myMath/multiplyMatrix([[F, [[F)V ;multiplies two two-dimensional matrices of floats

`invokespecial` 指令将栈顶的参数及对象引用依次出栈，然后将调用结果入栈。出栈顺序与代码书写顺序相反，从右至左的顺序出栈，反之，入栈顺序为从左至右与代码书写顺序一致。

> Invoke instance method; special handling for superclass, private, and instance initialization method invocations

`invokespecial` 用于调用实例构造方法；对超类，私有和实例初始化方法调用的特殊处理。`invokespecial` 不仅是调用构造函数，还有其它特殊用法，如果要深入了解，可以参考[此文档](https://www.jrebel.com/blog/using-objects-and-calling-methods-in-java-bytecode)。

`invokespecial` 指令需要指定构造函数方法签名，以`java.lang.Boolean`为例，调用构造函数的指令参数：`Method java/lang/Boolean."<init>":(Z)V`
`<init>` 为构造函数方法名称，`(Z)` 为参数类型列表，`Z` 为boolean参数类型，`V` 为返回值类型void。

> [Section 4.3.2](http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.3.2) of the JVM Spec:
>
> ```
> Character     Type          Interpretation
> ------------------------------------------
> B             byte          signed byte
> C             char          Unicode character
> D             double        double-precision floating-point value
> F             float         single-precision floating-point value
> I             int           integer
> J             long          long integer
> L<classname>; reference     an instance of class 
> S             short         signed short
> Z             boolean       true or false
> [             reference     one array dimension
> ```

这里的`invokespecial` 调用的是构造函数，无返回值入栈（关于这点，欢迎补充说明材料）。box对象构造函数只有1个参数，指令执行后栈内容变为：

| Stack                            |
| -------------------------------- |
| **box object ref (dup_x1 插入)** |
| index                            |
| object array ref                 |
| object array ref                 |

1. 执行语句`AsmOpUtils.arrayStore(instructions, AsmOpUtils.OBJECT_TYPE)`

> 参考前一小节 “数组元素赋值”

执行aastore指令，取出栈顶3个word，保存`box object ref`到数组中指定位置，栈内容恢复到循环遍历本地变量之前的状态。执行后的栈状态为：

| Stack            |
| ---------------- |
| object array ref |

**建议仔细阅读本小节内容，完全掌握后再看分支2的内容。**

#### 分支2：local var 为2个word

分支1已经包含了读取本地变量、保存到Object[]数组的整个流程，分支2是针对local var 为2个word（type size 为2）的情况进行特殊说明。本小节分析的代码片段：

```
if (type.getSize() == 2) {
    // Pp -> Ppo -> oPpo -> ooPpo -> ooPp -> o
    // dupX2
    dupX2(instructions);
    // dupX2
    dupX2(instructions);
    // pop
    pop(instructions);
}
```

dup_x2指令文档：

> Description: duplicate top word of stack and put duplicate beneath third word of stack
> **Stack**
>
> | Before    | After     |
> | --------- | --------- |
> | whatever1 | whatever1 |
> | whatever2 | whatever2 |
> | whatever3 | whatever3 |
> |           | whatever1 |

`dup_x2`指令复制栈顶第一个word，插入到第3个word下面。单独看这个指令是比较难理解其用法，结合本小节的local var 为2个word来理解，一下就豁然开朗了。

`dup_x2`之前的栈状态为（请回顾“加载本地变量”小节最后结论）：

| Stack                            |
| -------------------------------- |
| **box object ref (newInstance)** |
| local var word1                  |
| local var word2                  |
| index                            |
| object array ref                 |
| object array ref                 |

执行第一个`dup_x2`指令后的栈内容：

| Stack                            |
| -------------------------------- |
| **box object ref (newInstance)** |
| local var word1                  |
| local var word2                  |
| **box object ref (dup_x2)**      |
| index                            |
| object array ref                 |
| object array ref                 |

呵呵，是不是很惊喜，`dup_x2 `刚好解决了local var 2 word的问题，其作用和分支1的dup_x1类似，在local var后面插入box对象引用。
但这里的local var 是2 word，继续使用`swap`指令达不到交换 local var 和 `box object ref` 的目的，于是有了第2个`dup_x2 `指令，执行后栈内容为：

| Stack                                                        |
| ------------------------------------------------------------ |
| **box object ref (newInstance)**                             |
| local var word1                                              |
| local var word2                                              |
| **box object ref (dup_x2 [#2](https://github.com/alibaba/arthas/issues/2))** |
| **box object ref (dup_x2 [#1](https://github.com/alibaba/arthas/issues/1))** |
| index                                                        |
| object array ref                                             |
| object array ref                                             |

再执行`pop` 指令，移除栈顶多余的`box object ref` ，`dup_x2 `+ `pop` 合起来的作用就是交换了栈顶的`box object ref` 和 2个word的 local var !

| Stack                                                        |
| ------------------------------------------------------------ |
| local var word1                                              |
| local var word2                                              |
| **box object ref (dup_x2 [#2](https://github.com/alibaba/arthas/issues/2))** |
| **box object ref (dup_x2 [#1](https://github.com/alibaba/arthas/issues/1))** |
| index                                                        |
| object array ref                                             |
| object array ref                                             |

到这里，栈的内容已经准备好，后面继续执行下面两行语句：

调用box对象构造函数初始化，从栈取出 local var 2 word 和 第一个`box object ref`：

```
// 调用box对象构造函数初始化
invokeConstructor(instructions, boxed, new Method("<init>", Type.VOID_TYPE, new Type[] { type }));
```

当前栈数据：

| Stack                                                        |
| ------------------------------------------------------------ |
| **box object ref (dup_x2 [#1](https://github.com/alibaba/arthas/issues/1))** |
| index                                                        |
| object array ref                                             |
| object array ref                                             |

保存栈顶的变量到数组，从栈取出第二个`box object ref`和后面的`index`, `object array ref`，栈顶为剩下的一个`object array ref`，恢复到遍历本地变量之前的状态。

```
// 保存栈顶的变量到数组
AsmOpUtils.arrayStore(instructions, AsmOpUtils.OBJECT_TYPE);
```

当前栈数据：

| Stack            |
| ---------------- |
| object array ref |

至此，分支2执行后的栈状态与分支1是一致的，不影响后续遍历执行。后面有完整的代码及反汇编的字节码，可以对照理解。本文尽量深入细节讲解每一部分，可能导致整体连贯性不足，可以动手整理一下，画出每个语句执行后的栈内容，这样可以更加清晰的理解认识。

### 完整示例代码

1）ByteKit 代码：

```
public class LocalVarsDemo extends AbstractDemo {

    public static class SampleInterceptor {

        @AtLine(lines = {-1}, inline = false, suppressHandler = PrintExceptionSuppressHandler.class)
        public static void atEnter(@Binding.This Object object,
                                   @Binding.LocalVars Object[] vars,
                                   @Binding.MethodName String methodName) {
            System.out.println("atEnter, vars: " + Arrays.asList(vars)+", vars len: "+vars.length);
        }

    }

    public void testMain() throws Exception {

        // 对Sample.class字节码增强，返回增强后的字节码
        byte[] bytes = super.enhanceClass(Sample.class, new String[]{"hello"}, SampleInterceptor.class);

        //增强前测试
        System.out.println("Before reTransform ...");
        new Sample().hello("world", 50, 1.0 );

        // 通过 reTransform 增强类
        AgentUtils.reTransform(Sample.class, bytes);

        //测试结果
        System.out.println("After reTransform ...");
        new Sample().hello("world", 50, 1.0 );
    }

    public static void main(String[] args) throws Exception {
        new LocalVarsDemo().testMain();
    }

}
public class AbstractDemo {

    protected AbstractDemo() {
        AgentUtils.install();
    }

    protected byte[] enhanceClass(Class targetClass, String[] targetMethodNames, Class interceptorClass) throws Exception {
        // 解析定义的 Interceptor类 和相关的注解
        DefaultInterceptorClassParser interceptorClassParser = new DefaultInterceptorClassParser();
        List<InterceptorProcessor> processors = interceptorClassParser.parse(interceptorClass);

        // 加载字节码
        ClassNode classNode = AsmUtils.loadClass(targetClass);

        List<String> methodNameList = Arrays.asList(targetMethodNames);

        // 对加载到的字节码做增强处理
        for (MethodNode methodNode : classNode.methods) {
            if (methodNameList.contains(methodNode.name)) {
                MethodProcessor methodProcessor = new MethodProcessor(classNode, methodNode);
                for (InterceptorProcessor interceptor : processors) {
                    interceptor.process(methodProcessor);
                }
            }
        }

        // 获取增强后的字节码
        byte[] bytes = AsmUtils.toBytes(classNode);
        return bytes;
    }

    public static class PrintExceptionSuppressHandler {

        @ExceptionHandler(inline = true)
        public static void onSuppress(@Binding.Throwable Throwable e, @Binding.Class Object clazz) {
            System.out.println("exception handler: " + clazz);
            e.printStackTrace();
        }
    }
}
```

2）目标类原始代码：

```
 1 package com.example;
 2 
 3 public class Sample {
 4
 5   public String hello(String str, long num1, double num2) {
 6        Long num3 = 0L;
 7        num3 += num1;
 8        String result = "hello " + str;
 9        return result;
10   }
11
12 }
```

3）目标类原始字节码

```
  public java.lang.String hello(java.lang.String, long, double);
    descriptor: (Ljava/lang/String;JD)Ljava/lang/String;
    flags: ACC_PUBLIC
    Code:
      stack=4, locals=8, args_size=4
         0: lconst_0
         1: invokestatic  #2                  // Method java/lang/Long.valueOf:(J)Ljava/lang/Long;
         4: astore        6
         6: aload         6
         8: invokevirtual #3                  // Method java/lang/Long.longValue:()J
        11: lload_2
        12: ladd
        13: invokestatic  #2                  // Method java/lang/Long.valueOf:(J)Ljava/lang/Long;
        16: astore        6
        18: new           #4                  // class java/lang/StringBuilder
        21: dup
        22: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
        25: ldc           #6                  // String hello
        27: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        30: aload_1
        31: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        34: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        37: astore        7
        39: aload         7
        41: areturn
      LineNumberTable:
        line 6: 0
        line 7: 6
        line 8: 18
        line 9: 39
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      42     0  this   Lcom/example/Sample;
            0      42     1   str   Ljava/lang/String;
            0      42     2  num1   J
            0      42     4  num2   D
            6      36     6  num3   Ljava/lang/Long;
           39       3     7 result   Ljava/lang/String;
```

4）目标类增强后的字节码
javap生成的内容太长，节选前面的部分。

```
  public java.lang.String hello(java.lang.String, long, double);
    descriptor: (Ljava/lang/String;JD)Ljava/lang/String;
    flags: ACC_PUBLIC
    Code:
      stack=9, locals=16, args_size=4
         0: aload_0
         1: iconst_4
         2: anewarray     #4                  // class java/lang/Object
         5: dup
         6: iconst_0
         7: aload_0
         8: aastore
         9: dup
        10: iconst_1
        11: aload_1
        12: aastore
        13: dup
        14: iconst_2
        15: lload_2
        16: new           #17                 // class java/lang/Long
        19: dup_x2
        20: dup_x2
        21: pop
        22: invokespecial #20                 // Method java/lang/Long."<init>":(J)V
        25: aastore
        26: dup
        27: iconst_3
        28: dload         4
        30: new           #22                 // class java/lang/Double
        33: dup_x2
        34: dup_x2
        35: pop
        36: invokespecial #25                 // Method java/lang/Double."<init>":(D)V
        39: aastore
        40: ldc           #26                 // String hello
        42: invokestatic  #32                 // Method com/example/LocalVarsDemo$SampleInterceptor.atEnter:(Ljava/lang/Object;[Ljava/lang/Object;Ljava/lang/String;)V
        45: goto          88
        48: ldc           #2                  // class com/example/Sample
        50: astore        9
        52: astore        8
        54: getstatic     #38                 // Field java/lang/System.out:Ljava/io/PrintStream;
        57: new           #40                 // class java/lang/StringBuilder
        60: dup
        61: invokespecial #41                 // Method java/lang/StringBuilder."<init>":()V
        64: ldc           #43                 // String exception handler:
        66: invokevirtual #47                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        69: aload         9
        71: invokevirtual #50                 // Method java/lang/StringBuilder.append:(Ljava/lang/Object;)Ljava/lang/StringBuilder;
        74: invokevirtual #54                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        77: invokevirtual #60                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        80: aload         8
        82: invokevirtual #63                 // Method java/lang/Throwable.printStackTrace:()V
        85: goto          88
        88: lconst_0
        89: invokestatic  #67                 // Method java/lang/Long.valueOf:(J)Ljava/lang/Long;
        92: astore        6
        94: aload_0
        95: iconst_5
        96: anewarray     #4                  // class java/lang/Object
        99: dup
       100: iconst_0
       101: aload_0
       102: aastore
       103: dup
       104: iconst_1
       105: aload_1
       106: aastore
       107: dup
       108: iconst_2
       109: lload_2
       110: new           #17                 // class java/lang/Long
       113: dup_x2
       114: dup_x2
       115: pop
       116: invokespecial #20                 // Method java/lang/Long."<init>":(J)V
       119: aastore
...
      LineNumberTable:
        line 6: 0
        line 7: 94
        line 8: 199
        line 9: 313
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0     415     0  this   Lcom/example/Sample;
            0     415     1   str   Ljava/lang/String;
            0     415     2  num1   J
            0     415     4  num2   D
           94     321     6  num3   Ljava/lang/Long;
          313     102     7 result   Ljava/lang/String;
```

5）目标类增强后的字节码反编译得到的源码

```
package com.example;

import com.example.LocalVarsDemo.SampleInterceptor;

public class Sample {
   public String hello(String str, long num1, double num2) {
      try {
         SampleInterceptor.atEnter(this, new Object[]{this, str, new Long(num1), new Double(num2)}, "hello");
      } catch (Throwable var19) {
         Class var9 = Sample.class;
         System.out.println("exception handler: " + var9);// 6
         var19.printStackTrace();
      }

      Long num3 = 0L;

      try {
         SampleInterceptor.atEnter(this, new Object[]{this, str, new Long(num1), new Double(num2), num3}, "hello");
      } catch (Throwable var18) {
         Class var11 = Sample.class;
         System.out.println("exception handler: " + var11);
         var18.printStackTrace();
      }

      num3 = num3 + num1;// 7

      try {
         SampleInterceptor.atEnter(this, new Object[]{this, str, new Long(num1), new Double(num2), num3}, "hello");
      } catch (Throwable var17) {
         Class var13 = Sample.class;
         System.out.println("exception handler: " + var13);// 8
         var17.printStackTrace();
      }

      String result = "hello " + str;

      try {
         SampleInterceptor.atEnter(this, new Object[]{this, str, new Long(num1), new Double(num2), num3, result}, "hello");
      } catch (Throwable var16) {
         Class var15 = Sample.class;
         System.out.println("exception handler: " + var15);// 9
         var16.printStackTrace();
      }

      return result;
   }
}
```

### 参数绑定(ArgsBinding)

参数绑定的核心代码片段：

```
public static void loadArgArray(final InsnList instructions, MethodNode methodNode) {
	boolean isStatic = AsmUtils.isStatic(methodNode);
	// 获取参数类型数组
	Type[] argumentTypes = Type.getArgumentTypes(methodNode.desc);
	// 创建数组 Object[argumentTypes.length]
	push(instructions, argumentTypes.length);
	newArray(instructions, OBJECT_TYPE);
	// 变量参数类型列表
	for (int i = 0; i < argumentTypes.length; i++) {
		// 复制 object array ref
		dup(instructions);
		// 数组index
		push(instructions, i);
		// 加载指定参数var
		loadArg(isStatic, instructions, argumentTypes, i);
		// 转换为box对象
		box(instructions, argumentTypes[i]);
		// 保存到数组
		arrayStore(instructions, OBJECT_TYPE);
	}
}

public static void loadArg(boolean staticAccess, final InsnList instructions, Type[] argumentTypes, int i) {
    // 计算参数i在的LocalVariableTable 的index
    final int index = getArgIndex(staticAccess, argumentTypes, i);
    final Type type = argumentTypes[i];
    instructions.add(new VarInsnNode(type.getOpcode(Opcodes.ILOAD), index));
}
    
static int getArgIndex(boolean staticAccess, final Type[] argumentTypes, final int arg) {
    //非静态方法的第一个本地变量为对象自身的引用 this
    int index = staticAccess ? 0 : 1;
    for (int i = 0; i < arg; i++) {
    	  // 变量类型size（ long/double 为2，其它为1 ）
        index += argumentTypes[i].getSize();
    }
    return index;
}
```

处理过程与本地变量非常相似，唯一的区别是加载参数变量的方式不同。其中`loadArg()` 中需要计算参数i在的LocalVariableTable 的index (slot)，我们对比下面的方法签名及LocalVariableTable：

```
	public String hello(String str, long num1, double num2)
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0     415     0  this   Lcom/example/Sample;
            0     415     1   str   Ljava/lang/String;
            0     415     2  num1   J
            0     415     4  num2   D
           94     321     6  num3   Ljava/lang/Long;
          313     102     7 result   Ljava/lang/String;
```

- slot 0：this 引用，Object类型，type size=1
- slot 1：参数 str，String类型，type size=1
- slot 2：参数 num1，long类型，type size=2
- slot 4：参数 num2，double类型，type size=2
- slot 6：局部变量num3，Long类型，type size=1
- slot 7：局部变量result，String类型，type size=1

可以看到方法参数和方法体局部变量都是定义在LocalVariableTable中，按照slot排序后：

- this （静态方法无this）
- 参数1
- 参数n
- 局部变量1
- 局部变量n

分析发现asm LocalVariableNode.index的值实际上就是slot的值 （关于这点，欢迎补充说明材料），参数n+1的slot = 参数n slot + 参数n类型size。 注意局部变量num3为Long，box类型，类型size为1（按照OBJECT类型来计算）。

### 相关资源

推荐一个JVM字节码指令手册，包含每个指令栈变化的描述：[JVM Opcode Reference](http://homepages.inf.ed.ac.uk/kwxm/JVM/codeByFn.html)

### 结论

Arthas ByteKit 本地变量绑定实现逻辑是按需动态插入Java字节码，不同于常见的asm ClassReader/ClassWriter + ClassVisitor/MethodVisitor处理模式。动态插入Java字节码优点是比Visitor模式直观，可控性更强，缺点是要求对Java字节码有比较深厚的功力。Arthas ByteKit 本身只是提供10多种特定模式的变量绑定，有较好的通用性，只要保证每处修改字节码的功能稳定，整体来说ByteKit框架的代码质量是可以保证的。