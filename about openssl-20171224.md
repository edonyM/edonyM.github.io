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
#### OpenSSL命令行基本形式
OpenSSL命令行是OpenSSL项目三大功能之一，需要注意的是在使用OpenSSL命令行时需要较多的密码学背景，OpenSSL命令行的相关多余安全相关，所以建议事先了解一下密码学的背景知识，这主要是为了帮助在操作命令行时能够读懂OpenSSL的帮助信息。OpenSSL的命令形式如下：
```sh
# openssl + 标准命令 + 命令选项 + 命令参数
openssl standard command [command options] [command arguments]
```

#### OpenSSL基本信息查询
```sh
# openssl版本信息查询
openssl version -a

# openssl标准命令查询
openssl list-standard-commands

# openssl支持的加密算法命令
openssl list-cipher-commands

# openssl支持的摘要算法命令
openssl list-digest-commands

# openssl支持的加密算法
openssl list-cipher-algorithms

# openssl支持的摘要算法
openssl list-message-digest-algorithms

# openssl支持的公钥加密算法
openssl list-public-key-algorithms
```

#### OpenSSL命令行使用帮助
```sh
# openssl命令使用帮助查询
openssl standard command --help
```

### 关于OpenSSL源码须知
OpenSSL这对不同编译环境（架构）有不同的特殊配置，其配置工具为：Configure和config两个脚本。OpenSSL编译的时候最好不要自定义编译系统，否则会导致opensslconf.h和bn.h的生成有问题。
#### OpenSSL编译配置
运行如下命令，查看OpenSSL支持的平台。
```
$ ./Configure LIST
BC-32
BS2000-OSD
BSD-generic32
BSD-generic64
BSD-ia64
BSD-sparc64
BSD-sparcv8
BSD-x86
BSD-x86-elf
BSD-x86_64
Cygwin
Cygwin-x86_64
DJGPP
...
```
OpenSSL-fips-1.0.2k编译配置示例如下所示：
```sh
# ia64, x86_64, ppc are OK by default
# Configure the build tree.  Override OpenSSL defaults with known-good defaults
# usable on all platforms.  The Configure script already knows to use -fPIC and
# RPM_OPT_FLAGS, so we can skip specifiying them here.
./Configure \
    --prefix=%{_prefix} --openssldir=%{_sysconfdir}/pki/tls ${sslflags} \
    zlib sctp enable-camellia enable-seed enable-tlsext enable-rfc3779 \
    enable-cms enable-md2 enable-rc5 \
    no-mdc2 no-ec2m no-gost no-srp \
    --with-krb5-flavor=MIT --enginesdir=%{_libdir}/openssl/engines \
    --with-krb5-dir=/usr shared  ${sslarch} %{?!nofips:fips}

# Add -Wa,--noexecstack here so that libcrypto's assembler modules will be
# marked as not requiring an executable stack.
# Also add -DPURIFY to make using valgrind with openssl easier as we do not
# want to depend on the uninitialized memory as a source of entropy anyway.
RPM_OPT_FLAGS="$RPM_OPT_FLAGS -Wa,--noexecstack -DPURIFY"
make depend
make all

# Generate hashes for the included certs.
make rehash

# Overwrite FIPS README and copy README.legacy-settings
cp -f %{SOURCE5} %{SOURCE11} .

# Clean up the .pc files
for i in libcrypto.pc libssl.pc openssl.pc ; do
  sed -i '/^Libs.private:/{s/-L[^ ]* //;s/-Wl[^ ]* //}' $i
done
```

#### OpenSSL的Configure和config
OpenSSL的编译配置可以使用Configure和config来实现，两者的不同之处在于Configure能够合理的处理host，Arch， compiler。但是config不会做出这样的处理，它会猜测host，Arch， compiler这个三者，config有点像autotool的config.guess。
使用示例：
```sh
$ ./config 
Operating system: x86_64-whatever-linux2
Configuring for linux-x86_64
Configuring for linux-x86_64
    no-ec_nistp_64_gcc_128 [default]  OPENSSL_NO_EC_NISTP_64_GCC_128 (skip dir)
    no-gmp          [default]  OPENSSL_NO_GMP (skip dir)
    no-jpake        [experimental] OPENSSL_NO_JPAKE (skip dir)
    no-krb5         [krb5-flavor not specified] OPENSSL_NO_KRB5
    ...

./Configure darwin64-x86_64-cc
Configuring for darwin64-x86_64-cc
    no-ec_nistp_64_gcc_128 [default]  OPENSSL_NO_EC_NISTP_64_GCC_128 (skip dir)
    no-gmp          [default]  OPENSSL_NO_GMP (skip dir)
    no-jpake        [experimental] OPENSSL_NO_JPAKE (skip dir)
    no-krb5         [krb5-flavor not specified] OPENSSL_NO_KRB5
    ...
```

#### OpenSSL的配置选项 

|Option|Description|
|:-----|:----------|
|--prefix=XXX|See PREFIX and OPENSSLDIR in the next section (below).|
|--openssldir=XXX|See PREFIX and OPENSSLDIR in the next section (below).|
|-d|Debug build of the library. Optimizations are disabled (no -O3 or similar) and libefence is used (apt-get install electric-fence or yum install electric-fence). TODO: Any other features?|
|shared|Build a shared object in addition to the static archive. You probably need a RPATH when enabling shared to ensure openssl uses the correct libssl and libcrypto after installation.|
|enable-ec_nistp_64_gcc_128|Use on little endian platforms when GCC supports __uint128_t. ECDH is about 2 to 4 times faster. Not enabled by default because Configure can't determine it. Enable it if your compiler defines __SIZEOF_INT128__, the CPU is little endian and it tolerates unaligned data access.|
|enable-capieng|Enables the Microsoft CAPI engine on Windows platforms. Used to access the Windows Certificate Store. Also see Using Windows certificate store through OpenSSL on the OpenSSL developer list.|
|no-ssl2|Disables SSLv2. OPENSSL_NO_SSL2 will be defined in the OpenSSL headers.|
|no-ssl3|Disables SSLv3. OPENSSL_NO_SSL3 will be defined in the OpenSSL headers.|
|no-comp|Disables compression independent of zlib. OPENSSL_NO_COMP will be defined in the OpenSSL headers.|
|no-idea|Disables IDEA algorithm. Unlike RC5 and MDC2, IDEA is enabled by default|
|no-asm|Disables assembly language routines (and uses C routines)|
|no-dtls|Disables DTLS in OpenSSL 1.1.0 and above|
|no-dtls1|Disables DTLS in OpenSSL 1.0.2 and below|
|no-shared|Disables shared objects (only a static library is created)|
|no-hw|Disables hardware support (useful on mobile devices)|
|no-engine|Disables hardware support (useful on mobile devices)|
|no-threads|Disables threading support.|
|no-dso|Disables the OpenSSL DSO API (the library offers a shared object abstraction layer). If you disable DSO, then you must disable Engines also|
|no-err|Removes all error function names and error reason text to reduce footprint|
|no-npn/no-nextprotoneg|Disables Next Protocol Negotiation (NPN). Use no-nextprotoneg for 1.1.0 and above; and no-npn otherwise|
|no-psk|Disables Preshared Key (PSK). PSK provides mutual authentication independent of trusted authorities, but its rarely offered or used|
|no-srp|Disables Secure Remote Password (SRP). SRP provides mutual authentication independent of trusted authorities, but its rarely offered or used|
|no-ec2m|Used when configuring FIPS Capable Library with a FIPS Object Module that only includes prime curves. That is, use this switch if you use openssl-fips-ecp-2.0.5.|
|no-weak-ssl-ciphers|Disables RC4. Available in OpenSSL 1.1.0 and above.|
|-DXXX|Defines XXX. For example, -DOPENSSL_NO_HEARTBEATS.|
|-DOPENSSL_USE_IPV6=0|Disables IPv6. Useful if OpenSSL encounters incorrect or inconsistent platform headers and mistakenly enables IPv6. Must be passed to Configure manually.|
|-Lsomething, -lsomething, -Ksomething, -Wl,something|Linker options,will become part of LDFLAGS.|
|-anythingelse, +anythingelse|Compiler options, will become part of CFLAGS.|

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

### OpenSSL与数字证书
#### X.509数字证书
数字证书是网络世界的电子身份证，它由CA中心颁发，包含了证书所有者的姓名、序列号、失效日期、公钥和数字签名。数字证书大多遵循X.509标准，X.509标准实际上是基于ASN.1语言的公钥证书的一种格式。X.509标准中用到的加密算法和协议采用了PKCS标准（Public Key Cryptography Standards），PKCS定义了数字证书中用到的技术标准。

#### 数字签名
* 将报文按双方约定的HASH算法计算得到一个固定位数的报文摘要。在数学上保证:只要改动报文中任何一位，重新计算出的报文摘要值就会与原先的值不相符。这样就保证了报文的不可更改性。
* 将该报文摘要值用发送者的私人密钥加密，然后连同原报文一起发送给接收者，而产生的报文即称数字签名。
* 接收方收到数字签名后，用同样的HASH算法对报文计算摘要值，然后与用发送者的公开密钥进行解密解开的报文摘要值相比较。如相等则说明报文确实来自所称的发送者。

#### X.509证书解析
```sh
[root@localhost certs]# openssl x509 -in EulerOSSecureBoot_1.cer -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            9a:91:6d:18:be:15:8e:1d
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=CN, ST=Zhejiang, L=Hangzhou, O=HUAWEI TECHNOLOGIES CO.,LTD, OU=EulerOS, CN=EulerOS Secure Boot(Signing key)/emailAddress=euleros-security@huawei.com
        Validity
            Not Before: Jul  6 01:32:47 2017 GMT
            Not After : Jul  4 01:32:47 2027 GMT
        Subject: C=CN, ST=Zhejiang, L=Hangzhou, O=HUAWEI TECHNOLOGIES CO.,LTD, OU=EulerOS, CN=EulerOS Secure Boot(Signing key)/emailAddress=euleros-security@huawei.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:c6:fb:d6:cc:d2:94:74:f5:8b:7f:80:cf:00:a7:
                    3b:e7:b8:02:4d:0f:2d:5e:f8:71:e1:ef:53:30:72:
                    c5:b6:df:43:29:5f:d0:53:d3:34:9c:dc:7d:41:0a:
                    62:c8:06:3c:0b:d9:c6:31:f4:26:2d:31:c7:4e:4e:
                    4e:6f:32:60:d9:b6:f8:50:17:42:7e:9b:9a:ed:c5:
                    49:12:5b:cf:14:da:0e:cc:6a:ad:f5:bc:1b:84:dd:
                    71:8c:35:c3:7f:93:ff:51:2a:47:97:19:ce:3b:94:
                    a9:2f:86:6a:19:74:41:fb:21:91:6e:4c:fd:df:f2:
                    f7:d0:cb:ab:25:33:8c:a8:40:05:79:3f:b6:4d:5c:
                    4b:f5:31:53:48:6c:3f:fa:27:06:b6:78:ac:07:4d:
                    86:f1:ab:02:8a:e4:e1:a5:a7:d2:b1:35:58:51:5c:
                    7e:08:0c:9d:b1:a0:44:25:d2:e6:41:02:5d:94:21:
                    6d:b7:60:ca:de:f8:0e:70:6f:f4:a8:4c:54:6e:d7:
                    70:05:b4:86:d6:a3:9d:52:4d:2f:6b:ff:cd:5d:51:
                    75:85:e3:7f:ec:66:8f:d5:60:7d:22:90:42:39:95:
                    60:b1:58:dd:d9:35:34:85:ce:35:f9:dd:4c:54:35:
                    90:53:2d:7e:50:c2:50:27:de:3b:04:67:a0:8b:db:
                    24:99
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 Key Usage: 
                Digital Signature
            X509v3 Subject Key Identifier: 
                01:8C:F9:70:BB:D7:F0:B5:85:28:EF:1F:3A:61:FD:E2:75:C2:42:E6
            X509v3 Authority Key Identifier: 
                keyid:01:8C:F9:70:BB:D7:F0:B5:85:28:EF:1F:3A:61:FD:E2:75:C2:42:E6

    Signature Algorithm: sha256WithRSAEncryption
         49:ce:20:cb:b9:2f:71:cc:4a:1a:ed:7f:32:bc:16:23:e1:00:
         06:83:50:27:d6:e8:20:19:10:19:92:09:61:2e:df:c3:89:b2:
         65:a3:49:99:cb:9f:e3:46:59:de:ff:d9:d9:f0:2d:d1:b0:c6:
         0e:8c:ca:a5:8f:2a:9d:6b:a6:25:6d:13:5e:48:52:48:16:1c:
         da:8f:b2:c0:a0:45:22:70:ff:f7:8f:68:bd:18:83:f8:4a:96:
         fb:b9:0a:a0:b4:51:4e:d8:05:0d:12:15:7a:7a:a8:ad:82:db:
         c5:f3:d5:c4:b0:40:e7:a5:93:db:6c:bf:55:2f:1c:07:4a:77:
         8a:e1:49:aa:bc:85:0b:51:c5:a6:8a:19:3d:80:d6:28:42:a3:
         b6:51:35:41:b8:b9:b9:ca:d5:f9:3a:fb:c1:fb:99:dc:b9:72:
         aa:ed:9f:10:9b:ce:c8:d7:d2:f6:7c:14:dd:3c:b7:36:5d:db:
         f1:f1:cc:24:24:6b:ae:08:ec:c2:09:0a:74:ee:01:a6:52:2a:
         54:ac:06:74:df:7f:a1:b4:bf:20:0c:43:c6:12:58:8f:d4:11:
         9e:65:7b:d8:08:c9:c0:99:78:e2:a4:03:b0:ac:ef:43:6c:a2:
         54:ea:c4:1e:e7:d3:84:0b:ec:26:be:df:81:d5:f3:6c:cd:11:
         46:c4:a5:f0
```


### OpenSSL源码分析基础
#### OpenSSL堆栈数据结构
OpenSSL大量采用堆栈来存放数据。它实现了一个通用的堆栈，可以方便的存储任意数据。它实现了许多基本的堆栈操作，主要有：堆栈拷贝(`sk_dup`)，构建新堆栈(`sk_new_null`，`sk_new`)，插入数据(`sk_insert`)，删除数据(`sk_delete`)，查找数据(`sk_find`，`sk_find_ex`)，入栈(`sk_push`)，出栈(`sk_pop`)，获取堆栈元素个数(`sk_num`)，获取堆栈值(`sk_value`)，设置堆栈值(`sk_set`)和堆栈排序(`sk_sort`)。OpenSSL堆栈数据结构在 `crypto/stack/stack.h` 中定义如下：
```c
typedef struct stack_st {
    int num;
    char **data;
    int sorted;
    int num_alloc;
    int (*comp) (const void *, const void *);
} _STACK;                       /* Use STACK_OF(...) instead */

/* 参数说明 */
/*
* num: 堆栈中存放数据的个数。
* data: 用于存放数据地址，每个数据地址存放在 data[0]到 data[num-1]中。
* sorted: 堆栈是否已排序，如果排序则值为 1，否则为 0，堆栈数据一般是无序的，只有当用户调用了 sk_sort 操作，其值才为 1。
* comp: 堆栈内存放数据的比较函数地址，此函数用于排序和查找操作；当用户生成一个新堆栈时，可以指定 comp 为用户实现的一个比较函数；或当堆栈已经存在时通过调用 sk_set_cmp_func 函数来重新指定比较函数。
*/
```

注意，用户不需要调用底层的堆栈函数(`sk_sort`、`sk_set_cmp_func` 等)，而是调用他通过宏实现的各个函数。用户直接调用最底层的堆栈操作函数是一个麻烦的事情，对此OpenSSL提供了用宏来帮助用户实现接口。用户可以参考safestack.h来定义自己的上层堆栈操作函数，safestack.h定义了如下关于GENERAL_NAME数据结构的堆栈操作：
```c
# define sk_GENERAL_NAME_new(cmp) SKM_sk_new(GENERAL_NAME, (cmp))
# define sk_GENERAL_NAME_new_null() SKM_sk_new_null(GENERAL_NAME)
# define sk_GENERAL_NAME_free(st) SKM_sk_free(GENERAL_NAME, (st))
# define sk_GENERAL_NAME_num(st) SKM_sk_num(GENERAL_NAME, (st))
# define sk_GENERAL_NAME_value(st, i) SKM_sk_value(GENERAL_NAME, (st), (i))
# define sk_GENERAL_NAME_set(st, i, val) SKM_sk_set(GENERAL_NAME, (st), (i), (val))
# define sk_GENERAL_NAME_zero(st) SKM_sk_zero(GENERAL_NAME, (st))
# define sk_GENERAL_NAME_push(st, val) SKM_sk_push(GENERAL_NAME, (st), (val))
# define sk_GENERAL_NAME_unshift(st, val) SKM_sk_unshift(GENERAL_NAME, (st), (val))
# define sk_GENERAL_NAME_find(st, val) SKM_sk_find(GENERAL_NAME, (st), (val))
# define sk_GENERAL_NAME_find_ex(st, val) SKM_sk_find_ex(GENERAL_NAME, (st), (val))
# define sk_GENERAL_NAME_delete(st, i) SKM_sk_delete(GENERAL_NAME, (st), (i))
# define sk_GENERAL_NAME_delete_ptr(st, ptr) SKM_sk_delete_ptr(GENERAL_NAME, (st), (ptr))
# define sk_GENERAL_NAME_insert(st, val, i) SKM_sk_insert(GENERAL_NAME, (st), (val), (i))
# define sk_GENERAL_NAME_set_cmp_func(st, cmp) SKM_sk_set_cmp_func(GENERAL_NAME, (st), (cmp))
# define sk_GENERAL_NAME_dup(st) SKM_sk_dup(GENERAL_NAME, st)
# define sk_GENERAL_NAME_pop_free(st, free_func) SKM_sk_pop_free(GENERAL_NAME, (st), (free_func))
# define sk_GENERAL_NAME_deep_copy(st, copy_func, free_func) SKM_sk_deep_copy(GENERAL_NAME, (st), (copy_func), (free_func))
# define sk_GENERAL_NAME_shift(st) SKM_sk_shift(GENERAL_NAME, (st))
# define sk_GENERAL_NAME_pop(st) SKM_sk_pop(GENERAL_NAME, (st))
# define sk_GENERAL_NAME_sort(st) SKM_sk_sort(GENERAL_NAME, (st))
# define sk_GENERAL_NAME_is_sorted(st) SKM_sk_is_sorted(GENERAL_NAME, (st))
# define sk_GENERAL_NAMES_new(cmp) SKM_sk_new(GENERAL_NAMES, (cmp))
# define sk_GENERAL_NAMES_new_null() SKM_sk_new_null(GENERAL_NAMES)
# define sk_GENERAL_NAMES_free(st) SKM_sk_free(GENERAL_NAMES, (st))
# define sk_GENERAL_NAMES_num(st) SKM_sk_num(GENERAL_NAMES, (st))
# define sk_GENERAL_NAMES_value(st, i) SKM_sk_value(GENERAL_NAMES, (st), (i))
# define sk_GENERAL_NAMES_set(st, i, val) SKM_sk_set(GENERAL_NAMES, (st), (i), (val))
# define sk_GENERAL_NAMES_zero(st) SKM_sk_zero(GENERAL_NAMES, (st))
# define sk_GENERAL_NAMES_push(st, val) SKM_sk_push(GENERAL_NAMES, (st), (val))
# define sk_GENERAL_NAMES_unshift(st, val) SKM_sk_unshift(GENERAL_NAMES, (st), (val))
# define sk_GENERAL_NAMES_find(st, val) SKM_sk_find(GENERAL_NAMES, (st), (val))
# define sk_GENERAL_NAMES_find_ex(st, val) SKM_sk_find_ex(GENERAL_NAMES, (st), (val))
# define sk_GENERAL_NAMES_delete(st, i) SKM_sk_delete(GENERAL_NAMES, (st), (i))
# define sk_GENERAL_NAMES_delete_ptr(st, ptr) SKM_sk_delete_ptr(GENERAL_NAMES, (st), (ptr))
# define sk_GENERAL_NAMES_insert(st, val, i) SKM_sk_insert(GENERAL_NAMES, (st), (val), (i))
# define sk_GENERAL_NAMES_set_cmp_func(st, cmp) SKM_sk_set_cmp_func(GENERAL_NAMES, (st), (cmp))
# define sk_GENERAL_NAMES_dup(st) SKM_sk_dup(GENERAL_NAMES, st)
# define sk_GENERAL_NAMES_pop_free(st, free_func) SKM_sk_pop_free(GENERAL_NAMES, (st), (free_func))
# define sk_GENERAL_NAMES_deep_copy(st, copy_func, free_func) SKM_sk_deep_copy(GENERAL_NAMES, (st), (copy_func), (free_func))
# define sk_GENERAL_NAMES_shift(st) SKM_sk_shift(GENERAL_NAMES, (st))
# define sk_GENERAL_NAMES_pop(st) SKM_sk_pop(GENERAL_NAMES, (st))
# define sk_GENERAL_NAMES_sort(st) SKM_sk_sort(GENERAL_NAMES, (st))
# define sk_GENERAL_NAMES_is_sorted(st) SKM_sk_is_sorted(GENERAL_NAMES, (st))
OpenSSL栈应用的一个简单例子：
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <openssl/safestack.h>

#define sk_Student_new(cmp) SKM_sk_new(Student, (cmp))
#define sk_Student_new_null() SKM_sk_new_null(Student)
#define sk_Student_free(st) SKM_sk_free(Student, (st))
#define sk_Student_num(st) SKM_sk_num(Student, (st))
#define sk_Student_value(st, i) SKM_sk_value(Student, (st), (i))
#define sk_Student_set(st, i, val) SKM_sk_set(Student, (st), (i), (val))
#define sk_Student_zero(st) SKM_sk_zero(Student, (st))
#define sk_Student_push(st, val) SKM_sk_push(Student, (st), (val))
#define sk_Student_unshift(st, val) SKM_sk_unshift(Student, (st), (val))
#define sk_Student_find(st, val) SKM_sk_find(Student, (st), (val))
#define sk_Student_delete(st, i) SKM_sk_delete(Student, (st), (i))
#define sk_Student_delete_ptr(st, ptr) SKM_sk_delete_ptr(Student, (st), (ptr))
#define sk_Student_insert(st, val, i) SKM_sk_insert(Student, (st), (val), (i))
#define sk_Student_set_cmp_func(st, cmp) SKM_sk_set_cmp_func(Student, (st), (cmp))
#define sk_Student_dup(st) SKM_sk_dup(Student, st)
#define sk_Student_pop_free(st, free_func) SKM_sk_pop_free(Student, (st), (free_func))
#define sk_Student_shift(st) SKM_sk_shift(Student, (st))
#define sk_Student_pop(st) SKM_sk_pop(Student, (st))
#define sk_Student_sort(st) SKM_sk_sort(Student, (st))

typedef struct Student_st
{
	char *name;
	int age;
	char *otherInfo;
}Student;

typedef STACK_OF(Student) Students;

Student *Student_Malloc()
{
	Student *a = malloc(sizeof(Student));
	a->name = malloc(20);
	strcpy(a->name,"zpc");
	a->otherInfo = malloc(20);
	strcpy(a->otherInfo,"no info");
	return a;
}

void Student_Free(Student *a)
{
	free(a->name);
	free(a->otherInfo);
	free(a);
}

static int Student_cmp(Student *a,Student *b)
{
	int ret;
	ret = strcmp(a->name,b->name);
	return ret;
}

int main()
{
	Students *s,*snew;
	Student *s1,*one,*s2;
	int i,num;
	s = sk_Student_new_null();
	snew = sk_Student_new(Student_cmp);
	s2 = Student_Malloc();
	sk_Student_push(snew,s2);
	i = sk_Student_find(snew,s2);
	s1 = Student_Malloc();
	sk_Student_push(snew,s1);
	num = sk_Student_num(snew);
	printf("stack numbers: %d\n", num);
	for(i=0;i<num;i++)
	{
		one = sk_Student_value(snew,i);
		printf("student name : %s\n",one->name);
		printf("sutdent age : %d\n",one->age);
		printf("student otherinfo : %s\n\n\n",one->otherInfo);
	}
	sk_Student_pop_free(s,Student_Free);
	sk_Student_pop_free(snew,Student_Free);
	return 0;
}
```

#### OpenSSL内存数据结构
用户在使用内存时，容易犯的错误就是内存泄露。当用户调用内存分配和释放函数时，查找内存泄露比较麻烦。openssl 提供了内置的内存分配/释放函数。如果用户完全调用 openssl 的内存分配和释放函数，可以方便的找到内存泄露点。openssl 分配内存时，在其内部维护一个内存分配哈希表，用于存放已经分配但未释放的内存信息。当用户申请内存分配时，在哈希表中添加此项信息，内存释放时删除该信息。当用户通过 openssl函数查找内存泄露点时，只需查询该哈希表即可。用户通过 openssl 回调函数还能处理那些泄露的内存。openssl 供用户调用的内存分配等函数主要在 `crypto/mem.c` 中实现，其内置的分配函数在 `crypto/mem_dbg.c` 中实现。默认情况下 `mem.c` 中的函数调用 `mem_dbg.c` 中的实现。如果用户实现了自己的内存分配函数以及查找内存泄露的函数，可以通过调用`CRYPTO_set_mem_functions` 函数和 `CRYPTO_set_mem_debug_functions` 函数来设置。下面主要介绍了 openssl 内置的内存分配和释放函数。
```c
typedef struct app_mem_info_st
/*-
 * For application-defined information (static C-string `info')
 * to be displayed in memory leak list.
 * Each thread has its own stack.  For applications, there is
 *   CRYPTO_push_info("...")     to push an entry,
 *   CRYPTO_pop_info()           to pop an entry,
 *   CRYPTO_remove_all_info()    to pop all entries.
 */
{
    CRYPTO_THREADID threadid;
    const char *file;
    int line;
    const char *info;
    struct app_mem_info_st *next; /* tail of thread's stack */
    int references;
} APP_INFO;


typedef struct mem_st
/* memory-block description */
{
    void *addr;
    int num;
    const char *file;
    int line;
    CRYPTO_THREADID threadid;
    unsigned long order;
    time_t time;
    APP_INFO *app_info;
} MEM;
/* 参数说明 */
/*
 * addr：分配内存的地址。
* num：分配内存的大小。
* file：分配内存的文件。
* line：分配内存的行号。
* thread：分配内存的线程 ID。
* order：第几次内存分配。
* time：内存分配时间。
* app_info:用于存放用户应用信息，为一个链表，里面存放了文件、行号以及线程ID等信息。
* references：被引用次数。
*/

# define OPENSSL_malloc(num)     CRYPTO_malloc((int)num,__FILE__,__LINE__)
# define OPENSSL_strdup(str)     CRYPTO_strdup((str),__FILE__,__LINE__)
# define OPENSSL_realloc(addr,num) \
        CRYPTO_realloc((char *)addr,(int)num,__FILE__,__LINE__)
# define OPENSSL_realloc_clean(addr,old_num,num) \
        CRYPTO_realloc_clean(addr,old_num,num,__FILE__,__LINE__)
# define OPENSSL_remalloc(addr,num) \
        CRYPTO_remalloc((char **)addr,(int)num,__FILE__,__LINE__)
# define OPENSSL_freeFunc        CRYPTO_free
# define OPENSSL_free(addr)      CRYPTO_free(addr)

# define OPENSSL_malloc_locked(num) \
        CRYPTO_malloc_locked((int)num,__FILE__,__LINE__)
# define OPENSSL_free_locked(addr) CRYPTO_free_locked(addr)
```

#### OpenSSL动态加载数据结构
DSO可以让用户动态加载动态库来进行函数调用。各个平台下加载动态库的函数是不一样的，openssl的DSO对各个平台台下的动态库加载函数进行了封装，增加了源码的可移植性。Openssl的DSO功能主要用于动态加载压缩函数（ssl协议）和engine(硬件加速引擎)。Openssl的DSO功能除了封装基本的功能外还有其他辅助函数，主要用于解决不同系统下路径不同的表示方式以及动态库全名不一样的问题。比如windows系统下路径可以用“\\”和“/”表示，而 linux 下只能使用“/”，windows下动态库的后缀为.dll而linux下动态库名字一般为libxxx.so。
```c
struct dso_st {
    DSO_METHOD *meth;
    /*
     * Standard dlopen uses a (void *). Win32 uses a HANDLE. VMS doesn't use
     * anything but will need to cache the filename for use in the dso_bind
     * handler. All in all, let each method control its own destiny.
     * "Handles" and such go in a STACK.
     */
    STACK_OF(void) *meth_data;
    int references;
    int flags;
    /*
     * For use by applications etc ... use this for your bits'n'pieces, don't
     * touch meth_data!
     */
    CRYPTO_EX_DATA ex_data;
    /*
     * If this callback function pointer is set to non-NULL, then it will be
     * used in DSO_load() in place of meth->dso_name_converter. NB: This
     * should normally set using DSO_set_name_converter().
     */
    DSO_NAME_CONVERTER_FUNC name_converter;
    /*
     * If this callback function pointer is set to non-NULL, then it will be
     * used in DSO_load() in place of meth->dso_merger. NB: This should
     * normally set using DSO_set_merger().
     */
    DSO_MERGER_FUNC merger;
    /*
     * This is populated with (a copy of) the platform-independant filename
     * used for this DSO.
     */
    char *filename;
    /*
     * This is populated with (a copy of) the translated filename by which
     * the DSO was actually loaded. It is NULL iff the DSO is not currently
     * loaded. NB: This is here because the filename translation process may
     * involve a callback being invoked more than once not only to convert to
     * a platform-specific form, but also to try different filenames in the
     * process of trying to perform a load. As such, this variable can be
     * used to indicate (a) whether this DSO structure corresponds to a
     * loaded library or not, and (b) the filename with which it was actually
     * loaded.
     */
    char *loaded_filename;
};
```

#### OpenSSL抽象IO数据结构
OpenSSL抽象 IO(I/O abstraction，即BIO)是OpenSSL对于io类型的抽象封装，包括：内存、文件、日志、标准输入输出、socket（TCP/UDP）、加/解密、摘要和ssl通道等。OpenSSL BIO通过回调函数为用户隐藏了底层实现细节，所有类型的bio的调用大体上是类似的。Bio中的数据能从一个BIO传送到另外一个BIO或者是应用程序。
```c
typedef struct bio_method_st {
    int type;
    const char *name;
    int (*bwrite) (BIO *, const char *, int);
    int (*bread) (BIO *, char *, int);
    int (*bputs) (BIO *, const char *);
    int (*bgets) (BIO *, char *, int);
    long (*ctrl) (BIO *, int, long, void *);
    int (*create) (BIO *);
    int (*destroy) (BIO *);
    long (*callback_ctrl) (BIO *, int, bio_info_cb *);
} BIO_METHOD;
/* 参数说明 */
/*
 * type：具体 BIO 类型；
* name：具体 BIO 的名字；
* bwrite：具体 BIO 写操作回调函数；
* bread：具体 BIO 读操作回调函数；
* bputs：具体 BIO 中写入字符串回调函数；
* bgets：具体 BIO 中读取字符串函数；
* ctrl：具体 BIO 的控制回调函数；
* create：生成具体 BIO 回调函数；
* destroy：销毁具体 BIO 回调函数；
* callback_ctrl：具体 BIO 控制回调函数，与 ctrl 回调函数不一样，该函数可由调用者（而不是实现者）来实现，然后通过 BIO_set_callback 等函数来设置。
*/

struct bio_st {
    BIO_METHOD *method;
    /* bio, mode, argp, argi, argl, ret */
    long (*callback) (struct bio_st *, int, const char *, int, long, long);
    char *cb_arg;               /* first argument for the callback */
    int init;
    int shutdown;
    int flags;                  /* extra storage */
    int retry_reason;
    int num;
    void *ptr;
    struct bio_st *next_bio;    /* used by filter BIOs */
    struct bio_st *prev_bio;    /* used by filter BIOs */
    int references;
    unsigned long num_read;
    unsigned long num_write;
    CRYPTO_EX_DATA ex_data;
};
/* 参数说明 */
/*
 * init：具体句柄初始化标记，初始化后为 1。比如文件 BIO 中，通过 BIO_set_fp关联一个文件指针时，该标记则置 1；socket BIO 中通过 BIO_set_fd 关联一个链接时设置该标记为 1。
* shutdown：BIO 关闭标记，当该值不为 0 时，释放资源；改值可以通过控制函数来设置。
* flags：有些 BIO 实现需要它来控制各个函数的行为。比如文件 BIO 默认该值为 BIO_FLAGS_UPLINK，这时文件读操作调用 UP_fread 函数而不是调用 fread 函数。
* retry_reason：重试原因，主要用在 socket 和 ssl BIO 的异步阻塞。比如 socketbio 中，遇到 WSAEWOULDBLOCK 错误时，openssl 告诉用户的操作需要重试。
* num：该值因具体 BIO 而异，比如 socket BIO 中 num 用来存放链接字。
* ptr：指针，具体 bio 有不同含义。比如文件 BIO 中它用来存放文件句柄；membio 中它用来存放内存地址；connect bio 中它用来存放 BIO_CONNECT 数据，acceptbio 中它用来存放 BIO_ACCEPT 数据。
* next_bio：下一个 BIO 地址，BIO 数据可以从一个 BIO 传送到另一个 BIO，该值指明了下一个 BIO 的地址。
* references：被引用数量。
* num_read：BIO 中已读取的字节数。
* num_write：BIO 中已写入的字节数。
* ex_data：用于存放额外数据。

BIO 各个函数定义在 crypto/bio.h 中。所有的函数都由 BIO_METHOD 中的回调函
数来实现。函数主要分为几类：
	具体 BIO 相关函数，如：BIO_new_file（生成新文件）和 BIO_get_fd（设置网络链接）等。
	通用抽象函数，如 BIO_read 和 BIO_write 等。
	有很多函数是由宏定义通过控制函数 BIO_ctrl 实现，比如 BIO_set_nbio、BIO_get_fd 和 BIO_eof 等等。
5)	OpenSSL大数数据结构
密码学中采用了很多大数计算，为了让计算机实现大数运算，用户需要定义自己的大数表示方式并及实现各种大数运算。Openssl 为我们提供了这些功能，主要用于非对称算法。
typedef struct bignum_st BIGNUM;

struct bignum_st {
    BN_ULONG *d;                /* Pointer to an array of 'BN_BITS2' bit
                                 * chunks. */
    int top;                    /* Index of last used d +1. */
    /* The next are internal book keeping for bn_expand. */
    int dmax;                   /* Size of the d array. */
    int neg;                    /* one if the number is negative */
    int flags;
};
```

#### OpenSSL的大数数据结构
OpenSSL中用到的大数由`BIGNUM`结构体表示，整数值存放在`BN_ULONG *d`中，`d`是一个由`malloc`分配的字数组(array of words)。`BN_ULONG`类型的大小可能是16,32,64位，具体的数据取决于`openssl/bn.h`中指定的BITS2的大小。dmax是d数组的大小，top是被使用的字的个数，举例来说，对于大数4，bn.d[0] = 4，bn.top = 1。如果该大数为负数则neg = 1。当BIGNUM为0时，d == NULL并且top == 0。flags是一个在opnessl/bn.h中定义的标志位，该标志位以BN_FLG_开头。宏定义BN_set_flags(b,n)和BN_get_flags(b,n)可以获取BIGNUM结构体的标志位。OpenSSL库中很多地方需要使用临时的BIGNUM变量，但是BIGNUM在子线程同步调用时动态分配内存非常消耗资源，所以当，这种情况需要使用BN_CTX结构体，该结构体包含了BN_CTX_NUM大数据结构体。
```c
/***********/
/* BN_POOL */
/***********/

/* A bundle of bignums that can be linked with other bundles */
typedef struct bignum_pool_item {
    /* The bignum values */
    BIGNUM vals[BN_CTX_POOL_SIZE];
    /* Linked-list admin */
    struct bignum_pool_item *prev, *next;
} BN_POOL_ITEM;
/* A linked-list of bignums grouped in bundles */
typedef struct bignum_pool {
    /* Linked-list admin */
    BN_POOL_ITEM *head, *current, *tail;
    /* Stack depth and allocation size */
    unsigned used, size;
} BN_POOL;

/**********/
/* BN_CTX */
/**********/

/* The opaque BN_CTX type */
struct bignum_ctx {
    /* The bignum bundles */
    BN_POOL pool;
    /* The "stack frames", if you will */
    BN_STACK stack;
    /* The number of bignums currently assigned */
    unsigned int used;
    /* Depth of stack overflow */
    int err_stack;
    /* Block "gets" until an "end" (compatibility behaviour) */
    int too_many;
};
```

大数数据结构相关函数：
```c
BN_rand/BN_pseudo_rand	//生成一个随机的大数。
BN_rand_range/BN_pseudo_rand_range	//生成随机数，但是给出了随机数的范围。
BN_dup	//大数复制。
BN_generate_prime	//生成素数。
int BN_add_word(BIGNUM *a, BN_ULONG w)	//给大数a加上w，如果成功，返回1。
BIGNUM *BN_bin2bn(const unsigned char *s, int len, BIGNUM *ret)	//将内存中的数据转换为大数，为内存地址，len 为数据长度 ，ret 为返回值。
int BN_bn2bin(const BIGNUM *a, unsigned char *to)	//将大数转换为内存形式。输入参数为大数 a，to 为输出缓冲区地址，缓冲区需要预先分配，返回值为缓冲区的长度。
char *BN_bn2dec(const BIGNUM *a)	//将大数转换成整数字符串。返回值中存放整数字符串，它由内部分配空间，用户必须在外部用OPENSSL_free函数释放该空间。
char *BN_bn2hex(const BIGNUM *a)	//将大数转换为十六进制字符串。返回值为生成的十六进制字符串，外部需要用 OPENSSL_free 函数释放
BN_cmp	//比较两个大数。
BIGNUM *BN_mod_inverse(BIGNUM *in, const BIGNUM *a,const BIGNUM *n, BN_CTX *ctx)	//计算 ax=1(mod n)。
```

#### OpenSSL多线程支持
编写 openssl 多线程程序时，需要设置两个回调函数：
```c
CRYPTO_set_id_callback((unsigned long (*)())pthreads_thread_id);
CRYPTO_set_locking_callback((void (*)())pthreads_locking_callback);
```
对于多线程程序的写法，可以参考 `crypto/threads/mttest.c`


### 关于OpenSSL主要代码实现
OpenSSL命令行实现采用了面向对象的思想，它定义了FUNCTION对象，该对象表示OpenSSL命令行的功能（如asn1解析功能、签名校验功能），将具体的功能实现注册到FUNCTION对象中。主流程通过命令行参数来索引FUNCTION对象，然后do_cmd完成具体的功能。OpenSSL命令的实现的API调用流程如下图所示。

#### OpenSSL接口封装EVP的实现
##### EVP框架使用说明
OpenSSL实现了高级的结构封装EVP，用于各种密码功能。
* EVP_Seal###和EVP_Open###提供了公钥加解密的接口封装；
* EVP_DigestSign###和EVP_DigestVerify###实现了数字签名和MAC(Message Authentication Codes)，上述功能还可以参考接口EVP_Sign###和EVP_Verify###；
* EVP_PKEY###提供了所有的非对称算法的接口，通过`EVP_PKEY_new()`创建EVP_PKEY对象，EVP_PKEY对象可以将私钥与特定的非对称算法关联起来。秘钥协商使用接口`EVP_PKEY_derive`，签名校验使用接口`EVP_PKEY_sign`和`EVP_PKEY_verify`，加解密使用接口`EVP_PKEY_encrypt`和`EVP_PKEY_decrypt`；
* EVP_Encode###和EVP_Decode###实现了base64的解码和编码；

EVP框架的使用套路：
```c
#include <stdlib.h>
#include <openssl/evp.h>

int main()
{
    const char *alg = "sha256";
    const EVP_MD *m;
    EVP_MD_CTX ctx;
    unsigned char *ret;
    char buf[10] = {'0', '1', '2', '3', '4', '5','6','7','8','9'};
    int out_len;
    int i;

    // NOTE(edony)
    /* add all messeage digest algorithms into
     * OpenSSL internal method hash table so that
     * we can use EVP_get_cipher_byname() or EVP_get_digestbyname()
     * to fetch the method.
     */
    OpenSSL_add_all_digests();
    if (!(m = EVP_get_digestbyname(alg)))
        return -1;
    if (!(ret = (unsigned char *)malloc(EVP_MAX_MD_SIZE)))
        return -1;

    // NOTE(edony)
    /* EVP messeage digest methods to work:
     * EVP_DigestInit() --> EVP_DigestUpdate() --> EVP_DigestFinal()
     */
    EVP_DigestInit(&ctx, m);
    EVP_DigestUpdate(&ctx, buf, 10);
    EVP_DigestFinal(&ctx, ret, &out_len);

    for(i=0; i < out_len; i++)
        printf("%02x", ret[i]);
    printf("\n");

    return 0;
}
```

##### EVP框架的源码分析
evp源码位于`openssl/crypto/evp`目录，可以分为如下几类：
* 全局函数
主要包括`c_allc.c`、`c_alld.c`、`c_all.c`以及`names.c`。他们加载OpenSSL支持的所有的对称算法和摘要算法，放入到OpenSSL内部维护的哈希表中。实现了`OpenSSL_add_all_digests`、`OpenSSL_add_all_ciphers``以及OpenSSL_add_all_algorithms`(调用了前两个函数)函数。在进行计算时，用户也可以单独加载摘要函数（`EVP_add_digest`）和对称计算函数（`EVP_add_cipher`）。
* BIO扩充
包括`bio_b64.c`、`bio_enc.c`、`bio_md.c`和`bio_ok.c`，各自实现了`BIO_METHOD`方法，分别用于`base64`编解码、对称加解密以及摘要。
* 摘要算法 EVP 封装
由`digest.c`实现，实现过程中调用了对应摘要算法的回调函数。各个摘要算法提供了自己的 EVP_MD静态结构，对应源码为m_xxx.c。
* 对称算法 EVP 封装
由`evp_enc.c`实现，实现过程调用了具体对称算法函数，实现了Update操作。各种对称算法都提供了一个EVP_CIPHER静态结构，对应源码为e_xxx.c。需要注意的是，e_xxx.c中不提供完整的加解密运算，它只提供基本的对于一个block_size数据的计算，完整的计算由`evp_enc.c`来实现。当用户想添加一个自己的对称算法时，可以参考e_xxx.c的实现方式。一般用户至少需要实现如下功能：
> 1. 构造一个新的静态的`EVP_CIPHER`结构；
> 2. 实现`EVP_CIPHER`结构中的init函数，该函数用于设置iv，设置加解密标记、以及根据外送密钥生成自己的内部密钥；
> 3. 实现do_cipher函数，该函数仅对block_size字节的数据进行对称运算；
> 4. 实现cleanup函数，该函数主要用于清除内存中的密钥信息。
* 非对称算法EVP封装
主要是以p_开头的文件。其中，`p_enc.c`封装了公钥加密；`p_dec.c`封装了私钥解密；`p_lib.c`实现一些辅助函数；`p_sign.c`封装了签名函数；`p_verify.c`封装了验签函数；`p_seal.c`封装了数字信封；`p_open.c`封装了解数字信封。
* 基于口令的加密
包括`p5_crpt2.c`、`p5_crpt.c`和`evp_pbe.c`。

#### OpenSSL的RSA公钥算法实现
##### RSA公钥算法
RSA算法RSA算法基于一个十分简单的数论事实：将两个大质数相乘十分容易，但是想要对其乘积进行因式分解却极其困难，因此可以将乘积公开作为加密密钥。RSA算法描述：

OpenSSL的RSA实现源码在`crypto/rsa`目录下。它实现了RSA PKCS1标准。主要源码如下：
```c
rsa.h	//定义 RSA 数据结构以及 RSA_METHOD，定义了 RSA 的各种函数。
rsa_asn1.c	//实现了 RSA 密钥的 DER 编码和解码，包括公钥和私钥。
rsa_chk.c	//RSA 密钥检查。
rsa_eay.c	//Openssl 实现的一种 RSA_METHOD，作为其默认的一种 RSA 计算实现方式。此文件未实现 rsa_sign、rsa_verify 和 rsa_keygen 回调函数。
rsa_err.c	//RSA 错误处理。
rsa_gen.c	//RSA 密钥生成，如果 RSA_METHOD 中的 rsa_keygen 回调函数不为空，则调用它，否则调用其内部实现。
rsa_lib.c	//主要实现了 RSA 运算的四个函数(公钥/私钥，加密/解密)，它们都调用了RSA_METHOD 中相应都回调函数。
rsa_none.c	//实现了一种填充和去填充。
rsa_null.c	//实现了一种空的 RSA_METHOD。
rsa_oaep.c	//实现了 oaep 填充与去填充。
rsa_pk1.c	//实现了 pkcs1 填充与去填充。
rsa_sign.c	//实现了 RSA 的签名和验签。
rsa_ssl.c	//实现了 ssl 填充。
rsa_x931.c	//实现了一种填充和去填充。
```

##### RSA签名
RSA 签名过程：
1) 对用户数据进行摘要。
2) 构造 X509_SIG 结构并 DER 编码，其中包括了摘要算法以及摘要结果。
3) 对 2)的结果进行填充，填满 RSA 密钥长度字节数。比如 1024 位 RSA 密钥必须填满 128 字节。具体的填充方式由用户指定。
4) 对 3)的结果用 RSA 私钥加密。
其中OpenSSL中的RSA_eay_private_encrypt函数实现了3)和4)过程。

##### RSA验签
RSA验签过程是签名过程的逆过程：
1) 对数据用RSA公钥解密，得到签名过程中 2)的结果。
2) 去除1)结果的填充。
3) 从2)的结果中得到摘要算法，以及摘要结果。
4) 将原数据根据3)中得到摘要算法进行摘要计算。
5） 比较4)与签名过程中1)的结果。

其中OpenSSL中的RSA_eay_public_decrypt函数实现了1)和2)过程。

##### RSA的OpenSSL实现
RSA的OpenSSL实现的主要数据结构定义在`crypto/rsa/rsa.h`中
```c
struct rsa_meth_st {
    const char *name;
    int (*rsa_pub_enc) (int flen, const unsigned char *from,
                        unsigned char *to, RSA *rsa, int padding);
    int (*rsa_pub_dec) (int flen, const unsigned char *from,
                        unsigned char *to, RSA *rsa, int padding);
    int (*rsa_priv_enc) (int flen, const unsigned char *from,
                         unsigned char *to, RSA *rsa, int padding);
    int (*rsa_priv_dec) (int flen, const unsigned char *from,
                         unsigned char *to, RSA *rsa, int padding);
    /* Can be null */
    int (*rsa_mod_exp) (BIGNUM *r0, const BIGNUM *I, RSA *rsa, BN_CTX *ctx);
    /* Can be null */
    int (*bn_mod_exp) (BIGNUM *r, const BIGNUM *a, const BIGNUM *p,
                       const BIGNUM *m, BN_CTX *ctx, BN_MONT_CTX *m_ctx);
    /* called at new */
    int (*init) (RSA *rsa);
    /* called at free */
    int (*finish) (RSA *rsa);
    /* RSA_METHOD_FLAG_* things */
    int flags;
    /* may be needed! */
    char *app_data;
    /*
     * New sign and verify functions: some libraries don't allow arbitrary
     * data to be signed/verified: this allows them to be used. Note: for
     * this to work the RSA_public_decrypt() and RSA_private_encrypt() should
     * *NOT* be used RSA_sign(), RSA_verify() should be used instead. Note:
     * for backwards compatibility this functionality is only enabled if the
     * RSA_FLAG_SIGN_VER option is set in 'flags'.
     */
    int (*rsa_sign) (int type,
                     const unsigned char *m, unsigned int m_length,
                     unsigned char *sigret, unsigned int *siglen,
                     const RSA *rsa);
    int (*rsa_verify) (int dtype, const unsigned char *m,
                       unsigned int m_length, const unsigned char *sigbuf,
                       unsigned int siglen, const RSA *rsa);
    /*
     * If this callback is NULL, the builtin software RSA key-gen will be
     * used. This is for behavioural compatibility whilst the code gets
     * rewired, but one day it would be nice to assume there are no such
     * things as "builtin software" implementations.
     */
    int (*rsa_keygen) (RSA *rsa, int bits, BIGNUM *e, BN_GENCB *cb);
};
/* 参数说明（edony） */
/*
* name：RSA_METHOD 名称；
* rsa_pub_enc：公钥加密函数，padding 为其填充方式，输入数据不能太长，否则无法填充；
* rsa_pub_dec：公钥解密函数，padding 为其去除填充的方式，输入数据长度为 RSA 密钥长度的字节数；
* rsa_priv_enc：私钥加密函数，padding 为其填充方式，输入数据长度不能太长，否则无法填充；
* rsa_priv_dec：私钥解密函数，padding 为其去除填充的方式，输入数据长度为 RSA 密钥长度的字节数；
* rsa_sign：签名函数；
* rsa_verify：验签函数；
* rsa_keygen：RSA 密钥对生成函数。
* 用户可实现自己的 RSA_METHOD 来替换 openssl 提供的默认方法。
*/

struct rsa_st {
    /*
     * The first parameter is used to pickup errors where this is passed
     * instead of aEVP_PKEY, it is set to 0
     */
    int pad;
    long version;
    const RSA_METHOD *meth;
    /* functional reference if 'meth' is ENGINE-provided */
    ENGINE *engine;
    BIGNUM *n;
    BIGNUM *e;
    BIGNUM *d;
    BIGNUM *p;
    BIGNUM *q;
    BIGNUM *dmp1;
    BIGNUM *dmq1;
    BIGNUM *iqmp;
    /* be careful using this if the RSA structure is shared */
    CRYPTO_EX_DATA ex_data;
    int references;
    int flags;
    /* Used to cache montgomery values */
    BN_MONT_CTX *_method_mod_n;
    BN_MONT_CTX *_method_mod_p;
    BN_MONT_CTX *_method_mod_q;
    /*
     * all BIGNUM values are actually in the following data, if it is not
     * NULL
     */
    char *bignum_data;
    BN_BLINDING *blinding;
    BN_BLINDING *mt_blinding;
};
/* 参数说明（edony） */
/*
 * meth：RSA_METHOD 结构，指明了本 RSA 密钥的各种运算函数地址；
* engine：硬件引擎；
* n，e，d，p，q，dmp1，dmq1，iqmp：RSA 密钥的各个值；
* ex_data：扩展数据结构，用于存放用户数据；
* references：RSA 结构引用数
*/
```

主要函数：
```c
RSA_check_key	//检查 RSA 密钥。
RSA_new	//生成一个 RSA 密钥结构，并采用默认的 rsa_pkcs1_eay_meth RSA_METHOD 方法。
RSA_free	//释放 RSA 结构。
RSA_generate_key	//生成 RSA 密钥，bits 是模数比特数，e_value 是公钥指数 e，callback 回调函数由用户实现，用于干预密钥生成过程中的一些运算，可为空。
RSA_get_default_method	//获取默认的 RSA_METHOD，为 rsa_pkcs1_eay_meth。
RSA_get_ex_data	//获取扩展数据。
RSA_get_method	//获取 RSA 结构的 RSA_METHOD。
/*各种填充方式函数。*/
RSA_padding_add_none
RSA_padding_add_PKCS1_OAEP
RSA_padding_add_PKCS1_type_1//（私钥加密的填充）
RSA_padding_add_PKCS1_type_2//（公钥加密的填充）
RSA_padding_add_SSLv23
/*各种去除填充函数。*/
RSA_padding_check_none
RSA_padding_check_PKCS1_OAEP
RSA_padding_check_PKCS1_type_1
RSA_padding_check_PKCS1_type_2
RSA_padding_check_SSLv23
RSA_PKCS1_SSLeay
RSA_print	//将 RSA 信息输出到 BIO 中，off 为输出信息在 BIO 中的偏移量，比如是屏幕BIO，则表示打印信息的位置离左边屏幕边缘的距离。
RSA_print_fp	//将 RSA 信息输出到 FILE 中，off 为输出偏移量。
RSA_public_decrypt	//RSA 公钥解密。
RSA_public_encrypt	//RSA 公钥加密。
RSA_set_default_method/ RSA_set_method	//设置 RSA 结构中的 method，当用户实现了一个 RSA_METHOD 时，调用此函数来设置，使 RSA 运算采用用户的方法。
RSA_set_ex_data	//设置扩展数据。
RSA_sign	//RSA 签名。
RSA_sign_ASN1_OCTET_STRING	//另外一种RSA签 名，不涉及摘要算法，它将输入数据作为ASN1_OCTET_STRING进行DER编码，然后直接调用RSA_private_encrypt进行计算。
RSA_size	//获取 RSA 密钥长度字节数。
RSA_up_ref	//给 RSA 密钥增加一个引用。
RSA_verify	//RSA 验证。
RSA_verify_ASN1_OCTET_STRING	//另一种 RSA 验证，不涉及摘要算法，与 RSA_sign_ASN1_OCTET_STRING 对应。
RSAPrivateKey_asn1_meth	//获取 RSA 私钥的 ASN1_METHOD，包括 i2d、d2i、new 和 free 函数地址。
RSAPrivateKey_dup	//复制 RSA 私钥。
RSAPublicKey_dup	//复制 RSA 公钥。
```

#### SSL源码分析之数据结构
SSL协议源码位于`openssl/ssl`目录下。它实现了sslv2、sslv3、TLS以及DTLS（Datagram TLS，基于UDP的TLS实现）。ssl实现中，对于每个协议，都有客户端实现(`XXX_clnt.c`)、服务端实现(`XXX_srvr.c`)、加密实现(`XXX_enc.c`)、记录协议实现(`XXX_pkt.c`)、METHOD方法(`XXX_meth.c`)、客户端服务端都用到的握手方法实现(`XXX_both.c`)，以及对外提供的函数实现(`XXX_lib.c`)，比较有规律。

ssl的主要数据结构定义在`ssl.h`中。主要的数据结构有`SSL_CTX`、`SSL`和`SSL_SESSION`。`SSL_CTX`数据结构主要用于SSL握手前的环境准备，设置CA文件和目录、设置SSL握手中的证书文件和私钥、设置协议版本以及其他一些SSL握手时的选项。`SSL`数据结构主要用于 SSL握手以及传送应用数据。`SSL_SESSION`中保存了主密钥、session id、读写加解密钥、读写MAC密钥等信息。`SSL_CTX`中缓存了所有`SSL_SESSION`信息，`SSL`中包含`SSL_CTX`。一般`SSL_CTX`的初始化在程序最开始调用，然后再生成`SSL`数据结构。由于`SSL_CTX`中缓存了所有的SESSION，新生成的SSL结构又包含SSL_CTX数据，所以通过SSL数据结构能查找以前用过的SESSION id，实现SESSION重用。
```c
typedef struct ssl_st *ssl_crock_st;
typedef struct tls_session_ticket_ext_st TLS_SESSION_TICKET_EXT;
typedef struct ssl_method_st SSL_METHOD;
typedef struct ssl_cipher_st SSL_CIPHER;
typedef struct ssl_session_st SSL_SESSION;
typedef struct tls_sigalgs_st TLS_SIGALGS;
typedef struct ssl_conf_ctx_st SSL_CONF_CTX;
```

##### SSL源码分析之cipher suites
SSL/TLS中将各类基础加密算法组成cipher suites ，一个cipher suties包括：authentication算法；encryption算法；message authentication code算法；key exchange算法。每个cipher suites用一个2字节的数字标识。OpenSSL的实现总弃用了很多不安全的cipher suites，可以通过命令openssl ciphers -V查看。authentication算法常见的有RSA/DSA/ECDSA3种，目前最主流的是人民群众喜闻乐见，妇孺皆知的RSA ( 2048 bit及以上)，（ECDSA是新兴趋势，例如gmail，facebook都在迁移到ECDSA，当然目前用的还不多，DSA由于只能提供1024bit，已经没啥人敢用）。
encryption算法主流趋势是使用AES，128/256 bit都可以，加密模式的趋势是使用gcm，cbc由于被发现有BEAST攻击等，比较难以正确使用，至于ecb模式，请勿使用。加密算法还有RC4（不建议使用），3DES（不建议使用），Camellia(貌似日本人搞的) ，DES(已经被淘汰)等，
message authentication code (消息认证码简称MAC)算法 ，主流有sha256,sha384,sha1,等。tls中使用了HMAC模式，而不是原始的sha256,sha1等。google已经在淘汰MD5了。（gcm是一种特殊的称为aead的加密模式，不需要配合MAC。）
key exchange算法主流有两种：DH和ECDH，自从斯诺登爆料了NSA的https破解方案以后，现在的 key exchange(密钥交换)算法，普遍流行PFS，把DH，ECDH变成DHE，ECDHE 。
对于SSL/TLS的cipher suite选择，Mozilla的优先级选择考虑：
1. ECDHE+AESGCM最先选，目前没有已知漏洞。
2. PFS ciphersuite优先，其中ECDHE优先于DHE。
3. SHA256优先于SHA1。完全禁用MD5。
4. AES 128优先于AES 256。这个问题有一些讨论。
5. 在向后兼容模式中，AES优先于3DES。
6. 完全禁止RC4。3DES只用于兼容老版本。
OpenSSL的cipher suites在s3_lib.c里面的ssl3_ciphers数组中定义，所有的cipher suites均在此给出定义。
```c
const char ssl3_version_str[] = "SSLv3" OPENSSL_VERSION_PTEXT;

#define SSL3_NUM_CIPHERS        (sizeof(ssl3_ciphers)/sizeof(SSL_CIPHER))

/* list of available SSLv3 ciphers (sorted by id) */
OPENSSL_GLOBAL SSL_CIPHER ssl3_ciphers[] = {

/* The RSA ciphers */
/* Cipher 01 */
    {
     1,
     SSL3_TXT_RSA_NULL_MD5,
     SSL3_CK_RSA_NULL_MD5,
     SSL_kRSA,
     SSL_aRSA,
     SSL_eNULL,
     SSL_MD5,
     SSL_SSLV3,
     SSL_NOT_EXP | SSL_STRONG_NONE,
     SSL_HANDSHAKE_MAC_DEFAULT | TLS1_PRF,
     0,
     0,
     },

/* Cipher 02 */
    {
     1,
     SSL3_TXT_RSA_NULL_SHA,
     SSL3_CK_RSA_NULL_SHA,
     SSL_kRSA,
     SSL_aRSA,
     SSL_eNULL,
     SSL_SHA1,
     SSL_SSLV3,
     SSL_NOT_EXP | SSL_STRONG_NONE | SSL_FIPS,
     SSL_HANDSHAKE_MAC_DEFAULT | TLS1_PRF,
     0,
     0,
     },
```

SSL建立链接之前，客户端和服务器端用openssl函数来设置自己支持的加密套件。主要的函数有：
```c
int SSL_set_cipher_list(SSL *s,const char *str)；
int SSL_CTX_set_cipher_list(SSL_CTX *ctx, const char *str)；

// example(edony): set a cipher suites RC4 but MD5
int ret=SSL_set_cipher_list(ssl,"RC4-MD5");
```

##### SSL源码分析之key
SSL中的密钥相关信息包括：预主密钥、主密钥、读解密密钥及其iv、写加密密钥及其iv、读MAC密钥、写MAC密钥。
* 预主密钥
预主密钥是主密钥的计算来源。它由客户端生成，采用服务端的公钥加密发送给服务端。以SSLv3为例，预主密钥的生成在源代码s3_clnt.c的ssl3_send_client_key_exchange函数中，有源码如下(此处，`tmp_buf`中存放的就是预主密钥。)：
```c
            tmp_buf[0] = s->client_version >> 8;
            tmp_buf[1] = s->client_version & 0xff;
            if (RAND_bytes(&(tmp_buf[2]), sizeof tmp_buf - 2) <= 0)
                goto err;

            s->session->master_key_length = sizeof tmp_buf;

            q = p;
            /* Fix buf for TLS and beyond */
            if (s->version > SSL3_VERSION)
                p += 2;
            n = RSA_public_encrypt(sizeof tmp_buf,
                                   tmp_buf, p, rsa, RSA_PKCS1_PADDING);
```
* 主密钥
主密钥分别由客户端和服务端根据预主密钥、客户端随机数和服务端随机数来生成，他们的主密钥是相同的。主密钥用于生成各种密钥信息，它存放在SESSION数据结构中。由于协议版本不同，生成方式也不同。SSLv3的源代码中，它通过`ssl3_generate_master_secret`函数生成，TLSv1中它通过`tls1_generate_master_secret`函数来生成。
* 对称密钥和MAC密钥
对称密钥（包括IV）和读写MAC密钥通过主密钥、客户端随机数和服务端随机数来生成。 SSLv3源代码中，它们在`ssl3_generate_key_block`中生成，在`ssl3_change_cipher_state`中分配。

##### SSL源码分析之SESSION
当客户端和服务端在握手中新建了session，服务端生成一个session ID，通过哈希表缓存 SESSION信息，并通过server hello消息发送给客户端。此ID是一个随机数，SSLv2版本时长度为16字节，SSLv3和TLSv1长度为32字节。此ID与安全无关，但是在服务端必须是唯一的。当需要session重用时，客户端发送包含session ID的clientHello消息（无sesion重用时，此值为空）给服务端，服务端可用根据此ID来查询缓存。session重用可以免去诸多SSL握手交互，特别是客户端的公钥加密和服务端的私钥解密所带来的性能开销。session的默认超时时间为 60*5+4秒，5分钟。
session相关函数有：
```c
int SSL_has_matching_session_id(const SSL *ssl, const unsigned char * id,unsigned int id_len)
/*
 * SSL 中查询 session id，id 和 id_len 为输入的要查询的 session id，查询哈希表
 * ssl->ctx->sessions，如果匹配，返回 1，否则返回 0。
 */
 
int ssl_get_new_session(SSL *s, int session)
/*
 * 生成 ssl 用的 session，此函数可用被服务端或客户端调用，当服务端调用时，
 * 传入参数 session 为 1，生成新的 session；当客户端调用时，传入参数 session 为 0，
 * 只是简单的将 session id 的长度设为 0。
 */
 
int ssl_get_prev_session(SSL *s, unsigned char *session_id, int len)
/*
 * 获取以前用过的 session id，用于服务端 session 重用，本函数由服务端调用，
 * session_id 为输入 senssion ID 首地址，len 为其长度，如果返回 1，表明要 session
 * 重用；返回 0，表示没有找到；返回-1 表示错误。
 */
 
int SSL_set_session(SSL *s, SSL_SESSION *session)
/*
 * 设置 session，本函数用于客户端，用于设置 session 信息；如果输入参数 session
 * 为空值，它将置空 s->session；如果不为空，它将输入信息作为 session 信息。
 */
 
void SSL_CTX_flush_sessions(SSL_CTX *s, long t)
/*
 * 清除超时的 SESSION，输入参数 t 指定一个时间，如果 t=0,则清除所有
 * SESSION，一般用 time(NULL)取当前时间。此函数调用了哈希表函数 lh_doall_arg
 * 来处理每一个 SESSION 数据。
 */

int ssl_clear_bad_session(SSL *s)
/*
 * 清除无效 SESSION。
 */
```

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
