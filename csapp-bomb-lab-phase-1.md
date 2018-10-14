# 菜鸟做 bomb lab 之第一关

第一题比较简单，但本菜鸡也做了两个小时(╯‵□′)╯︵┻━┻。。。

首先打开事先已经反汇编的 bomb.s 文件，通过 `bomb.c` 已经知道每一关都是一个函数，它们的命名都是 `phase_x`，x 代表该关卡的数字，如果某个关卡输入的不正确，就会引爆炸弹 `explode_bomb`。首先看 main 函数的这几行

```asm
400e1e:   bf 38 23 40 00          mov    $0x402338,%edi
400e23:   e8 e8 fc ff ff          callq  400b10 <puts@plt>
400e28:   bf 78 23 40 00          mov    $0x402378,%edi
400e2d:   e8 de fc ff ff          callq  400b10 <puts@plt>
400e32:   e8 67 06 00 00          callq  40149e <read_line>
400e37:   48 89 c7                mov    %rax,%rdi
400e3a:   e8 a1 00 00 00          callq  400ee0 <phase_1>
400e3f:   e8 80 07 00 00          callq  4015c4 <phase_defused>
400e44:   bf a8 23 40 00          mov    $0x4023a8,%edi
```

打开 gdb，先给这一行打上断点 `break *0x400e23`，然后 `run` 起来。这里可以看到调用了 `puts` 这个函数，寄存器 `%edi` 存储的是函数的第一个参数，我们把它的结果打印出来 `x/s 0x402338`、`x/s 0x402378`，发现得到了运行 bomb 后输出的字符串。说明第一关就是从这里开始的。

由于返回值是存在 `%rax` 中的，这里 `mov %rax %rdi`，说明输入的内容传参给了 `phase_1`。在 gdb 里给 phase_1 打断点 `break phase_1`。

```asm
0000000000400ee0 <phase_1>:
    400ee0:   48 83 ec 08             sub    $0x8,%rsp
    400ee4:   be 00 24 40 00          mov    $0x402400,%esi
    400ee9:   e8 4a 04 00 00          callq  401338 <strings_not_equal>
    400eee:   85 c0                   test   %eax,%eax
    400ef0:   74 05                   je     400ef7 <phase_1+0x17>
    400ef2:   e8 43 05 00 00          callq  40143a <explode_bomb>
    400ef7:   48 83 c4 08             add    $0x8,%rsp
    400efb:   c3                      retq
```

通过这里的代码，就可以分析出来，通过调用 `string_not_equal` 比较输入的字符串与 `0x402400` 存储的字符串是否相等，来决定是不是 `explode_bomb`。通过这个函数名也可以知道一定要输入与 `0x402400` 相同的字符串就可以通过第一关了。所以在这里打个断点 `break *0x400ee9`，然后 `x/s 0x402400` 打印出来这里的字符串，我这里是 `Border relations with Canada have never been better.`，然后输入这个字符串，第一关就过了~

## string_not_equal

虽然这样就过关了，但是我还是对这里的代码好奇，毕竟是学习嘛，看看这里的代码熟悉熟悉汇编。

```asm
0000000000401338 <strings_not_equal>:
  401338:   41 54                   push   %r12
  40133a:   55                      push   %rbp
  40133b:   53                      push   %rbx
  40133c:   48 89 fb                mov    %rdi,%rbx
  40133f:   48 89 f5                mov    %rsi,%rbp
  401342:   e8 d4 ff ff ff          callq  40131b <string_length>
  401347:   41 89 c4                mov    %eax,%r12d
  40134a:   48 89 ef                mov    %rbp,%rdi
  40134d:   e8 c9 ff ff ff          callq  40131b <string_length>
  401352:   ba 01 00 00 00          mov    $0x1,%edx
  401357:   41 39 c4                cmp    %eax,%r12d
  40135a:   75 3f                   jne    40139b <strings_not_equal+0x63>
  40135c:   0f b6 03                movzbl (%rbx),%eax
  40135f:   84 c0                   test   %al,%al
  401361:   74 25                   je     401388 <strings_not_equal+0x50>
  401363:   3a 45 00                cmp    0x0(%rbp),%al
  401366:   74 0a                   je     401372 <strings_not_equal+0x3a>
  401368:   eb 25                   jmp    40138f <strings_not_equal+0x57>
  40136a:   3a 45 00                cmp    0x0(%rbp),%al
  40136d:   0f 1f 00                nopl   (%rax)
  401370:   75 24                   jne    401396 <strings_not_equal+0x5e>
  401372:   48 83 c3 01             add    $0x1,%rbx
  401376:   48 83 c5 01             add    $0x1,%rbp
  40137a:   0f b6 03                movzbl (%rbx),%eax
  40137d:   84 c0                   test   %al,%al
  40137f:   75 e9                   jne    40136a <strings_not_equal+0x32>
  401381:   ba 00 00 00 00          mov    $0x0,%edx
  401386:   eb 13                   jmp    40139b <strings_not_equal+0x63>
  401388:   ba 00 00 00 00          mov    $0x0,%edx
  40138d:   eb 0c                   jmp    40139b <strings_not_equal+0x63>
  40138f:   ba 01 00 00 00          mov    $0x1,%edx
  401394:   eb 05                   jmp    40139b <strings_not_equal+0x63>
  401396:   ba 01 00 00 00          mov    $0x1,%edx
  40139b:   89 d0                   mov    %edx,%eax
  40139d:   5b                      pop    %rbx
  40139e:   5d                      pop    %rbp
  40139f:   41 5c                   pop    %r12
  4013a1:   c3                      retq
```

看代码，发现很符合书上讲的，`%r12`、`%rbp`、`%rbx` 都是被调用者保存的寄存器。

首先 0x401342 ~ 0x40135a，判断了它们的长度是不是相同，如果长度不相同，那么它们必然不是同一个字符串。`mov    $0x1,%edx` 和 `mov    %edx,%eax` 返回了 1。

0x40135c ~ 0x401361 这几行，判断了所输入的字符串的第一个字符是不是 `\0`。因为走到这条命令，已经判断过长度是相同的了，如果其中的一个字符串的首字符是 `\0`，那么另外一个必然是一样的（所有的字符串一定都包含一个 `\0`），所以这里直接就返回 0。

0x401361 ~ 0x40137f 是一个循环，它遍历了两个字符串，每一个字符是不是相同的，直到遇到 `\0`。

## string_length

```asm
000000000040131b <string_length>:
  40131b:   80 3f 00                cmpb   $0x0,(%rdi)
  40131e:   74 12                   je     401332 <string_length+0x17>
  401320:   48 89 fa                mov    %rdi,%rdx
  401323:   48 83 c2 01             add    $0x1,%rdx
  401327:   89 d0                   mov    %edx,%eax
  401329:   29 f8                   sub    %edi,%eax
  40132b:   80 3a 00                cmpb   $0x0,(%rdx)
  40132e:   75 f3                   jne    401323 <string_length+0x8>
  401330:   f3 c3                   repz retq
  401332:   b8 00 00 00 00          mov    $0x0,%eax
  401337:   c3                      retq
```

这个函数就比较简单了，其实就是找到 `\0` 的位置，然后返回其余首地址的差，即长度。这个翻译成 C 语言可以这么写。

```c
int string_length(char *s)
{
    char *b = a;
    
    while (*b != 0)
        b = b + 1;
    
    return (int) (b - a);
}
```

> 本文是作者在看《深入理解计算机系统》以及完成 bomb lab 时的理解与总结，谨此记录下来已被日后翻阅。同时，也分享给各位希望了解这些知识的同道者们。由于作者水平有限，如有错误之处，望不吝赐教，深表感谢。

