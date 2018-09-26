---
title: Define 宏
date: 2018-01-18 16:21:20
tags: [define, amber, blog, inline]
---

# define用法集锦

Syntax
 #define identifier token-stringopt
 #define identifier(identifieropt , ... , identifieropt) token-stringopt


# 宏的基本内容

1. 程序第一步是在预编译之前会有一些操作，例如删除反斜线和换行符的组合，将每个注释用一个空格替代...
2. 然后在进入预编译的时候，寻找可能存在的预处理指定（由#开头），例如 C 中常用的 #include，或者 oc 中的 #import,#define...
3. 处理 #define 的时候，然后预处理器会从 # 开始，一直执行到第一个换行符（写代码的时候换行的作用），自然，#define 只会允许定义一行的宏，不过正因为上面提到的预处理之前会删除反斜线和换行符的组合，所以可以利用反斜线定义多行宏，在删除反斜线和换行符的组合后，逻辑上就成了一行的宏了
4. 红作用在预编译使其，其真正的效果就是代码替换，而且是直接替换（内联函数），这个和函数有着很大的区别，并且正因为是直接替换，在使用的时候就会有一些的注意点
5. 红可以被称为类对象红，类函数宏
6. 宏在预处理阶段只进行文本的替换，不会进行具体的计算（发生在编译使其）


# 用法
 #define A(x) T_##x     => T_1
 #define B(x) #@x       => '1'
 #define C(x) #x        => "1"
 
 
# 多行定义

# 定义宏，取消宏
//定义宏 #define [MacroName] [MacroValue] 
//取消宏 #undef [MacroName] 
//普通宏 #define PI (3.1415926)
//带参数的宏 #define max(a,b) ((a)>(b)? (a),(b)) 

通过条件编译开关来避免重复包含(重复定义) 
例如 #ifndef __headerfileXXX__ #define __headerfileXXX__ … //文件内容 … #endif

1、防止一个头文件被重复包含
    #ifndef COMDEF_H
    #define COMDEF_H
     //头文件内容
    #endif
2、重新定义一些类型,防止由于各种平台和编译器的不同,而产生的类型字节数差异,方便移植。
typedef  unsigned char      boolean;     // Boolean value type. 
typedef  unsigned long int  uint32;      // Unsigned 32 bit value 
typedef  unsigned short     uint16;      // Unsigned 16 bit value 
typedef  unsigned char      uint8;       // Unsigned 8  bit value 
typedef  signed long int    int32;       // Signed 32 bit value 
typedef  signed short       int16;       // Signed 16 bit value 
typedef  signed char        int8;        // Signed 8  bit value 
//下面的不建议使用
typedef  unsigned char     byte;         // Unsigned 8  bit value type. 
typedef  unsigned short    word;         // Unsinged 16 bit value type. 
typedef  unsigned long     dword;        // Unsigned 32 bit value type. 
typedef  unsigned char     uint1;        // Unsigned 8  bit value type. 
typedef  unsigned short    uint2;        // Unsigned 16 bit value type. 
typedef  unsigned long     uint4;        // Unsigned 32 bit value type. 
typedef  signed char       int1;         // Signed 8  bit value type. 
typedef  signed short      int2;         // Signed 16 bit value type. 
typedef  long int          int4;         // Signed 32 bit value type. 
typedef  signed long       sint31;       // Signed 32 bit value 
typedef  signed short      sint15;       // Signed 16 bit value 
typedef  signed char       sint7;        // Signed 8  bit value 

3、得到指定地址上的一个字节或字
    #define  MEM_B( x )  ( *( (byte *) (x) ) )
    #define  MEM_W( x )  ( *( (word *) (x) ) )
    
4、求最大值和最小值
   #define  MAX( x, y ) ( ((x) > (y)) ? (x) : (y) )
   #define  MIN( x, y ) ( ((x) < (y)) ? (x) : (y) )
   
5、得到一个field在结构体(struct)中的偏移量
    #define FPOS( type, field ) /
//lint -e545 // 
( (dword) &;(( type *) 0)-> field ) 
//lint +e545 //

6、得到一个结构体中field所占用的字节数
    #define FSIZ( type, field ) sizeof( ((type *) 0)->field )
    
7、按照LSB格式把两个字节转化为一个Word
    #define  FLIPW( ray ) ( (((word) (ray)[0]) * 256) + (ray)[1] )
    
8、按照LSB格式把一个Word转化为两个字节
    #define  FLOPW( ray, val ) /
  (ray)[0] = ((val) / 256); /
  (ray)[1] = ((val) &; 0xFF)
  
9、得到一个变量的地址(word宽度)
    #define  B_PTR( var )  ( (byte *) (void *) &;(var) )
    #define  W_PTR( var )  ( (word *) (void *) &;(var) )
    
10、得到一个字的高位和低位字节
    #define  WORD_LO(xxx)  ((byte) ((word)(xxx) &; 255))
    #define  WORD_HI(xxx)  ((byte) ((word)(xxx) >> 8))
    
11、返回一个比X大的最接近的8的倍数
    #define RND8( x )       ((((x) + 7) / 8 ) * 8 )
    
12、将一个字母转换为大写
    #define  UPCASE( c ) ( ((c) >= 'a' &;&; (c) <= 'z') ? ((c) - 0x20) : (c) )
    
13、判断字符是不是10进值的数字
    #define  DECCHK( c ) ((c) >= '0' &;&; (c) <= '9')
    
14、判断字符是不是16进值的数字
    #define  HEXCHK( c ) ( ((c) >= '0' &;&; (c) <= '9') ||/
                       ((c) >= 'A' &;&; (c) <= 'F') ||/
((c) >= 'a' &;&; (c) <= 'f') )

15、防止溢出的一个方法
    #define  INC_SAT( val )  (val = ((val)+1 > (val)) ? (val)+1 : (val))
    
16、返回数组元素的个数
    #define  ARR_SIZE( a )  ( sizeof( (a) ) / sizeof( (a[0]) ) )
    
17、返回一个无符号数n尾的值MOD_BY_POWER_OF_TWO(X,n)=X%(2^n)
    #define MOD_BY_POWER_OF_TWO( val, mod_by ) /
           ( (dword)(val) &; (dword)((mod_by)-1) )
           
18、对于IO空间映射在存储空间的结构,输入输出处理

  #define inp(port)         (\*((volatile byte \*) (port)))
  #define inpw(port)        (\*((volatile word \*) (port)))
  #define inpdw(port)       (\*((volatile dword \*)(port)))
  #define outp(port, val)   (\*((volatile byte \*) (port)) = ((byte) (val)))
  #define outpw(port, val)  (\*((volatile word \*) (port)) = ((word) (val)))
  #define outpdw(port, val) (\*((volatile dword \*) (port)) = ((dword) (val)))
  
19、使用一些宏跟踪调试
ANSI标准说明了五个预定义的宏名。它们是:
__LINE__
__FILE__
__DATE__
__TIME__
__STDC__

C++中还定义了 __cplusplus

如果编译器不是标准的,则可能仅支持以上宏名中的几个,或根本不支持。记住编译程序也许还提供其它预定义的宏名。
__LINE__ 及 __FILE__ 宏指示,#line指令可以改变它的值,简单的讲,编译时,它们包含程序的当前行数和文件名。
__DATE__ 宏指令含有形式为月/日/年的串,表示源文件被翻译到代码时的日期。
__TIME__ 宏指令包含程序编译的时间。时间用字符串表示,其形式为:分:秒
__STDC__ 宏指令的意义是编译时定义的。一般来讲,如果__STDC__已经定义,编译器将仅接受不包含任何非标准扩展的标准C/C++代码。如果实现是标准的,则宏__STDC__含有十进制常量1。如果它含有任何其它数,则实现是非标准的。
__cplusplus 与标准c++一致的编译器把它定义为一个包含至少6为的数值。与标准c++不一致的编译器将使用具有5位或更少的数值。
可以定义宏,例如:
当定义了_DEBUG,输出数据信息和所在文件所在行

    #ifdef _DEBUG
    #define DEBUGMSG(msg,date) printf(msg);printf(“%d%d%d”,date,_LINE_,_FILE_)
    #else
    #define DEBUGMSG(msg,date)
    #endif
 
20、宏定义防止错误使用小括号包含。
例如:
有问题的定义:#define DUMP_WRITE(addr,nr) {memcpy(bufp,addr,nr); bufp += nr;}
应该使用的定义: #difne DO(a,b) do{a+b;a++;}while(0)
例如:
if(addr)
    DUMP_WRITE(addr,nr);
else
    do_somethong_else();
宏展开以后变成这样:
if(addr)
    {memcpy(bufp,addr,nr); bufp += nr;};
else
    do_something_else();
gcc在碰到else前面的“;”时就认为if语句已经结束,因而后面的else不在if语句中。而采用do{} while(0)的定义,在任何情况下都没有问题。而改为 #difne DO(a,b) do{a+b;a++;}while(0) 的定义则在任何情况下都不会出错
 
21. define中的特殊标识符
 #define Conn(x,y) x##y 
 #define ToChar(x) #@x 
 #define ToString(x) #x
int a=Conn(12,34); char b=ToChar(a); char c[]=ToString(a); 结果是 a=1234,c='a',c='1234';
可以看出 ## 是简单的连接符,#@用来给参数加单引号,#用来给参数加双引号即转成字符串


# define 和 inline 的区别

define：定义预编译时的处理宏
只进行简单的字符替换，无类型检测

## static
一、产生背景
引出原因：函数内部定义的变量，在程序执行到它的定义处时，编译器为它在栈上分配空间，大家知道，函数在栈上分配的空间在此函数执行结束时会释放掉，这样就产生了一个问题: 如果想将函数中此变量的值保存至下一次调用时，如何实现？
最容易想到的方法是定义一个全局的变量，但定义为一个全局变量有许多缺点，最明显的缺点是破坏了此变量的访问范围（使得在此函数中定义的变量，不仅仅受此函数控制）。类的静态成员也是这个道理。
解决方案：因此C++ 中引入了static，用它来修饰变量，它能够指示编译
器将此变量在程序的静态存储区分配空间保存，这样即实现了目的，又使得此变量的存取范围不变。
二、具体作用
Static作用分析总结：static总是使得变量或对象的存储形式变成静态存储，连接方式变成内部连接，对于局部变量（已经是内部连接了），它仅改变其存储方式；对于全局变量（已经是静态存储了），它仅改变其连接类型。(1 连接方式：成为内部连接；2 存储形式：存放在静态全局存储区)

## const
一、产生背景
C++有一个类型严格的编译系统，这使得C++程序的错误在编译阶段即可发现许多，从而使得出错率大为减少，因此，也成为了C++与C相比，有着突出优点的一个方面。
C中很常见的预处理指令 #define VariableName VariableValue 可以很方便地进行值替代，这种值替代至少在三个方面优点突出：
一是避免了意义模糊的数字出现，使得程序语义流畅清晰，如下例：
　　#define USER_NUM_MAX 107 这样就避免了直接使用107带来的困惑。
二是可以很方便地进行参数的调整与修改，如上例，当人数由107变为201时，改动此处即可；
三是提高了程序的执行效率，由于使用了预编译器进行值替代，并不需要为这些常量分配存储空间，所以执行的效率较高。

然而，预处理语句虽然有以上的许多优点，但它有个比较致命的缺点，即，预处理语句仅仅只是简单值替代，缺乏类型的检测机制。这样预处理语句就不能享受C++严格类型检查的好处，从而可能成为引发一系列错误的患。

Const 推出的初始目的，正是为了取代预编译指令，消除它的缺点，同时继承它的优点。
现在它的形式变成了：
Const DataType VariableName = VariableValue ;
2) 具体作用
1.const 用于指针的两种情况分析：
　    int const \*A; 　//A可变，\*A不可变
　    int \*const A; 　//A不可变，\*A可变
　分析：const 是一个左结合的类型修饰符，它与其左侧的类型修饰符和为一个类型修饰符，所以，int const 限定 \*A,不限定A。int \*const 限定A,不限定*A。
2.const 限定函数的传递值参数：
　    void Fun(const int Var);
　    分析：上述写法限定参数在函数体中不可被改变。
3.const 限定函数的值型返回值：
const int Fun1();
const MyClass Fun2();
　    分析:上述写法限定函数的返回值不可被更新，当函数返回内部的类型时（如Fun1），已经是一个数值，当然不可被赋值更新，所以，此时const无意义，最好去掉，以免困惑。当函数返回自定义的类型时（如Fun2），这个类型仍然包含可以被赋值的变量成员，所以，此时有意义。
4. 传递与返回地址： 此种情况最为常见，由地址变量的特点可知，适当使用const，意义昭然。
5.　const 限定类的成员函数：
class ClassName {
　public:
　　int Fun() const;
　.....
}
　　注意：采用此种const 后置的形式是一种规定，亦为了不引起混淆。在此函数的声明中和定义中均要使用const,因为const已经成为类型信息的一部分。
获得能力：可以操作常量对象。
失去能力：不能修改类的数据成员，不能在函数中调用其他不是const的函数。

## inline

1) 产生背景
inline这个关键字的引入原因和const十分相似，inline 关键字用来定义一个类的内联函数，引入它的主要原因是用它替代C中
表达式形式的宏定义。
表达式形式的宏定义一例：
　　　#define ExpressionName(Var1,Var2) (Var1+Var2)*(Var1-Var2)
这种表达式形式宏形式与作用跟函数类似，但它使用预编译器，没有堆栈，使用上比函数高效。但它只是预编译器上符号表的简单替换，不能进行参数有效性检测及使用C++类的成员访问控制。
inline 推出的目的，也正是为了取代这种表达式形式的宏定义，它消除了它的缺点，同时又很好地继承了它的优点。inline代码放入预编译器符号表中，高效；它是个真正的函数，调用时有严格的参数检测；它也可作为类的成员函数。
2) 具体作用
直接在class类定义中定义各函数成员，系统将他们作为内联函数处理；成员函数是内联函数，意味着：每个对象都有该函数一份独立的拷贝。
在类外，如果使用关键字inline定义函数成员，则系统也会作为内联函数处理；


## C关键字
    #define 宏名要替换的代码    
宏定义，保存在预编译器的符号表中，执行高效；作为一种简单的符号替换，不进行其中参数有效性的检测
typedef 已有类型 新类型
别名, 常用于创建平台无关类型, typedef 在编译时被解释，因此让编译器来应付超越预处理器能力的文本替换

## 参考

https://msdn.microsoft.com/en-us/library/teas0593.aspx

https://www.jianshu.com/p/f3917ae186f4

http://www.cnblogs.com/iloveyoucc/archive/2012/03/18/2404658.html

