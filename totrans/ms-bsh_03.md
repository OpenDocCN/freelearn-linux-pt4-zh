# 测试

到目前为止，我们强调了给脚本提供结构的重要性，使它们更具灵活性，并让它们响应一些条件和情况，以便帮助我们自动化某些日常任务，做出决策并代表我们执行操作。我们在前几章中所看到的使我们能够赋值变量，以不同的方式更改它们的值，并且还能够保存它们；但从展示的例子来看，我们需要更多的功能，而这正是本章的主题。我们将学习如何进行测试，做比较，并根据结果做出响应，我们将为脚本赋予第一个结构，当某些事情发生时做出决策。

# 如果...否则

让我们以之前的一个例子为例，详细分析：

```
#!/bin/bash    
echo "Hello user, please give me a number between 10 and 20: "
read user_input
if [ ${user_input} -ge 10 ] && [ ${user_input} -le 20 ]
then
echo "Great! The number ${user_input} is what we were looking for!"
else
echo "The number ${user_input} is not what we are looking for..."
fi  

```

作为一个练习，帮助理解，让我们尝试将其写成自然语言：

1.  打印一个问候语，要求输入一个 10 到 20 之间的数字

1.  读取用户输入并将其保存在 `user_input` 变量中

1.  如果 `user_input` 的值大于或等于 `10` 且 `user_input` 的值小于或等于 `2`，则打印一条 OK 消息给用户

1.  否则（else），如果条件不满足，打印一条不 OK 的消息

1.  `Fi` 条件结束

这些是条件语句的基础，它让你根据条件进行探索：如果成功，执行一条指令；如果失败，则调用另一块指令。我们还可以使它更灵活，介绍一个备用条件，在第一个条件失败时进行检查（`elif`）：

+   `if` 测试条件的退出代码是 0

+   然后

+   执行某个操作

+   `elif` 这个其他测试条件的退出代码是 0

+   执行某个操作

+   `else if` 之前的任何条件返回 0

+   执行某个操作

+   `fi` 我们退出条件语句

所以在这种更复杂的形式下，条件提供了更多的灵活性，记住，你可以有尽可能多的 `elif` 块，甚至将 `if` 嵌套在 `if` 中，尽管为了清晰起见不推荐这么做。现在，让我们通过返回代码来举一个实际生活中的例子，从创建三个 `test` 文件开始：

```
zarrelli:~$ touch test1 test2 test3  

```

现在，让我们创建一个小脚本，检查这三个文件是否存在：

```
#!/bin/sh    
echo "We are going to test for files test1 test2 test3"    
if ls test1
then
echo "File test1 exists so the ls test1 execution returns $?"
elif ls test2
then
echo "File test2 exists so the ls test2 execution returns $?"
elif ls test3
then
echo "File tes3 exists so the ls test3 execution returns $?"
else
echo "Neither or test1 or test2 or test3 exist so the the exit 
code is $?"
fi
echo "End of the script"  

```

现在，让我们运行它并看看发生了什么：

```
zarrelli:~$ ./test-files.sh 
We are going to test for files test1 test2 test3
test1
File test1 exists so the ls test1 execution returns 0
End of the script 

```

发生了什么？第一次对 `test1` 执行 `ls` 返回了 `0`，所以它成功了，条件语句没有继续测试其他选项，而是退出了语句，执行了条件外的下一条指令，这就是：

```
echo "End of the script"  

```

现在是时候看看如果遇到第一个条件失败时会发生什么，因此我们将删除 `test1` 文件：

```
rm test1  

```

然后再次执行脚本：

```
zarrelli:~$ ./test-files.sh 
We are going to test for files test1 test2 test3
ls: cannot access 'test1': No such file or directory
test2
File test2 exists so the ls test2 execution retuns 0
End of the script  

```

还是发生了什么？第一条指令`ls test1`失败，因为没有`test1`文件可以用`ls`显示，因此该指令返回`1`。脚本然后进入条件语句的第二个条件，执行`ls test2`。在这种情况下，由于`file2`存在，命令返回`0`，脚本跳出了条件语句，执行了条件语句外的第一条指令：

```
echo "End of the script"  

```

让我们继续，删除`test2`：

```
rm test1  

```

然后调用脚本：

```
zarrelli:~$ ./test-files.sh 
We are going to test for files test1 test2 test3
ls: cannot access 'test1': No such file or directory
ls: cannot access 'test2': No such file or directory
test3
File test3 exists so the ls test3 execution retuns 0
End of the script  

```

由于`test1`和`test2`不存在，前两个`ls`失败，因此前两个条件也失败，但第三个`ls`没有失败，因为`test3`仍然存在。第三个`ls`成功返回`0`，脚本退出条件语句，重新执行了条件语句外的第一条指令：

```
echo "End of the script"  

```

最后一次测试，是时候删除`test3`了：

```
rm test3  

```

并执行脚本：

```
zarrelli:$ ./test-files.sh 
We are going to test for files test1 test2 test3
ls: cannot access 'test1': No such file or directory
ls: cannot access 'test2': No such file or directory
ls: cannot access 'test3': No such file or directory
Neither or test1 or test2 or test3 exist so the the exit code is 2
End of the script  

```

现在应该清楚发生了什么。所有的`if...then`条件都失败了，因此最后的手段是`else`部分，它报告`lstest3`的退出代码。完成后，脚本退出条件语句，执行了条件语句外的第一条指令，重复如下：

```
echo "End of the script"  

```

请注意，条件语句的总体退出状态是最后执行的指令的退出状态，而脚本的总体退出代码是脚本本身执行的最后一条指令的退出状态：

```
zarrelli:~$ ./test-files.sh ; echo $?
We are going to test for files test1 test2 test3
ls: cannot access 'test1': No such file or directory
ls: cannot access 'test2': No such file or directory
ls: cannot access 'test3': No such file or directory
Neither or test1 or test2 or test3 exist so the the exit code is 2
End of the script
0  

```

我们看到这里脚本返回了`0`，这是正确的，因为最后执行的指令`echo End of the script`成功了。现在让我们将脚本的最后一条指令更改为：

```
else
:  

```

冒号实际上意味着*什么也不做*，让我们看看：

```
zarrelli$ ./test-files.sh ; echo $?
We are going to test for files test1 test2 test3
ls: cannot access 'test1': No such file or directory
ls: cannot access 'test2': No such file or directory
ls: cannot access 'test3': No such file or directory
0  

```

还是`0`。现在，让我们进行一次逆向检查，修改第三个条件，添加一个`!`。

```
elif !ls test3
then
echo "File test3 exists so the ls test3 execution retuns $?"  

```

所以，如果`ls test3`返回`1`，则检查成功：

```
zarrelli:~./test-files-not.sh ; echo $?
We are going to test for files test1 test2 test3
ls: cannot access 'test1': No such file or directory
ls: cannot access 'test2': No such file or directory
ls: cannot access 'test3': No such file or directory
File test3 exists so the ls test3 execution retuns 0
0  

```

好吧，打印的消息其实是一个迷惑性信息，因为`ls test3`的执行不成功，不能返回`0`：

```
zarrelli:~$ ls test3 ; echo $?
ls: cannot access 'test3': No such file or directory
2  

```

实际上返回`0`的是我们对反向条件所做的检查：

```
elif !ls file3  

```

它可以理解为，只有当`ls file3`没有通过时，`if`条件才会被验证。因此，对于我们来说，验证是成功的，成功由返回值`0`表示，条件只有在`if ls file3`未通过时才会验证（`-ne 0`）。因此，在使用这样的条件时要小心，因为你可能会遇到一些意想不到的结果。

我们刚刚看到如何逐一检查条件，但我们可以在单个检查中结合运算符，以便获得更有趣的结果。看看下面的脚本：

```
#!/bin/bash    
echo "Hello user, please give me a number between 10 and 20, 
it must be even: "
read user_input
if [[ ${user_input} -ge 10 && ${user_input} -le 20 && 
$(( $user_input % 2 )) -eq 0 ]]
then
echo "Great! The number ${user_input} is what we were looking for!"
else
echo "The number ${user_input} is not what we are looking for..."
fi  

```

我们在这里做的是同时测试三个不同的条件，因此只有当用户输入一个`10`到`20`之间的数字且必须是偶数时，`if`才会被验证。换句话说，它必须能被 2 整除，我们通过检查该值的模数是否为`0`来测试它。让我们尝试一些值：

```
zarrelli:~$ ./userinput-and.sh 
Hello user, please give me a number between 10 and 20, it 
must be even: 
8
The number 8 is not what we are looking for...  

```

这个数字满足第三个和第二个条件，因为它是偶数且小于 `20`，但是不满足第一个条件，因为它不等于或大于 `10`。因此，`if` 条件不成立，触发了 `else` 操作：

```
zarrelli:~$ ./userinput-and.sh 
Hello user, please give me a number between 10 and 20, it 
must be even: 
9
The number 9 is not what we are looking for...  

```

现在，这个数字不满足第一个和第三个条件，但满足第二个条件，它不等于或大于 `10`，它不是偶数，但小于 `20`，因此触发了 `else` 块的操作。

```
zarrelli:~$ ./userinput-and.sh 
Hello user, please give me a number between 10 and 20, it 
must be even: 
10
Great! The number 10 is what we were looking for!  

```

数字 `10` 是合适的。它满足第一个和第二个条件，因为它等于 `10` 且小于 `20`，并且满足第三个条件，因为它是偶数，因此触发了 `then` 块的操作。

```
zarrelli:~$ ./userinput-and.sh 
Hello user, please give me a number between 10 and 20, it 
must be even: 
15
The number 15 is not what we are looking for...  

```

在这种情况下，第一个和第二个条件被验证，但第三个条件没有。`15` 不是偶数，因此触发了 `else` 块的操作：

```
zarrelli:~$ ./userinput-and.sh
Hello user, please give me a number between 10 and 20, it 
must be even: 
20 Great! The number 20 is what we were looking for!  

```

`20` 是合适的，它大于 `10`，等于 `20`，并且可以被 `2` 整除，因此所有三个条件都被验证，触发了 `if` 块的操作：

```
zarrelli:~$ ./userinput-and.sh
Hello user, please give me a number between 10 and 20, it 
must be even: 
21
The number 21 is not what we are looking for...  

```

这个数字满足第一个条件，即大于 `10`，但不满足其他两个条件，因为它不是等于或小于 `20`，也不是偶数。因此触发了 `else` 块的操作：

```
zarrelli:~$ ./userinput-and.sh
Hello user, please give me a number between 10 and 20, it 
must be even: 
22
The number 22 is not what we are looking for...  

```

甚至这个数字也不合适。它满足第一个条件，即大于 `10`，但不满足第二个条件，因为它不是等于或小于 `20`。第三个条件满足，但这还不够。所以触发了 `else` 块的操作。

如我们所见，当处理多个条件时，我们必须非常小心我们写的内容并考虑结果，因为有时我们可能会得到我们实际上并不想要的结果。一个经验法则是，尽量保持条件的简单，或者花时间彻底检查它们。我们说简单吗？看看这个：

```
#!/bin/bash
echo "Hello user, please give me an even number: "
read user_input
if ! (( $user_input % 2 ))
then
echo "Great! The number ${user_input} is what we were looking for!"
fi  

```

我们正在使用一个算术评估复合命令，并对其进行否定以检查一个数字是否为偶数：如果模运算不失败，那么条件就成立。但看， 我们没有 `else` 块，我们只是评估了 `if` 条件并退出了条件语句，因为我们不关心对其他情况作出反应。这是典型的例子，比如计数器退出条件，就像我们之前看到的：我们希望当计数器达到特定值时退出循环，否则就让循环继续运行。让我们看看结果：

```
zarrelli:~$ ./userinput-and-simple.sh
Hello user, please give me an even number: 
20
Great! The number 2 is what we were looking for!
The modulo of 20 by 2 is 0:
zarrelli:~$ a=$((20 % 2 )) ; echo ${a}
0  

```

模运算没有给出结果。看看这个操作的返回代码：

```
zarrelli:~/$ ((20 % 2 )) ; echo $? 1 

```

结果就是这样，返回代码是 `1`，这意味着对我们来说不合适。所以，如果一个数字通过模运算并得到 `0`，返回代码就是失败，这意味着将该数字除以二没有余数。这一切意味着，如果一个数字能被 `2` 整除；例如，它不会给我们除法的余数，而且它是偶数。现在，让我们试试一个奇数：

```
zarrelli:~$ ./userinput-and-simple.shsimple.sh
Hello user, please give me an even number: 
25  

```

没有触发 `then` 块的操作，因为这是一个奇数。让我们验证一下：

```
zarrelli:~$ ((25 % 2 )) ; echo $?
0  

```

操作成功，所以我们必须得到一些余数。再检查一下：

```
zarrelli:~$ a=25 ; b=2 ; c=$((a/b)) ; echo ${c}
12  

```

我们有 `12` 作为 `25` 除以 `2` 的余数。所以现在我们在之前的脚本中看到的条件更加清晰了：

```
if ! (( $user_input % 2 ))  

```

`if` 语句会在模除二的算术操作失败时被满足。所以，如果一个数不能被 `2` 整除，它就是偶数，简单明了。现在是时候看看我们如何测试我们的条件了。

# 测试命令回顾

正如我们在之前的一些例子中所看到的，我们使用了 shell 的 `built-in` 测试来对变量和文件进行一些检查，并结合条件语句 `if...then` 使脚本能对条件作出反应：如果测试成功，它返回 `0`，如果失败，返回 `1`，这些值触发了我们到目前为止的反应。

我们可以使用几种不同的符号来执行测试，我们已经看过这些：

```
[expression]  

```

或者

```
[[expression]]  

```

我们已经讨论了这两者之间的区别，但在继续之前，让我们快速回顾一下：

+   单括号实现了标准的符合 POSIX 的 `test` 命令，并且在所有 POSIX shell 中都可用。`[` 实际上是一个命令，它的参数是 `]`，这会防止单括号接收更多的参数。

    +   一些 Linux 版本仍然有 `/bin/[` 命令，但 `built-in` 版本在执行时具有优先权。

+   双括号只在 Bash、`zsh` 和 `korn` shell 中可用。

+   双括号是一个关键字，不是一个程序，从 Bash 的 2.02 版本开始提供，并且具有一些很棒的功能，如下所示：

+   +   用于正则表达式匹配的 `=~` 操作符

    +   `=` 或 `==` 可以用于模式匹配。

    +   你可以使用 `<>` 而无需转义 `\> \<`。

    +   你可以使用 `&&` 替代 `-a`，使用 `||` 替代 `-o`。

    +   你不需要转义括号 `\( \)` 来分组表达式。

    +   通配符扩展，所以 `*` 可以扩展为任何内容，这在模式匹配中非常有用。

    +   你不必引用变量以确保变量内部的空格安全。

所以，看起来双括号给我们比旧命令多了一些灵活性，但在广泛使用之前，请考虑脚本的受众。如果你希望它们在不同的 shell 之间共享，并且在同一个 Bash 中在不同版本之间共享，尽量避免使用只有部分版本可用的命令或内建命令。遵循 POSIX 标准会让你的脚本更加可共享，但作为缺点，它们缺乏一些关键字（如双括号）所提供的高级功能。因此，要明智地平衡你的写作风格，并采用最适合你目标的策略。我们在可能的情况下，会使用单括号符号，以确保尽可能的兼容性。

# 测试文件

关于测试有很多要说的，其中最常见的任务之一是检查文件系统中的文件或目录是否可用或具有某些权限。因此，想象一下一个需要在目录中写入一些数据的脚本：首先，我们应该检查目录是否存在，然后是否可以向其写入，最后检查我们将要打开以写入的文件与已存在的文件之间是否存在名称冲突。让我们看看我们用来执行一些文件和设备上测试的操作符，并记住它们在条件满足时返回 `true`：

+   `-e`: 如果文件存在，则返回 true：

```
zarrelli:~$ ls test-files.sh 
test-files.sh  

```

我们刚刚验证了文件 `test-files.sh` 的存在，因为 `ls` 显示了它：

```
zarrelli:~$ if [ -e test-files.sh ] ; then 
echo "Yes, this is a file!" ; fi
Yes, this is a file device!  

```

我们的测试确认了这一点，并显示了一个良好的消息。

现在让我们验证一下，我们当前目录中没有名为 `aaaaa` 的文件：

```
zarrelli:~$ lsaaaaa
ls: cannot access 'aaaaa': No such file or directory 

```

好的，没有这样名称的文件；让我们进行测试：

```
zarrelli:~$ if [ -e aaaaa ] ; then echo "Yes, this is a file!"; 
else 
echo "There is not such a file!" ; fi
There is not such a file!  

```

好吧，正如您所见，我们使用分号来分隔语句的不同部分。在脚本中，我们会看到以下内容：

```
#!/bin/bash
if [ -e aaaaa ]
then
echo "Yes, this is a file!"
else
echo "There is not such a file!"
fi  

```

每个单独的命令必须适当地以新行或 `;` 终止。在 `;` 分隔的代码块之前执行每个命令，而无需新行。

+   `-a`: 具有与 `-e` 相同的目的，但已弃用。

+   `-b`: 这检查文件是否实际上是块设备，例如磁盘、CD-ROM 或磁带设备：

```
zarrelli:~$ if [ -b /dev/nvme0n1p1 ] ; 
then 
echo "Yes, this is a block device!" ; fi
Yes, this is a block device!  

```

+   `-d`: 这检查文件是否实际上是一个目录：

```
zarrelli:~$ if [ -d test ] ; 
then echo "Yes, this is a directory!" ; 
else echo "There is not such a directory!" ; fi
Yes, this is a directory!  

```

+   `-f`: 检查文件是否为常规文件，而不是类似字符设备、目录或块设备的东西：

```
zarrelli:~$ if [ -f /dev/tty7 ] ; 
then echo "Yes, this is a regular file!" ; 
else echo "There is not a regular file!" ; fi
There is not a regular file!  

```

好的，这是一个代表终端的文件，所以显然不是像 `test.file` 可能是的常规文件：

```
zarrelli:~$ touch test.file    
zarrelli:~$ if [ -f test.file ] ; 
then echo "Yes, this is a regular file!" ; 
else echo "There is not a regular file!" ; fi
Yes, this is a regular file!  

```

+   `-c`: 测试参数是否为字符文件：

```
zarrelli:~$ if [ -c /dev/tty7 ] ; 
then echo "Yes, this is a character file!" ; 
else echo "There is not a character file!" ; fi
Yes, this is a character file!  

```

+   `-s`: 如果文件不为 `0` 大小，则为 true：

```
zarrelli:~$ if [ -s test.file ] ; 
then echo "Yes, the size of this file is not 0!" ; 
else echo "The size of this file is 0!" ; fi
The size of this file is 0!  

```

好吧，我们刚刚 *touch* 了这个文件，所以创建了一个大小为 `0` 字节的文件。让我们用一个字符填充它：

```
zarrelli:~$ echo 1 >>test.file 

```

现在，让我们重复测试：

```
zarrelli:~$ if [ -s test.file ] ; 
then echo "Yes, the size of this file is not 0!" ; 
else echo "The size of this file is 0!" ; fi
Yes, the size of this file is not 0!  

```

+   `-g`: 如果目录设置了 `sgid` 标志，则为 true。正如我们所见，设置组 ID 在目录上强制新创建的文件归属于拥有该目录的组：

```
zarrelli:~$ if [ -g test ] ; 
then echo "Yes, this dir has a sgid bit" ; 
else echo "No sgid bit on this dir" ; fi
No sgid bit on this dir  

```

现在：

```
zarrelli:~$ chmodg+s test
zarrelli:~$ if [ -g test ] ; 
then echo "Yes, this dir has a sgid bit" ; 
else echo "No sgid bit on this dir" ; fi
Yes, this dir has a sgid bit  

```

+   `-G`: 如果组 ID 与您的相同，则为 true。先在文件上进行测试：

```
zarrelli:~$ if [ -G test.file ] ; 
then echo "Yes, this file has your same group owner" ; 
else echo "No the group owner is not the same of yours" ; fi
Yes, this file has your same group owner  

```

现在针对目录：

```
zarrelli:~$ if [ -G test ] ; 
then echo "Yes, this file has your same group owner" ; 
else echo "No the group owner is not the same of yours" ; fi
Yes, this file has your same group owner  

```

让我们再次验证更改 `test.file` 的组所有者：

```
zarrelli:~$ su
Password: 
root:# chgrp root test.file
root:# ls -lahtest.file
-rw-r--r-- 1 zarrelli root 2 Feb  6 18:23 test.file
root:# exit
exit
zarrelli:~$ if [ -G test.file ] ; 
then echo "Yes, this file has your same group owner" ; 
else echo "No the group owner is not the same of yours" ; fi
No the group owner is not the same of yours  

```

+   `-O`: 如果您是所有者，则为 true：

```
zarrelli$ if [ -O test.file ] ; 
then echo "Yes, you are the owner" ; 
else echo "No you are not the owner" ; fi
Yes, you are the owner
zarrelli:~$ ls -lahtest.file
-rw-r--r-- 1 zarrellizarrelli 2 Feb  6 18:23 test.file  

```

+   `-N`: 如果文件自上次读取以来已修改，则为 true。当您想要备份文件或仅查看是否添加了新信息时，这可能会非常有用。典型的场景可能是一个由进程或服务馈送的日志文件或数据文件：如果在一定的时间内文件未被修改，则可能意味着该进程未运行或未正常工作，因此我们可以做一些像重新启动它的操作。让我们看看我们之前的某个脚本：

```
zarrelli:~$ if [ -N userinput-or.sh ] ; 
then echo "Yes, it has been modified since last read" ; 
else echo "No modifications since last read " ; fi
No modifications since last read   

```

好的，看起来文件似乎最近没有修改过，所以现在是修改它的时候了：

```
zarrelli:~$ echo 1 >> userinput-or.sh 
zarrelli:~$ if [ -N userinput-or.sh ] ; 
then echo "Yes, it has been modified since last read" ; 
else echo "No modifications since last read " ; fi
Yes, it has been modified since last read  

```

就是这样。请记住，在所有测试中，当我们说条件验证时测试为真时，我们暗示了第二个条件，即文件必须存在。所以在这种情况下，我们会说，如果文件存在且自上次读取后没有被修改，则验证通过。另外，请记住，在 Unix 中一切皆文件，包括目录。

+   `-u`：如果设置了`suid`位，则为真。这种测试可以非常有用，原因有很多，尤其是因为当你运行一个可执行文件时，它通常会以调用它的用户的权限运行。而如果设置了`suid`位，该可执行文件将以该文件所有者的权限运行，而不是以调用者的权限运行。因此，带有`suid`位的 root 拥有的程序可能会对系统安全构成严重威胁，因为调用它的任何人都将获得 root 权限。另一方面，一些程序，尤其是那些必须拥有 root 权限才能访问设备的程序，必须设置`suid`位，因为这可以允许普通用户像 root 一样访问设备，而无需访问完整的 root 环境：

```
zarrelli:$ su
Password: 
root:# chown root test.file
root:# ls -lahtest.file
-rw-r--r-- 1 root zarrelli 2 Feb  6 18:23 test.file
root:# chmod +s test.file
root:# ls -lahtest.file
-rwSr-Sr-- 1 root zarrelli 2 Feb  6 18:23 test.file
root:# exit
exit
zarrelli:~$ if [ -u test.file ] ; 
then echo "Yes, it has the suid bit flagged" ; 
else echo "No suid bit found" ; fi  

```

+   `-k`：如果设置了`sticky`位，则为真。这种权限非常有趣，因为如果它应用于文件，它会使文件保持在内存中，从而加速访问；但如果应用于目录，则限制用户权限：只有目录所有者或目录内文件的所有者，才能删除该文件。这在协作环境中非常有用，因为多个用户可以将工作文件放在同一目录中，并且对该目录应用 sticky 位后，只有文件所有者才能删除他们自己的文件：

```
zarrelli:~$ chmod +t test
zarrelli:~$ if [ -k test ] ; 
then echo "Yes, it has the sticky bit set" ; 
else echo "No sticky bit set" ; fi
Yes, it has the sticky bit set  

```

+   `-r`：如果为执行测试的用户设置了读取权限，则为真：

```
zarrelli:~$ if [ -r test.file ] ; 
then echo "Yes, this user can read the file" ; 
else echo "No this user cannot read the file" ; fi
Yes, this user can read the file  

```

所以，用户可以读取文件，让我们检查接下来的内容：

```
zarrelli$ ls -lahtest.file
-rwSr-Sr-- 1 root zarrelli 2 Feb  6 18:23 test.file  

```

哦，文件由 root 拥有，并且 root 有读取权限，那么为什么测试在 root 有读取权限的情况下成功呢？很简单：

```
zarrelli:~$ su
Password: 
root:# chmodog-r test.file
root:# exit
exit
zarrelli:~$ if [ -r test.file ] ; 
then echo "Yes, this user can read the file" ; 
else echo "No this user cannot read the file" ; fi
No this user cannot read the file  

```

发生了什么？第一次我们尝试测试时，所有者是 root，但用户`zarrelli`仍然通过组权限和其他权限能够读取文件。因此，清除这些位使得文件只能由 root 用户读取，其他人无法访问。

+   `-w`：如果写入位被设置，则为真：

```
zarrelli:~$ if [ -w test.file ] ; 
then echo "Yes, this user can write to the file" ; 
else echo "No this user cannot write to the file" ; fi
No this user cannot write to the file  

```

有趣，让我们看看文件：

```
zarrelli:~$ ls -lahtest.file
-rw---S--- 1 root zarrelli 2 Feb  6 18:23 test.file  

```

确实，只有 root 用户可以写入它。你想尝试修复问题然后再次运行测试吗？

+   `-x`：如果执行位被设置，则为真：

```
zarrelli:~$ if [ -x test.file ] ; 
then echo "Yes, this user can execute it" ; 
else echo "No this user cannot execute it" ; fi
No this user cannot execute it  

```

让我们看看文件的访问权限：

```
zarrelli$ ls -lah test.file
-rw---S--- 1 root zarrelli 2 Feb  6 18:23 test.file  

```

所以没有给用户执行位。现在，我们来测试一个目录：

```
zarrelli:~$ if [ -x test ] ; 
then echo "Yes, this user can execute it" ; 
else echo "No this user cannot execute it" ; fi
Yes, this user can execute it  

```

有趣，是时候看看目录的权限了：

```
zarrelli:~$ ls -lah test
total 8.0K
drwxr-sr-t 2 zarrellizarrelli 4.0K Feb  6 18:12 .
drwxr-xr-x 3 zarrellizarrelli 4.0K Feb  7 08:12 ..  

```

那些点和双点是什么？让我们仔细看看：

```
zarrelli$ ls -lai
total 8
5900830 drwxr-sr-t 2 zarrellizarrelli 4096 Feb  6 18:12 .
5899440 drwxr-xr-x 3 zarrellizarrelli 4096 Feb  7 08:12 ..  

```

请记住第一列。这些是与` .`和`..`相关的**inode**编号：

```
zarrelli@:~$ cd ..
zarrelli@:~$ ls -lai
total 36
5899440drwxr-xr-x 3 zarrellizarrelli 4096 Feb  7 08:12 .
5899435 drwxr-xr-x 3 zarrellizarrelli 4096 Feb  7 20:47 ..
5900830 drwxr-sr-t 2 zarrellizarrelli 4096 Feb  6 18:12 
test
5899311 -rw---S--- 1 root     zarrelli    2 Feb  6 18:23 
test.file
5899447 -rwxr--r-- 1 zarrellizarrelli  319 Feb  5 11:42 
test-files-not.sh
5899450 -rwxr--r-- 1 zarrellizarrelli  317 Feb  5 11:21 
test-files.sh
5898796 -rw-r--r-- 1 zarrellizarrelli    0 Feb  7 08:00 
test.modified
5899448 -rwxrwxr-x 1 zarrellizarrelli  352 Feb  5 12:31 
userinput-and.sh
5899449 -rwxr-xr-x 1 zarrellizarrelli  190 Feb  5 12:41 
userinput-and-simple.sh
5899444 -rwxrwxr-x 1 zarrellizarrelli  305 Feb  7 08:07 
userinput-or.sh  

```

`test` 目录中的 `.` 的 inode 值是 `5900830`，现在我们向上移动一层目录，我们可以看到 `test` 目录的 inode 值是 `5900830`。所以我们可以安全地说，`.` 指向我们所在的目录。那么 `..` 呢？看看父目录中 `.` 的值，它是 `5899440`。现在看看 `test` 目录中 `..` 的值，它是 `5899440`，因此我们可以安全地说，`..` 指向父目录，因为两者都指向相同的 inode。

简单来说，要理解 inode 与文件之间的关系，我们可以说，在 Unix 风格的文件系统中，inode 是一个元数据结构，描述文件和目录的属性，例如类型、时间戳、大小、访问权限、链接计数，以及指向磁盘块的指针，这些磁盘块存储构成文件或目录内容的数据。每个 inode 编号实际上是一个索引，允许内核访问文件或目录及其内容和属性，就像数组中的索引一样。实际上，如果你知道如何查看文件，你可以了解它的很多信息：

```
zarrelli:~$ stat test
 File: test
 Size: 4096      Blocks: 8          IO Block: 4096   
  directory
Device: fd01h/25025aInode: 5900830     Links: 2
Access: (3755/drwxr-sr-t)  Uid: ( 1200/zarrelli)   Gid: ( 1200/zarrelli)
Access: 2017-02-06 18:12:53.376827639 +0000
Modify: 2017-02-06 18:12:53.376827639 +0000
Change: 2017-02-07 19:26:15.936253432 +0000
 Birth: -  

```

所以，我们可以说，指向相同 inode 编号的东西，指向的是相同的数据结构、相同的文件或目录，这就是将 `..` 和 `.` 链接到父目录和当前目录表示的原因，就像我们通过跟踪 inode 编号所证明的那样。

+   `-h -L`：如果文件是链接，则此选项为真。链接是指向另一个文件的指针，您可以拥有软链接或硬链接：

+   +   符号链接是指向文件的引用，可以跨越不同的文件系统。它是一种特殊类型的文件，保存对另一个文件的引用，因此当操作系统试图访问该链接时，它会将其识别为链接，并将所有操作重定向到实际的目标文件。如果目标文件被删除，链接仍然存在，但指向空值。符号链接的主要限制是它在操作中会产生额外的开销；操作系统必须将所有操作从符号链接重定向到目标文件。

    +   硬链接是指向相同 inode 的另一个文件，由于 inode 是文件系统的元结构，硬链接无法跨越文件系统。一旦原始文件被删除，硬链接不受影响，因为它指向一个有效的 inode，而该 inode 即使文件被删除后仍然存在于文件系统中。一个限制是它不能指向目录。

所以，为了简单起见，我们可以说符号链接是指向文件的名称指针，而硬链接是指向 inode 的指针。我们从一个软链接开始，看看一些实际差异：

```
zarrelli:~$ ln -s test.filenew.test.file
zarrelli@:~$ ls -lah | grep new
lrwxrwxrwx 1 zarrellizarrelli    9 Feb  7 21:48 new.test.file ->test.file  

```

让我们 `cat` 链接：

```
zarrelli:~$ su
Password: 
root:# chmoda+rtest.file
root:# exit
exit
zarrelli:~$ cat new.test.file
1  

```

现在，让我们删除原始文件：

```
zarrelli:~$ rmtest.file

```

并验证我们是否可以通过链接访问内容：

```
zarrelli:~$ cat new.test.file
cat: new.test.file: No such file or directory  

```

使用 `ls` 可以清楚地看到问题：

```
zarrelli:~$ ls -lah
total 32K
drwxr-xr-x 3 zarrellizarrelli 4.0K Feb  7 22:06 .
drwxr-xr-x 3 zarrellizarrelli 4.0K Feb  7 22:02 ..
lrwxrwxrwx 1 zarrellizarrelli    9 Feb  7 21:48 
new.test.file ->test.file
drwxr-sr-t 2 zarrellizarrelli 4.0K Feb  6 18:12 
test
-rwxr--r-- 1 zarrellizarrelli  319 Feb  5 11:42 
test-files-not.sh
-rwxr--r-- 1 zarrellizarrelli  317 Feb  5 11:21 
test-files.sh
-rw-r--r-- 1 zarrellizarrelli    0 Feb  7 08:00 
test.modified
-rwxrwxr-x 1 zarrellizarrelli  352 Feb  5 12:31 
userinput-and.sh
-rwxr-xr-x 1 zarrellizarrelli  190 Feb  5 12:41 
userinput-and-simple.sh
-rwxrwxr-x 1 zarrellizarrelli  305 Feb  7 08:07 
userinput-or.sh  

```

链接仍然存在，但原始文件已经消失，因此当操作系统尝试访问它时，它会失败。让我们重新创建原始文件和链接：

```
zarrelli:~$ rmnew.test.file
zarrelli:~$ rmnew.test.file
zarrelli:~$ echo 1 >test.file
zarrelli:~$ ln -s test.filenew.test.file  

```

现在，让我们跨越 `/boot` 文件系统创建链接。首先，让我们检查 `/boot` 是否挂载在自己的分区和文件系统上：

```
zarrelli:~$ mount  | grep boot
/dev/nvme0n1p1 on /boot type ext2 (rw,relatime,block_validity,barrier,user_xattr,acl)  

```

好的，它确实是另一个文件系统，所以让我们使用软链接：

```
zarrelli:~$ su
Password:     
root:# ln -s test.file /boot/boot.test.file  

```

然后访问该链接：

```
root:# cat /boot/boot.test.file
cat: /boot/boot.test.file: No such file or directory  

```

嗯，似乎出了点问题。发生了什么？用 `ls -lah` 查看一下：

```
root:# ls -lah /boot/boot.test.file
lrwxrwxrwx 1 root root 9 Feb  7 22:19 /boot/boot.test.file 
->test.file  

```

嗯，尽管我们从另一个目录进行了链接，但链接指向 `/boot` 中的 `test.file`。我们需要一个绝对引用：

```
root:# ln -s /home/zarrelli/test.file /boot/new.test.file
root:# cat /boot/new.test.file
1  

```

现在，它可以工作了。最后，让我们看看如果尝试链接一个目录会发生什么：

```
zarrelli:~$ ln -s test new.test  

```

我们将 `new.test` 链接到 `test` 目录，现在只需检查 `test` 目录是否为空：

```
zarrelli:~$ ls -lah test
total 8.0K
drwxr-sr-t 2 zarrellizarrelli 4.0K Feb  7 22:29 .
drwxr-xr-x 3 zarrellizarrelli 4.0K Feb  7 22:26 ..  

```

现在，让我们使用链接创建一个文件：

```
zarrelli:~$ echo 2 >new.test/testing
zarrelli:~$ cat test/testing 
2  

```

然后让我们看看 test 中有什么：

```
zarrelli:~$ cat test/testing 
2  

```

这里，我们可以通过 `new.test` 软链接访问 test 中的数据。

现在是时候跨文件系统使用硬链接了：

```
zarrelli:~$ su
Password: 
root:# ln  /home/zarrelli/test.file /boot/hard.test.file
ln: failed to create hard link '/boot/hard.test.file' =>'/home/zarrelli/test.file': Invalid cross-device link  

```

不行！我们无法跨文件系统使用硬链接，因为 inode 限制阻止了我们这么做。

现在，让我们尝试链接一个目录：

```
zarrelli:~$ ln test hard.test
ln: test: hard link not allowed for directory  

```

再次，无法继续。现在，让我们尝试在同一个文件系统中进行硬链接：

```
zarrelli:~$ lntest.filehard.test.file  

```

没问题，但让我们看看 inode 情况：

```
zarrelli:~$ ls -lai
total 40
5899440 drwxr-xr-x 3 zarrellizarrelli 4096 Feb  7 22:35 .
5899435 drwxr-xr-x 3 zarrellizarrelli 4096 Feb  7 22:33 ..
5899465 -rw-r--r-- 2 zarrellizarrelli    2 Feb  7 22:11 
hard.test.file
5899355 lrwxrwxrwx 1 zarrellizarrelli    4 Feb  7 22:26 
new.test -> test
5900839 lrwxrwxrwx 1 zarrellizarrelli    9 Feb  7 22:12 
new.test.file ->test.file
5900830 drwxr-sr-t 2 zarrellizarrelli 4096 Feb  7 22:31 
test
5899465 -rw-r--r-- 2 zarrellizarrelli    2 Feb  7 22:11 
test.file
5899447 -rwxr--r-- 1 zarrellizarrelli  319 Feb  5 11:42 
test-files-not.sh
5899450 -rwxr--r-- 1 zarrellizarrelli  317 Feb  5 11:21 
test-files.sh
5898796 -rw-r--r-- 1 zarrellizarrelli    0 Feb  7 08:00 
test.modified
5899448 -rwxrwxr-x 1 zarrellizarrelli  352 Feb  5 12:31 
userinput-and.sh
5899449 -rwxr-xr-x 1 zarrellizarrelli  190 Feb  5 12:41 
userinput-and-simple.sh
5899444 -rwxrwxr-x 1 zarrellizarrelli  305 Feb  7 08:07 
userinput-or.sh  

```

硬链接指向原始文件的相同 inode，软链接则不指向。现在再次删除原始文件并尝试 `cat` 硬链接和软链接：

```
zarrelli:~$ rmtest.file
zarrelli:~$ cat new.test.file
cat: new.test.file: No such file or directory
zarrelli:~$ cat hard.test.file
1  

```

正如我们预期的那样，软链接失败了，因为没有文件名可以指向，而硬链接成功了，因为 inode 仍然存在：

```
zarrelli$ ls -lahi
total 36K
5899440 drwxr-xr-x 3 zarrellizarrelli 4.0K Feb  7 22:40 .
5899435 drwxr-xr-x 3 zarrellizarrelli 4.0K Feb  7 22:33 ..
5899465 -rw-r--r-- 1 zarrellizarrelli    2 Feb  7 22:11 
hard.test.file
5899355 lrwxrwxrwx 1 zarrellizarrelli    4 Feb  7 22:26 
new.test -> test
5900839 lrwxrwxrwx 1 zarrellizarrelli    9 Feb  7 22:12 
new.test.file ->test.file
5900830 drwxr-sr-t 2 zarrellizarrelli 4.0K Feb  7 22:31 
test
5899447 -rwxr--r-- 1 zarrellizarrelli  319 Feb  5 11:42 
test-files-not.sh
5899450 -rwxr--r-- 1 zarrellizarrelli  317 Feb  5 11:21 
test-files.sh
5898796 -rw-r--r-- 1 zarrellizarrelli    0 Feb  7 08:00 
test.modified
5899448 -rwxrwxr-x 1 zarrellizarrelli  352 Feb  5 12:31 
userinput-and.sh
5899449 -rwxr-xr-x 1 zarrellizarrelli  190 Feb  5 12:41 
userinput-and-simple.sh
5899444 -rwxrwxr-x 1 zarrellizarrelli  305 Feb  7 08:07 
userinput-or.sh  

```

一个简单的经验法则是，不要使用软链接来引用频繁访问的文件，如网页。例如，可以将它们用于 `config` 文件，因为这些文件通常只在启动时读取。软链接比较慢。硬链接适合引用文件，并且能够保留其内容，无论原文件发生什么：

+   `-p`：如果文件是管道，则为真。回想一下我们在第一章中关于管道和命名管道的内容；它们是让不同进程之间进行通信的一种方式。如果一个进程是由父进程生成的，这并不算什么大事，它们共享相同的环境和文件描述符。在父进程的打开描述符中，子进程将能够按写入顺序读取数据，使用缓冲区内核来保存等待读取的位。如果进程之间没有共享相同的环境，我们可以使用管道，它利用一个文件，进程可以依附在该文件上：该文件通常会比使用它的进程存在得更久，通常直到重启时才会被删除或重定向。这类进程间通信结构被称为 **命名管道** 或 **先进先出** (**FIFO**) 管道，基于数据处理的顺序。让我们做个示例，创建一个管道，可以使用 `mkfifo` 或 `mknode`：

```
zarrelli:~$ mkfifomyfifo  

```

现在，让我们打开第二个终端，并在相同目录下执行以下命令：

```
zarrelli:~$ cat myfifo  

```

该进程将挂起，等待读取某些内容。现在，让我们回到第一个终端并向命名管道写入一些内容：

```
zarrelli:~$ echo hello >myfifo  

```

现在回到第二个终端查看发生了什么：

```
zarrelli:~$ cat myfifo
hello  

```

我们将在接下来的章节中进一步了解命名管道，现在我们只需检查 `myfifo` 是否真的是一个命名管道：

```
zarrelli:~$ if [ -p myfifo ] ; 
then echo "Yes, it is a named pipe" ; 
else echo "No it is not a pipe" ; fi
Yes, it is a named pipe  

```

+   `-S`：如果它是一个套接字，则返回为真。正如我们之前看到的，套接字是同一系统内或跨网络的两个设备之间的端点，它允许设备交换数据，正如我们看到的管道和命名管道一样。正如你可以轻易猜到的，Linux 系统有很多套接字：

```
    zarrelli:~$ if [ -S /tmp/OSL_PIPE_1000_SingleOfficeIPC_39e ] ; then echo "Yes, it is a named pipe" ; else echo "No it is not a pipe" ; fi
Yes, it is a named pipe

```

我们可以通过查看`ls`列出的文件属性来进行双重检查：

```
zarrelli:~$ ls -lah /tmp/OSL_PIPE_1000_SingleOfficeIPC_39e 
srwxr-xr-x 1 zarrellizarrelli 0 Feb 12 09:02 /tmp/OSL_PIPE_1000_SingleOfficeIPC_39e

```

我们可以看到权限列表的开头有一个*s*字符，这表明这个文件正如我们预期的那样，是一个套接字。

+   `-t`：如果文件链接到终端，则返回为真。几乎所有指南中都会有一个经典的用途：检查脚本中的`stdin`或`stderr`是否链接到终端。让我们检查一下我们的 shell 的`stdout`：

```
zarrelli:~$ if [ -t 1 ] ; 
then echo "Yes, it is associated to a terminal" ; 
else echo "No it is not associated to a terminal" ; fi
Yes, it is associated to a terminal  

```

需要再双重检查一下吗？很简单，因为我们在当前 shell 中执行的命令的输出已经打印到连接到终端的显示器上。

+   `file -ntother_file`：如果文件比其他文件更新（通过修改日期进行比较），返回为真。让我们看一下：

```
zarrelli:~$ touch other_file ; 
for i in {1..10}; do : ; done ; touch file; 
if [ file -ntother_file ] ; 
then echo "Yes, file is newer then other_file" ; 
else echo "No file is not newer than other_file" ; fi
Yes, file is newer then other_file  

```

那么，我们做了什么？我们简单地串联了一些命令，并且由于`other_file`一定是较旧的文件，我们从创建它开始。然后，我们设置了一个简单的"for"循环，迭代从`1`到`10`，什么都不做，正如双冒号所暗示的那样，因为我们需要让一些时间过去，然后才创建较新的文件。经过`10`次迭代后，我们创建了一个新文件并进行比较。这个例子中有一点值得注意的是：

```
{1..10}  

```

这是一个自 Bash 3.0 版本以来就有的构造。让我们快速设置一个范围，写出起始数字和结束数字。从 Bash 4.0 版本开始，我们还可以定义一个带有增量的范围，形式如下：

```
{start..end..step}  

```

就像下面的例子一样：

```
zarrelli:~$ touch other_file ; 
for i in {1..120..2}; do if [ $(($i%5)) -eq 0 ] ; 
then echo "Waiting...cycle $i" ; fi ; sleep 1 ; done ; 
touch file; if [ file -ntother_file ] ; 
then echo "Yes, file is newer then other_file" ; 
else echo "No file is not newer than other_file" ; fi
Waiting...cycle 5
Waiting...cycle 15
Waiting...cycle 25
Waiting...cycle 35
Waiting...cycle 45
Waiting...cycle 55
Waiting...cycle 65
Waiting...cycle 75
Waiting...cycle 85
Waiting...cycle 95
Waiting...cycle 105
Waiting...cycle 115
Yes, file is newer then other_file  

```

好吧，这里加了一些东西：循环现在从`1`到`120`，步长为 2，只有在循环数字能被 5 整除时才打印消息，这样就不会使屏幕太乱。最后，我们使用`sleep`命令让每个循环等待 1 秒，以确保在循环结束时，我们在创建最后一个文件并进行比较之前，已经花费了 60 秒的时间。

我们还可以使用特定的日期修改文件的时间，即使是回到过去，使用`touch`命令和格式`-t YYMMDDHHmm`：

`zarrelli:~$ ls -lahtest.file`

`-rw-r--r-- 3 zarrellizarrelli 0 Feb 12 14:56 test.file`

`touch -t 8504251328 test.file`

`zarrelli:~$ ls -lahtest.file`

`-rw-r--r-- 3 zarrellizarrelli 0 Apr 25 1985 test.file`

+   `file -otother_file`：如果文件比`other_file`旧，返回为真：

```
zarrelli:~/$ if [ userinput-or.sh -otother_file ] ; 
then echo "Yes, the first file is older than the second" ; 
else echo "No the first file is not older than the second" ; fi  

```

确实如此：

```
zarrelli:~/$ ls -laht userinput-or.sh other_file
-rw-r--r-- 1 zarrellizarrelli   0 Feb 12 14:16 other_file
-rwxrwxr-x 1 zarrellizarrelli 305 Feb  7 08:07 userinput-or.sh  

```

现在只需触摸`userinput-or.sh`：

```
zarrelli:~$ if [ userinput-or.sh -otother_file ] ; 
then echo "Yes, the first file is older than the second" ; 
else echo "No the first file is not older than the second" ; fi
No the first file is not older than the second  

```

+   `file -efother_file`：如果文件和其他文件共享相同的 inode 号和设备，则返回为真。你是否已经听说过具有相同 inode 号的文件，并且它们不能位于不同设备上？是的，我们称之为硬链接：

```
zarrelli$ rmtest.file
zarrelli:~$ rmnew.test.file
zarrelli:~$ touch test.file
zarrelli$ lntest.filenew.test.file
zarrelli$ lntest.fileother.new.test.file
zarrelli:~$ if [ new.test.file -efother.new.test.file ] ; 
then echo "Yes, the files share the same inode number and device" ; else echo "No the files do not share the same inode number and device" ; fi
Yes, the files share the same inode number and device  

```

现在让我们再做一次双重检查：

```
zarrelli:~$ lls -laihtest.fileother.new.test.fileother
.new.test.file
5900839 -rw-r--r-- 3 zarrellizarrelli 0 Feb 12 14:56 other.new.test.file
5900839 -rw-r--r-- 3 zarrellizarrelli 0 Feb 12 14:56 other.new.test.file
5900839 -rw-r--r-- 3 zarrellizarrelli 0 Feb 12 14:56 
test.file  

```

这三个文件共享相同的 inode，因为 inode 是文件系统的元结构，所以它们都在同一个文件系统中。

我们已经完成了文件比较，但还有一些其他内容需要了解，例如如何测试整数，因为它们在我们看到的所有脚本中都是相当常见的。

# 测试整数

正如我们在文件比较中看到的，我们可以使用一些二进制操作符对整数执行类似的操作。这在我们想根据变量的值做出决策时非常有用，正如我们在之前的示例中所看到的，它是处理脚本时常见的一种操作：

+   `-eq`：当第一个整数等于第二个整数时为真：

```
#!/bin/bash    
echo "Hello user, please type in one integer and press enter:"
read user_input1 
echo "Now type in the number again and press enter:"
read user_input2 
if [ ${user_input1} -eq ${user_input2} ]
then
echo "Great! The integer ${user_input1} is equal to ${user_input2}"
else
echo "The integer ${user_input1} is not equal to ${user_input2}..."
fi  

```

代码相当简单，它没有对输入进行任何检查，但即使如此，它的简洁性也足以满足我们的需求：

```
zarrelli:~$ ./eq.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
10
Great! The integer 10 is equal to 10
zarrelli:~$ ./eq.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
20
The integer 10 is not equal to 20...  

```

+   `-ne`：当第一个整数不等于第二个整数时为真。让我们稍微修改一下之前的脚本：

```
if [ ${user_input1} -ne ${user_input2} ]
then
echo "Great! The integer ${user_input1} is not equal to 
${user_input2}"
else
echo "The integer ${user_input1} is equal to ${user_input2}..."
fi  

```

现在，让我们执行这段新的代码：

```
zarrelli:~$ ./ne.sh 
Hello user, please type in one integer and press enter:
100
Now type in the number again and press enter:
50
Great! The integers 100 is not equal to 50  

```

+   `-gt`：当第一个整数大于第二个整数时为真。再一次，修改几行代码：

```
if [ ${user_input1} -gt ${user_input2} ]
then
echo "Great! The integer ${user_input1} is greater than 
${user_input2}"
else
echo "The integer ${user_input1} is not greater than 
${user_input2}..."
fi  

```

现在，让我们试试：

```
zarrelli:~$ ./gt.sh 
Hello user, please type in one integer and press enter:
999
Now type in the number again and press enter:
333
Great! The integer 999 is greater than 333
zarrelli:~$ ./gt.sh 
Hello user, please type in one integer and press enter:
222
Now type in the number again and press enter:
888
The integer 222 is not greater than 888...  

```

+   `-ge`：当第一个整数大于或等于第二个整数时为真。以下是一些修改：

```
if [ ${user_input1} -ge ${user_input2} ]
then
echo "Great! The integer ${user_input1} is greater than or 
equal to ${user_input2}"
else
echo "The integer ${user_input1} is not greater than or 
equal to ${user_input2}..."
fi  

```

现在，这里有一些测试：

```
zarrelli:~$ ./ge.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
5
Great! The integer 10 is greater than or equal to 5
zarrelli@moveaway:~/Documents/My books/Mastering bash/Chapter 3/Scripts$ ./ge.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
10
Great! The integer 10 is greater than or equal to 10
zarrelli@moveaway:~/Documents/My books/Mastering bash/Chapter 3/Scripts$ ./ge.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
11
The integer 10 is not greater than or equal to 11...  

```

+   `-lt`：当第一个整数小于第二个整数时为真。所以，做一下小修改如下：

```
if [ ${user_input1} -lt ${user_input2} ]
then
echo "Great! The integer ${user_input1} is less than 
${user_input2}"
else
echo "The integer ${user_input1} is not less than 
${user_input2}..."
fi  

```

现在来做几个测试：

```
zarrelli:~$ ./lt.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
5
The integer 10 is not less than 5...
zarrelli:~$ ./lt.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
20
Great! The integer 10 is less than 20  

```

+   `-le`：当第一个整数小于或等于第二个整数时为真：

```
if [ ${user_input1} -le ${user_input2} ]
then
echo "Great! The integer ${user_input1} is less than or 
equal to ${user_input2}"
else
echo "The integer ${user_input1} is not less than or 
equal to ${user_input2}..."
fi  

```

现在，常规测试如下：

```
zarrelli:~$ ./le.sh 
Hello user, please type in one integer and press enter:
30
Now type in the number again and press enter:
20
The integer 30 is not less than or equal to 20...
zarrelli:~$ ./le.sh 
Hello user, please type in one integer and press enter:
60
Now type in the number again and press enter:
70
Great! The integer 60 is less than or equal to 70
zarrelli:~$ ./le.sh 
Hello user, please type in one integer and press enter:
70
Now type in the number again and press enter:
70
Great! The integer 70 is less than or equal to 70 

```

我们还有一组可以与双括号符号一起使用的操作符，它会执行算术扩展和计算：

+   `<`：当第一个整数小于第二个整数时为真。这是修改后的代码：

```
#!/bin/bash    
echo "Hello user, please type in one integer and press enter:"
read user_input1
echo "Now type in the number again and press enter:"
read user_input2
if (($user_input1 < $user_input2))
then
echo "Great! The integer $user_input1 is less than $user_input2"
else
echo "The integer $user_input1 is not less than $user_input2..."
fi  

```

现在，让我们测试一下：

```
zarrelli:~$ ./minor.sh 
Hello user, please type in one integer and press enter:
100
Now type in the number again and press enter:
50
The integer 100 is not less than 50...
zarrelli:~$ ./minor.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
100
Great! The integer 10 is less than 100  

```

+   `<=`：当第一个整数小于或等于第二个整数时为真。对之前的代码做一个小改动：

```
if (($user_input1 <= $user_input2))
then
echo "Great! The integer $user_input1 is less than or equal to $user_input2"
else
echo "The integer $user_input1 is not less than or equal to $user_input2..."
fi  

```

以及常规测试：

```
zarrelli:~$ ./minequal.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
20
Great! The integer 10 is less than or equal to 20
zarrelli:~$ ./minequal.sh 
Hello user, please type in one integer and press enter:
20
Now type in the number again and press enter:
10
The integer 20 is not less than or equal to 10...
zarrelli:~$ ./minequal.sh 
Hello user, please type in one integer and press enter:
20
Now type in the number again and press enter:
20
Great! The integer 20 is less than or equal to 20  

```

+   `>`：当第一个整数大于第二个整数时为真。稍作修改：

```
if (($user_input1 > $user_input2))
then
echo "Great! The integer $user_input1 is greater than 
$user_input2"
else
echo "The integer $user_input1 is not greater than 
$user_input2..."
fi  

```

以及常规测试，因为我们正在测试我们所写的所有内容：

```
    zarrelli:~$ ./greater.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
20
The integer 10 is not greater than 20...
zarrelli:~$ ./greater.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
5
Great! The integer 10 is greater than 5

```

+   `>=`：当第一个整数大于或等于第二个整数时为真：

```
if (($user_input1 >= $user_input2))
then
echo "Great! The integer ${user_input1} is greater than or equal to ${user_input2}"
else
echo "The integer ${user_input1} is not greater than or equal to ${user_input2}..."
fi  

```

现在，让我们运行一些检查：

```
zarrelli:~$ ./greaqual.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
10
Great! The integer 10 is greater than or equal to 10
zarrelli:~$ ./greaqual.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
5
Great! The integer 10 is greater than or equal to 5
zarrelli:~$ ./greaqual.sh 
Hello user, please type in one integer and press enter:
10
Now type in the number again and press enter:
20
The integer 10 is not greater than or equal to 20...  

```

# 测试字符串

我们刚才看到了一些有趣的比较，但现在事情变得更加有趣，因为我们将引入一些字符串测试，这将让我们在使脚本更加反应灵敏方面迈出一步：

+   `=`：当第一个字符串等于第二个字符串时为真：

```
#!/bin/bash 
echo "Hello user, please type a string and press enter:"
read user_input1 
echo "Now type another string and press enter:"
read user_input2 
if [ ${user_input1} = ${user_input2} ]
then
echo "Great! The string ${user_input1} is equal to ${user_input2}"
else
echo "The string ${user_input1} is not equal to ${user_input2}..."
fi

```

现在做一些测试：

```
zarrelli:~$ ./equal.sh 
Hello user, please type a string and press enter:
hello
Now type another string and press enter:
hello
Great! The string hello is equal to hello
zarrelli:~$ ./equal.sh 
Hello user, please type a string and press enter:
hello
Now type another string and press enter:
Hello
The string hello is not equal to Hello...  

```

现在，看看这个：

```
if [ ${user_input1}=${user_input2} ]
then
echo "Great! The string ${user_input1} is equal to ${user_input2}"
else
echo "The string ${user_input1} is not equal to ${user_input2}..."
fi  

```

现在做一些检查：

```
zarrelli:~$ ./equal-wrong.sh 
Hello user, please type a string and press enter:
hello
Now type another string and press enter:
hello
Great! The string hello is equal to hello
zarrelli:~$ ./equal-wrong.sh 
Hello user, please type a string and press enter:
hello
Now type another string and press enter:
Hello
Great! The string hello is equal to Hello  

```

有点奇怪，不是吗？我们刚刚去除了等号周围的空格，现在它不再起作用了：

```
zarrelli:~$ ./equal-wrong.sh 
Hello user, please type a string and press enter:
hello
Now type another string and press enter:
cat
Great! The string hello is equal to cat  

```

这些空格其实比你想象的更重要。我们的比较变成了赋值！所以，务必小心！

+   `==`：当第一个字符串等于第二个字符串时为真，但在双括号中使用时，它的行为与`=`不同：

```
#!/bin/bash    
echo "Hello user, please type a string and press enter:"
read user_input1 
echo "Now type another string and press enter:"
read user_input2 
if [[ ${user_input1} == ${user_input2}* ]]
then
echo "Great! The string ${user_input1} is equal to ${user_input2}"
else
echo "The string ${user_input1} is not equal to ${user_input2}..."
fi  

```

只要第一个字符串以第二个字符串输入的字符开始，`*`将匹配第二个字符串中输入字符后面的所有字符：

```
zarrelli:~$ ./equalequal.sh 
Hello user, please type a string and press enter:
match
Now type another string and press enter:
mi
The string match is not equal to mi...  

```

现在我们匹配带空格的情况：

```
zarrelli:~$ ./equalequal.sh 
Hello user, please type a string and press enter:
this should match
Now type another string and press enter:
this
Great! The string this should match is equal to this  

```

小心，在使用单括号时，由于文件模式匹配和单词拆分生效，这个操作符的行为会发生变化。

+   `!=`: 如果第一个字符串不等于第二个字符串，则为真，但请记住，存在模式匹配；因此，我们将稍微修改前面的脚本：

```
if [[ ${user_input1} != ${user_input2}* ]]
then
echo "Great! The string ${user_input1} is not equal 
to ${user_input2}"
else
echo "The string ${user_input1} is equal 
to ${user_input2}..."
fi  

```

现在测试如下：

```
zarrelli:~/Documents/My books/Mastering bash/Chapter 3/Scripts$ ./equalnot.sh 
Hello user, please type a string and press enter:
this should match
Now type another string and press enter:
this
The string this should match is equal to this...
zarrelli:~$ ./equalnot.sh 
Hello user, please type a string and press enter:
this should match
Now type another string and press enter:
not this
Great! The string this should match is not equal to not this  

```

+   `<`: 如果第一个字符串在 ASCII 字母顺序中小于第二个字符串，则为真。那么，让我们修改脚本：

```
if [[ ${user_input1} < ${user_input2} ]]
then
echo "Great! The string ${user_input1} is less than ${user_input2}"
else
echo "The string ${user_input1} is not less than ${user_input2}..."
fi  

```

现在进行一些测试：

```
zarrelli:~$ ./less.sh 
Hello user, please type a string and press enter:
abcd
Now type another string and press enter:
bcde
Great! The string abcd is less than bcde
zarrelli:~$ ./less.sh 
Hello user, please type a string and press enter:
zcf
Now type another string and press enter:
rst
The string zcf is not less than rst...  

```

小心，在使用单括号时，`<`符号必须转义，因此条件变为：

`if [ ${user_input1} \< ${user_input2} ]`

+   `>`: 如果第一个字符串在 ASCII 字母顺序中大于第二个字符串，则为真。那么，让我们修改脚本：

```
if [[ ${user_input1} > ${user_input2} ]]
then
echo "Great! The string ${user_input1} is greater 
than ${user_input2}"
else
echo "The string ${user_input1} is not greater 
than ${user_input2}..."
fi  

```

现在进行一些测试：

```
zarrelli:~$ ./greater.sh 
Hello user, please type a string and press enter:
@
Now type another string and press enter:
A
The string @ is not greater than A...  

```

实际上，`@`符号在 ASCII 表中的值为`40`，而`A`的值为`41`，所以`A`大于`@`。

小心，在使用单括号时，`>`符号必须转义，因此条件变为：

`if [ ${user_input1} \> ${user_input2} ]`

+   `-z`: 如果字符串为空，则为真。这里有一堆行来验证这个条件：

```
#!/bin/bash  
echo "Hello user, please type a string and press enter:" 
read user_input1  
if [[ -z ${user_input1} ]] 
then 
echo "Great! The string ${user_input1} is null" 
else 
echo "The string ${user_input1} is not null..." 
fi 

```

现在进行几个检查：

```
zarrelli:~$ ./null.sh 
Hello user, please type a string and press enter:    
Great! The string  is null
zarrelli:~$ ./null.sh 
Hello user, please type a string and press enter:
Hello
The string Hello is not null...  

```

正如你所看到的，在`null`值的情况下，当我们`echo`变量值时，屏幕上不会显示任何内容。

+   `-n`: 如果字符串不为空，则为真。对之前代码的一个小修改：

```
if [[ -n ${user_input1} ]]
then
echo "Great! The string ${user_input1} is not null"
else
echo "The string ${user_input1} is null..."
fi

```

现在进行一些测试

```
zarrelli:~$ ./notnull.sh 
Hello user, please type a string and press enter:    
The string  is null...
zarrelli:~$ ./notnull.sh 
Hello user, please type a string and press enter:
Hello
Great! The string Hello is not null  

```

每次测试变量时，请用引号引起来！未加引号的变量可能会导致奇怪的结果，特别是在处理空值时。

看看在单括号和未引用变量的测试中结果如何变化。请记住，`user_input2`未实例化，因此其值为`null`：

```
#!/bin/bash
echo "Hello user, please type a string and press enter:"
read user_input1 
if [ -n ${user_input1} ]
then
echo "Great! The string ${user_input1} is not null"
else
echo "The string ${user_input1} is null..."
fi
echo "But look at this..."    
if [ -n ${user_input2} ]
then
echo "Great! The string ${user_input2} is not null"
else
echo "The string ${user_input2} is null..."
fi    
echo "And now at this..."    
if [ -n "${user_input2}" ]
then
echo "Great! The string ${user_input2} is not null"
else
echo "The string ${user_input2} is null..."
fi  

```

现在，运行它：

```
zarrelli:~$ ./nullornot.sh 
Hello user, please type a string and press enter:    
Great! The string  is not null
But look at this...
Great! The string  is not null
And now at this...
The string  is null...  

```

好吧，正如你所看到的，引号确实很重要！

# 更多关于测试的内容

我们刚才看到了一些每次检查一个条件的测试，但我们也有`&&`和`||`的等效操作符，因此我们可以进行复合检查并一次测试更多条件，如下所示：

+   `-a`: 这是逻辑与，当两个条件都为真时，它为真。我们将这个操作符与测试命令一起使用，或者在单括号中使用。让我们重写之前的一个脚本：

```
#!/bin/bash    
echo "Hello user, please give me a number between 10 and 20, 
it must be even: "
read user_input
if [ ${user_input} -ge 10 -a ${user_input} -le 20 -a 
$(( $user_input % 2 )) -eq 0 ]
then
echo "Great! The number ${user_input} is what we were looking for!"
else
echo "The number ${user_input} is not what we are looking for..."
fi  

```

现在进行一些测试：

```
zarrelli@moveaway:~/Documents/My books/Mastering bash
/Chapter 3/Scripts$ ./userinput-a.sh 
Hello user, please give me a number between 10 and 20, 
it must be even: 
10
Great! The number 10 is what we were looking for!
zarrelli@moveaway:~/Documents/My books/Mastering bash
/Chapter 3/Scripts$ ./userinput-a.sh 
Hello user, please give me a number between 10 and 20, 
it must be even: 
13
The number 13 is not what we are looking for...  

```

+   `-o`: 这是逻辑或，如果其中一个条件为真，则为真。我们将这个操作符与测试命令一起使用，或者在单括号中使用。让我们重写前一个示例中的一些行：

```
if [ ${user_input} -ge 10 -a ${user_input} -le 20 -o 
$(( $user_input % 2 )) -eq 0 ]  

```

现在进行一些检查：

```
zarrelli:~$ ./userinput-o.sh 
Hello user, please give me a number between 10 and 20 OR even: 
15
Great! The number 15 is what we were looking for!
zarrelli:~$ ./userinput-o.sh
Hello user, please give me a number between 10 and 20 OR even: 
70
Great! The number 70 is what we were looking for!  

```

因此，现在如果数字在`10`和`20`之间或是偶数，则是可接受的：

```
zarrelli:~$ ./userinput-o.sh
Hello user, please give me a number between 10 and 20 OR even: 
23
The number 23 is not what we are looking for...
zarrelli:~$ ./userinput-o.sh
Hello user, please give me a number between 10 and 20 OR even: 
24
Great! The number 24 is what we were looking for!  

```

# 总结

我们终于进入了一个有趣的主题。并不是说前面的章节不重要，而是通过测试，我们看到现在可以创建一些条件语句，让脚本对不同的情况做出反应。这不仅仅是变量的回声，我们可以获取它的值，对其进行处理，决定接下来要做什么，并据此采取行动。在这一章中，我们简要了解了脚本中的灵活性意味着什么：它必须是一个工具，这也是编写脚本的主要目标之一：它必须是一个能够代表我们做出决策并根据我们提前设定的条件作出反应的工具。
