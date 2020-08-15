### 调用栈：为什么js代码会栈溢出

一、 调用栈

        用来管理函数调用关系的一种数据结构。

        执行上下文栈，又称调用栈

        例：

        `
            var a = 2
            function add(b,c){ 
                return b+c
            }

            function addAll(b,c){
                var d = 10
                result = add(b,c)
                return a+result+d
            }

            addAll(3,6)
        `

        1. 创建全局上下文变量环境。压入栈底

            变量 a、函数 add 和 addAll 都保存到了全局上下文的变量环境对象中

            全局执行上下文压入到调用栈后，JavaScript 引擎便开始执行全局代码了

            执行 a=2 的赋值操作，执行该语句会将全局上下文变量环境中 a 的值设置为 2



        2. 调用addAll函数

            addAll 函数的执行上下文创建好之后，便进入了函数代码的执行阶段了，

            这里先执行的是 d=10 的赋值操作，执行语句会将 addAll 函数执行上下文中的 d 由 undefined 变成了 10



        3. 执行add函数

            该函数的执行上下文就会从栈顶弹出，并将 result 的值设置为 add 函数的返回值，也就是 9

            紧接着 addAll 执行最后一个相加操作后并返回，addAll 的执行上下文也会从栈顶部弹出，此时调用栈中就只剩下全局上下文了


二、 函数调用

        函数调用就是运行一个函数

        例：
        `
            var a = 2
            function add(){
                var b = 10
                return a+b
            }
            add()
        `

        1、创建全局执行上下文变量环境

            var a;
            function add = function(){ 此处省略 }

        2、全局执行上下文中，取出 add 函数代码执行

        3、对 add 函数的这段代码进行编译，并创建该函数的执行上下文和可执行代码
            var b
            b = 10

        4、执行代码，输出结果 执行的是add()


三、 什么是栈
        
        一条单行道，一头被堵，车子往里面开。

        堵住的单行线就可以被看作是一个栈容器，车子开进单行线的操作叫做入栈，车子倒出去的操作叫做出栈。


四、 栈溢出

        调用栈是有大小的，当入栈的执行上下文超过一定数目，JavaScript 引擎就会报错，我们把这种错误叫做栈溢出

        写递归代码的时候，就很容易出现栈溢出的情况

        函数是递归的，并且没有任何终止条件

        `
            function division(a,b){ 
                return division(a,b)
            }

            console.log(division(1,2))
        `












            
