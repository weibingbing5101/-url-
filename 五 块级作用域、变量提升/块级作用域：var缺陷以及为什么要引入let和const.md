### 仅支持全局作用域、函数作用域的老版js，现有问题

`
  function varTest() { 
    var x = 1; 
    if (true) { 
      var x = 2; // 同样的变量! 
      console.log(x); // 2 
    } 
    
    console.log(x); // 2
  }

`

有两个地方都定义了变量 x，

第一个地方在函数块的顶部，

第二个地方在 if 块的内部，

由于 var 的作用范围是整个函数，所以在编译阶段，会生成 同一个x变量，见 md文件同名-图1 png图片

从执行上下文的变量环境中可以看出，最终只生成了一个变量 x，函数体内所有对 x 的赋值操作都会直接改变变量环境中的 x 值



let + 块级作用域改造后，纠正了上面的问题

`
function letTest() { 
  let x = 1; 
  if (true) { 
    let x = 2; // 不同的变量 
    console.log(x); // 2 
  } 
  
  console.log(x); // 1
}
`


### ES6 又是如何在函数级作用域的基础之上，实现对块级作用域的支持呢

`

function foo(){
    var a = 1
    let b = 2
    {
      let b = 3
      var c = 4
      let d = 5
      console.log(a)
      console.log(b)
    }
    console.log(b) 
    console.log(c)
    console.log(d)
}   
foo()

`
上面代码执行时

1 编译并创建执行上下文 

    var 声明的变量，全被放到变量环境中。(这就是所谓变量提升)
    
        var a  var c 放到变量环境中，赋值undefined

    let 声明的变量，放到词法环境中的栈底

        let b (49行) 放到词法环境中的栈底，赋值undefined

    let b (51行)   let d ，在这步未做任处理


2 执行代码 foo()

    2-1 执行到50上面，变量环境中 a 赋值 1， 词法环境 b(49行) 赋值 2

    2-2 执行到50行后的块级，词法环境栈中再放入一层，b = undefined(51行) ， d = undefined

    2- 3执行 let b ， let d 赋值

    2-4 执行console.log a b , 由于词法环境栈中没有a，则继续向变量环境中查找。找到后返给javascript引擎。没有找到继续在变量环境中查找。

    2-5 50 - 56 作用域块执行结束后，定义变量从词法环境栈顶弹出。

    附图 见     块级作用域：var缺陷以及为什么要引入let和const-图2.png    







