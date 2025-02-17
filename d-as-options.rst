汇编器选项
==========

* `命令行格式`_
* `选项汇总`_
* `i386 选项`_

命令行格式
-----------

在程序名称 as 之后，命令行可以包含选项和文件名。选项可以以任意顺序出现，并且可以在文件
名之前、之后或之间。文件名的顺序是重要的。单独的 “--”（两个连字符）可以指定标准输入文
件，作为要由 as 进行汇编的文件之一。除了“--”之外，任何以连字符（“-”）开头的命令行参数
都是一个选项。每个选项都会改变 as 的行为。一个选项不会改变另一个选项的工作方式。选项是
一个 “-” 后面跟着一个或多个字母；字母的大小写是重要的。所有选项都是可选的。有些选项需要
紧随其后恰好有一个文件名。文件名可以紧跟在选项字母之后（与旧的汇编器兼容），也可以是下一
个命令参数（GNU标准）。这两条命令行是等效的： ::

    as -o my-object-file.o mumble.s
    as -omy-object-file.o mumble.s

输入文件，或者我们使用 “源程序”（source program）这个短语，简称 “源”（source），来描
述输入到一次 as 运行中的程序。该程序可以是一个或多个文件；源代码被分割成文件的方式不会
改变源代码的含义。源程序是所有文件中文字的合集，按照指定的顺序排列。每次运行 as 时，它
都会汇编成一个源程序。源程序由一个或多个文件组成，标准输入也是一个文件。你可以给 as 一
个命令行，其中包含零个或多个输入文件名，输入文件按从左到右的顺序读取。任何位置的命令行参
数，如果没有特殊含义，则被视为输入文件名。如果你没有给 as 任何文件名，它会尝试从 as 的
标准输入读取一个输入文件。你可能需要输入 ctl-D 来告诉 as 没有更多的程序需要汇编。如果
你需要在命令行中明确指定标准输入文件，请使用 ‘--’。如果源代码为空，as 将生成一个小型的
空目标文件。

文件名和行号，有两种方法可以定位输入文件中的某一行，这两种方法都可以用于报告错误消息。一
种方法是指向物理文件中的行号；另一种是指向 “逻辑” 文件中的行号。物理文件是在给 as 的命
令行中命名的文件。逻辑文件仅仅是通过汇编指令（assembler directives）显式声明的名称，它
们与物理文件无关。逻辑文件名有助于将错误消息反映到原始源文件，当 as 的源代码本身是从其
他文件同步过来的情况下。as 理解 gcc 预处理器发出的 ‘#’ 指令。参见 [.file] 部分。

输出（目标）文件，每次运行 as 时，它都会生成一个输出文件，这是你的汇编语言程序被翻译成
数字后的结果。这个文件就是目标文件。其默认名称是 a.out。你可以通过使用 -o 选项来给它指
定另一个名称。按照惯例，目标文件名以 .o 结尾。默认名称的使用出于历史原因：早期的汇编器
能够将自包含的程序直接汇编成可运行的程序。对于某些格式，目前还无法做到这一点，但对于
a.out 格式是可以实现的。目标文件是为链接器 ld 准备的输入文件。它包含已汇编的程序代码、
帮助 ld 将已汇编的程序整合到一个可执行文件中的信息，以及（可选的）供调试器使用的符号信
息。

错误和警告消息，as 可能会将警告和错误消息写入标准错误文件。当编译器自动运行 as 时，这种
情况不应该发生。警告报告了一个假设，以便 as 能够继续汇编一个有缺陷的程序；而错误则报告
了一个严重的问题，导致汇编无法继续。警告消息的格式为：
file_name:NNN:Warning Message Text，其中NNN是行号。如果指定了逻辑文件名（参见
[.file]）和逻辑行号（参见 [.line]），则会使用它们；否则，将使用当前汇编源文件中的文件
名和行号。请注意，文件名必须通过逻辑版本的 .file 指令设置，而不是 DWARF2 版本的 .file
指令。错误消息的格式为：file_name:NNN:FATAL:Error Message Text，文件名和行号的获取方
式与警告消息相同。实际的消息文本可能不太具有说明性，因为其中许多错误本不应发生。

如果你是通过 GNU C 编译器调用 as，你可以使用 ‘-Wa’ 选项将参数传递给汇编器。汇编器的参
数必须用逗号与彼此（以及 ‘-Wa’）分隔。例如：gcc -c -g -O -Wa,-alh,-L file.c，这将两
个选项传递给汇编器：‘-alh’（向标准输出生成包含高级别源代码和汇编代码的列表内容）和
‘-L’（在符号表中保留本地符号）。通常你不需要使用这种 ‘-Wa’ 机制，因为许多编译器命令行
选项会自动由编译器传递给汇编器。你可以用 ‘-v’ 选项调用 GNU 编译器驱动程序，以准确查看
它传递给每个编译阶段（包括汇编器）的选项。

选项汇总
---------

汇编命令行选项： ::

    as  [-a[cdghilns][=file]]
        [–alternate]
        [–compress-debug-sections] [–nocompress-debug-sections]
        [-D]
        [–dump-config]
        [–debug-prefix-map old=new]
        [–defsym sym=val]
        [–elf-stt-common=[no|yes]]
        [–emulation=name]
        [-f]
        [-g] [–gstabs] [–gstabs+]
        [–gdwarf-<N>] [–gdwarf-sections]
        [–gdwarf-cie-version=VERSION]
        [–generate-missing-build-notes=[no|yes]]
        [–gsframe]
        [–hash-size=N]
        [–help] [–target-help]
        [-I dir]
        [-J]
        [-K]
        [–keep-locals]
        [-L]
        [–listing-lhs-width=NUM]
        [–listing-lhs-width2=NUM]
        [–listing-rhs-width=NUM]
        [–listing-cont-lines=NUM]
        [–multibyte-handling=[allow|warn|warn-sym-only]]
        [–no-pad-sections]
        [-o objfile] [-R]
        [–scfi=experimental]
        [–sectname-subst]
        [–size-check=[error|warning]]
        [–statistics]
        [-v] [-version] [–version]
        [-W] [–no-warn] [–warn] [–fatal-warnings]
        [-w] [-x]
        [-Z] [@FILE]
        [target-options]
        [–-|files ...]

    Target AArch64 options: 64-bit mode of the ARM Architecture (AArch64)
        [-EB|-EL]
        [-mabi=ABI]

    Target ARM options:
        [-mcpu=processor[+extension...]]
        [-march=architecture[+extension...]]
        [-mfpu=floating-point-format]
        [-mfloat-abi=abi]
        [-meabi=ver]
        [-mthumb]
        [-EB|-EL]
        [-mapcs-32|-mapcs-26|-mapcs-float|-mapcs-reentrant]
        [-mthumb-interwork] [-k]

    Target i386 options:
        [–32|–x32|–64] [-n]
        [-march=CPU[+EXTENSION...]] [-mtune=CPU]

**@file** ::

    从文件读取命令行选项，插入到 @file 选择所在位置，如果文件不存在或不能读取，那么
    @file 这个选项将被当作普通的文字处理，不会被移除，也不会从文件中读取任何内容。文件
    中的选项使用空白进行分隔，一个选项中如果包含空白，那么整个选项必须使用单引号或双引
    号括起来。任何字符（包括反斜杠）都可以通过前置一个反斜杠字符的方式进行包含。该文件
    本身可能包含额外的 @file 选项；任何此类选项都将被递归处理。

**-nocpp -w -X -Qy -Qn -k -s** ::

    被忽略。

**--hash-size N** ::

    被忽略，为了与其他汇编器在命令行上兼容而提供支持。

**--reduce-memory-overheads** ::

    被忽略。为了与同时向汇编器和链接器传递相同选项的工具保持兼容而提供支持。

**-a[cdghilmns][=file]** ::

    启用列表输出功能，可以通过多种方式定制输出内容。单独使用-a时，默认等同于-ahls，即
    包含高级别源代码、汇编代码、符号表和表单处理。这些选项可以组合使用，例如，使用-aln
    表示输出汇编列表但不进行表单处理。

    -ac：省略没有汇编的条件为假的代码行（omit false conditionals）。
    -ad：省略调试指令（omit debugging directives）。
    -ag：包含一般信息，例如版本号和传递的选项（like version and options passed）。
    -ah：包含高级别源代码（include high-level source）。
    -al：包含汇编代码（include assembly）。
    -ali：包含带有 ginsn 的汇编代码（include assembly with ginsn）。
    -am：包含宏展开（include macro expansions）。
    -an：省略表单处理（omit forms processing）。
    -as：包含符号表（include symbols）。
    =file：设置列表文件的名称。如果使用此选项，它必须是最后一个选项。

    高级语言列表（-ah）需要使用编译器调试选项（如 -g），并且还需要请求汇编列表
    （-al）。一旦你指定了这些选项之一，你就可以使用指令 .list、.nolist、.psize、
    .eject、.title 和 .sbttl来 进一步控制列表输出及其外观。‘-an’ 选项关闭所有表格处
    理。如果你没有使用 ‘-a’ 选项之一请求列表输出，那么列表控制指令将没有任何效果。请注
    意，如果汇编源代码来自标准输入（例如它是由 gcc 创建的，并且使用了 ‘-pipe’ 命令行开
    关），那么列表将不包含任何注释或预处理器指令。这是因为列表代码只在汇编器预处理后才
    从 stdin 缓冲输入源代码行。这减少了内存使用并使代码更加高效。

**--alternate** ::

    以交替宏模式开始，参见 [.altmacro] 部分。

**-D** ::

    如果支持的话，-D 选项会在目标特定的后端启用调试功能。如果目标后端不支持调试功能，该
    选项将被忽略。即使目标后端不支持调试功能，-D 选项仍然被接受不会报错，这是为了确保脚
    本的兼容性。

**--debug-prefix-map old=new** ::

    当在目录 old 中汇编文件时，记录调试信息，将它们描述为位于目录 new 中。

**--defsym sym=value** ::

    定义符号 sym 的值为 value，在对输入文件进行汇编之前。value 必须是一个整数常量。如
    同 C 语言语法，以 0x 开头表示十六进制值，以 0 开头表示八进制值。符号的值可以通过在
    源文件中使用 .set 伪操作符来覆盖。

**--dump-config** ::

    显示汇编器的配置信息，然后退出。

**--emulation=name** ::

    如果汇编器被配置为支持多种不同的目标配置，则可以使用此选项选择所需的形式。

**-f** ::

    “快速” 模式 —— 跳过空白和注释的预处理（假设源代码是编译器的输出）。‘-f’ 选项仅应在
    汇编由可信编译器生成的程序时使用。‘-f’ 会阻止汇编器在汇编输入文件之前对它们进行空白
    和注释的预处理。警告：如果你实际需要对文件进行预处理的情况下使用选项 ‘-f’（例如，文
    件中包含注释），as 将无法正确工作。

**-g --gen-debug** ::

    为每个汇编源代码行生成调试信息，使用目标系统所偏好的调试格式。这目前意味着可能是
    STABS、ECOFF 或 DWARF2。当调试格式为 DWARF 时，只有当汇编文件本身没有生成
    .debug_info 和 .debug_line 段时，才会输出这些段。

**--gsframe** ::

    通过 CFI 指令创建 .sframe 段。

**--help** ::

    打印命令行选项的摘要并退出。

**--target-help** ::

    打印所有目标特定选项的摘要并退出。

**-I dir** ::

    将目录 dir 添加到 .include 指令的搜索列表中。你可以根据需要多次使用 -I 来包含各种
    路径。当前工作目录总是首先被搜索，之后 as 按照它们在命令行中指定的顺序（从左到右）
    搜索 -I 目录。

**-J** ::

    不对有符号溢出发出警告。

**-K** ::

    当不同形式因长位移而被修改时发出警告。as 有时会更改形式为 ‘.word sym1-sym2’ 的指
    令所产生的代码。如果你希望在这种情况下发出警告，可以使用 -K 选项。

**-L --keep-locals** ::

    在符号表中保留局部符号。这些符号以系统特定的局部标签前缀开头，通常在 ELF 系统中为
    ‘.L’，在传统的 a.out 系统中为 ‘L’。通常在调试时你不会看到这样的符号，因为它们是为
    编写汇编程序的程序（如编译器）使用的，而不是让你注意到的。通常，as 和 ld 都会丢弃
    这样的符号，所以你通常不会用它们来调试。这个选项告诉 as 在目标文件中保留这些本地符
    号。通常，如果你这样做，你也会告诉链接器 ld 保留这些符号。

**–listing-** ::

    汇编器的列表功能可以通过命令行开关 ‘-a’ 启用，此功能将输入源文件与输出目标文件中相
    应位置的十六进制转储结合起来，并将它们显示为一个列表文件。此列表的格式可以通过汇编
    源代码中的指令 .list、.title、.sbttl、.psize 和 .eject 以及以下开关来控制：
    --listing-lhs-width=‘number’，--listing-lhs-width2=‘number’，
    --listing-rhs-width=‘number’，--listing-cont-lines=‘number’。

**-M --mri** ::

    -M 或 --mri 选项选择 MRI 兼容模式。这会改变 as 的语法和伪操作处理，使其与
    Microtec Research 的 ASM68K 汇编器兼容。MRI 语法的具体性质在此不作详细说明；请参
    阅 MRI 手册以获取更多信息。特别要注意的是，宏及其参数的处理方式有所不同。此选项的目
    的是允许使用 as 汇编现有的 MRI 汇编代码。MRI 兼容性并不完整，MRI 汇编器的某些操作
    依赖于其目标文件格式，无法支持使用其他目标文件格式。

**--MD FILE** ::

    依赖跟踪，as 可以为它创建的文件生成一个依赖文件。这个文件由一条适合 make 规则组
    成，描述了目标源文件的依赖关系，该规则被写入其参数中指定的文件。此功能用于
    makefile 的自动更新。

**--multibyte-handling=allow|warn|warn-sym-only|warn_sym_only** ::

    控制汇编器如何处理输入中的多字节字符。默认行为（可以通过使用 allow 参数来恢复）是
    允许这样的字符而不发出警告。使用 warn 参数会使汇编器在遇到任何多字节字符时生成一条
    警告消息。使用 warn-sym-only 参数则仅在定义符号时，如果符号名称包含多字节字符，才
    会生成警告，对未定义符号的引用不会生成警告。

**--no-pad-sections** ::

    停止汇编器对输出段的末尾进行填充以对齐该段。默认情况下，汇编器会对段进行填充，但这
    可能会浪费空间，对于那些内存受限的目标系统来说，这些空间可能是必需的。

**-o objfile** ::

    将汇编器的输出目标文件命名为 objfile。每次运行 as 时，总会生成一个目标文件。默认情
    况下，它的名称是 a.out。你可以使用此选项（它恰好需要一个文件名）来为目标文件指定一
    个不同的名称。无论目标文件叫什么名字，as 都会覆盖任何同名的现有文件。

**-R** ::

    将数据段合并到文本段中。合并数据段和文本段：-R 告诉 as 在写目标文件时，就像所有数
    据段的数据都存储在文本段中一样。这仅在最后时刻完成：你的二进制数据保持不变，但数据
    段的部分被重新定位。你的目标文件的数据段部分长度为零，因为它的所有字节都被追加到文
    本段。当你指定 -R 时，理论上可以生成更短的地址偏移量，因为我们不需要跨越文本段和数
    据段。我们没有这样做，仅仅是为了与旧版本的 as 保持兼容。在未来，-R 可能会以这种方
    式工作。当 as 配置为输出 COFF 或 ELF 格式时，此选项仅在你使用名为 ‘.text’ 和
    ‘.data’ 的段时才有用。-R 不支持任何 HPPA 目标，使用 -R 时 as 会发出警告。

**--sectname-subst** ::

    允许在段名中使用替换序列，参见 [.section name] 部分。

**--size-check=error --size-check=warning** ::

    对于无效的 ELF .size 指令，发出错误或警告。

**--statistics** ::

    打印汇编过程中使用的最大空间（以字节为单位）和总时间（以秒为单位）。

**--strip-local-absolute** ::

    从输出的符号表中移除局部绝对符号。

**-v -V** ::

    打印汇编器的版本信息。

**-version --version** ::

    打印汇编器的版本信息并退出。

**-W --no-warn** ::

    抑制警告信息。

**--warn** ::

    不抑制警告信息，也不将警告视为错误。

**--fatal-warnings** ::

    将警告视为错误。在汇编编译器输出时，as 不应该发出任何警告或错误消息。但是，由人编写
    的程序常常会导致 as 发出警告，指出某个特定的假设被采用。所有这样的警告都输出到标准
    错误文件。如果你使用 -W 或 --no-warn 选项，将不会发出任何警告。这仅影响警告消息：
    它不会改变 as 汇编你的文件的任何特定方式，仍然会报告导致汇编停止的错误。警告默认是
    开启的，可以使用 -W 或 --no-warn 关闭警告。在命令行后面再次指定 --warn 将重新开启
    警告，并像往常一样输出它们。如果你使用 --fatal-warnings 选项，as 会将产生警告的文
    件视为存在错误。

**-Z** ::

    即使出现错误，也生成目标文件。在发出错误消息后，as 通常不会产生任何输出。如果你出于
    某种原因，即使在 as 对你的程序发出错误消息后，也对目标文件输出感兴趣，请使用 -Z 选
    项。如果有任何错误，as 仍将继续，并在最后发出一条以下形式的警告消息后，写入一个目标
    文件。‘n errors, m warnings, generating bad object file.’

**--|files ...** ::

    标准输入，或要汇编的源文件。

i386 选项
----------

i386 版本的 as 支持原始的 Intel 386 架构，包括 16 位和 32 位模式，以及扩展 Intel 架
构到 64 位的 AMD x86-64 架构。 ::

    Target i386 options:
        [–32|–x32|–64] [-n]
        [-march=CPU[+EXTENSION...]] [-mtune=CPU]

**--32|--x32|--64** ::

    生成 32bit/x32bit/64bit 代码。‘--32’ 表示 Intel i386 架构，而 ‘--x32’ 和
    ‘--64’ 分别表示 AMD x86-64 架构的 32 位或 64 位字大小。这些选项仅在 ELF 目标文件
    格式中可用，并且需要包含必要的 BFD 支持，在 32 位平台上你必须在配置中添加
    –enable-64-bit-bfd 以启用 64 位使用，并使用 x86-64 作为目标平台。

**-n** ::

    不优化代码对齐。默认情况下，x86 GAS 会用多字节 nop 指令（如
    leal 0(%esi,1), %esi）替换代码段中用于对齐的多个 nop 指令。此开关将禁用该优化，如
    果将单字节 nop（0x90）明确指定作为对齐的填充字节。

**--divide** ::

    在基于 SVR4 的平台上，字符 ‘/’ 被视为注释字符，这意味着它不能在表达式中使用。
    ‘--divide’ 选项将 ‘/’ 变为普通字符。这不会禁用行首的 ‘/’ 作为注释的开始，也不会影
    响使用 ‘#’ 来开始注释。

**-march=CPU[+EXTENSION...]** ::

    选择要使用的 CPU 架构，生成针对指定 CPU 和扩展的代码。此选项指定目标处理器，如果尝
    试汇编一条无法在目标处理器上执行的指令，汇编器将发出错误消息。以下处理器名称被识
    别：generic32、generic64、i386、i486、i586、i686、pentium、pentiumpro、
    pentiumii、pentiumiii、pentium4、prescott、nocona、core、core2、corei7、
    iamcu、k6、k6_2、athlon、opteron、k8、amdfam10、bdver1、bdver2、bdver3、
    bdver4、znver1、znver2、znver3、znver4、znver5、btver1、btver2。
    除了基本指令集外，还可以指示汇编器接受各种扩展助记符。例如，-march=i686+sse4+vmx
    会在 i686 的基础上扩展 sse4 和 vmx。当前支持的扩展有：8087, 287, 387, 687,
    cmov, fxsr, mmx, sse, sse2, sse3, sse4a, ssse3, sse4.1, sse4.2, sse4, avx,
    avx2, lahf_sahf, monitor, adx, rdseed, prfchw, smap, mpx, sha, rdpid,
    ptwrite, cet, gfni, vaes, vpclmulqdq, prefetchwt1, clflushopt, se1, clwb,
    movdiri, movdir64b, enqcmd, serialize, tsxldtrk, kl, widekl, hreset,
    avx512f, avx512cd, avx512er, avx512pf, avx512vl, avx512bw, avx512dq,
    avx512ifma, avx512vbmi, avx512_4fmaps, avx512_4vnniw, avx512_vpopcntdq,
    avx512_vbmi2, avx512_vnni, avx512_bitalg, avx512_vp2intersect, tdx,
    avx512_bf16, avx_vnni, avx512_fp16, prefetchi, avx_ifma, avx_vnni_int8,
    cmpccxadd, wrmsrns, msrlist, avx_ne_convert, rao_int, fred, lkgs,
    avx_vnni_int16, sha512, sm3, sm4, pbndkb, avx10.1, avx10.1/512, avx10.1/256,
    avx10.1/128, user_msr, apx_f, amx_int8, amx_bf16, amx_fp16, amx_complex,
    amx_tile, vmx, vmfunc, smx, xsave, xsaveopt, xsavec, xsaves, aes, pclmul,
    fsgsbase, rdrnd, f16c, bmi2, fma, movbe, ept, lzcnt, popcnt, hle, rtm, tsx,
    invpcid, clflush, mwaitx, clzero, wbnoinvd, pconfig, waitpkg, uintr,
    cldemote, rdpru, mcommit, sev_es, lwp, fma4, xop, cx16, syscall, rdtscp,
    3dnow, 3dnowa, sse4a, sse5, snp, invlpgb, tlbsync, svme, padlock。
    请注意，这些扩展助记符可以加上 no 前缀来撤销相应的（以及任何依赖的）功能。进一步注
    意，-march=avx10.<N> 上的后缀强制执行向量长度限制，即尽管这些是 “启用” 选项，但使
    用这些后缀将禁用所有具有更宽向量或掩码寄存器操作数的指令。当与 -march 一起使用
    .arch 指令时，.arch 指令将优先。

**-mtune=CPU** ::

    此选项指定要针对优化的处理器。与 -march 选项一起使用时，仅会生成由 -march 选项指定
    的处理器的指令。有效的 CPU 值与 -march=CPU 的处理器列表完全相同。

**-moperand-check=none|warning|error** ::

    这些选项控制汇编器是否应该检查某些指令的操作数或操作数组合。例如，有些指令的操作数
    大小无法从其操作数推断出来，也没有通过指令后缀指定。-moperand-check=none 会使汇编
    器不执行这些检查。-moperand-check=warning 会使汇编器在相应的检查失败时发出警告，
    这是默认行为。-moperand-check=error 会使汇编器在相应的检查失败时发出错误。

**-mmnemonic=att|intel** ::

    这些选项指定用于匹配指令的指令助记符。.att_mnemonic 和 .intel_mnemonic指令将优
    先。默认为 att 助记符。

**-msyntax=att|intel** ::

    这些选项指定在处理指令时使用的指令语法。.att_syntax 和 .intel_syntax 指令将优
    先。默认为 att 语法。

**-mnaked-reg** ::

    此选项指定寄存器不需要 ‘%’ 前缀。.att_syntax 和 .intel_syntax 指令将优先。

**-madd-bnd-prefix** ::

    此选项强制汇编器为所有跳转分支添加 BND 前缀，即使源代码中没有明确指定这样的前缀。

**-mno-shared** ::

    在 ELF 目标上，汇编器通常会优化掉针对已定义的非弱全局分支目标（具有默认可见性）的非
    PLT 重定位。‘-mshared’ 选项告诉汇编器生成可能进入共享库的代码，在共享库中，所有具
    有默认可见性的非弱全局分支目标都可以被抢占，生成的代码会稍微大一些。此选项仅影响分
    支指令的处理。

**-mbig-obj** ::

    在PE/COFF目标上，此选项强制使用大目标文件格式，该格式允许超过 32768 个段。

**-momit-lock-prefix=no|yes** ::

    这些选项控制汇编器如何编码 lock 前缀。此选项旨在作为某些在 lock 前缀上失败的处理器
    的变通方法。此选项只能安全地用于单核、单线程计算机。-momit-lock-prefix=yes 将省略
    所有 lock 前缀。-momit-lock-prefix=no 将像往常一样编码 lock 前缀，这是默认行为。

**-mfence-as-lock-add=no|yes** ::

    这些选项控制汇编器如何编码 lfence、mfence 和 sfence。-mfence-as-lock-add=yes 会
    在 64 位模式下将 lfence、mfence 和 sfence 编码为 ‘lock addl $0x0, (%rsp)’，在
    32 位模式下编码为 ‘lock addl $0x0, (%esp)’。-mfence-as-lock-add=no 会像往常一
    样编码 lfence、mfence 和 sfence，这是默认行为。

**-mrelax-relocations=no|yes** ::

    这些选项控制汇编器是否生成放松重定位（relax relocations）。在 32 位模式下，放松重
    定位包括 R_386_GOT32X；在 64 位模式下，包括 R_X86_64_GOTPCRELX 和
    R_X86_64_REX_GOTPCRELX。-mrelax-relocations=yes 将生成放松重定位，
    -mrelax-relocations=no 将不生成放松重定位。默认行为可以通过配置选项
    --enable-x86-relax-relocations 来控制。

**-malign-branch-boundary=NUM** ::

    此选项控制汇编器如何使用段前缀或 NOP 对齐分支。NUM 必须是 2 的幂。它应该是 0 或不
    小于 16。分支将在 NUM 字节边界内对齐。-malign-branch-boundary=0 是默认值，不对齐
    分支。

**-malign-branch=TYPE[+TYPE...]** ::

    此选项指定要对齐的分支类型。TYPE 是 ‘jcc’（对齐条件跳转）、‘fused’（对齐融合条件
    跳转）、‘jmp’（对齐无条件跳转）、‘call’（对齐调用）、‘ret’（对齐返回）、
    ‘indirect’（对齐间接跳转和调用）的组合。默认值是 -malign-branch=jcc+fused+jmp。

**-malign-branch-prefix-size=NUM** ::

    此选项指定用于对齐分支的指令上的最大前缀数量。NUM 应在 0 到 5 之间。默认的 NUM 是
    5。

**-mbranches-within-32B-boundaries** ::

    此选项将条件跳转、融合条件跳转和无条件跳转对齐到 32 字节边界内，最多使用 5 个段前
    缀对齐指令。它等同于-malign-branch-boundary=32 -malign-branch=jcc+fused+jmp
    -malign-branch-prefix-size=5。默认情况下不对齐分支。

**-mlfence-after-load=no|yes** ::

    这些选项控制汇编器是否在加载指令后生成 lfence。-mlfence-after-load=yes 将生成
    lfence。-mlfence-after-load=no 将不生成 lfence，这是默认行为。

**-mlfence-before-indirect-branch=none|all|register|memory** ::

    这些选项控制汇编器是否在间接近分支指令之前生成 lfence。
    -mlfence-before-indirect-branch=all 将在通过寄存器的间接近分支之前生成 lfence，
    并在通过内存的间接近分支之前发出警告。它还会隐式地设置 -mlfence-before-ret=shl，
    除非明确指定了 -mlfence-before-ret。-mlfence-before-indirect-branch=register
    将在通过寄存器的间接近分支之前生成 lfence。
    -mlfence-before-indirect-branch=memory 将在通过内存的间接近分支之前发出警告。
    -mlfence-before-indirect-branch=none 将不生成 lfence，也不发出警告，这是默认行
    为。请注意，当 -mlfence-after-load=yes 时，不会在通过寄存器的间接近分支之前生成
    lfence，因为 lfence 将在加载分支目标寄存器之后生成。

**-mlfence-before-ret=none|shl|or|yes|not** ::

    这些选项控制汇编器是否在 ret 指令之前生成 lfence。-mlfence-before-ret=or 将在
    ret 之前生成带 lfence 的 or 指令。-mlfence-before-ret=shl 或
    -mlfence-before-ret=yes 将在 ret 之前生成带 lfence 的 shl 指令。
    -mlfence-before-ret=not 将在 ret 之前生成带 lfence 的 not 指令。
    -mlfence-before-ret=none 将不生成 lfence，这是默认行为。

**-mx86-used-note=no|yes** ::

    这些选项控制汇编器是否生成 GNU PROPERTY X86 ISA 1 USED 和 GNU PROPERTY X86
    FEATURE 2 USED GNU 属性注释。默认行为可以通过配置选项 --enable-x86-used-note
    来控制。

**-mevexrcig=rne|rd|ru|rz** ::

    这些选项控制汇编器如何编码仅使用 SAE（Suppress All Exceptions）的 EVEX 指令。
    -mevexrcig=rne 将 EVEX 指令的 RC 位编码为 00，这是默认行为。-mevexrcig=rd、
    -mevexrcig=ru 和 -mevexrcig=rz 分别将仅使用 SAE 的 EVEX 指令的 RC 位编码为
    01、10 和 11。

**-mamd64 -mintel64** ::

    这些选项指定汇编器在 64 位模式下仅接受 AMD64 或 Intel64 ISA。默认情况下，汇编器接
    受通用、仅 Intel64 和 AMD64 ISAs。

**-O0 | -O | -O1 | -O2 | -Os** ::

    优化指令编码以减小指令大小。‘-O’ 和 ‘-O1’ 将 64 位寄存器加载指令（带有 64 位立即
    数）编码为 32 位寄存器加载指令（带有 31 位或 32 位立即数），将 64 位寄存器清零指令
    编码为 32 位寄存器清零指令，将 256/512 位 VEX/EVEX 向量寄存器清零指令编码为 128
    位 VEX 向量寄存器清零指令，将 128/256 位 EVEX 向量寄存器加载/存储指令编码为 VEX
    向量寄存器加载/存储指令，并将 128/256 位 EVEX 打包整数逻辑指令编码为 128/256 位
    VEX 打包整数逻辑指令。‘-O2’ 包括 ‘-O1’ 的优化，并且将 256/512位 EVEX 向量寄存器
    清零指令编码为 128 位 EVEX 向量寄存器清零指令。在 64 位模式下，如果交换源操作数可
    以使用 2 字节 VEX 前缀形式而不是 3 字节形式，则 VEX 编码指令的交换源操作数也会被交
    换。某些形式的 AND 以及具有相同（寄存器）操作数的 OR 也会被改为 TEST。‘-Os’ 包括
    ‘-O2’ 的优化，并且将 16 位、32 位和 64 位寄存器与立即数的测试指令编码为 8 位寄存
    器与立即数的测试指令。‘-O0’ 关闭这种优化。
