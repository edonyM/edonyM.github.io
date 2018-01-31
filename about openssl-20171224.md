### 关于OpenSSL社区
OpenSSL功能简介，OpenSSL外围包提供了如下三种功能：
1. 命令行工具，用来完成各种各样密码学相关的任务。例如，创建证书、解析证书、加密文件、加密算法测试等。
2. 全面的、可扩展的密码学库（libcrypto）。覆盖了大多数标准定义的加密算法，可用硬件扩展或者加速。
3. 符合SSL/TLS协议的加密通讯库（libssl）。提供客户端和服务端进行加密通讯的的能力。

### OpenSSL版本号规定
OpenSSL的版本号的格式为：n1.n2.n3x（例如OpenSSL-1.0.2k）。其中n1, n2, n3是数字，x是字母。OpenSSL的版本分为：大版本(Major Release)、小版本(Minor Release)、字母版本(Letter Release)。
* 大版本(Major Release)：n1或者n2变化，该版本的变化会导致OpenSSL相关的二进制兼容性问题，升级时要注意。
* 小版本(Minor Release)：n3变化，该版本变化是因为新增一些特性，但是这些特性不会导致OpenSSL二进制的兼容性有问题。
* 字母版本(Letter Release)：x变化，该版本变化是因为修复bug和安全问题（CVE修复），不包含任何新特性。

### OpenSSL发布计划
OpenSSL社区对1.0.0大版本高度认可，认为其对开发者和厂商都十分友好。OpenSSL项目的发布策略 如下：
* Version 1.1.0 will be supported until 2018-08-31.
* Version 1.0.2 will be supported until 2019-12-31 (LTS).
* Version 1.0.1 is no longer supported.
* Version 1.0.0 is no longer supported.
* Version 0.9.8 is no longer supported.

OpenSSL社区每四年会指定一个LTS的OpenSSL版本，OpenSSL社区对LTS版本的OpenSSL至少会支持5年。非LTS版本的OpenSSL设置至少会支持2年。OpenSSL社区支持的意思是超过支持年之后，社区只接纳CVE相关的commit。

### 关于OpenSSL的安全议题
#### 安全隐患
OpenSSL作为系统的基础组件，一旦发现安全问题影响范围都会很大。例如，CVE-2014-3512，该漏洞是2014 年OpenSSL版本更新时对于TLS连接的ClientHello的回传消息未做长度检查，从而引入溢出攻击。这个CVE导致了全球60%的网站受到攻击（包括Google，Yahoo，Facebook）。

#### 专利隐患
OpenSSL中用到的一些加密算法是有专利授权的，如IDEA（部分地区进行商业使用时需要获得专利授权，否则会面临诉讼），MDC2（该算法作者持有专利权，因为专利问题目前大多数Linux发行版都不适用改算法），RC5（该算法专利被RSA Security公司持有）。针对上述有专利隐患的算法，在OpenSSL编译时，建议做如下配置：

```sh
./Configure no-idea no-mdc2 no-rc5
```
#### 算法隐患
OpenSSL大多数加密算法和协议（如AES、RSA、ECC、DH、TLS）均按照美国标准实现（NIST），斯诺登之后由算法专家分析称NIST颁发的标准故意留有对美国政府监控有用的漏洞，甚至包括一些破解算法的方法和工具。一个比较有说服力的例子：NSA曾经要求RSA公司在RSA加密算法总安置后门 （Dual_EC_DRBG随机数生成算法）。

### OpenSSL与FIPS
Federal Information Processing Standards(FIPS)是由美国NIST颁发的加密算法相关的标准，由于美国政府（美国商务部、NSA等）的影响，许多地区涉及安全的商品必须符合FIPS标准，否则无法销售。OpenSSL是首个通过FIPS-140认证的开源软件，所以可以认为OpenSSL是加密算法的现实标准，但是出于“阴谋论”和对NSA的不信任，许多科技公司都自己实现了SSL（如MacOS）。
 
OpenSSL社区这对这一个需求，设计开发了名为OpenSSL FIPS Object Module的组件 ，将OpenSSL的动态库和API转换为符合FIPS标准，社区仅仅对OpenSSL FIPS Object Module进行了认证。目前OpenSSL社区有两组源码tar包，如下图所示，其中openssl-fips-xxx即为OpenSSL FIPS Object Module 2.0的源码tar包，openssl-1.xxx为openssl源码tar包。


### OpenSSL分层软件架构
OpenSSL项目采用了分层软件架构，分层的软件架构具备高可测试性、对开发者友好等特性。

OpenSSL软件架构设计为三层：
* 第一层为用户层，该层由OpenSSL软件、或者使用了OpenSSL动态库的软件组成，它们之间彼此是独立的。
* 第二层为抽象层，该层抽象出独立的三种功能：SSL/TLS加密通讯功能（SSL）；加密功能（EVP）；I/O功能（BIO）。EVP(Envelope)模块是一个OpenSSL提供的高级加密功能接口，可以用于加解密和签名检验，EVP隐藏了加密算法的细节。BIO模块提供能抽象I/O功能，隐藏了程序的I/O细节，BIO类似于C语言中FILE*。SSL/TLS模块提供了建立安全连接的功能，该模块不仅提供了功能相关的API接口，同时还实现了建立连接或“握手”等功能。
* 第三层为实现层，Engine模块提供了使用软件实现的加密算法和硬件实现的加密算法的功能。软件实现的加密算法模块是OpenSSL的核心，它包含了多个相关联的加密算法子模块（例如加解密算法模块，它同样创建和处理加密秘钥。）。Engine模块提供了ENGINE对象来创建和控制加密模块，同时它还包含了对硬件加密算法实现的抽象以便能够使用硬件加密算法实现。Hardware Crypto模块有加密算法的硬件实现组成，这个硬件不由OpenSSL提供，但是OpenSSL允许使用这个硬件的功能（例如Intel的RSA加密硬件）。

OpenSSL的分层示意图如下所示。
![openssl arch](https://edonymu.files.wordpress.com/2018/01/arch_view_nice.png)

#### OpenSSL软件架构的发展
![openssl new arch](https://edonymu.files.wordpress.com/2018/01/arch_view_act.png)

### 关于OpenSSL使用方法


### 关于OpenSSL的Coding Style
```

                OpenSSL coding style
		Jan 12 2015

This document describes the coding style for the OpenSSL project. It is
derived from the Linux kernel coding style, which can be found at:

    https://www.kernel.org/doc/Documentation/CodingStyle

This guide is not distributed as part of OpenSSL itself. Since it is
derived from the Linux Kernel Coding Style, it is distributed under the
terms of the kernel license, available here:

    https://www.kernel.org/pub/linux/kernel/COPYING

Coding style is all about readability and maintainability using commonly
available tools. OpenSSL coding style is simple. Avoid tricky expressions.


                Chapter 1: Indentation

Indentation is four space characters. Do not use the tab character.

Pre-processor directives use one space for indents:

    #if
    # define
    #else
    # define
    #endif


                Chapter 2: Breaking long lines and strings

Don't put multiple statements, or assignments, on a single line.

    if (condition) do_this();
    do_something_everytime();

The limit on the length of lines is 80 columns. Statements longer
than 80 columns must be broken into sensible chunks, unless exceeding
80 columns significantly increases readability and does not hide
information. Descendants are always substantially shorter than the parent
and are placed substantially to the right. The same applies to function
headers with a long argument list. Never break user-visible strings,
however, because that breaks the ability to grep for them.


                Chapter 3: Placing Braces and Spaces

The other issue that always comes up in C styling is the placement
of braces. Unlike the indent size, there are few technical reasons to
choose one placement strategy over the other, but the preferred way,
following Kernighan and Ritchie, is to put the opening brace last on the
line, and the closing brace first:

    if (x is true) {
        we do y
    }

This applies to all non-function statement blocks (if, switch, for,
while, do):

    switch (suffix) {
    case 'G':
    case 'g':
        mem <<= 30;
        break;
    case 'M':
    case 'm':
        mem <<= 20;
        break;
    case 'K':
    case 'k':
        mem <<= 10;
        /* fall through */
    default:
        break;
    }

Note, from the above example, that the way to indent a switch statement
is to align the switch and its subordinate case labels in the same column
instead of "double-indenting" the case bodies.

There is one special case, however. Functions have the
opening brace at the beginning of the next line:

    int function(int x)
    {
        body of function
    }

Note that the closing brace is empty on a line of its own, EXCEPT in the
cases where it is followed by a continuation of the same statement, such
as a "while" in a do-statement or an "else" in an if-statement, like this:

    do {
        ...
    } while (condition);

and

    if (x == y) {
        ...
    } else if (x > y) {
        ...
    } else {
        ...
    }

In addition to being consistent with K&R, note that that this brace-placement
also minimizes the number of empty (or almost empty) lines. Since the
supply of new-lines on your screen is not a renewable resource (think
25-line terminal screens here), you have more empty lines to put comments on.

Do not unnecessarily use braces around a single statement:

    if (condition)
        action();

and

    if (condition)
        do_this();
    else
        do_that();

If one of the branches is a compound statement, then use braces on both parts:

    if (condition) {
        do_this();
        do_that();
    } else {
        otherwise();
    }

Nested compound statements should often have braces for clarity, particularly
to avoid the dangling-else problem:

    if (condition) {
        do_this();
        if (anothertest)
            do_that();
    } else {
        otherwise();
    }


                Chapter 3.1:  Spaces

OpenSSL style for use of spaces depends (mostly) on whether the name is
a function or keyword. Use a space after most keywords:

    if, switch, case, for, do, while, return

Do not use a space after sizeof, typeof, alignof, or __attribute__.
They look somewhat like functions and should have parentheses
in OpenSSL, although they are not required by the language. For sizeof,
use a variable when at all possible, to ensure that type changes are
properly reflected:

    SOMETYPE *p = OPENSSL_malloc(sizeof(*p) * num_of_elements);


Do not add spaces around the inside of parenthesized expressions.
This example is wrong:

    s = sizeof( struct file );

When declaring pointer data or a function that returns a pointer type,
the asterisk goes next to the data or function name, and not the type:

    char *openssl_banner;
    unsigned long long memparse(char *ptr, char **retptr);
    char *match_strdup(substring_t *s);

Use one space on either side of binary and ternary operators,
such as this partial list:

    =  +  -  <  >  *  /  %  |  &  ^  <=  >=  ==  !=  ?  : +=

Do not put a space after unary operators:

    &  *  +  -  ~  !  defined

Do not put a space before the postfix increment and decrement unary
operators or after the prefix increment and decrement unary operators:

    foo++
    --bar

Do not put a space around the '.' and "->" structure member operators:
    foo.bar
    foo->bar

Do not leave trailing whitespace at the ends of lines. Some editors with
"smart" indentation will insert whitespace at the beginning of new lines
as appropriate, so you can start typing the next line of code right away.
But they may not remove that whitespace if you leave a blank line, however,
and you end up with lines containing trailing, or nothing but, whitespace.

Git will warn you about patches that introduce trailing whitespace, and
can optionally strip the trailing whitespace; however, if applying
a series of patches, this may make later patches in the series fail by
changing their context lines.


                Chapter 4: Naming

C is a Spartan language, and so should your naming be. Do not use long
names like ThisVariableIsATemporaryCounter. Use a name like tmp, which
is much easier to write, and not more difficult to understand.

Except when otherwise required, avoid mixed-case names.

Do not encode the type into a name (so-called Hungarian notation).

Global variables (to be used only if you REALLY need them) need to
have descriptive names, as do global functions. If you have a function
that counts the number of active users, you should call that
count_active_users() or similar, you should NOT call it cntusr().

Local variable names should be short, and to the point. If you have
some random integer loop counter, it should probably be called i.
Calling it loop_counter is non-productive, if there is no chance of it
being mis-understood. Similarly, tmp can be just about any type of
variable that is used to hold a temporary value.

If you are afraid that someone might mix up your local variable names,
perhaps the function is too long; see Chapter 6.


                Chapter 5: Typedefs

OpenSSL uses typedef's extensively. For structures, they are all uppercase
and are usually declared like this:

    typedef struct name_st NAME;

For examples, look in ossl_type.h, but note that there are many exceptions
such as BN_CTX. Typedef'd enum is used much less often and there is no
convention, so consider not using a typedef. When doing that, the enum
name should be lowercase and the values (mostly) uppercase.

The ASN.1 structures are an exception to this. The rationale is that if
a structure (and its fields) is already defined in a standard it's more
convenient to use a similar name. For example, in the CMS code, a CMS_
prefix is used so ContentInfo becomes CMS_ContentInfo, RecipientInfo
becomes CMS_RecipientInfo etc. Some older code uses an all uppercase
name instead. For example, RecipientInfo for the PKCS#7 code uses
PKCS7_RECIP_INFO.

Be careful about common names which might cause conflicts. For example,
Windows headers use X509 and X590_NAME. Consider using a prefix, as with
CMS_ContentInfo, if the name is common or generic. Of course, you often
don't find out until the code is ported to other platforms.

A final word on struct's. OpenSSL has historically made all struct
definitions public; this has caused problems with maintaining binary
compatibility and adding features. Our stated direction is to have struct's
be opaque and only expose pointers in the API. The actual struct definition
should be defined in a local header file that is not exported.


                Chapter 6: Functions

Ideally, functions should be short and sweet, and do just one thing.
A rule of thumb is that they should fit on one or two screenfuls of text
(25 lines as we all know), and do one thing and do that well.

The maximum length of a function is often inversely proportional to the
complexity and indentation level of that function. So, if you have a
conceptually simple function that is just one long (but simple) switch
statement, where you have to do lots of small things for a lot of different
cases, it's OK to have a longer function.

If you have a complex function, however, consider using helper functions
with descriptive names. You can ask the compiler to in-line them if you
think it's performance-critical, and it will probably do a better job of
it than you would have done.

Another measure of complexity is the number of local variables. If there are
more than five to 10, consider splitting it into smaller pieces. A human
brain can generally easily keep track of about seven different things;
anything more and it gets confused. Often things which are simple and
clear now are much less obvious two weeks from now, or to someone else.
An exception to this is the command-line applications which support many
options.

In source files, separate functions with one blank line.

In function prototypes, include parameter names with their data types.
Although this is not required by the C language, it is preferred in OpenSSL
because it is a simple way to add valuable information for the reader.
The name in the prototype declaration should match the name in the function
definition.


                Chapter 7: Centralized exiting of functions

The goto statement comes in handy when a function exits from multiple
locations and some common work such as cleanup has to be done. If there
is no cleanup needed then just return directly. The rationale for this is
as follows:

    - Unconditional statements are easier to understand and follow
    - It can reduce excessive control structures and nesting
    - It avoids errors caused by failing to updated multiple exit points
      when the code is modified
    - It saves the compiler work to optimize redundant code away ;)

For example:

    int fun(int a)
    {
        int result = 0;
        char *buffer = OPENSSL_malloc(SIZE);

        if (buffer == NULL)
            return -1;

        if (condition1) {
            while (loop1) {
                ...
            }
            result = 1;
            goto out;
        }
        ...
    out:
        OPENSSL_free(buffer);
        return result;
    }

                Chapter 8: Commenting

Use the classic "/* ... */" comment markers.  Don't use "// ..." markers.

Comments are good, but there is also a danger of over-commenting. NEVER try
to explain HOW your code works in a comment. It is much better to write
the code so that it is obvious, and it's a waste of time to explain badly
written code. You want your comments to tell WHAT your code does, not HOW.

The preferred style for long (multi-line) comments is:

    /*-
     * This is the preferred style for multi-line
     * comments in the OpenSSL source code.
     * Please use it consistently.
     *
     * Description:  A column of asterisks on the left side,
     * with beginning and ending almost-blank lines.
     */

Note the initial hyphen to prevent indent from modifying the comment.
Use this if the comment has particular formatting that must be preserved.

It's also important to comment data, whether they are basic types or
derived types. To this end, use just one data declaration per line (no
commas for multiple data declarations). This leaves you room for a small
comment on each item, explaining its use.


                Chapter 9: Deleted


                Chapter 10: Deleted


                Chapter 11: Deleted


                Chapter 12: Macros and Enums

Names of macros defining constants and labels in enums are in uppercase:

    #define CONSTANT 0x12345

Enums are preferred when defining several related constants.

Macro names should be in uppercase, but macros resembling functions may
be written in lower case. Generally, inline functions are preferable to
macros resembling functions.

Macros with multiple statements should be enclosed in a do - while block:

    #define macrofun(a, b, c)   \
        do {                    \
            if (a == 5)         \
                do_this(b, c);  \
        } while (0)

Do not write macros that affect control flow:

    #define FOO(x)                 \
        do {                       \
            if (blah(x) < 0)       \
                return -EBUGGERED; \
        } while(0)

Do not write macros that depend on having a local variable with a magic name:

    #define FOO(val) bar(index, val)

It is confusing to the reader and is prone to breakage from seemingly
innocent changes.

Do not write macros that are l-values:

    FOO(x) = y

This will cause problems if, e.g., FOO becomes an inline function.

Be careful of precedence. Macros defining constants using expressions
must enclose the expression in parentheses:

    #define CONSTANT 0x4000
    #define CONSTEXP (CONSTANT | 3)

Beware of similar issues with macros using parameters. The GNU cpp manual
deals with macros exhaustively.


                Chapter 13: Deleted


                Chapter 14: Allocating memory

OpenSSL provides the following general purpose memory allocators:
OPENSSL_malloc(), OPENSSL_realloc(), OPENSSL_strdup() and OPENSSL_free().
Please refer to the API documentation for further information about them.


                Chapter 15: Deleted


                Chapter 16: Function return values and names

Functions can return values of many different kinds, and one of the
most common is a value indicating whether the function succeeded or
failed. Usually this is:

    1: success
    0: failure

Sometimes an additional value is used:

    -1: something bad (e.g., internal error or memory allocation failure)

Other APIs use the following pattern:

    >= 1: success, with value returning additional information
    <= 0: failure with return value indicating why things failed

Sometimes a return value of -1 can mean "should retry" (e.g., BIO, SSL, et al).

Functions whose return value is the actual result of a computation,
rather than an indication of whether the computation succeeded, are not
subject to these rules. Generally they indicate failure by returning some
out-of-range result. The simplest example is functions that return pointers;
they return NULL to report failure.


                Chapter 17:  Deleted


                Chapter 18:  Editor modelines

Some editors can interpret configuration information embedded in source
files, indicated with special markers. For example, emacs interprets
lines marked like this:

    -*- mode: c -*-

Or like this:

    /*
    Local Variables:
    compile-command: "gcc -DMAGIC_DEBUG_FLAG foo.c"
    End:
    */

Vim interprets markers that look like this:

    /* vim:set sw=8 noet */

Do not include any of these in source files. People have their own personal
editor configurations, and your source files should not override them.
This includes markers for indentation and mode configuration. People may
use their own custom mode, or may have some other magic method for making
indentation work correctly.


                Chapter 19:  Processor-specific code

In OpenSSL's case the only reason to resort to processor-specific code
is for performance. As it still exists in a general platform-independent
algorithm context, it always has to be backed up by a neutral pure C one.
This implies certain limitations. The most common way to resolve this
conflict is to opt for short inline assembly function-like snippets,
customarily implemented as macros, so that they can be easily interchanged
with other platform-specific or neutral code. As with any macro, try to
implement it as single expression.

You may need to mark your asm statement as volatile, to prevent GCC from
removing it if GCC doesn't notice any side effects. You don't always need
to do so, though, and doing so unnecessarily can limit optimization.

When writing a single inline assembly statement containing multiple
instructions, put each instruction on a separate line in a separate quoted
string, and end each string except the last with \n\t to properly indent
the next instruction in the assembly output:

        asm ("magic %reg1, #42\n\t"
             "more_magic %reg2, %reg3"
             : /* outputs */ : /* inputs */ : /* clobbers */);

Large, non-trivial assembly functions go in pure assembly modules, with
corresponding C prototypes defined in C. The preferred way to implement this
is so-called "perlasm": instead of writing real .s file, you write a perl
script that generates one. This allows use symbolic names for variables
(register as well as locals allocated on stack) that are independent on
specific assembler. It simplifies implementation of recurring instruction
sequences with regular permutation of inputs. By adhering to specific
coding rules, perlasm is also used to support multiple ABIs and assemblers,
see crypto/perlasm/x86_64-xlate.pl for an example.

Another option for processor-specific (primarily SIMD) capabilities is
called "compiler intrinsics." We avoid this, because it's not very much
less complicated than coding pure assembly, and it doesn't provide the
same performance guarantee across different micro-architecture. Nor is
it portable enough to meet our multi-platform support goals.


                Chapter 20:  Portability

To maximise portability the version of C defined in ISO/IEC 9899:1990
should be used. This is more commonly referred to as C90. ISO/IEC 9899:1999
(also known as C99) is not supported on some platforms that OpenSSL is
used on and therefore should be avoided.


                Chapter 21: Miscellaneous

Do not use ! to check if a pointer is NULL, or to see if a str...cmp
function found a match.  For example, these are wrong:

    if (!(p = BN_new())) ...
    if (!strcmp(a, "FOO")) ...

Do this instead:

    if ((p = BN_new()) == NULL)...
    if (strcmp(a, "FOO") == 0) ...


                Appendix A: References

The C Programming Language, Second Edition
by Brian W. Kernighan and Dennis M. Ritchie.
Prentice Hall, Inc., 1988.
ISBN 0-13-110362-8 (paperback), 0-13-110370-9 (hardback).
URL: http://cm.bell-labs.com/cm/cs/cbook/

The Practice of Programming
by Brian W. Kernighan and Rob Pike.
Addison-Wesley, Inc., 1999.
ISBN 0-201-61586-X.
URL: http://cm.bell-labs.com/cm/cs/tpop/

GNU manuals - where in compliance with K&R and this text - for cpp, gcc,
gcc internals and indent, all available from https://www.gnu.org/manual/

WG14 is the international standardization working group for the programming
language C, URL: http://www.open-std.org/JTC1/SC22/WG14/
```

### 关于OpenSSL源码须知

### 关于OpenSSL主要代码实现

### 关于OpenSSL的Iusse Tracking
___Timing attacks on RSA Keys___

On March 14, 2003, a timing attack on RSA keys was discovered, indicating a vulnerability within OpenSSL versions 0.9.7a and 0.9.6. This vulnerability was assigned the identifier CAN-2003-0147 by the Common Vulnerabilities and Exposures (CVE) project. RSA blinding was not turned on by default by OpenSSL, since it is not easily possible to when providing SSL or TLS using OpenSSL. Almost all SSL enabled Apaches were affected, along with many other applications of OpenSSL. Timing differences on the number of extra reductions along and use of Karatsuba and normal integer multiplication algorithms meant that it was possible for local and remote attackers to obtain the private key of the server.[citation needed]

___Denial of Service ASN.1 parsing___

OpenSSL 0.9.6k had a bug where certain ASN.1 sequences triggered a large number of recursions on Windows machines, discovered on November 4, 2003. Windows could not handle large recursions correctly, so OpenSSL would crash as a result. Being able to send arbitrary large numbers of ASN.1 sequences would cause OpenSSL to crash as a result. A client certificate to a SSL/TLS enabled server could accept ASN.1 sequences and crash.[citation needed]

___OCSP stapling vulnerability___

When creating a handshake, the client could send an incorrectly formatted ClientHello message, leading to OpenSSL parsing more than the end of the message. Assigned the identifier CVE-2011-0014 by the CVE project, this affected all OpenSSL versions 0.9.8h to 0.9.8q and OpenSSL 1.0.0 to 1.0.0c. Since the parsing could lead to a read on an incorrect memory address, it was possible for the attacker to cause a DOS. It was also possible that some applications expose the contents of parsed OCSP extensions, leading to an attacker being able to read the contents of memory that came after the ClientHello.

___ASN.1 BIO vulnerability___

When using Basic Input/Output (BIO) or FILE based functions to read untrusted DER format data, OpenSSL is vulnerable. This vulnerability was discovered on April 19, 2012, and was assigned the CVE identifier CVE-2012-2110. While not directly affecting the SSL/TLS code of OpenSSL, any application that was using ASN.1 functions (particularly d2i_X509 and d2i_PKCS12) were also not affected.[26]

___SSL, TLS and DTLS Plaintext Recovery Attack___

In handling CBC cipher-suites in SSL, TLS, and DTLS, OpenSSL was found vulnerable to a timing attack during the MAC processing. Nadhem Alfardan and Kenny Paterson discovered the problem, and published their findings on February 5, 2013. The vulnerability was assigned the CVE identifier CVE-2013-0169. The vulnerability affected all OpenSSL versions, and was only partially mitigated by using the OpenSSL FIPS Object Module and enabling FIPS mode.[citation needed]

___Predictable private keys (Debian-specific)___

OpenSSL's pseudo-random number generator acquires entropy using complex programming methods, described[by whom?] as poor coding practice. To keep the Valgrind analysis tool from issuing associated warnings, a maintainer of the Debian distribution applied a patch to the Debian's variant of the OpenSSL suite, which inadvertently broke its random number generator by limiting the overall number of private keys it could generate to 32,768. The broken version was included in the Debian release of September 17, 2006 (version 0.9.8c-1), also compromising other Debian-based distributions, for example Ubuntu. Any key generated with the broken random number generator was compromised, as well as the data encrypted with such keys;[citation needed] moreover, ready-to-use exploits are easily available.

The error was reported by Debian on May 13, 2008. On the Debian 4.0 distribution (etch), these problems were fixed in version 0.9.8c-4etch3, while fixes for the Debian 5.0 distribution (lenny) were provided in version 0.9.8g-9.

___Heartbleed___

OpenSSL versions 1.0.1 through 1.0.1f had a severe memory handling bug in their implementation of the TLS Heartbeat Extension that could be used to reveal up to 64 KB of the application's memory with every heartbeat (CVE-2014-0160). By reading the memory of the web server, attackers could access sensitive data, including the server's private key. This could allow attackers to decode earlier eavesdropped communications if the encryption protocol used does not ensure perfect forward secrecy. Knowledge of the private key could also allow an attacker to mount a man-in-the-middle attack against any future communications. The vulnerability might also reveal unencrypted parts of other users' sensitive requests and responses, including session cookies and passwords, which might allow attackers to hijack the identity of another user of the service.

At its disclosure on April 7, 2014, around 17% or half a million of the Internet's secure web servers certified by trusted authorities were believed to have been vulnerable to the attack. However, Heartbleed can affect both the server and client.

___CCS Injection Vulnerability___

CCS Injection Vulnerability (CVE-2014-0224) is a security bypass vulnerability that exists in OpenSSL. The vulnerability is due to a weakness in OpenSSL methods used for keying material.

This vulnerability can be exploited through the use of a man-in-the-middle attack, where an attacker may be able to decrypt and modify traffic in transit. A remote unauthenticated attacker could exploit this vulnerability by using a specially crafted handshake to force the use of weak keying material. Successful exploitation could lead to a security bypass condition where an attacker could gain access to potentially sensitive information. The attack can only be performed between a vulnerable client and server.

OpenSSL clients are vulnerable in all versions of OpenSSL before the versions 0.9.8za, 1.0.0m and 1.0.1h. Servers are only known to be vulnerable in OpenSSL 1.0.1 and 1.0.2-beta1. Users of OpenSSL servers earlier than 1.0.1 are advised to upgrade as a precaution.

___ClientHello sigalgs DoS___

This vulnerability (CVE-2015-0291) allows anyone to take a certificate, read its contents and modify it accurately to abuse the vulnerability causing a certificate to crash a client or server. If a client connects to an OpenSSL 1.0.2 server and renegotiates with an invalid signature algorithms extension, a null-pointer dereference occurs. This can cause a DoS attack against the server.

A Stanford Security researcher, David Ramos, had a private exploit and presented it before the OpenSSL team where they patched the issue.

OpenSSL classified the bug as a high-severity issue, noting version 1.0.2 was found vulnerable.

___Key Recovery Attack on Diffie Hellman small subgroups___

This vulnerability (CVE-2016-0701) allows, when some particular circumstances are met, to recover the OpenSSL server's private Diffie–Hellman key. An Adobe System Security researcher, Antonio Sanso, privately reported the vulnerability.

OpenSSL classified the bug as a high-severity issue, noting only version 1.0.2 was found vulnerable

### OpenSSL周边参考
* https://www.openssl.org/
* https://www.openssl.org/docs/
* https://www.feistyduck.com/books/openssl-cookbook/
* https://www.openssl.org/docs/standards.html
* https://github.com/openssl
* https://en.wikipedia.org/wiki/OpenSSL