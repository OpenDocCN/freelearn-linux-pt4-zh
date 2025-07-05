# 操作符

到目前为止，我们所做的是处理来自变量扩展和描述符的值，并以巧妙的方式使用它们。因此，这是一个不错的操作，但我们还不能做太多，因为我们没有办法真正关联值、进行比较，甚至按我们的意图修改它们。

这是操作符发挥作用的地方，我们将看到如何修改变量的值，以便它能持有一个值，并随着时间推移逐步修改并收集新的信息。所以，让我们从简单的开始，先从基础数学入手，然后逐步过渡到更复杂的内容。

在继续之前，我们必须记住的一件事是，操作符遵循一个优先级顺序：

+   复合逻辑操作符 `-a`、`-o` 和 `&&` 的优先级较低

+   算术操作符的优先级如下：

+   +   乘

    +   除

    +   加

    +   减

+   具有相同优先级的操作符按从左到右的顺序进行求值

# 算术操作符

算术操作符做的就是你想的那样，也就是加、减、除等。即使没有特定的编程知识，我们也很熟悉这些操作。让我们来看一下它们每一个是如何用来操作变量的值的。

在继续之前，请记住，对于 shell 脚本，数字默认是十进制，除非在前面加上 `0` 表示八进制，`0x` 表示十六进制，或者使用 `base#number` 来表示基数为 base 的数值。

# + 操作符

这就像我们在小学时学到的内容；这个操作符允许我们将一个整数加到变量的值上，如下面的示例所示：

```
#!/bin/bash
echo "Hello user, please give me a number: "    
read user_input    
echo "And now another one, please: "    
read adding    
addition=$((user_input+adding))    
echo "The number is: ${user_input:-99}"
echo "The number added of ${adding} is: ${addition}"  

```

现在是时候调用脚本了：

```
zarrelli:$ ./useraddition.sh 
Hello user, please give me a number: 
120
And now another one, please: 
30
The number is: 120
The number added of 30 is: 150  

```

正如你可能已经注意到的，我们使用了双括号结构 `$(( ))` 来执行这种算术扩展和求值：简而言之，就是扩展并求值，然后返回值。这是二进制操作符中常见的符号，它还允许我们像用双引号括起来一样引用特殊字符，因此我们不必强制进行转义。唯一的例外是双引号，它仍然需要进行转义。别担心，我们稍后会进一步了解特殊字符以及如何引用它们。

现在，尝试运行脚本并且不提供数字：

```
The number is: 99
The number added of is: 0  

```

数字 `99` 不是赋值，它只是我们在变量没有有效值的情况下返回的默认值，但第一个和第二个变量没有赋值，因此将一个值加到另一个值上会导致 `0`。

# - 操作符

好了，在这种情况下，我们将从变量的值中减去一些东西，有一个小的警告：这是一个左结合操作，从左到右进行求值，这意味着我们正在将减号右侧的值从左侧的值中减去：

```
zarrelli:~$ a=20 ; b=5 ; c=$((a-b)) ; echo ${c}
15  

```

# * 操作符

对于乘法，我们不需要关注顺序；一个值与另一个值相乘，无论我们采取哪个方向：

```
zarrelli:~$ a=20 ; b=5 ; c=$((${a}*${b})) ; echo ${c}
100 

```

# / 操作符

除法是另一个左关联的运算，因此我们将除号左边的数字除以右边的数字：

```
zarrelli:~$ a=20 ; b=5 ; c=$((a/b)) ; echo ${c}
4   

```

# % 运算符

模除运算符给我们提供了两个整数相除的余数：

```
zarrelli:~$ a=29 ; b=5 ; c=$((a/b)) ; echo ${c}
5
zarrelli:~$ a=29 ; b=5 ; c=$((a%b)) ; echo ${c}
4  

```

# ** 运算符

正如我们在学校看到的那样，指数运算是一个数乘以它自己若干次，次数由指数决定：

```
zarrelli:~$ a=4 ; b=5 ; c=$((a**b)) ; echo ${c}
1024    
zarrelli:~$ a=4 ; c=$((a*a*a*a*a)) ; echo ${c}
1024  

```

在这种情况下，我们面对的是一个左关联操作，顺序很重要。请注意，在任何情况下，变量会在评估之前展开，所有我们到目前为止看到的运算符都会被应用。我们使用了`$a`和`${a}`，是为了让你习惯在实际生活中，查看你在互联网上会遇到的脚本。

# 赋值运算符

到目前为止，我们已经看到如何操作赋值给变量的值和整数，然后将这个值重新赋给另一个变量或相同的变量。但为什么要使用两个操作，而不是同时使用赋值运算符来修改变量的值并重新赋值呢？

# += 运算符

这个运算符将一个数量加到变量的值上，并将结果赋值给变量本身，但为了澄清它的用法，让我们重写我们之前见过的一个例子：

```
#!/bin/bash    
echo "Hello user, please give me a number: "    
read user_input    
echo "And now another one, please: "  

```

加法

```
echo "The user_input variable value is: ${user_input}"
echo "The adding variable value is: ${adding}"
echo "${user_input} added of ${adding} is: $((user_input+=adding))"
echo "And the user_input variable has now  the value of 
${user_input}"
echo"But the adding variable has still the value of ${adding}"  

```

现在，让我们按如下方式运行它：

```
zarrelli:~$ ./userreassign.sh 
Hello user, please give me a number: 
150
And now another one, please: 
50
The user_input variable value is: 150
The adding variable value is: 50
150 added of 50 is: 200
And the user_input variable has now the value of 200
But the adding variable has still the value of 50  

```

很简单！我们不需要显式地重新赋值 200。

# -= 运算符

事实上，这与前一个运算符非常相似，只是在这种情况下，我们做了减法并重新赋值。让我们用运算符重写我们的最后一个脚本，看看会发生什么：

```
zarrelli:~$ ./userreassign-subtract.sh 
Hello user, please give me a number: 
200
And now another one, please: 
50
The user_input variable value is: 200
The adding variable value is: 50
200 subtracted of 50 is: 150
And the user_input variable has now  the value of 150
But the adding variable has still the value of 50  

```

# *= 运算符

在这种情况下，我们将变量的值乘以给定的数字，并重新赋值：

```
zarrelli:~$ ./userreassign-multiply.sh 
Hello user, please give me a number: 
-1
And now another one, please: 
9223372036854775808
The user_input variable value is: -1
The adding variable value is: 9223372036854775808
-1 multiplied for  9223372036854775808 is: 
-9223372036854775808
And the user_input variable has now the value of 
-9223372036854775808
But the adding variable has still the value of 
9223372036854775808  

```

太好了！ 事实上，我们已经达到了现代 Bash 的一个边界：过去，变量保存的值可以用一个 32 位带符号长整型表示，但从 2.05b 版本开始，它切换到一个 64 位带符号整数，范围如下：

```
−9,223,372,036,854,775,808 to 9,223,372,036,854,775,807  

```

记住，包含逗号的值会被解释为字符字符串。

# /= 运算符

在这种情况下，我们将变量的值除以给定的数字，并重新赋值新值：

```
zarrelli@:~$ ./userreassign-division.sh 
Hello user, please give me a number: 
10
And now another one, please: 
2
The user_input variable value is: 10
The adding variable value is: 2
10 divided for 2 is: 5
And the user_input variable has now the value of 5
But the adding variable has still the value of 2  

```

# %= 运算符

使用模赋值，我们将变量的值除以给定的数字，并重新赋值余数。我们只需修改我们脚本中的几行：

```
echo "The value of ${user_input} divided by ${adding} is: $((user_input/=adding))"
echo "The remainder of ${user_input} divided by ${adding} is: $((user_input%=adding))"

```

然后执行它：

```
zarrelli@:~$ ./userreassign-modulo.sh 
Hello user, please give me a number: 
324
And now another one, please: 
12
The user_input variable value is: 324
The adding variable value is: 12
The value of 324 divided by 12 is: 27
The remainder of 27 divided by 12 is: 3
And the user_input variable has now  the value of 3
But the adding variable has still the value of 12  

```

# ++ 或 -- 运算符

这是一个一元运算符`++`（或`--`），它允许我们将变量的值增加/减少`1`并重新赋值，但要小心，运算符的位置很重要：

```
zarrelli:~$ a=15 ; echo $((a++)) ; echo ${a}
15
16
zarrelli:~$ a=15 ; echo $((++a)) ; echo ${a}
16
16  

```

你能弄清楚发生了什么吗？简单来说，在第一个例子中，第一个运算符返回了值，然后才加了 1；在第二个例子中，它首先加了 1 到值，然后返回它。请注意这个运算符，因为它在循环内部广泛使用，用于计数循环次数并最终跳出循环：

```
zarrelli:~$ cat loop.sh 
#!/bin/bash   
counter=10    
while [ $counter -gt 0 ]; 
do
echo"Loop number: $((counter--))"
done  

```

我们实例化一个名为`counter`的变量，初始值为`10`，然后定义一个循环，当`counter`的值大于`0`时，打印`counter`的值并将其减少，每次减少`1`。每个循环周期中，变量的值被打印，然后减小。当`counter`达到`0`时，`while`条件不再有效，循环停止：

```
zarrelli:~$ ./loop.sh 
Loop number: 10
Loop number: 9
Loop number: 8
Loop number: 7
Loop number: 6
Loop number: 5
Loop number: 4
Loop number: 3
Loop number: 2
Loop number: 1  

```

# 位运算符

位运算符在处理位掩码时非常有用，但在日常实践中，它们并不那么容易使用，因此你不太可能经常遇到它们。然而，由于它们在 Bash 中可用，我们将通过一些示例来查看它们。

# 左移 (<<)

位运算左移运算符每移位一次就将值乘以`2`；以下示例将使一切变得更清楚：

```
zarrelli:~$ x=10 ; echo $((x<<1))
20
zarrelli:~$ x=10 ; echo $((x<<2))
40
zarrelli:~$ x=10 ; echo $((x<<3))
80
zarrelli:~$ x=10 ; echo $((x<<4))
160  

```

发生了什么？正如我们之前所说，位运算符作用于位掩码，那么我们就从将整数`10`转换为其 16 位二进制表示开始，并使用一个`2`的幂表来检查其值。

在这种情况下，表示十进制数的二进制形式的简单方法是使用二的幂表示法，从将整数分解为幂和数的和开始。在我们的示例中，适合`10`的最大二的幂是`2³`，即`8`，加上`2¹`，即`2`。因此，在二的幂中，我们只选择`23`和`21`，并可以像下面的表格那样表示：

| **2⁷** | **2⁶** | **2⁵** | **2⁴** | **2³** | **2²** | **2¹** | **2⁰** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
| 0 | 0 | 0 | 0 | 1 | 0 | 1 | 0 |
|  |  |  |  | 8 |  | 2 |  |

如果我们用`1`标记`2`的幂，表示我们用来表示数字`10`的部分，用`0`标记未使用的部分，结果将是`1010`，或者如果使用 8 位表示数字，则为`00001010`。

现在我们要做的是将所有数字向左移动一个位置，但这会给我们带来一个问题，即右边的数字没有右边的数字可以替代它的位置。所以，对于最后一个右边的槽，我们将使用一个`0`：

```
10100  

```

| **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 0 | 0 | 1 | 0 | 1 | 0 | 0 |
|  |  |  | 16 |  | 4 |  |  |

而这个数字转换为十进制后是`20`。所以现在，让我们看看`echo$((x<<2))`会变成什么：

| **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 0 | 1 | 0 | 1 | 0 | 0 | 0 |
|  |  | 32 |  | 8 |  |  |  |

所以，我们得到了`40`。现在是时候进行`$((x<<3))`，将第一个`1`移出左侧，添加一个尾随的`0`并在开头添加一个符号位：

```
1010000  

```

| **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 1 | 0 | 1 | 0 | 0 | 0 | 0 |
|  | 64 |  | 16 |  |  |  |  |

我们达到了`80`；现在，让我们进行`$((x<<4))`，使用相同的步骤：

```
10100000  

```

| **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| 128 |  | 32 |  |  |  |  |  |

好的，`160` 就是结果，正如你能想象的那样。所以现在我们知道了一种非常快速的方式来将一个数字乘以二的幂，但是如果我们想要除法呢？

# 右移 (>>)

按位右移是将数字除以 `2` 的一种很好的方法，每次右移一位，即通过二的幂进行除法。请注意，这个操作会在左边填充最高有效位，也就是符号位，所以所有的位都会填充为 `1`，但下面的例子会让一切变得更简单：

```
zarrelli:~$ x=160 ; echo $((x>>1))
80
zarrelli:~$ x=160 ; echo $((x>>2))
40
zarrelli:~$ x=160 ; echo $((x>>3))
20
zarrelli:~$ x=160 ; echo $((x>>4))
10  

```

那么，我们从 `160` 开始：

```
10100000  

```

| **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| 128 |  | 32 |  |  |  |  |  |

接下来我们做的是将其右移一位：

```
1010000  

```

| **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 1 | 0 | 1 | 0 | 0 | 0 | 0 |
|  | 64 |  | 1 |  |  |  |  |

现在我们得到 `80`，但再次，左移一位：

| **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 0 | 1 | 0 | 1 | 0 | 0 | 0 |
|  |  | 32 |  | 8 |  |  |  |

现在，作为练习，自己计算剩下的值。

# 按位与

这是按位与运算符，它类似于逻辑与，但它在整数的二进制表示的位掩码上进行操作。二进制位从左到右读取，并在两个数字之间进行比较：如果在每个数字的相同位置上找到一个 `1`，结果将是 `1`，否则将是 `0`：

```
zarrelli:~$ x=50 ; y=20; echo $((x&y))
16  

```

我们是如何得到这个值的？让我们创建一个包含 `50` 和 `20` 二进制值的矩阵：

|  | **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 50 | 0 | 0 | 1 | 1 | 0 | 0 | 1 | 0 |
| 20 | 0 | 0 | 0 | 1 | 0 | 1 | 0 | 0 |
| & | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |

结果是二进制 `00010000`，十进制是 `16`。

# 按位或（|）

按位或运算符类似于包含性或，它通过二进制表示检查两个整数：如果在相同位置上每个数字都有一个 `1`，结果将是 `1`：

```
zarrelli:~$ x=50 ; y=20; echo $((x|y))
54 

```

正如我们从下面的表格中看到的：

|  | **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 50 | 0 | 0 | 1 | 1 | 0 | 0 | 1 | 0 |
| 20 | 0 | 0 | 0 | 1 | 0 | 1 | 0 | 0 |
| &#124; | 0 | 0 | 1 | 1 | 0 | 1 | 1 | 0 |

# 按位异或 (^)

这就是我们所谓的异或（XOR）：只有在同一位置上只有一个 `1` 时，结果才为 `1`，如果有两个 `1`，结果将是 `0`：

```
zarrelli:~$ x=50 ; y=20; echo $((x^y))
38  

```

所以，让我们再次检查我们的矩阵：

|  | **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 50 | 0 | 0 | 1 | 1 | 0 | 0 | 1 | 0 |
| 20 | 0 | 0 | 0 | 1 | 0 | 1 | 0 | 0 |
| ^ | 0 | 0 | 1 | 0 | 0 | 1 | 1 | 0 |

而结果就是 `36`。

# 按位非（~）

按位非是一个一元运算符，这意味着它只与一个操作符一起使用，翻转用于表示整数的二进制位：

```
zarrelli:~$ x=50 ; echo $((~x))
-51  

```

看一下下面的表格：

|  | **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 50 | 0 | 0 | 1 | 1 | 0 | 0 | 1 | 0 |
| ~ | 1 | 1 | 0 | 0 | 1 | 1 | 0 | 1 |
|  | -128 | 64 |  |  | 8 | 4 |  | 1 |

我们必须记住，对于一个带符号的整数（尽管我们没有写`+50`，因为 Bash 以 64 位符号长整型表示整数），最左边的最高位表示符号值。因此，翻转第一个有效位实际上反转了该值。经验法则是，按位取反的结果等于该数的二进制补码减去 1：

```
zarrelli:~$ x=30 ; echo $((~x))
-31  

```

|  | **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 30 | 0 | 0 | 0 | 1 | 1 | 1 | 1 | 0 |
| ~ | 1 | 1 | 1 | 0 | 0 | 0 | 0 | 1 |
|  | -128 | 64 | 32 |  |  |  |  | 1 |

然而，我们也可以通过不首先将整数转换为二进制，直接取反位并在结果中加上一个`1`来计算按位运算，这就是所谓的二进制补码：

| **50** | **0** | **0** | **1** | **1** | **0** | **0** | **1** | **0** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 反转 | 1 | 1 | 0 | 0 | 1 | 1 | 0 | 1 |
| 加 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 |
| -51 | 1 | 1 | 0 | 0 | 1 | 1 | 0 | 1 |

所以，正如你所看到的，在二进制补码中，最左边的位为`1`的二进制数是负数，而以`0`开头的是正整数。

# 逻辑运算符

这里，我们进入了一些对我们的脚本非常有用的内容，一些操作符将使我们能够执行测试并做出响应。因此，我们将能够让脚本对某些变化或用户输入作出反应，变得更加灵活。让我们看看有哪些操作符可用。

# 逻辑非（!）

非运算符用于测试表达式是否为真，当表达式为假时它为真：

```
[! expression ]  

```

让我们回到之前的一个脚本，并让它变得更加用户友好：

```
#!/bin/bash    
echo "Hello user, please give me a number between 10 and 12: "    
read user_input    
if [ ! ${user_input} -eq11 ]
then
echo "The number ${user_input} is not what we are looking for..."
else
echo "Great! The number ${user_input} is what we were looking for!"
fi  

```

我们在这里做的事情是要求用户输入一个介于`10`和`12`之间的数字。我们从用户的输入中读取值并进行评估：如果用户输入的值不等于`11`，那么我们输出一个“错误”语句；否则，我们找到了我们的数字。别担心，我们将在本书后面讨论`if...then...else`，现在只需将`if`理解为一个简单的条件语句。让我们运行脚本，看看当我们输入正确答案和错误答案时会发生什么：

```
zarrelli:~$ ./userinput-not.sh
Hello user, please give me a number between 10 and 12: 
11
Great! The number 11 is what we were looking for!    
zarrelli:~$ ./userinput-not.sh
Hello user, please give me a number between 10 and 12: 
12
The number 12 is not what we are looking for...  

```

太好了！脚本变得非常互动，根据我们施加的条件和给出的输入，它的输出发生了变化。

# 逻辑与

与运算符测试两个或多个表达式的成功性，当所有条件都为真时，它为真。这个操作符非常有用，可以让我们的脚本变得更加复杂，这样我们必须满足至少几个条件才能触发某些操作：

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

在这种情况下，我们要求几个条件必须同时成立：用户输入的数字必须大于等于`10`并且小于等于`20`。让我们看看：

```
zarrelli:~$ ./userinput-and.sh 
Hello user, please give me a number between 10 and 20: 
9
The number 9 is not what we are looking for...   
The number 9 is not a valid value: it is less than 20 but 
not bigger than 10.    
zarrelli:~$ ./userinput-and.sh 
Hello user, please give me a number between 10 and 20: 
10
Great! The number 10 is what we were looking for!  

```

是的，`10`是有效的，因为它等于`10`且小于`20`。这两个条件同时为真。我们有如下代码：

```
zarrelli:~$ ./userinput-and.sh 
Hello user, please give me a number between 10 and 20: 
11
Great! The number 11 is what we were looking for!  

```

对于`11`，我们是 OK 的，因为它大于`10`且小于`20`：

```
zarrelli:~$ ./userinput-and.sh 
Hello user, please give me a number between 10 and 20: 
19
Great! The number 19 is what we were looking for!  

```

仍然是一个有效的答案，因为它大于`10`且小于`20`：

```
zarrelli:~$ ./userinput-and.sh 
Hello user, please give me a number between 10 and 20: 
20
Great! The number 20 is what we were looking for!  

```

这应该是最后一个有效的答案。值大于`10`并等于`20`，但是`20`是我们的上限，再多一个就超出了我们的边界：

```
zarrelli:~$ ./userinput-and.sh 
Hello user, please give me a number between 10 and 20: 
21
The number 21 is not what we are looking for..

```

这里是：大于`10`但也大于`20`，而且只有当值小于等于`20`时，第二个条件才成立。所以，只有一个条件成立，另一个不成立，`21`不是我们要找的数字。

# 逻辑“或”运算符（||）

OR 运算符测试两个或多个表达式的成功，并且如果至少一个条件成立，它就成立：

```
#!/bin/bash    
echo "Hello user, please give me a number between 10 and 20 
or between 50 and 10: "    
read user_input  
if [[ ${user_input} -ge 10 && ${user_input} -le 20 ]] || 
[[ ${user_input} -ge 50 && ${user_input} -le 100 ]]
then
echo "Great! The number ${user_input} is what we were looking 
for!"
else
echo "The number ${user_input} is not what we are looking for..."
fi  

```

我们在这里看到了一些有趣的事情。首先，我们使用了复合条件测试，现在可以在四个不同的条件之间进行检查，按 2 分组。如果用户给我们的数字大于等于`10`且小于等于`20`，或者他输入一个大于等于`50`且小于等于`100`的数字，我们的测试就成立。

请注意，我们不得不使用带双括号`[[`的测试命令；这是 Bash 对单括号的改进，应该优先使用。实际上，`[` 是一个实际的二进制命令，你可以在操作系统中找到，而 `[[` 只是 Bash、**Zsh** 和 **Korn** shell 中可用的关键字。

双括号有一些非常有趣的改进。例如，它不会受到词分割或通配符扩展的影响，因此能更好地处理空格和空字符串，且你不需要给变量加引号。其他优点包括你不必在双括号内转义任何括号，并且你可以在其中使用`!`、`&&`和`||`来组合不同的表达式。在我们的示例中使用了`[]`测试运算符，只是为了熟悉它们，但你会发现，在大多数脚本中，你通常会遇到括号用于文件或字符串测试，而测试数字时，你更喜欢使用算术操作`$(())`，因为前者在算术操作中已被弃用。

# 逗号运算符（,）

另一个实际上不属于任何其他类别的运算符是逗号运算符，它用于将算术操作连接起来。所有操作都会被评估，但只有最后一个操作的值会被返回：

```
zarrelli:~$ echo $((x=1, 7-2))
5
zarrelli@moveaway:~$ echo ${x}
1  

```

# 运算符评估顺序和优先级，按降序排列

运算符的评估顺序是非常精确的，在使用它们时我们必须牢记这一点。记住哪个先评估哪个后评估并不容易，所以下面的表格将帮助我们记住运算符的顺序和优先级：

| **运算符** | **评估顺序** |
| --- | --- |
| ++ -- | 用于递增/递减的单目运算符，从左到右评估 |
| +- !~ | 单目加号和减号，从右到左评估 |
| * / % | 乘法、除法、取余，从左到右进行计算，并且在之后进行计算 |
| + - | 加法和减法从左到右进行计算 |
| <<>> | 位移运算符从左到右进行计算 |
| <= =><> | 比较运算符，从左到右 |
| == != | 等式运算符，从左到右 |
| & | 位运算 AND，从左到右 |
| ^ | 位运算 XOR，从左到右 |
| &#124; | 位运算 OR，从左到右 |
| && | 逻辑 AND，从左到右 |
| &#124;&#124; | 逻辑 OR，从左到右 |
| = += -+ */ /= %= &= ^= <<= =>> }= | 赋值运算符，从左到右 |

# 退出代码

我们已经看到，当程序遇到问题时，它会*退出*，通常会带有错误信息。退出意味着什么呢？简单来说，就是代码执行终止，程序或脚本返回一个退出代码，通知系统发生了什么。这对我们非常有用，因为我们可以捕获程序的退出代码，根据其值决定接下来要做什么。

| 0 | 成功 |
| --- | --- |
| 1 | 失败 |
| 2 | 内建命令滥用 |
| 126 | 命令不可执行 |
| 127 | 命令未找到 |
| 128 | 无效参数 |
| 128+x | 致命错误，退出时信号 x |
| 130 | 执行被 Ctrl + C 终止 |
| 255 | 退出状态超出边界（0-255） |

所以，或许你已经猜到，每次执行都会以退出代码结束，无论成功与否，是否带有错误信息或静默退出：

```
zarrelli:~$ date ; echo $?
Thu  2 Feb 19:17:48 GMT 2017
0  

```

如你所见，退出代码是 `0`，因为命令没有出现问题地执行完毕。现在，让我们尝试这个：

```
zarrelli:~$ asrw ; echo $?
bash: asrw: command not found
127  

```

它是一个`command not found`，因为我们刚刚输入了一串无意义的字符。现在，除非我们按下 Ctrl + C，否则 while 循环不会终止。

```
while true ; do echo 1 ; done  

```

你将看到屏幕上充满了一列无限的 1。按下 Ctrl + C，你会看到：

```
^C  

```

现在，让我们检查退出代码：

```
zarrelli:~$ echo $?
130  

```

现在，让我们创建一个永无止境的脚本：

```
#!/bin/bash    
while true; do    
echo ${$}    
done  

```

虽然 true 是一个永无止境的循环，因为条件始终为真，它将打印当前 shell 的 PID 到 `stdout`，脚本就在这个 shell 中运行。让我们打开第二个终端，并从第一个终端启动脚本；你将看到相同的 PID 无限重复：

```
1764
1764
1764
1764
1764  

```

现在，从第二个终端，使用相同的用户或 root 用户，执行：

```
zarrelli:~$ kill -9 1764  

```

返回第一个终端，你将看到你的脚本已终止：

```
1764
1764
1764
1764
Killed  

```

是时候检查脚本的退出状态了：

```
zarrelli:~$ echo $?
137  

```

这就是 `128 + 9`，`9` 是我们用来杀死进程的信号。让我们再次运行脚本：

```
1778
1778
1778
1778  

```

现在通过第二个终端使用以下命令结束它：

```
zarrelli:~$ kill -15 1778    
1778
1778
1778
1778 Terminated  

```

回到第一个终端，检查脚本的退出代码：

```
zarrelli:~$ echo $?
143 

```

而 `143` 正好是 `128 + 15`，如我们预期的那样。

# 退出脚本

到目前为止，我们已经看到脚本如何根据最后一个命令的退出状态来终止，以及`$?`如何让我们读取退出值。这是可能的，因为每个命令都会返回一个退出代码，无论是在命令行中发出的命令，还是在脚本内部发出的命令，甚至我们可以将其视为多个命令的组合的函数也会返回一个值。现在，我们将看到脚本如何根据其自身的情况返回一个退出码，而不依赖于最后一个命令的结果：

```
#!/bin/bash    
counter=10    
while [ $counter -gt 0 ]; 
do
echo "Loop number: $((counter--))"
done
exit 20  

```

我们拿取了之前的一个脚本并加上了这个：

```
exit 20  

```

如你所见，我们使用了`exit`命令，并跟随一个正整数来给出退出码。记住，你可以使用任何介于以下范围内的代码：

```
0-255  

```

除去上一章中我们看到的保留值后，现在让我们运行脚本：

```
zarrelli:~$ ./loop-exit.sh ; echo $?
Loop number: 10
Loop number: 9
Loop number: 8
Loop number: 7
Loop number: 6
Loop number: 5
Loop number: 4
Loop number: 3
Loop number: 2
Loop number: 1
20   

```

就是这样。由于最后的指令成功，脚本退出值是`0`，但因为`exit`命令的存在，我们得到了`20`作为退出值。

让我们稍微修改一下脚本，使得退出命令位于我们的`echo`命令之前：

```
#!/bin/bash    
counter=10
exit_at=5    
while (( counter > 0 ));
do
echo "Loop number: $((counter--))"
if (( $counter <exit_at )); then
exit 18
fi
done  

```

注意以下几点：

+   我们使用了`$(())`和`(())`。第一个是算术扩展，会给我们一个数字，第二个是一个命令，它会返回一个退出状态，以便我们可以读取*如果它为真（0）*，即计数器的值小于`exit_at`的值。

+   我们使用了一个条件来跳出这个无限循环。一旦计数器的值小于`exit_at`的值，我们就使用`18`的退出码退出整个脚本，而不管最后一个命令，即`if`条件的评估是失败的（值为 1）。

现在，执行以下脚本：

```
zarrelli:~$ /loop-premature-exit.sh ; echo $?
Loop number: 10
Loop number: 9
Loop number: 8
Loop number: 7
Loop number: 6
Loop number: 5
18  

```

就是这样。脚本一旦超过`5`的边界就退出了，所以剩下的五个`echo`命令根本没有被执行，我们得到了`18`作为退出值。所以，现在你有了一个方便的循环，可以用它遍历项目，并在满足某个条件时停止。

我们曾说过，`exit`命令会阻止脚本的进一步执行，之前的示例让我们对这一点有所了解，但现在让我们修改之前的循环脚本，把`exit 20`命令移动到前面几行：

```
#!/bin/bash    
counter=10    
while [ $counter -gt 0 ];
do
exit 20
echo"Loop number: $((counter--))"
done  

```

现在，让我们执行它：

```
zarrelli:$ ./loop-upper-exit.sh ; echo $?
20  

```

好的，没有输出，退出值为`20`。脚本没有时间到达`echo`这一行，它被迫在之前就退出了。现在，让我们看看`exit`命令是如何掩盖一个“命令未找到”的退出码的：

```
#!/bin/bash    
counter=10    
while [ $counter -gt 0 ]; 
do
echo "Loop number: $((counter--))"
fsaapoiwe
done
exit 20  

```

看看`echo`下方的那一行，也就是一堆没有任何意义的字符，它肯定会引发一个错误：

```
zarrelli:~$ ./loop-error-exit.sh ; echo $?
Loop number: 10
./loop-error-exit.sh: line 8: fsaapoiwe: command not found
Loop number: 9
./loop-error-exit.sh: line 8: fsaapoiwe: command not found
Loop number: 8
./loop-error-exit.sh: line 8: fsaapoiwe: command not found
Loop number: 7
./loop-error-exit.sh: line 8: fsaapoiwe: command not found
Loop number: 6
./loop-error-exit.sh: line 8: fsaapoiwe: command not found
Loop number: 5
./loop-error-exit.sh: line 8: fsaapoiwe: command not found
Loop number: 4
./loop-error-exit.sh: line 8: fsaapoiwe: command not found
Loop number: 3
./loop-error-exit.sh: line 8: fsaapoiwe: command not found
Loop number: 2
./loop-error-exit.sh: line 8: fsaapoiwe: command not found
Loop number: 1
./loop-error-exit.sh: line 8: fsaapoiwe: command not found
20  

```

它确实有用，在每个循环中我们都有错误，但总体的退出码仍然是`20`，因为我们通过`exit`命令强制了这一点。

使用`exit`命令我们能获得哪些好处？嗯，我们刚刚看到了一个简单易用的计数器，它可以在遍历数字、项目、数组、列表时非常有用，但从更广泛的角度来看，我们可以利用退出码检查我们创建的函数的结果，检查我们调用的命令的结果，并根据得到的值做出相应的反应。不过，要做到这一点，我们需要一种方法来验证并做出反应，检查一个条件是否满足，然后根据结果执行某些操作或其他操作。为了做到这一点，我们需要更深入地了解`if...else`语句以及测试运算符。

# 总结

在上一章，我们学习了变量；现在我们刚刚了解了如何关联它们。为变量赋值并能够对其进行一些数学或逻辑运算为我们提供了更多的灵活性，因为我们不仅仅是收集某些东西，而是将其转化为不同且全新的东西。我们还了解到，退出码有时可能成为陷阱，迷惑我们并引导我们误入歧途，这告诉我们一个重要的事情：永远不要把任何事情视为理所当然，始终仔细检查你在代码中写的内容，并始终尝试捕捉所有可能的结果和例外情况。这个看似显而易见，但由于它是如此常识，我们往往忽视了这种简单却有效的编码风格建议。
