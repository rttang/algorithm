#1简介
因为递归很适合解决分治问题所以下面首先会介绍**分治**的思想，然后会比较详细的介绍递归，并通过解决几个问题诸如斐波那契数列、阶乘、全排、n皇后等问题来更深入的加强对递归的认识。最后我们会回到js上，用递归解决一些实际问题，这里接受使用递归遍历DOM树以及使用递归实现沉复制。
#2分治简介
这里主要引用《算法笔记》里的定义（其实这个系列算是这本书的读书笔记吧。。）

> 分治（divide and conquer）全称分而治之，即分治法将原问题划分为若干个规模较小儿结构与原问题相同或者相似的子问题，然后分别解决这些子问题，最后合并子问题的解，即可得到原问题的解

步骤：
![图片描述][1]
注意：
* 分解的子问题应该相互独立、没有交叉。不然应该选择其它解决方法。
* 分治是一种思想，既可以使用递归也可以使用**非递归**手段实现

#3递归
递归就是反复调用自身函数，但每次调用时会吧范围缩小，直到范围缩小到可以直接得到边界数据的结果，然后在返回的路上求出对应的解。
递归式的**重要概念**：
* 递归边界：没有递归边界会导致无限递归，然后就会报错 `Maximum call stack size exceeded`
* 递归式
下面通过几个例子来深入了解一下递归：
##3.1使用递归求解n的阶乘
n!的计算公式：
$$ n!=1*2*3*.....*n $$ 
n!的递归式：
$$ F(n)=F(n-1)*n $$
有了递归式就可以很方便的写出递归函数：
```javascript
 function F(n=3){
       if(n==0) return 1;
        else return F(n-1)*n;
    }
    console.log(F())
```
为了方便理解，这里选取了3！数量较少递归树好画，同时给出了图来描述这个递归过程：
![3！递归图][2]
同时我们结合实际执行时的gif来动态的了解一下：
![实际执行][3]

这里着重注意一下最右侧的Call Stack,俗称调用堆栈，这个堆栈有个特点就是谁最后进去出来的时候最先出来，简称后进先出（LILO），调用堆栈存放的是函数。因为这个递归过程中每次递归都要保存一下当前函数，不然递归结束的时候没办法进行回溯。因为断点加到了函数里所有一进去Call Stack就有一个F，图一里的四层就对应Call Stack里放的四个F，然后我们再观察一下那个变量n，每次单步执行的时候n都在变化，步骤1时n=2...步骤3时n=0，当n=0的时候达到了当前的递归边界，结果开始返回，这时我们观察Call Stack下方的Scope：
![图片描述][4]

那个红框标识出来的变量Return value,这时就开始进入回溯阶段，就是步骤4至步骤7，Call Stack开始弹出之前保存的递归函数，每次都返回一个计算好的值，最后合并成最终结果。
##3.2求Fibonacci数列的第n项
Fibonacci数列是满足F(0)=1,F(1)=1,F(n)=F(n-1)+F(n-2)(n>=2)的数列。因此可以得出
* 递归边界：为F(0)=1,F(1)=1
* 递归式：为F(n)=F(n-1)+F(n-2)(n>=2)
从上面可以看出和3.1中的阶乘很像，我们参照着就可很快写出：
```javascript
 function Fib(n=4){
        if(n==0||n==1) return 1;
        else return Fib(n-1)+Fib(n-2);
    }
    console.log(Fib())
```
现在画一下它的递归树
![图片描述][5]

黑线是递归函数入栈，黄线是出栈，步骤1~17表示顺序。从这种图中我们可以小窥一下画同时调用多个自己的递归树的方法，因为代码都是顺序执行的，所以递归也要按顺序入栈出栈，遇到这种情况就先指着优先级最高的那个一直递归到递归边界，然后在返回的过程中按顺序递归剩下的即可。
##3.3全排
引用百度百科的解释：
>从n个不同元素中任取m（m≤n）个元素，按照一定的顺序排列起来，叫做从n个不同元素
 中取出m个元素的一个排列。当m=n时所有的排列情况叫全排列。
公式：全排列数f(n)=n!(定义0!=1)，如1,2,3三个元素的全排列为：
1,2,3
1,3,2
2,1,3
2,3,1
3,1,2
3,2,1
共3\*2*1=6种。

本例使用的是**字典序**（从小到大顺序排序）实现全排，而上段中的1，2，3的全排就是按照字典序排列的。
从递归角度分析，输出n的全排就可以分解为输出以1开头的全排、2开头的全排.....输出以n开头的全排。
定义数组save用于存放当前的排列，设定一个flag数组当flag[x]为true时表示整数x已经存在save中，当递归结束时需要还原。index表示排列位置。递归边界就是index==n+1，表示1~n位置都已经填好。
```javascript
function Permutation(n) {
        let flag = new Array(n + 1).fill(false);
        let save = new Array(n + 1).fill(0);
        let index = 1;
        return (function innerPer(index) {
            if (index == n + 1) {
                for (let i = 1; i <= n; i++) {
                    console.log(save[i]);
                }
                console.log('\n')
                return;
            }
            for (let x = 1; x <= n; x++) {
                if (!flag[x]) {
                    flag[x] = true;//每向下递归一次，进入if的次数少一
                    save[index] = x;//index在每层循环过程中是不变的
                    innerPer(index + 1);//仅在递归时发生变化
                    flag[x] = false;//递归结束还原状态
                }
            }
        })(index)
    }

    Permutation(3);
```
这个是正序输出，大家也可以实现一下逆序输出，或者画一画递归树加深一下印象。
3.4n皇后问题
n皇后问题很经典，该问题指的是在n*n的棋盘上放置n个皇后，使得这n个皇后不再同一行、同一列、同一对角线上，求合法的方案数量，下图就是n=5的一个合法方案。
![图片描述][6]

因为每一行每一列只能放置一个皇后，所以这个问题就转换为n的排列问题，例如上图按照行号就是24513。这样我们把3.3中的代码稍作修改就能解决现在的问题。（所以说数学好的人就是nb，唉。。）
我们在全排的代码上加上判断每两个皇后是否在对角线上的代码即可。
```javascript
 function Queen(n) {
        let flag = new Array(n + 1).fill(false);
        let save = new Array(n + 1).fill(0);
        let index = 1;
        let cnt = 0;

        return (function innerQ(index) {
            if (index == n + 1) {
                let judge = true;
                for (let i = 1; i <= n; i++) {
                    for (let j = i + 1; j <= n; j++) {
                        if (Math.abs(i - j) == Math.abs(save[i] - save[j])) {
                            judge = false;
                        }
                    }
                }
                if (judge) {
                    console.log(save, ++cnt);
                    console.log('\n');

                }
                return;
            }
            for (let x = 1; x <= n; x++) {
                if (!flag[x]) {
                    flag[x] = true;
                    save[index] = x;
                    innerQ(index + 1);
                    flag[x] = false;
                }
            }
        })(index)

    }
    Queen(10)
```
这里有个问题就是判断对角线冲突，这里采用两个一维数组解决了二维数组的问题，外层的for循环i，j为列号，一位数组里存的值为行号，通过观察可以知道，*如果两个棋盘格子处在对角的位置，那么他们的横坐标之差等于他们的纵坐标之差的绝对值*（斜字部分引用自[《运用全排列的方法解决八皇后问题》][7]）。
上面这种枚举所有情况然后挨个判断合法性的手段被称之为**暴力法**，通过观察可以发现只要发现第一次不合法那整个递归就可以返回，无须将递归进行到底再去判断，直接返回上层即可。这就引出了**回溯法**
**回溯法**就是在达到递归边界前的某层，由于一些事实导致已经不需要前往任何一个子问题递归，就可以直接返回上一层。
下面举出回溯法的代码，请与上方进行对比。
```javascript
    function ReQueen(n) {
        let flag = new Array(n + 1).fill(false);
        let save = new Array(n + 1).fill(0);
        let index = 1;
        let cnt = 0;

        return (function innerQ(index) {
            if (index == n + 1) {
                console.log(save, ++cnt);
                return;
            }
            for (let x = 1; x <= n; x++) {
                if (!flag[x]) {
                    let judge = true;
                    for (let pre = 1; pre < index; pre++) {//再次强调index代表位置
                        if (Math.abs(pre - index) == Math.abs(save[pre] - x)) {
                            judge = false;
                            break;
                        }
                    }
                    if (judge) {
                        flag[x] = true;
                        save[index] = x;
                        innerQ(index + 1);
                        flag[x] = false;
                    }
                }
            }
        })(index)
    }

    ReQueen(8)
```


  [1]: /img/bV8A4G
  [2]: /img/bV8B0h
  [3]: /img/bV8B0p
  [4]: /img/bV8B3E
  [5]: /img/bV8GbU
  [6]: /img/bV8Kbs
  [7]: https://blog.csdn.net/liyuxing6639801/article/details/76153225