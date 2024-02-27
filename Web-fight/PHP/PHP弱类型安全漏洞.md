# PHP弱类型安全漏洞

> 由于php是弱类型语言，不用在变量前面加类型，因此容易发生变量的类型转换问题，并产生许多意想不到的问题，我们可以在任何地时候将变量赋值给任意的其他类型的变量，同时变量也可以转换成任意地其他类型的数据



## 1.PHP中的比较运算符

| 例子       | 名称                       | 结果                                                         |
| ---------- | -------------------------- | ------------------------------------------------------------ |
| \$a == $b  | 等于                       | **`TRUE`**，如果类型转换后 $a 等于 $b。                      |
| \$a===$b   | 全等                       | **`TRUE`**，如果 $a 等于 $b，并且它们的类型也相同。          |
| \$a != $b  | 不等                       | **`TRUE`**，如果类型转换后 $a 不等于 $b。                    |
| \$a <> $b  | 不等                       | **`TRUE`**，如果类型转换后 $a 不等于 $b。                    |
| \$a !== $b | 不全等                     | **`TRUE`**，如果 $a 不等于 $b，或者它们的类型不同。          |
| \$a < $b   | 小与                       | **`TRUE`**，如果 $a 严格小于 $b。                            |
| \$a > $b   | 大于                       | **`TRUE`**，如果 $a 严格大于 $b。                            |
| \$a <= $b  | 小于等于                   | **`TRUE`**，如果 $a 小于或者等于 $b。                        |
| \$a >= $b  | 大于等于                   | **`TRUE`**，如果 $a 大于或者等于 $b。                        |
| \$a <=> $b | 太空船运算符（组合比较符） | 当\$a小于、等于、大于$b时,分别返回一个小于、等于、大于0的[integer] 值。PHP7开始提供. |

## 2.类型转换规则

> 变量声明时不需要定义类型，变量的类型由存储的值决定，PHP可能会尝试在某些上下文中自动将值转换为另一种类型

### 2.1 Numeric

当使用算数运算符时，任一对象为`float`，则两个运算对象和结果都将解释为`float`，否则，全部解释成int。

自PHP8.0开始，如果无法解释其中一个运算对象，则会抛出 `TypeError`

### 2.2 String

当使用echo，print时值会解释成`string`

### 2.3 Logical

当使用条件，逻辑语句时值会解释成`bool`

### 2.4 Integral & String

当使用位运算符时，如果所有的运算对象都是`string`,那运算结果也是`string`，否则都将解释为`int`

### 2.5 ==Comparative==

| 运算数 1 类型                                                | 运算数 2 类型                                                | 结果                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [null](https://www.php.net/manual/zh/language.types.null.php) 或 [string](https://www.php.net/manual/zh/language.types.string.php) | [string](https://www.php.net/manual/zh/language.types.string.php) | 将 **`null`** 转换为 ""，进行数字或词汇比较                  |
| [bool](https://www.php.net/manual/zh/language.types.boolean.php) 或 [null](https://www.php.net/manual/zh/language.types.null.php) | 任何其它类型                                                 | 转换为 [bool](https://www.php.net/manual/zh/language.types.boolean.php)，**`false`** < **`true`** |
| [object](https://www.php.net/manual/zh/language.types.object.php) | [object](https://www.php.net/manual/zh/language.types.object.php) | 内置类可以定义自己的比较，不同类不能比较，相同的类查看[对象比较](https://www.php.net/manual/zh/language.oop5.object-comparison.php) |
| [string](https://www.php.net/manual/zh/language.types.string.php)、[float](https://www.php.net/manual/zh/language.types.float.php)、[resource](https://www.php.net/manual/zh/language.types.resource.php)、[int](https://www.php.net/manual/zh/language.types.integer.php) | [string](https://www.php.net/manual/zh/language.types.string.php)、[float](https://www.php.net/manual/zh/language.types.float.php)、[resource](https://www.php.net/manual/zh/language.types.resource.php)、[int](https://www.php.net/manual/zh/language.types.integer.php) | 将字符串和资源转换成数字，按普通数学比较                     |
| [array](https://www.php.net/manual/zh/language.types.array.php) | [array](https://www.php.net/manual/zh/language.types.array.php) | 成员越少的数组越小，如果运算数 1 中的键不存在于运算数2 中则数组无法比较，否则挨个值比较 |
| [object](https://www.php.net/manual/zh/language.types.object.php) | 任何其它类型                                                 | [object](https://www.php.net/manual/zh/language.types.object.php) 总是更大 |
| [array](https://www.php.net/manual/zh/language.types.array.php) | 任何其它类型                                                 | [array](https://www.php.net/manual/zh/language.types.array.php) 总是更大 |

常见例子：

```php
// Bool 和 null 总是作为 bool 比较
var_dump(1 == TRUE);  // TRUE - same as (bool)1 == TRUE
var_dump(0 == FALSE); // TRUE - same as (bool)0 == FALSE
var_dump(100 < TRUE); // FALSE - same as (bool)100 < TRUE
var_dump(-10 < FALSE);// FALSE - same as (bool)-10 < FALSE

var_dump(0 == "a"); // 0 == 0 -> true
var_dump("1" == "01"); // 1 == 1 -> true
var_dump("10" == "1e1"); // 10 == 10 -> true （解释：e是exponent(指数)的缩写）
var_dump(100 == "1e2"); // 100 == 100 -> true
var_dump(100 == "10a0"); // 100 == 10 -> false （string从前往后取数字遇到非e的就停了）

''==0==false==NULL
'123'==123
'abc'==0
'123a'==123
'0x01'==1
'0e123456789'=='0e987654321'
[false] ==[0]==[NULL]==['']
true == 1
```

