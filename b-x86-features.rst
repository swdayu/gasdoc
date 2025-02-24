x86平台特性
===========

* `特殊命令`_
* `语法风格`_
* `指令助记符`_
* `寄存器名称`_
* `指令前缀`_
* `内存引用`_
* `跳转指令`_
* `浮点指令`_
* `位操作指令`_
* `16位代码`_
* `指定处理器`_
* `指令集架构`_
* `乘法指令`_

x86 版本的汇编器支持原本的 Intel386 架构的16位或32位模式，同时也支持 AMD x86-64 架构
对 Intel 架构的64位扩展。

特殊命令
---------

.lcomm symbol, length[, alignment]
    为符号（symbol）指定的局部通用块预留 length（绝对值表达式）字节。符号的分区和值是
    新的局部通用块对应的分区和值。地址在 bss 分区中分配，因此开始运行时这些字节为全零。
    由于符号没有声明为是全局的，它通常对链接器不可见。可以选的第三个参数（alignment）
    指定了符号在 bss 分区中的地址对齐需求。该命令仅在 x86 平台上的 COFF 格式可用。
.largecomm symbol, length[, alignment]
    跟 comm 命令类似，不同的是数据会输出到 .lbss 分区而不是 .bss 分区；这个命令仅用于
    数据需要大量空间的情况，并仅在 x86_64 平台上的 ELF 格式可用
.value expression[, expression]
    跟 .short 命令的行为一样，将一系列两字节宽度的值存到当前段中
.noopt
    关闭指令大小的优化
.insn [prefix[, ...]] [encoding] major-opcode[+r|/extension][, operand[, ...]]
    这个命令允许编写汇编器可能还不了解的，或者无法表达的指令（例如某些替代编码）。它对
    操作数是怎样被编码的只能假设某些基本结构，并且它也只能识别（除了下面的一些扩展之
    外）对指令有效的操作数。因此不能保证任何内容都能够被表达，例如原始的 Intel Xeon 
    Phi 的 MVEX 编码就无法表示。

其中 .insn 命令的参数解释如下：

参数 prefix 以通常的方式表示一个或多个操作数前缀。改变含义的旧编码前缀（0x66，0xF2，
0xF3）可能在 major-opcode 的高字节指定（也许已经包含了一个编码空间前缀），注意只能有一
个这样的前缀。只要存在对应的内存操作数，段寄存器最好在其中指定。

参数 encoding 用来指定 VEX，XOP，或者 EVEX 编码。语法与文档中使用的相似： ::

    VEX[.len][.prefix][.space][.w]
    EVEX[.len][.prefix][.space][.w]
    XOPspace[.len][.prefix][.w]

其中，len 可以是 LIG、128、256、512（仅 EVEX）、L0/L1（VEX/XOP）、L0...L3（EVEX）；
prefix 可以是 NP、66、F3、F2；space 可以是 VEX 的 0f、0f38、0f3a、M0...M31，XOP 的
08...1f，EVEX 的 0f、0f38、0f3a、M0...M15；w 可以是 WIG、W0、W1。默认值为，如果不指
定 len 表示在至少存在一个已知大小的向量操作数的情况下从操作数推断，否则值为 LIG（显然当
操作数后面指定了 EVEX 舍入控制时，必须省略 len）；不指定 prefix 表示 NP；不指定 space
（仅 VEX/EVEX）表示编码空间从 major-opcode 中获取；不指定 w 表示在64位代码中当至少有
一个 GPR（或相似）操作数时，从 GPR 操作数的大小推导，否则值为 WIG。

参数 major-opcode 是一个绝对值表达式，用来指定指令的操作码。改变编码空间的旧编码前缀
（0x0f，0x0f38，0x0f3a）必须在高字节中指定。像某些 FPU 操作码或像 0x0f01 这样的主操作
码（major opcode）的子空间，其中出现的退化 ModR/M 字节，通常希望作为立即数操作数进行编
码（这些操作码通常不会有非立即数操作数）；一些情况下也可以将这些编码在 major-opcode 的
低字节中，但存在潜在的歧义。还要注意，在剥离编码前缀后，剩余的部分必须适合到两个字节中。
可以在主操作码表达式后添加 +r 后缀，来指定仅寄存器的编码形式，但不使用 ModR/M 字节。或
者可以在主操作码表达式后加上 /extension 后缀，以指定扩展操作码，该操作码由 ModR/M 字节
的第3~5比特位定义。

参数 operand 是以通常方式表示的指令操作数。寄存器操作数主要用于表示 ModR/M 字节和 
REX/VEX/XOP/EVEX 前缀中编码的寄存器编号。在某些情况下，寄存器类型（实际上是大小）也用
于派生其他编码属性，如果这些属性没有显式指定的话。注意操作数之间没有一致性检查，因此操作
数组合可能完全错误。另外，只有在指令中实际编码的操作数才应该被指定。像移位或循环移位指令
中的像 %cl 这样的操作数必须省略，否则它们将被编码位普通（寄存器）操作数。操作数的顺序也
可能与实际指令的顺序不匹配，见下文。

操作数的编码：虽然对于内存操作数（只能有一个）在结果中的 ModR/M 字节中如何编码是明确
的，但寄存器操作数严格按照以下顺序编码。操作数计数不包含下面所列表中的立即数，如果指定了
扩展操作码则它被算作是寄存器操作数，VEX.vvvv 也同时涵盖 XOP 和 EVEX：

1. 对于1个寄存器操作数的 VEX/XOP/EVEX 指令，使用 VEX.vvvv
2. 对于2个操作数的指令，使用 ModR/M.rm 和 ModR/M.reg
3. 对于3个操作数的指令，使用 ModR/M.rm、VEX.vvvv、ModR/M.reg
4. 对于4个操作数的指令，使用 Imm{4,5}、ModR/M.rm、VEX.vvvv、ModR/M.reg

显然，如果有内存操作数需要跳过 ModR/M.rm，如果有扩展操作码需要跳过 ModR/M.reg。对于
Intel 风格语法，要使用相反的操作数顺序。使用 +r 时（即没有 ModR/M 字节），旧编码只能有
一个寄存器操作数。VEX 以及类似的可以有两个寄存器操作数，其中第二个（在 Intel 风格中是
第一个）编码为 VEX.vvvv。

立即操作数（包括类似立即数的移位，即当不是 ModR/M 寻址的一部分时）会按照指定的顺序编
码，不管 AT&T 还是 Intel 风格。由于不可能推断出立即数的类型大小，它们可以通过 {:sn} 或 
{:un} 后缀来表示给定位数的有符号或无符号立即数。在编码这样的操作数时，它的位数会向上取
整到最小的8、16、32、或64位，只有在64位代码中立即数的宽度才允许超过32位。

对于 EVEX 编码，具有移位的内存操作数需要知道 Disp8 的缩放大小，以便使用8位移位。对于很
多指令，这可以从指定的其他操作数的类型推断。在 Intel 语法中，可以使用 dword ptr 等类似
的前缀指定操作数的大小。在 AT&T 语法中，可以通过后缀 {:dn} 来指定内存操作数的字节大
小。这可以和内嵌指定符一起结合使用：``8(%eax){1to8:d8}``。

语法风格
---------

使用 # 字符作为行注释字符，如果该字符出现在一行开头，整行都是注释，但是这种情况下，该行
也可能是一个逻辑行号命令，或者是一个预处理控制命令。如果没有设置 --divide 命令行选项，
斜杠（/）字符也可以用作行注释字符。另外一个特殊字符，分号（;），可以用来在同一行分隔多
条语句。

汇编器支持 AT&T 语法，该语法与 gcc 的输出兼容，使用 .att_syntax 命令切换。还支持
Intel 汇编语法，使用 .intel_syntax 命令切换。这两个命令都有一个可选的参数，prefix 或
noprefix 指定是否在寄存器的名字前使用 % 前缀。这两种语法有很大的不同，这里介绍它们的不
同是因为几乎所有的 x86 文档都使用的是 Intel 汇编语法。这两种语法显著的区别如下：

1. AT&T 立即数使用 $ 字符开头 ``pushl $4``，而 Intel 不需要 ``push 4``
2. AT&T 寄存器操作数使用 % 字符开头，而 Intel 不需要
3. AT&T 绝对值（不同于PC相对值）jump/call 操作数 ``使用*字符`` 开头，而Intel不需要
4. AT&T 使用与 Intel 相反的源操作数和目标操作数顺序，AT&T 的源操作数在前
   ``addl $4, %eax``，而 Intel 源操作数在后 ``add eax, 4``；这种顺序是为了兼容以前
   的 Unix 汇编器，另外注意 bond，invlpga，以及有两个立即数操作数的指令（例如
   enter），不用相反顺序
5. AT&T 内存操作数的大小，通过指令助记符的最后一个字符确定，助记符后缀 ``b、w、l、q``
   分别表示一个字节（byte）、两个字节（word）、四个字节（long）、八个字节（quadruple
   word）；当没有其他方法区分指令时，使用助记符后缀 x、y、z 来表示 xmm（128位向量）、
   ymm（256位向量）、zmm（512位向量），即操作数的大小为 16/32/64 个字节；例如
   ``movb foo,%al``
6. 而 Intel 使用内存操作数前缀（不是助记符）达到相同的目的，byte ptr、word ptr、
   dword ptr、qword ptr、xmmword ptr、ymmword ptr、zmmword ptr 分别表示一个字节到
   64个字节大小；例如 ``mov al, byte ptr foo``；另外 Intel 还是用 fword ptr、
   tbyte ptr、和 oword ptr 表示48位、80位、128位长度大小；在64位代码中，movabs 可以
   用来编码具有64位位移或立即操作数的 mov 指令
7. AT&T 直接形式的长跳转或调用 ``lcall/ljmp $section,$offset``，而 Intel 对应的语法
   为 ``call/jmp far section:offset``
8. AT&T 远返回指令 ``lret $stack-adjust``，而 Intel 为 ``ret far stack-adjust``

指令助记符
----------

指令助记符后会加上一个字符修饰符，该修饰符用于指定操作数的大小。字母 “b”、“w”、“l” 和
“q” 分别表示字节、字、长字和四倍字操作数。如果一条指令没有指定后缀，那么汇编器会尝试根
据目标寄存器操作数（按照惯例是最后一个操作数）来补充缺失的后缀。因此，“mov %ax, %bx”
等同于 “movw %ax, %bx”；同样，“mov $1, %bx” 等同于 “movw $1, %bx”。请注意，这与
AT&T Unix 汇编器不兼容，AT&T Unix 汇编器假定缺失的助记符后缀意味着操作数大小为长字。
这种不兼容性不会影响编译器的输出，因为编译器总是会明确指定助记符后缀。

当没有大小后缀，并且没有（合适的）寄存器操作数来推断内存操作数的大小时，除了少数例外情
况，内存操作数的默认大小在32位或64位模式是4字节（long），在16位模式为2字节（short）。
显著的例外情况如下：

1. 在64位模式下，具有隐式栈上操作数的指令还有分支指令，操作数的大小位8自己（quad）
2. 有符号并且零扩展的 mov 指令，源操作数的默认大小位1字节（byte）
3. 使用整数操作数的浮点指令，由于历史原因操作数默认大小为2字节（short）
4. 使用64位目标操作数的 crc32 指令，源操作数的默认大小为8字节（quad）

通过伪前缀，可以指定不同的编码选项：

1. {disp8} - 优先使用8位移位
2. {disp16} - 优先使用16位移位
3. {disp32} - 优先使用32位移位
4. {load} - 优先使用 load 形式的指令
5. {store} - 优先使用 store 形式的指令
6. {vex} - 使用 VEX 前缀编码
7. {vex3} - 使用3字节的 VEX 前缀编码
8. {evex} - 使用 EVEX 前缀编码
9. {rex} - 整型指令以及旧向量指令优先使用 REX 前缀（仅 x86-64）。注意这不同于 rex 前
   缀，rex 前缀会无条件生成 REX 前缀
10. {rex2} - 整型指令以及旧向量指令优先使用 REX2 前缀（仅 APX_F）
11. {nooptimize} - 关闭对指令大小的优化

Intel VNNI/IFMA 指令助记符默认使用 EVEX 前缀进行编码。伪前缀 {vex} 可以指定使用 VEX
前缀对这些指令助记符进行编码。

下面这些 Intel 风格的类型转换指令，在 AT&T 上被称为 cbtw、cwtl、cwtd、cltd、cltq、
cqto，汇编器接受这两种方式的名字。其中 byte 表示1字节，word 和 short 表示2字节，dword
和 long 表示4字节，qword 和 quad 表示8字节，oword 和 octu 表示16字节。

1. cbw  -  byte  %al 转换成  word       %ax （byte to word）
2. cwde -  word  %ax 转换成 dword      %eax （word to long）
3. cwd  -  word  %ax 转换成 dword   %dx:%ax （word to double word）
4. cdq  - dword %eax 转换成 qword %edx:%eax （long to double long）
5. cdqe - dword %eax 转换成 qword      %rax （long to quad）
6. cqo  - qword %rax 转换成 oword %rdx:%rax （quad to octuple）

以下是 Intel 风格的位扩展指令，每一行最后是对应的 AT&T 风格指令：

1. movsx  - reg8/mem8 符号位扩展到 reg16    - movsbw movsxb movsx
2. movsx  - reg8/mem8 符号位扩展到 reg32    - movsbl movsxb movsx
3. movsx  - reg8/mem8 符号位扩展到 reg64    - movsbq movsxb movsx
4. movsx  - reg16/mem16 符号位扩展到 reg32  - movswl movsxw
5. movsx  - reg16/mem16 符号位扩展到 reg64  - movswq movsxw
6. movsxd - reg32/mem32 符号位扩展到 reg64  - movslq movsxl
7. movzx  - reg8/mem8 高位零扩展到 reg16    - movzbw movzxb movzx
8. movzx  - reg8/mem8 高位零扩展到 reg32    - movzbl movzxb movzx
9. movzx  - reg8/mem8 高位零扩展到 reg64    - movzbq movzxb movzx
10. movzx - reg16/mem16 高位零扩展到 reg32  - movzwl movzxw
11. movzx - reg16/mem16 高位零扩展到 reg64  - movzwq movzxw

Intel 风格的长调用或长跳转指令 ``call far``，``jump far``；而 AT&T 风格的指令为
``lcall``， ``ljmp``。

汇编器支持使用 .intel_mnemonic 命令选择 Intel 风格的助记符，支持使用 .att_mnemonit
命令切换回与 gcc 输出兼容的 AT&T 风格助记符。一些 x87 指令，fadd、fdiv、fdivp、
fdivr、fdivrp、fmul、fsub、fsubp、fsubr、fsubrp，是与 Intel 不同的 AT&T 助记符，
gcc 使用 AT&T 助记符来生成这些指令。AT&T 助记符 movslq 只能接受64位目的寄存器。AT&T 
和 Intel 的助记符 movsxd 可以用来编码16位或32位目的寄存器。

寄存器名称
----------

寄存器操作数总是使用 % 字符开头，80386 的寄存器包括：

1. 8个32位寄存器：%eax %ebx %ecx %edx %edi %esi %ebp %esp
2. 8个低16位寄存器：%ax %bx %cx %dx %di %si %bp %sp
3. 8个高低8位寄存器：%ah %al %bh %bl %ch %cl %dh %dl
4. 6个16位段寄存器：%cs %dx %ss %es %fs %gs
5. 5个处理器控制寄存器：%cr0 %cr2 %cr3 %cr4 %cr8
6. 6个调试寄存器：%db0 %db1 %db2 %db3 %db6 %db7
7. 2个测试寄存器：%tr6 %tr7
8. 8个浮点寄存器：%st 或 %st(0)，%st(1) ~ %st(7)，它们与 MMX 寄存器重叠 %mm0 ~ %mm7
9. 8个128位的 SSE 寄存器：%xmm0 ~ %xmm7

x86-64 架构扩展的寄存器包括：

1. 8个64位寄存器：%rax %rdx %rcx %rdx %rdi %rsi %rbp %rsp
2. 8个额外的64位寄存器：%r8 ~ %r15
3. 8个低32位寄存器：%r8d ~ %r15d
4. 8个低16位寄存器：%r8w ~ %r15w
5. 8个低8位寄存器：%r8b ~ %r15b
6. 4个低8位寄存器：%sil %dil %bpl %spl
7. 8个额外的调试寄存器：%db8 ~ %db15
8. 8个额外的 SSE 寄存器：%xmm8 ~ %xmm15

AVX 扩展包含的寄存器：

1. 16个256位 SSE 寄存器：%ymm0 ~ %ymm15，在32位模式下只能使用前8个
2. 其中的低128位对应 %xmm0 ~ %xmm15

AVX512 扩展包含的寄存器：

1. 32个512位寄存器：%zmm0 ~ %zmm31，在32位模式下只能使用前8个
2. 其中低256位对应 %ymm0 ~ %ymm31
3. 其中低128位对应 %xmm0 ~ %xmm31
4. 8个向量掩码寄存器：%k0 ~ %k7

指令前缀
---------

指令前缀可以用来重复字符串指令，提供段基地址，执行锁总线操作，改变操作数和地址大小。大多
数在正常情况下使用32位操作数的指令，如果有 operand size 前缀，会使用16位大小的操作数。
指令前缀最好与指令写到同一行，例如字符串扫描指令 scas（scan string）使用重复执行前缀：
``repne scas %es:(%edi),%al``。

以下是指令前缀列表：

1. 使用段寄存器名称前缀 cs ds ss es fs gs，会自动以 section:memory-operand 形式为内
   存数据添加段基地址
2. 操作数或地址大小前缀 data16 和 addr16 将32位操作数或地址转换成16位操作数或地址，
   data32 和 addr32 将16位值（在 .code16 分区）转换成32位操作数或地址。例如在16位代
   码分区可以使用： ``addr32 jmpl *(%ebx)``
3. 锁总线前缀 lock 在执行指令的过程中禁止中断，只对特定的以下指令合法
4. 重复执行前缀 rep repe repne，可以加到字符串指令之前让指令执行 %ecx 次（16位模式执
   行 %cx 次）
5. x86-64架构上用来对i386指令集编码扩展的 rex 系列前缀，rex 有四个比特，一个操作数大
   小比特（64）将32位大小的操作数变成64位，另外 X、Y、Z 扩展比特用来扩展寄存器集合。可
   以直接属性 rex 前缀，例如 rex64xyz 设置了所有的比特。通常不需要显式写该前缀，因为汇
   编器会根据指令的操作数自动生成。

内存引用
---------

间接内存引用的 Intel 风格语法 ``section:[base + index*scale + disp]``，AT&T 风格的
语法 ``section:disp(base, index, scale)``。其中 base 和 index 是可选的32位寄存器，
disp 是一个可选的偏移，scale 是 1、2、4、8 用来乘以 index 计算操作数的地址，如果没有
指定 scale，其值为 1。section 指定为内存操作数指定可选的段寄存器，用来覆盖默认的段寄存
器。在 AT&T 风格中，段寄存器必须使用 % 前缀。如果使用的段寄存器与默认的相同，汇编器不会
输出指令的段寄存器。下面是一些例子：

1. AT&T ``-4(%ebp)`` Intel ``[ebp - 4]``，默认段寄存器是 %ss
2. AT&T ``foo(,%eax,4)`` Intel ``[eax * 4 + foo]``，默认段寄存器是 %ds
3. AT&T ``foo(,1)`` Intel ``[foo]``
4. AT&T ``%gs:foo`` Intel ``gs:foo``，段 %gs 中变量 foo 的值

**绝对地址调用或跳转**

绝对地址（不同于PC相对地址）调用或跳转的操作数必须使用 * 字符前缀，如果没有该前缀，汇编
器总会选择 PC 相对地址用于调用或跳转。如果一条指令没有寄存器操作数，只有内存操作数，必
须使用操作数大小后缀（b、w、l、q）。

x86-64 架构添加了 rip 指令指针相对寻址，这种寻址模式使用 rip 寄存器作为基寄存器，并且
只允许常量地址偏移：

1. AT&T ``1234(%rip)`` Intel ``[rip + 1234]``，当前指令之后 1234 个字节对应的地址
2. AT&T ``symbol(%rip)`` Intel ``[rip + symbol]``，使用 rip 相对寻址方式指向符号，
   比默认的绝对寻址方式更短

其他的寻址模式在 x86-64 架构上保持不变，除了寄存器使用的是64位而不是32位。

跳转指令
---------

跳转指令总是被优化使用最小可能的地址偏移，如果目标足够近会使用一个字节表示的偏移，如果不
够则使用4字节表示偏移。在32位模式下不支持2字节偏移的跳转（即不能使用 data16 指令前
缀），因为这样会导致 80386 坚持将 %eip 掩码到16位。

注意 jcxz jecxz loop loopz loope loopnz loopne 指令仅使用单字节编译，如果你使用这些
指令（gcc 不会使用这些指令），你可能会得到错误消息和不正确代码。AT&T 80386 汇编器通过
将 ``jcxz foo`` 进行扩展来规避这个问题： ::

                jcxz cx_zero
                jmp cx_nonzero
    cx_zero:    jmp foo
    cx_nonzero:

浮点指令
--------

所有的 80387 浮点类型除了 packed BCD 都支持，添加对 BCD 的支持也不太难。这些数据类型
是16位、32位、64位整型，和单精度（32位）、双精度（64位）、扩展精度（80位）浮点。每种支
持的类型都有一个对应的指令助记符后缀和一个关联的构造器，助记符后缀指定操作数的数据类型，
构造器用于将这些数据类型构建到内存：

1. 浮点构造器有 .float 或 .single（32位）、.double（64位），.tfloat（80位），
   .hfloat 或 .bfloat16（16位），前三种对应的助记符后缀为 s、l、t（代表80位，ten
   byte）；对于扩展精度浮点，80387 仅支持 fldt 加载80位浮点到栈顶，fstpt 存储80位浮点
   并出栈
2. 整数构造器有 .word（16位）、.long 或 .int（32位）、.quad（64位），对应的助记符后
   缀为 s（短整型，short）、l、q。与80位浮点格式类似，64位的 q 格式仅存在于 fildq 和
   fistpq 指令中

寄存器到寄存器的操作不应该使用指令助记符后缀，例如 ``fstl %st,%st(1)`` 会警告，会汇编
成 ``fst %st,%st(1)``，因为都是80位浮点操作数。而相反的， ``%fstl %st,mem`` 则表示将
80位浮点转换成64位浮点并将结果保存到32位的内存位置中。

所有源自 AT&T 的 ix86 Unix 汇编器，在某些情况下生成的浮点指令，其源寄存器和目的寄存器
是相反的。不幸的是，gcc 以及可能许多其他程序都是用这种相反的语法，所以我们只能接受它。

例如 ``fsub %st,%st(3)`` 是将 ``%st-%st(3)`` 的结果保存到 ``%st(3)``，而不是指令期
待的 ``%st(3)-%st``。这种情况发生在所有使用两个寄存器作为操作数的非交换算术浮点运算
中，其中源寄存器是 %st，目的寄存器是 %st(i)。

位操作指令
----------

汇编器支持 BMI（Bit Manipulation）位操作指令集，也支持 AMD 的 TBM（Trailing Bit
Manipulation）指令集。TBM 指令集在 AMD BDVER2 处理器（Trinity 和 Viperfish）上可
用。TBM 指令集不仅支持隔离、掩码、置位、清位操作，还支持取反以及尾部的比特0和比特1操
作。

16位代码
---------

根据默认的配置，汇编器正常仅支持纯32位的 i386 代码以及64位的 x86-64 代码。但是它还支持
编写代码在实模式或者16位保护模式的代码段中运行，只需要使用 .code16 或 .code16gcc 命令
将汇编语言指令的运行模式切换到16位。可以使用 .code23 和 .code64 切换回32位代码模式和
64位代码模式。

.code16gcc 命令提供一种实验环境，为 gcc 生成16位代码，并且与 .code16 不同的是，
call、ret、enter、leave、push、pop、pusha、popa、pushf、popf 这些指令默认都是用32位
大小操作数。因此，在函数调用上可以使用相同的方式操作栈指针，允许以32位模式在相同的栈偏
移处访问函数参数。在 gcc 生成的需要使用32位地址模式的地方，.code16gcc 还会自动添加地址
大小前缀。

汇编器生成的16位代码不一定要在80386之前的16位处理器上运行，但是如果一定要在这些处理器上
运行，必须避免使用任何32位用来让汇编器输出地址或操作数大小前缀的构造器。

注意在16位代码段中使用显式指定的大小前缀或指令助记符后缀，最后产生的机器码与对应的32位
代码不同。在32位代码段中，代码 ``pushw $4`` 产生的机器码是 66 6a 04，它将 4 入栈并将
栈顶指针 %esp 减 2。但相同的代码在16位代码段中产生的机器码是 6a 04，即去掉了操作数大小
后缀，这才是正确的因为16位代码段中处理器默认的操作数大小被假定为16位。

指定处理器
----------

通过是使用 .arch cpu_type 命令，可以指定一个特定的处理器架构。如果存在该处理器不支持的
指令，汇编器会报警告。处理器类型（cpu_type）如下： ::

    ‘default’ ‘push’ ‘pop’
    ‘i8086’ ‘i186’ ‘i286’ ‘i386’
    ‘i486’ ‘i586’ ‘i686’ ‘pentium’
    ‘pentiumpro’ ‘pentiumii’ ‘pentiumiii’ ‘pentium4’
    ‘prescott’ ‘nocona’ ‘core’ ‘core2’
    ‘corei7’ ‘iamcu’
    ‘k6’ ‘k6_2’ ‘athlon’ ‘k8’
    ‘amdfam10’ ‘bdver1’ ‘bdver2’ ‘bdver3’
    ‘bdver4’ ‘znver1’ ‘znver2’ ‘znver3’
    ‘znver4’ ‘znver5’ ‘btver1’ ‘btver2’
    ‘generic32’
    ‘generic64’ ‘.cmov’ ‘.fxsr’ ‘.mmx’
    ‘.sse’ ‘.sse2’ ‘.sse3’ ‘.sse4a’
    ‘.ssse3’ ‘.sse4.1’ ‘.sse4.2’ ‘.sse4’
    ‘.avx’ ‘.vmx’ ‘.smx’ ‘.ept’
    ‘.clflush’ ‘.movbe’ ‘.xsave’ ‘.xsaveopt’
    ‘.aes’ ‘.pclmul’ ‘.fma’ ‘.fsgsbase’
    ‘.rdrnd’ ‘.f16c’ ‘.avx2’ ‘.bmi2’
    ‘.lzcnt’ ‘.popcnt’ ‘.invpcid’ ‘.vmfunc’
    ‘.monitor’ ‘.hle’ ‘.rtm’ ‘.tsx’
    ‘.lahf_sahf’ ‘.adx’ ‘.rdseed’ ‘.prfchw’
    ‘.smap’ ‘.mpx’ ‘.sha’ ‘.prefetchwt1’
    ‘.clflushopt’ ‘.xsavec’ ‘.xsaves’ ‘.se1’
    ‘.avx512f’ ‘.avx512cd’ ‘.avx512er’ ‘.avx512pf’
    ‘.avx512vl’ ‘.avx512bw’ ‘.avx512dq’ ‘.avx512ifma’
    ‘.avx512vbmi’ ‘.avx512_4fmaps’‘.avx512_4vnniw’
    ‘.avx512_vpopcntdq’‘.avx512_vbmi2’ ‘.avx512_vnni’
    ‘.avx512_bitalg’ ‘.avx512_bf16’ ‘.avx512_vp2intersect’
    ‘.tdx’ ‘.avx_vnni’ ‘.avx512_fp16’ ‘.avx10.1’
    ‘.clwb’ ‘.rdpid’ ‘.ptwrite’ ‘.ibt’
    ‘.prefetchi’ ‘.avx_ifma’ ‘.avx_vnni_int8’
    ‘.cmpccxadd’ ‘.wrmsrns’ ‘.msrlist’
    ‘.avx_ne_convert’ ‘.rao_int’ ‘.fred’ ‘.lkgs’
    ‘.avx_vnni_int16’ ‘.sha512’ ‘.sm3’ ‘.sm4’
    ‘.pbndkb’ ‘.user_msr’
    ‘.wbnoinvd’ ‘.pconfig’ ‘.waitpkg’ ‘.cldemote’
    ‘.shstk’ ‘.gfni’ ‘.vaes’ ‘.vpclmulqdq’
    ‘.movdiri’ ‘.movdir64b’ ‘.enqcmd’ ‘.tsxldtrk’
    ‘.amx_int8’ ‘.amx_bf16’ ‘.amx_fp16’
    ‘.amx_complex’ ‘.amx_tile’
    ‘.kl’ ‘.widekl’ ‘.uintr’ ‘.hreset’
    ‘.3dnow’ ‘.3dnowa’ ‘.sse4a’ ‘.sse5’
    ‘.syscall’ ‘.rdtscp’ ‘.svme’
    ‘.lwp’ ‘.fma4’ ‘.xop’ ‘.cx16’
    ‘.padlock’ ‘.clzero’ ‘.mwaitx’ ‘.rdpru’
    ‘.mcommit’ ‘.sev_es’ ‘.snp’ ‘.invlpgb’
    ‘.tlbsync’ ‘.apx_f’

除了报警告，在操作上还有两个不同的效果。第一，如果你指定除 i486 之外的处理器，移动1位的
移位指令（例如 ``sarl $1,%eax``）会自动使用两字节操作码序列。更大的三字节序列被用在
486 处理器上（或没特别指定处理器），因为这样在 486 机器上执行得更快。注意，可以这样使用
指令 ``sarl %eax`` 显式地要求使用两字节操作码。第二，如果指定 i8086、i186、i286，并且
使用 .code16 或 .code16gcc，那么字节偏移条件跳转会在必要时被提升为由两个指令组成的序
列，包括一个相反意义的条件跳转和一个到目标的无条件跳转。

注意以点字符开头的子架构指定符，可以添加前缀 no 来撤销对应的以及任何依赖的功能。另外注
意 ``.avx10.<N>`` 可以加上向量长度限制后缀 /256 或 /128，并且使用 /512 可以恢复默认
的向量长度。尽管这些通常是用来启用对应的指定符，但是使用这些后缀会禁用所有具有更宽长度的
向量或掩码寄存器操作数。在 SVR4 派生平台上，分隔符（/）可以使用冒号（:）代替。

在处理器架构后面（不是以点字符开始的子架构），可以指定 jumps 或者 nojumps 控制条件跳转
的自动提升。jumps 是默认值，即开启跳转提升，所有外部跳转都是一个长跳转，文件局部跳转将
在需要时提升。而 nojumps 将外部条件跳转视为字节偏移跳转，并且警告汇编器提升的文件局部条
件跳转。无条件跳转按照 jumps 方式处理。例如 ``.arch i8086,nojumps``。

指令集架构
----------

ADM64 指令集架构（ISA, instruction set architecture）与 Intel64 指令集结构由一些差
异，如下：

1. movsxd 操作16位目的寄存器，AMD64 支持32位源操作数，Intel64支持16位源操作数
2. 显式使用内存操作数的长分支指令，两个指令集架构都支持32位和16位大小的操作数，Intel64
   额外支持64位大小操作数，用 AT&T 语法 ``ljmpq/lcallq`` 和 Intel 风格的使用 tbyte
   ptr 操作数前缀的指令编码
3. 指令 lfs、lgs、lss 类似，允许16位和32位大小的操作数（即32位和48位内存操作数），而
   Intel64 额外支持64位大小的操作数（即80位内存操作数）

乘法指令
--------

关于 mul 和 imul 指令有一些值得注意的技巧，16位、32位、64位、和128位的扩展乘法（基操作
码为 0xf6，mul 的扩展码为 4，imul 的扩展码为 5）只能以单操作数形式输出。因此
``imul %ebx,%eax`` 并不会扩展乘法，因为扩展乘法会影响 %edx 寄存器导致混淆 gcc 输出。
可以使用 ``imul %ebx`` 获得 %edx:%eax 中的乘积结果。

当第一个操作数是立即数表达式，第二个操作数是寄存器时，我们为 imul 添加了上操作数模式。
这只是一种简写方式，因此将 %eax 乘以 69，可以使用 ``imul $69,%eax`` 而不是
``imul $69,%eax,%eax``。
