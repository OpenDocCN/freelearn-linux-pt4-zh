# 迭代

到目前为止，我们所看到的使我们能够与用户互动，处理输入，并根据我们设定的条件提供输出。所有这些都很好；如果用户带着一些参数调用我们的脚本，我们可以将它们存储在数组中并进行处理，前提是我们知道他们传递给命令行的选项数量。我们必须事先知道用户将提供多少项，否则我们会丢失多余的项。这时，迭代结构就派上用场了。因为我们已经看过一些示例，它可以枚举数组的内容，并让我们在不知道存储了多少项的情况下处理这些内容。在本章中，我们将看看如何使用 `for` 循环和 *while/until* 循环来充分利用用户提供的数据。

# `for` 循环

`for` 循环是 Bash 脚本中最常用的结构之一，它使我们能够对列表中的每个单独项执行一个或多个操作。它的基本结构可以概括如下：

```
for placeholder in list_of_items do
 action_1 $placeholder action_2 $placeholder action_n $placeholderdone

```

所以，我们使用了一个占位符，它将在循环的每一轮中获取列表项中的一个值，然后在 `do` 部分进行处理。当列表被扫描完后，循环结束，我们退出它。让我们从一个简单且漂亮的例子开始：

```
#!/bin/bash for i in 1 2 3 4 5 do
 echo "$i"done

```

现在让我们执行它：

```
zarrelli:~$ ./counter-simple.sh 1 2 3 4 5

```

其实，这非常简单，但要注意列表可以是任何操作的结果：

```
#!/bin/bash for i in {10..1..2} do
 echo "$i"done

```

在这种情况下，我们使用了大括号扩展来得到一个步长为 2 的倒计时：

```
zarrelli:~$ ./counter-brace.sh 10 8 6 4 2

```

我们也可以将 `for` 循环写在一行中：

```
zarrelli:~$ for i in *; do echo "Found the following file: $i"; done Found the following file: counter-brace.sh Found the following file: counter-simple.sh

```

漂亮，不是吗？现在让我们做一些更复杂的事情。假设我们要写出以下列表：

```
Belfast is in UK Redwood is in USA Milan is in ITALY Paris is in FRANCE

```

我们该怎么做呢？让我们尝试用一个简单的循环，看看会发生什么：

```
#!/bin/bash for cities in Belfast UK Redwood USA Milan ITALY Paris FRANCE do
 echo "$cities is in $cities" done exit 0

```

现在让我们运行它：

```
zarrelli:$ ./for-pair.sh Belfast is in Belfast UK is in UK Redwood is in Redwood USA is in USA Milan is in Milan ITALY is in ITALY Paris is in Paris FRANCE is in FRANCE

```

并不是我们想要的结果。嗯，完全不是，因为脚本不知道如何区分城市、国家以及它们之间的关系。我们必须找到一种方法来标定这些项；我们可以通过它们的位置来做到这一点。这里我们使用了内置的 set 命令，它使我们能够将变量的内容分配给位置参数。我们将以一种有趣的方式使用它：

```
#!/bin/bash for cities in "Belfast UK" "Redwood USA" "Milan ITALY" "Paris FRANCE" do
 set -- $cities echo "$1 is in $2" done exit 0

```

现在让我们运行脚本：

```
zarrelli:~$ ./for-pair-set.sh 
  Belfast is in UK
 Redwood is in USA Milan is in ITALY Paris is in FRANCE

```

这要好得多，正是我们想要的结果；但是我们是怎么实现的呢？第一步是将相关项分组并加上双引号，比如，`Belfast` 和 `UK` 一起。棘手的部分是使用 `set` 内置命令与 `--`，它强制将后续的值分配给位置参数，即使它们以短横线开头，如果没有提供参数，位置参数将被清空。因此，由于我们有两个一组的项：城市和国家，我们有 `$1` 和 `$2`；一个表示城市，另一个表示国家。从那时起，剩下的就是打印这些位置参数的事了。我们甚至可以不指定列表来继续：

```
zarrelli:~$ cat for-pair-input.sh #!/bin/bash i=0 for cities do
 echo "City $((i++)) is: $cities" done exit 0

```

然后，我们可以在命令行上提供参数；脚本将从 `$@` 获取输入：

```
zarrelli:~$ ./for-pair-input.sh 
Belfast Redwood Milan Paris City 0 is: Belfast City 1 is: Redwood City 2 is: Milan City 3 is: Paris

```

如前所述，列表可以是任何东西：一个变量，一个大括号表达式，固定的值，命令替换的结果，任何通过迭代的值创建列表的东西：

```
zarrelli:~/$ cat counter-function.sh #!/bin/bash counter() {
 echo {10..0..2} } for i in $(counter) do
 echo "$i" done

```

在这个示例中，列表是由计数器函数提供的，该函数输出大括号展开的结果。然后，我们通过`echo`命令获取函数返回的值，并将其作为列表进行迭代：

```
zarrelli:~$ ./counter-function.sh 10 8 6 4 2 0

```

不要忘记 C 风格：

```
zarrelli:~$ cat c-for.sh #!/bin/bash for ((i=20;i > 0;i--)) { if (( i % 2 == 0 )) then
 echo "$i is divisible by 2"fi } exit 0

```

所以，我们有一个递减的计数器：

```
zarrelli:~$ ./c-for.sh 20 is divisible by 2 18 is divisible by 2 16 is divisible by 2 14 is divisible by 2 12 is divisible by 2 10 is divisible by 2 8 is divisible by 2 6 is divisible by 2 4 is divisible by 2 2 is divisible by 2

```

到目前为止，我们所看到的内容让我们能够遍历数据结构并对其进行操作，只要我们有一些项目需要处理，但我们还不知道如何在某个条件成立或不成立之前进行操作，所以下一段将讨论如何保持脚本在某些事情发生之前保持运行。

# 让我们做一些事情，直到…… 

`for` 循环是一个很好的选择，用来枚举用户提供的内容，但当需要处理一组预先无法知道数量的选项时，它就不太方便了。在这种情况下，我们会发现一些更有趣的循环结构，它们允许我们在满足某个条件之前或在某种情况持续存在时进行循环，例如，用户输入某些内容时，或者直到达到某个阈值为止。所以，让我们看看哪些结构可以帮助我们：

```
while condition do
 command_1 command_2 command_n done

```

一开始，`while` 和 `for` 循环的区别很明显：后者基于一个占位符，每次从列表中获取一个值并对该值进行操作，而前者在条件满足时触发。让我们通过一个从 `for` 循环开始的示例来说明：

```
#!/bin/bash for i in 1 2 3 4 5 do
 echo "$i" done

```

这是一个简单的计数器，从 1 到 5，我们已经看过它：

```
zarrelli:~$ ./counter-simple.sh 1 2 3 4 5

```

现在，让我们使用 `while` 循环重写它：

```
#!/bin/bash i=1 while (( i <= 5)) do
 echo "$i"((i++)) done

```

让我们看看输出是否相同：

```
zarrelli:~$ ./while-simple.sh 1 2 3 4 5 

```

好吧，输出是完全相同的。接下来，我们将讨论另一种循环结构，`until` 循环，它在条件满足之前对列表进行循环。其结构如下：

```
until condition do
 command_1 command_2 command_n done

```

结构与 `while` 循环类似，只是条件不同：`while` 在条件成立时持续运行，`until` 在条件成立时停止运行。为了更好地理解区别，让我们将示例重写为 `until` 形式：

```
#!/bin/bash i=1 until (( i > 5)) do echo "$i"((i++))done

```

从代码中可以看到，直到 `i` 的值大于 `5` 之前，我们打印它的值并增加它：

```
zarrelli:~$ ./until-simple.sh 12345

```

看起来很熟悉，不是吗？那么我们可以总结出三种类型的循环，条件如下：

+   `for` 在从列表中获取的值上进行迭代

+   `while` 在条件为 `false` 时执行循环

+   `until` 在条件为 `false` 时执行循环

# 使用 break 和 continue 退出循环

这给了我们一些不错的机会，比如无限循环：

```
while true ; do echo "Hello" ; done

```

由于`true`始终计算为真，因此条件始终会被验证，从而导致`do/done`语句块的无限执行；按*Ctrl+C*退出循环。无限循环看起来像是一个麻烦的事情，但它为我们的脚本打开了一个新的场景，因为我们可以让它们运行或等待某个事件，直到我们希望的时间。实际上，如果我们不使用一些循环控制命令：`break`将退出循环，`continue`将重新启动循环，跳过剩余的命令。让我们看看创建一个假设的备份程序菜单的示例：

```
#!/bin/bash while true do
 clear cat <<MENU BACKUP UTIL v 1.0 ------------------ 1\. Backup a file/directory 2\. Restore a file/directory 0\. Quit ------------------ MENU
 read -p "Please select an option, 0 or Q to exit: " option case $option in 1 | [Bb]) echo "You chose the first option, Backup" sleep 3 ;; 2 | [Rr])  echo "You chose the second option, Restore"
 sleep 3 ;; 0 | [Qq]) echo "You chose the third options, Quit, so we quit!" break ;; *) echo "Not a valid choice, please select an option..." sleep 3 ;; esac done 

```

来看看我们做了什么。我们打开了一个`while true`循环，所以其中的内容会一遍又一遍地执行。然后，我们使用了*here*文档来展示一个漂亮的菜单给用户，并使用`read`选项让用户选择并评估输入。除非用户选择`quit`，否则任何选择都会什么也不做，仅显示一条消息并等待 3 秒，之后循环重新开始，清空屏幕并再次显示菜单（这就是`sleep`命令的作用）。唯一的例外是如果客户选择了`0`、`Q`或`q`：在这种情况下，会显示一条消息并退出循环：

```
zarrelli:~$ ./menu.sh 
  BACKUP UTIL v 1.0
 ------------------ 1\. Backup a file/directory 2\. Restore a file/directory 0\. Quit ------------------ Please select an option, 0 or Q to exit: 0 You chose the third options, Quit, so we quit!

```

请注意，我们退出的是循环，而不一定是整个脚本。这是一个经典的老式菜单，它相较于图形菜单有一些优势：更容易编写、更容易维护、消耗的资源更少，但最重要的是，它不需要图形显示器：它在字符显示器和串行连接上运行良好。对于`continue`指令，操作流程非常不同，因为它会恢复主`for`、`while`、`until`或`select`循环的迭代。当用于`for`循环时，变量将取列表中条件的下一个元素的值：

```
zarrelli:~$ cat for-continue.sh #!/bin/bash for i in {0..10} do
 if (( i == 4 )) then continue else echo $i fi done exit 0

```

我们的代码将从 0 枚举到 10 并打印出遇到的值，除了当它遇到数字 4 时：在这种情况下，`continue`将迫使`for`循环跳过该值并从 5 继续：

```
zarrelli:~$ ./for-continue.sh 0 1 2 3 5 6 7 8 9 10

```

# 现在是时候给我们的客户端一个菜单了

在本章中，我们将探讨不同的方式来处理循环，以便处理用户提供给我们的信息。从一个简单的菜单开始，我们转向了更华丽、外观更佳的方式；现在，是时候更进一步，看看`select`构造，它的任务是让我们轻松创建菜单。它的语法与`for`构造相似：

```
select placeholder [in list] do command_1 command_2 command_n done

```

因此，正如我们所看到的，这个结构与`for`非常相似，并支持列表，在标准错误中展开为一系列以数字开头的元素。如果省略了`in list`部分，列表将从命令行中给定的位置参数构建，例如，如果我们使用了`[in $@]`。一旦打印出列表中的元素，将显示`PS3`提示，并读取并存储到`REPLY`变量中的`stdin`行。如果读取了行上的内容，则每个单词都会显示出来并附带一个数字；如果行为空，则再次显示提示，但给出`if`和`EOF`字符作为输入（*Ctrl+D*）时退出循环。作为快捷方式，您可以使用`break`退出。让我们看一个例子：

```
#!/bin/bash echo "Just select the fruit you like:" select fruit in apple banana orange mango do
 echo "You picked $fruit (Option $REPLY)" done

```

这个简单的脚本将向您展示一个从`in list`中获取的选项菜单，并等待选择。一旦用户输入选择，它就会被回显，并且循环再次开始，显示可用选项：

```
zarrelli:~$ ./simple-select.sh Just select the fruit you like: 1) apple 2) banana 3) orange 4) mango #? 3 You picked orange (Option 3) #? o You picked (Option o) #? pear You picked (Option pear) #? 

```

正如我们所看到的，我们无法控制用户向我们提供的内容，所以这是我们必须自己实现的事情。此外，提示是我们见过的最不性感的东西，但我们可以通过为`PS3`变量赋值来更改它，所以只需在 sha-bang 下面添加`PS3="Your choice is: "`。保存并重新运行脚本：

```
Just select the fruit you like: Enter the number of the file you want to protect: 1) apple 2) banana 3) orange 4) mango Your choice is: 

```

现在好多了。很好，现在另一个小问题：脚本永远不会退出，所以我们如何强制它退出呢？让我们看一些有趣的修改：

```
zarrelli:~$ cat case-select.sh #!/bin/bash PS3="Your choice is: " echo "Just select the fruit you like:" select fruit in apple banana orange mango do
 case "$fruit" in mango) echo "You chose $fruit, so we wanna break free!" break ;; *) echo "You chose $fruit" ;; esac done

```

我们在`select`内嵌套了一个`case`结构，这样我们就可以评估用户给出的选择并做出相应反应。无论如何，我们只是打印出用户的选择，但如果他选择了`4`，我们会打印出选择并使用`break`退出：

```
zarrelli:~$ ./case-select.sh Just select the fruit you like: 1) apple 2) banana 3) orange 4) mango Your choice is: 2 You chose banana Your choice is: 4 You chose mango, so we wanna break free!

```

但是我们可以做得更有趣，特别是如果我们想与系统进行交互。让我们再次尝试制作一个备份脚本，并利用我们刚刚做的事情：

```
#!/bin/bash while true do
 clear cat <<MENU BACKUP UTIL v 1.0 ------------------ 1\. Backup a file/directory 2\. Restore a file/directory 0\. Quit ------------------ MENU PS3="Which file do you want to backup? " touch EXIT 
  read -p "Please select an option, 0 or Q to exit: " option
 case $option in 1 | [Bb]) echo "You chose the first option, Backup" clear select file in * do case "$file" in                    EXIT)
 echo "Ok, we exit!" rm EXIT break ;; *)
 echo "Compressing file $file" tar cvzf "${file}".tgz "$file" || exit 1 echo "File $file compressed." ls "${file}".tgz echo "Press a key to return to main menu..." read                break
 ;; esac done ;; 2 | [Rr])         echo "You chose the second option, Restore"
 sleep 3 ;; 0 | [Qq]) echo "You chose the third options, Quit, so we quit!" break ;; *) echo "Not a valid choice, please select an option..." sleep 3 ;; esac done rm EXIT

```

我们使用`while true`创建了主循环，这样脚本将始终运行，除非我们明确退出它，然后使用*here*文档，向用户显示一个整洁的菜单，然后使用`case`结构来评估用户在下一步中给出的答案。`case`语句中的第一个选项清除屏幕并嵌入一个选择结构，还提供了使用文件名扩展来操作文件列表的方法。由于我们事先不知道当前目录中会有多少文件，并且不能修改列表，我们可以依赖一个技巧，在脚本开始时创建一个名为`EXIT`的文件，并在脚本结束时删除它。输出将类似于这样：

```
BACKUP UTIL v 1.0
------------------
1\. Backup a file/directory
2\. Restore a file/directory
0\. Quit
------------------
Please select an option, 0 or Q to exit:

```

然后选择选项 1：

![](img/00007.jpeg)

文件会自动编号，而没有 EXIT 占位符。目录中的文件都会被编号，并且我们的 EXIT 策略会显示出来，尽管它不是最后一个选项；但是重命名它会让我们将它放在任何我们想要的位置。在`select`中，我们找到另一个 case，因为我们要评估用户给出的答案，并处理正确的文件。所以，如果用户没有选择与 EXIT 对应的选项，我们不会从循环中退出并清理文件，而是继续压缩它，并退出到主循环，这会显示主菜单。所有这些都很好，但你意识到这里存在一个大问题吗？如果你选择一个在 select 菜单中并不存在的选项会发生什么？

```
1) backup-menu.sh 2) case-select.sh 3) c-for.sh 4) counter-brace.sh 5) counter-function.sh 6) counter-simple.sh 7) EXIT 8) for-continue.sh 9) for-pair-input.sh 10) for-pair-set.sh 11) for-pair.sh 12) simple-select.sh 13) until-simple.sh 14) while-simple.sh Which file do you want to backup? 15 Compressing file tar: Substituting `.' for empty member name tar: : Cannot stat: No such file or directory tar: Exiting with failure status due to previous errors

```

好吧，这是可以预料的，因为我们没有对输入进行真正的检查，或者更准确地说，我们只检查了我们预期接收的内容，而不是意外的内容。所以让我们修改默认选项以适应内部的 case 语句：

```
*)
 if [ -z "$file" ] then       echo "Please, select one of the number displayed"
 sleep 3 continue fi echo "Compressing file $file" tar cvzf "${file}".tgz "$file" || exit 1 echo "File $file compressed." ls "${file}".tgz echo "Press a key to return to main menu..." read break ;;

```

我们仅仅在传递给`$file variable`的内容中添加了一个检查：如果该变量为空且不指向任何文件名，我们将显示一条消息，等待三秒钟，然后重新启动循环。我们也可以使用`read`，而不仅仅是等待，这样可以强制用户按下一个键继续；这将使警告消息保持显示，直到用户做出反应。

从各个示例中我们可以看到，创建用户菜单在 Bash 中有不止一种方法；在接下来的章节中，我们将使用这些方法并让它们变得更加华丽。但谈到用户交互时，我们还有一个话题需要面对，而且它非常有趣：如何处理传递给脚本的命令行参数。所以，如果我们不想显示菜单，而是希望在命令行接收参数，我们该如何操作呢？我们已经看到过一些内容，但有一个很好的内建功能可以让我们的工作更加轻松，现在是时候看看`getops`了。

# CLI，将参数传递给命令行

Geopts 是一个 Bash 内建命令，广泛用于高效地解析传递给脚本的开关和参数。我们已经看到过其他完成此任务的方法，但`getops`使得处理它变得非常简单，因为它可以自动识别传递给脚本的开关和参数。它的语法如下：

```
getops options variable

```

我们传递给`getops`的第一个是选项的字符串，即经典的`-a -x -f`之类的，没有任何前导的破折号，比如`getops axf`或`getops ax:f`。如果你看到某个选项后面跟着一个冒号，这意味着该选项需要一个参数，如下所示：

```
./our_script.sh -x our_argument -a

```

在我们的示例中，`-x`有一个参数，而`-a`是一个简单的开关，或者我们也可以称之为**标志**，它可以存在也可以不存在，但不需要任何参数。选项可以用小写或大写字母，或者数字来指定。`getops`内建有一些预定义变量供其内部使用：

+   `OPTARG`保存着选项的参数或未知选项的标志。

+   `OPTBIND`保存下一个要解析的选项的索引。

+   `OPTERR`的值为 0 或 1，用来设置`getops`的错误信息显示。默认值为 1，所以这里会显示错误信息。

很明显，`getops`适用于解析短选项，但它无法处理长选项样式，因此`-a`是可以的，但`--all`则无法解析。这是一个限制，但也只是风格问题。我们来看一个简单的例子，边注释边看`getops`在实际应用中是如何工作的：

```
#!/bin/bash while getopts ":ax:f" option do
 case $option in a | f) echo "You selected $option!" ;;
 x) echo "You selected $option with argument $OPTARG" ;;
 ?) echo "Invalid switch: -$OPTARG"
 ;; :) echo "No arguments provided: -$OPTARG"
 ;; esac done

```

所以，我们的脚本以一个 while 循环开始，这是因为当`getops`遇到无法解析的内容时，它会退出并返回一个名为`fail`的状态，这个条件会在遇到第一个非选项参数或遇到`--`时触发。接下来，我们看到的是`getops`内建命令，后面跟着`ax:f`，意味着它期望如下：

```
-a -x argument -f

```

`getops`会读取所有选项，直到遇到第一个非选项参数，并将其存储在一个变量中，在我们的例子里叫做 option。现在，最后这一部分有点复杂。看一下`getopts ":ax:f"`。

你注意到第一个选项前的`:`了吗？它禁用了 getops 的标准错误信息，并改变了标准变量的使用方式。

如果选项无效，变量（在我们的例子中是`option`）会用`?`字符来存储错误信息；同时，`OPTARG`会存储用户提供的无效字符。

如果有参数，变量会通过冒号`:`进行实例化，而`OPTARG`会保存选项字符。让我们用不同的选项运行这个脚本，看看它是如何工作的：

```
zarrelli:~/$ ./getops-simple.sh -a You selected a! zarrelli:~/$ ./getops-simple.sh -f You selected f! zarrelli:~/$ ./getops-simple.sh -f -a You selected f! You selected a! zarrelli:~/$ ./getops-simple.sh -x No arguments provided: -x zarrelli:~/$ ./getops-simple.sh -x hello You selected x with argument hello zarrelli:~/$ ./getops-simple.sh -x hello -a -f You selected x with argument hello You selected a! You selected f! zarrelli:~/$ ./getops-simple.sh -z Invalid switch: -z

```

很好，但不要被愚弄，我们这里有两个主要问题，你能看出来吗？我们来看看第一个问题：

```
zarrelli:~/$ ./getops-simple.sh zarrelli:~/$

```

好吧，没有给出任何开关，没有输出，这样可不行：记住，绝不能让用户没有任何反馈。永远让你的脚本给用户显示一些东西，让他们知道他们做了什么，否则他们可能会反复尝试调用。我们可以通过在 while 循环前添加一个小片段代码来处理这种情况，该片段用于统计命令行上的参数个数：

```
if (( $# == 0 )) then 
echo "Please, give at least one option on the command line" exit 1 fi

```

没什么特别的，我们只是检查传递给命令行的参数数量是否等于`0`，如果是的话，我们就输出一条信息并以错误状态退出：

```
zarrelli:~$ ./getops-arguments.sh Please, give at least one option on the command line Is it this all about our errors? Not precisely: zarrelli:~$ ./getops-arguments.sh -a Hello You selected a!

```

`Hello`是传递给命令行的一个参数，但`-a`不接受`options`参数，那么我们该如何获取`Hello`呢？请注意，这两个调用之间是有区别的：

```
zarrelli:~$ ./getops-arguments.sh -x Hello You selected x with argument Hello zarrelli:~$ ./getops-arguments.sh -a Hello You selected a!

```

第一个`Hello`是一个选项的参数，而第二个是命令行上的一个简单参数，它与选项无关，因为`-a`不接受任何参数。因此，我们第一次能够处理到`Hello`，但在第二种情况下却无法处理。我们该如何克服这个限制呢？让我们重写之前的例子，并在脚本的末尾添加以下几行：

```
echo "And the argument was $*" shift "$((OPTIND-1))" echo "And the argument was $*"

```

现在让我们再次运行脚本：

```
zarrelli:~$ ./getops-arguments.sh -x whatever -a Hello You selected x with argument whatever You selected a! And the argument was -x whatever -a Hello And the argument was Hello

```

就是这样。注意到我们使用了 `shift`，它帮助我们获取了参数；这个技巧基于 `OPTIND` 存储的值，它对应于 `getops` 最后一次调用时解析的选项数量。如果我们回顾一下 `getops` 的工作原理：每次调用时，它将下一个选项放入用于存储选项的变量中，如果该变量不存在，则会进行初始化，并将下一个要解析的参数的索引存入 `OPTIND` 变量。因此，第一次运行时，`OPTIND` 的值为 `1`。请记住，`OPTIND` 永远不会被 shell 重置，因此如果你需要多次调用 `getops`，你需要自己将该变量重新初始化为 `1`。然后，我们使用 `shift` 来处理位置参数，因为这个内建函数可以根据指定的参数将位置参数向左移动。因此，`shift "$((OPTIND-1))"` 会将下一个 `getops` 参数的位置向左移一个位置。让我们重写上一部分脚本：

```
case $option in
 a | f) echo "You selected $option with $OPTIND=$OPTIND and the command line argument $*!" ;; x) echo "You selected $option with argument $OPTARG with $OPTIND=$OPTIND and the command line $*!"
 ;; ?) echo "Invalid switch: -$OPTARG with $OPTIND=$OPTIND" ;; :) echo "No arguments provided: -$OPTARG with $OPTIND=$OPTIND" ;; esac done echo "$OPTIND at the end of the loop is $OPTIND" shift "$((OPTIND-1))" echo "But at the end of the script we have this left on the command line: $*"

```

现在，再次运行它：

```
zarrelli:~$ ./getops-arguments.sh -f -a Hello You selected f with $OPTIND=2 and the command line argument -f -a Hello! You selected a with $OPTIND=3 and the command line argument -f -a Hello! $OPTIND at the end of the loop is 3

```

但是在脚本的最后，我们在命令行上留下了这个：`Hello`。

那么，发生了什么呢？当你运行脚本时，`OPTIND` 的初始值为 `1`，每次调用 `getops` 时，`OPTIND` 会增加 `1`。因此，既然我们在命令行上有两个选项需要处理，在 `getops` 循环结束时，`OPTIND` 的值将是 2+1，也就是 3。现在，如果我们对命令行使用 `"$((OPTIND-1))"` 进行移位，这意味着我们将命令行的参数向左移动了 2 个位置（3-1）。请记住，当你将位置参数向左移时，它们基本上会丢失，剩下的就是其余的参数。在我们的例子中，如果我们将位置参数向左移动 2 个位置，我们就去掉了 `-f` 和 `-a`，剩下的就是第一个非选项参数，`"Hello"`。就是这样！如果我们在循环后打印命令行内容 `$@`，结果就是 `Hello`。现在，是时候建立我们将在接下来的章节中使用的工具，并将到目前为止学到的所有部分汇集起来了。首先：脚本将变得更复杂，并且可能会变得有些凌乱。所以，如果我们回顾一下前几章关于引用文件的内容，我们现在做的是创建一个库来存放所有我们将频繁使用的公共函数和设置。采用这种风格将帮助我们保持脚本的整洁和简洁，并更轻松地掌握其内容。所以，首先，创建一个我们称之为 `library.lib` 的库文件，并开始在其中编写一些函数：

```
# Library file holding common functions and setting # Functions non_zero_input() {
 if (( $1 == 0 )) then  echo "Please, give at least one option on the command line"
 exit 1 fi }

```

现在，让我们以如下方式重写前面脚本的第一部分：

```
#!/bin/bash source library.lib non_zero_input "$#"

```

现在，是时候在没有任何选项的情况下运行脚本了：

```
zarrelli:~$ ./getops-library.sh 
Please, give at least one option on the command line.

```

如我们所见，零长度输入的检查已被移至库中，然后再引用回主脚本。所以，现在我们有了两个优点：

+   我们的主脚本中行数更少了

+   `non_zero_function` 现在对所有将引用我们刚创建的库的脚本都可用

但是现在，是时候来点花样了。有没有想过为输出增添一些亮点？只需按以下方式修改这个库：

```
# Library file holding common functions and setting # Functions #---------- non_zero_input() {
 if (( $1 == 0 )) then  echo "Please, give at least one option on the command line"
 exit 1 fi } color_print() {
 printf "$1$2${CReset}n" } # Colors - foreground #-------------------- Black='330;30m' Red='33[0;31m' Green='33[0;32m' Yellow='33[0;33m' Blue='33[0;34m' Purple='33[0;35m' Cyan='33[0;36m' White='33[0;37m' # Colors - Reset #--------------- CReset='33[0m'

```

我们向库中添加了一些 ANSI 颜色码，并将它们分配给有意义的变量名，还创建了一个小的 `color_print` 函数。ANSI 转义码或序列是用来管理文本终端上颜色和属性的方法，它们以 ESC 字符（八进制 033）开头，后面跟着一个 ASCII 范围内 64 到 95 之间的字符。我们仅添加了一些前景颜色，但通过谷歌搜索，你会找到一长串可以分配给背景色、粗体字符、反转等的转义字符。`color_print` 函数仅仅是利用这些控制码能够做的事情的一个小例子，它使用了 `printf`，虽然 `echo -e` 也能处理转义字符，足以用来打印颜色和属性，但 `printf` 更加灵活。请注意，`color_print` 函数在 `printf` 结束时使用了 `'33[0m'`，这是一个重置控制字符，会将你对输出所做的所有更改恢复为默认设置：一旦你修改了颜色或属性，所有内容都会按照这些修改进行打印，直到你明确使用重置转义序列恢复为默认值。现在，是时候利用我们刚刚学到的东西，修改我们刚刚创建的脚本，让它以这种方式结束：

```
done echo "$OPTIND at the end of the loop is $OPTIND" shift "$((OPTIND-1))" echo $@ echo -e "${Green}But${CReset} at the end of the script we have this left on the command line: ${Red}$@${CReset}" color_print ${Yellow} "But we can use our color_print function to have a fancy output: $@"

```

这里有两个不同的示例，展示了如何使用转义码来管理输出：

```
echo -e "${Green}But${CReset} at the end of the script we have this left on the command line: ${Red}$@${CReset}"

```

单词 `But` 前面是我们从库中获取的 `Green` 变量的值，后面跟着 `CReset` 变量值，所以 echo 会在打印 `But` 之前将输出切换到绿色前景色，并在重置转义序列的作用下，在打印完成后恢复为标准颜色（通常是白色）。然后，在打印命令行参数之前，它会切换到 `Red`，完成后再次恢复。最后一行是使用从库文件中获取的 `color_print` 函数打印的：

```
color_print ${Yellow} "But we can use our color_print function to have a fancy output: $@"

```

从函数定义中我们可以看到，它接受两个参数，一个是转义码，另一个是要打印的字符串，后面跟着一个重置码；在我们的案例中我们选择了 `Yellow`，那么让我们看看接下来的截图中会有什么样的结果：

![内联转义码或临时函数让我们的输出更具表现力不错吧？其实使用 Bash 给输出加上颜色有很多方法，比如通过与 dialog 程序交互，它会给你一个类似 curses 的界面：```zarrelli:~$ dialog --begin 10 30 --backtitle "Example menu" --title "This is a Message Box" --msgbox 'Your message goes here!' 10 30 ```![](img/00009.jpeg)

这是一个简单的消息框，它让我想起了老版 Linux 安装程序中的 zenity，它会为你提供一个 GTK+ 界面：

```
zarrelli:~$ ls -l | zenity --text-info --height=600 --width 800 

```

![](img/00010.jpeg)

Zenity 允许你使用 GTK+ 装饰创建漂亮的界面

我们可以使用的东西取决于我们希望与客户进行的交互级别；例如，处理服务的脚本可能不需要任何花哨的东西。关于可移植性的考虑：要使用 dialog 和 zenity，你必须安装它们；它们并不默认随 Linux 系统一起提供。对于 zenity，请记住，它只在图形界面上显示其优点；如果通过串口或基于文本的终端运行，它最多只会显示像 curses 这样的界面。如果你想使用比 ANSII 转义码更先进的东西，可以使用 `tput` 命令，它随 Linux 提供；通过使用 `terminfo` 或 `termcap` 数据库，它可以让你以更有趣的方式与终端进行交互：

```
#!/bin/bash
fred=$(tput setaf 1)
fgreen=$(tput setaf 2)
fwhite=$(tput setaf 7)
bblue=$(tput setab 4)
esmso=$(tput smso)
xsmso=$(tput rmso)
dim=$(tput dim)
reset=$(tput sgr0)
hide=$(tput civis)
box() {
printf ${hide} 
printf ${bblue} 
width=$(tput cols)
height=$(tput lines)
message="Width is: ${esmso}${fgreen}$width${fwhite} Height is: ${dim}${fred}$height${reset}"
length=${#message}
clear
tput cup $((height / 2)) $(((width / 2) - ((length - 29) / 2)))
printf "$message"
}
trap box WINCH
box
while true
do
:
done 

```

![](img/00011.jpeg)

使用 tput 显示的自动更新消息

这个简单的脚本使用 `tput` 和一系列数字来改变输出的颜色。`tput setaf x` 设置前景色为与 `x` 整数对应的值：

```
0 black 1 red 2 green 3 yellow 4 blue 5 magenta 6 cyan 7 white

```

对于背景，我们使用 `tput setbg x` 和相同的代码列表。我们使用命令替换获取命令的输出，并使用 `printf` 来相应地修改输出。其他 `tput` 命令的值是显而易见的：

+   `tput smso` 进入突出显示模式

+   `tput rmso` 退出突出显示模式

+   `tput dim` 使输出变得不那么亮

+   `tput sgro` 恢复到标准终端输出

+   `tput cvis` 隐藏光标

然后，我们创建了一个名为 `box` 的函数，它隐藏光标，并将背景设置为蓝色。这里的有趣部分是：

+   `tput cols` 获取终端的列数

+   `tput lines` 获取终端的行数

我们将两个命令的输出存储在宽度和高度变量中，并使用消息变量来组成一个字符串，输出我们终端的尺寸，使用 `tput` 的颜色来格式化输出，背景有一个漂亮的旗帜。我们以非常规的方式使用了各种属性，但你可以自己尝试，看看能得到什么样的输出。

查看 `man terminfo`、`man termcap` 和 `man tput`，了解你可以使用 `tput` 做的所有事情。继续编写脚本，我们获取消息的长度并清除脚本，清除屏幕，最后使用 `tput cup x y`。

我们将光标移动到正确的位置，以便将消息居中显示在终端上。我们需要补偿用于` tput` 属性的字符。在函数外部，我们使用一个陷阱，拦截发送到进程的窗口变化信号，当控制该进程的终端改变其大小时。这样，每次用户改变窗口大小时，陷阱就会调用`box`函数，从而计算并打印新的高度和宽度到终端。我们在最后留下了一个无限循环，一旦第一个框架迭代完成，它便开始执行：这个无限循环保持脚本处于空闲状态，等待窗口变化信号的捕获。一旦捕获到信号，就会调用`box`函数，计算新的高度和宽度，并显示最新的消息。

# 摘要

我们的脚本开始变得更加复杂且有趣；我们正在从处理 Shell 脚本转向编程，并利用它创造一些实用的东西。本书的下一部分将深入到一些实际编程，创建一些应用程序，展示如何为我们的日常生活作为系统管理员或好奇的用户，打造一些可靠且有用的工具。
