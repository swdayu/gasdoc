ARM平台特性
===========

ARM汇编选项
-----------

ARM汇编选项汇总： ::

    -mcpu=processor[+extension...]
    -march=architecture[+extension...]
    -mfpu=floating-point-format
    -mfp16-format=format
    -mthumb -mthumb-interwork
    -mimplicit-it=never|always|arm|thumb
    -mapcs-26|-mapcs-32
    -matpcs -mapcs-float -mapcs-reentrant
    -mfloat-abi=abi -meabi=ver
    -EB -EL -k --fix-v4bx
    -mwarn-deprecated -mno-warn-deprecated
    -mccs -mwarn-syms -mno-warn-syms

ARM汇编语法
------------

ARM汇编命令
------------

ARM汇编命令汇总： ::

    .align expression [, expression]
    .arch name .arch_extension name
    .arm .cantunwind .code [16|32]
    .cpu name
    name .dn register name [.type] [[index]]
    name .qn register name [.type] [[index]]
    .eabi_attribute tag, value
    .even
    .extend expression [, expression]*
    .ldouble expression [, expression]*
    .float16 value [,...,value_n]
    .float16_format format
    .fnend
    .fnstart
    .force_thumb
    .fpu name
    .handlerdata
    .inst opcode [ , ... ]
    .inst.n opcode [ , ... ]
    .inst.w opcode [ , ... ]
    .ldouble expression [, expression]*
    .ltorg
    .movsp reg [, #offset]
    .object_arch name
    .packed expression [, expression]*
    .pacspval
    .pad #count
    .personality name
    .personalityindex index
    .pool
    name .req register name
    .save reglist
    .setfp fpreg, spreg [, #offset]
    .secrel32 expression [, expression]*
    .syntax [unified | divided]
    .thumb
    .thumb_func
    .thumb_set
    .tlsdescseq tls-variable
    .unreq alias-name
    .unwind_raw offset, byte1, ...
    .vsave vfp-reglist

AArch64汇编选项
----------------

AArch64汇编选项汇总： ::

    -EB -EL -mabi=abi
    -mcpu=processor[+extension...]
    -march=architecture[+extension...]
    -mverbose-error
    -mno-verbose-error

AArch64汇编语法
----------------

**特殊字符**

行中出现的 // 表示从该位置到当前行末尾的注释开始。如果 # 出现在行的第一个字符，则整行被
视为注释。可以使用 ; 字符代替换行符来分隔语句。# 可以选择性地用于表示立即数操作数。

**寄存器名称**

请参阅 ARMv8 指令集概述中的寄存器名称部分。

**重定位**

可以通过在标签前加上 #:abs_g2: 等前缀来为 MOVZ 和 MOVK 指令生成重定位。例如，将 foo
的 48 位绝对地址加载到 x0 中： ::

    movz x0, #:abs_g2:foo // bits 32-47, overflow check
    movk x0, #:abs_g1_nc:foo // bits 16-31, no overflow check
    movk x0, #:abs_g0_nc:foo // bits 0-15, no overflow check

可以通过在标签前加上 :pg_hi21: 和 #:lo12: 来分别为 ADRP 和 ADD、LDR 或 STR 指令生成
重定位。例如，使用 33 位（+/-4GB）PC 相对寻址将 foo 的地址加载到 x0 中： ::

    adrp x0, :pg_hi21:foo
    add x0, x0, #:lo12:foo

或者将 foo 的值加载到 x0 中： ::

    adrp x0, :pg_hi21:foo
    ldr x0, [x0, #:lo12:foo]

注意，:pg_hi21: 是可选的。 ``adrp x0, foo`` 等同于 ``adrp x0, :pg_hi21:foo``。

**浮点数**

AArch64 架构使用 IEEE 浮点数。

**操作码**

GAS（GNU 汇编器）实现了所有标准的 AArch64 操作码。它还实现了一些伪操作码，包括几个合成的加载指令。

LDR = ::

    ldr <register>, =<expression>

    如果常量表达式尚未在最近的字面量池中，则将其放入最近的字面量池中，并生成一条相对于
    PC 的 LDR 指令。

有关 AArch64 指令集和汇编语言符号的更多信息，请参阅 ARMv8 指令集概述（ARMv8
Instruction Set Overview）部分，对应网站 http://infocenter.arm.com/。

**映射符号**

AArch64 ELF 规范要求在目标文件中插入特殊符号以标记某些特性：

* $x 在包含 AArch64 指令的代码区域的开头。
* $d 在数据区域的开头。

AArch64汇编命令
---------------

AArch64汇编命令汇总： ::

    .arch name
    .arch_extension name
    .cpu name
    .dword expressions
    .even
    .float16 value [,...,value_n]
    .inst expressions
    .ltorg
    .pool
    name .req register name
    .tlsdescadd
    .tlsdesccall
    .tlsdescldr
    .unreq alias-name
    .variant_pcs symbol
    .xword expressions
    .cfi_b_key_frame
