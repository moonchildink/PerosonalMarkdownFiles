# JVM笔记

## jvm汇编

使用

```shell
 javap -v  out/production/algorithm/example.class
```

指令完成对class文件的反汇编。可以发现jvm无关平台、处理器架构，产生的汇编代码是相同的。

```assembly
Classfile /D:/code/Java/algorithm/out/production/algorithm/example.class
  Last modified 2024年4月27日; size 700 bytes
  SHA-256 checksum 75ce2b61ff6ae358a6bbd52f04aa7536b65345865d961b3c51b4acff98b78d15
  Compiled from "example.java"
public class example
  minor version: 0
  major version: 61
  flags: (0x0021) ACC_PUBLIC, ACC_SUPER
  this_class: #29                         // example
  super_class: #2                         // java/lang/Object
  interfaces: 0, fields: 0, methods: 2, attributes: 1
Constant pool:
   #1 = Methodref          #2.#3          // java/lang/Object."<init>":()V
   #2 = Class              #4             // java/lang/Object
   #3 = NameAndType        #5:#6          // "<init>":()V
   #4 = Utf8               java/lang/Object
   #5 = Utf8               <init>
   #6 = Utf8               ()V
   #7 = Methodref          #8.#9          // java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
   #8 = Class              #10            // java/lang/Integer
   #9 = NameAndType        #11:#12        // valueOf:(I)Ljava/lang/Integer;
  #10 = Utf8               java/lang/Integer
  #11 = Utf8               valueOf
  #12 = Utf8               (I)Ljava/lang/Integer;
  #13 = Fieldref           #14.#15        // java/lang/System.out:Ljava/io/PrintStream;
  #14 = Class              #16            // java/lang/System
  #15 = NameAndType        #17:#18        // out:Ljava/io/PrintStream;
  #16 = Utf8               java/lang/System
  #17 = Utf8               out
  #18 = Utf8               Ljava/io/PrintStream;
  #19 = Methodref          #8.#20         // java/lang/Integer.equals:(Ljava/lang/Object;)Z
  #20 = NameAndType        #21:#22        // equals:(Ljava/lang/Object;)Z
  #21 = Utf8               equals
  #22 = Utf8               (Ljava/lang/Object;)Z
  #23 = Methodref          #24.#25        // java/io/PrintStream.println:(Z)V
  #24 = Class              #26            // java/io/PrintStream
  #25 = NameAndType        #27:#28        // println:(Z)V
  #26 = Utf8               java/io/PrintStream
  #27 = Utf8               println
  #28 = Utf8               (Z)V
  #29 = Class              #30            // example
  #30 = Utf8               example
  #31 = Utf8               Code
  #32 = Utf8               LineNumberTable
  #33 = Utf8               LocalVariableTable
  #34 = Utf8               this
  #35 = Utf8               Lexample;
  #36 = Utf8               main
  #37 = Utf8               ([Ljava/lang/String;)V
  #38 = Utf8               args
  #39 = Utf8               [Ljava/lang/String;
  #40 = Utf8               a1
  #41 = Utf8               Ljava/lang/Integer;
  #42 = Utf8               a2
  #43 = Utf8               SourceFile
  #44 = Utf8               example.java
{
  public example();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 1: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lexample;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=3, args_size=1
         0: sipush        200
         3: invokestatic  #7                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
         6: astore_1
         7: sipush        200
        10: invokestatic  #7                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        13: astore_2
        14: sipush        2000
        17: invokestatic  #7                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        20: astore_1
        21: sipush        2000
        24: invokestatic  #7                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        27: astore_2
        28: getstatic     #13                 // Field java/lang/System.out:Ljava/io/PrintStream;
        31: aload_1
        32: aload_2
        33: invokevirtual #19                 // Method java/lang/Integer.equals:(Ljava/lang/Object;)Z
        36: invokevirtual #23                 // Method java/io/PrintStream.println:(Z)V
        39: return
      LineNumberTable:
        line 3: 0
        line 4: 7
        line 5: 14
        line 6: 21
        line 7: 28
        line 8: 39
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      40     0  args   [Ljava/lang/String;
            7      33     1    a1   Ljava/lang/Integer;
           14      26     2    a2   Ljava/lang/Integer;
}
SourceFile: "example.java"

```

`Code:`给出了汇编代码，发现jvm使用了栈结构（传统的x86汇编使用寄存器）。产生的汇编代码使用Java解释器执行。然后