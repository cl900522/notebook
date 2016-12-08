# 第一张

## 基本语法
### 基础类型
> 除了null,可以通过typeof运算符识别

类型        | 名称           | 备注
--------- | ------------ | -------------------------------------
string    | 字符串          |
number    | 数字           |
boolean   | 布尔           |
undefined | 未定义          | 使用var声明变量但未对其加以初始化时，这个变量的值就是undefined
null      | 空            |
object    | 对象,是一种复杂数据类型 |

1. **数组和函数不是不是**
2. **null值表示一个空对象指针，而这也正是使用typeof操作符检测null时会返回"object"**
3. **undefined值是派生自null值的，因此ECMA-262规定对它们的相等性测试要返回true,管null和undefined有这样的关系，但它们的用途完全不同**(undefined == null)

### 数据对比和转换
```javascript
    var value = 12;
    console.log(value == "12"); //true
    console.log(value === "12"); //false
```

```javascript
    var effect = true;
    console.log(effect.toString()); // "true"
    console.log(effect == "true"); //false
    console.log(effect === "true"); //false
    console.log(effect == 1); //true
    console.log(effect === 1); //false
    console.log(effect == "1"); //true
    console.log(effect === "1"); //false
```

```javascript
    var name = null;
    name = (name)? "default": name; //name="defalut"
```

```javascript
    var a1=undefined;
    var a2=null;
    var a3="";
    var a4=0
    var a5="ddddd";
    var result=a1||a2||a3||a4||a5;
    console.log(result); //"ddddd"

    var result=!!(a1||a2||a3||a4||a5);
    console.log(result); //true
```

### 作用域
>**javascript只有函数作用域和全局作用域，函数内部变量申明不加var即可提升变量作用域为全局**

```javascript
    function fun1() {
        globe = 8;
        console.log(globe);
    }
    function fun2() {
        var temp = 12;
        console.log(temp);
    }
    function fun3() {
        globe = 10;
        console.log(globe);
    }
    fun1(); //8
    fun2(); //12
    fun3(); //10
    console.log(window.globe); //10
```

## 函数
### 函数简介
>函数也是对象（可作为参数进行调用、传参），函数变量和函数执行是两个完全不同的概念。函数申明时可以不指定参数列表，所以**javascript不存在函数重载**
```javascript
    function fun1(arg) {
        alert(arg);
    }
    function fun1(arg1, callBack) {
        if (!callBack) {
            console.log(arg1);
        } else {
            callBack(arg1);
        }
    }
    function autIncrease(arg) {
        arg++;
        alert(arg);
    }
    fun1(1, autIncrease);
```

### 函数执行方式
1. 函数名加括号，括号内引入参数即可
2. call，第一个参数为执行对象，后面的参数为入参。逐个传入
3. apply，第一个参数为执行对象，后面的参数为入参。数组形式

```javascript
    function say() {
        if(arguments.length == 0) {
            console.log("你好:" + this.name);
        } else {
            var preFix = "";
            for(var i=0; i<arguments.length; i++) {
                preFix += arguments[i];
            }
            console.log(preFix + ":" + this.name)
        }
    }
    window.name = "Window";
    var xiaoming = {name:"xiaoming"};
    say();
    say.call(xiaoming, "Hi,", "Good morning");
    say.apply(xiaoming, ["Hi,", "Good night"]);
```

### 请思考以下问题
```javascript
    for (var i=0; i<10; i++) {
        setTimeout(function() {
            console.log("out:" + i);
        }, 10);
    }
```

```javascript
    function createConsole(i) {
        return function() {
            console.log("out:" + i);
        }
    }
    for (var i=0; i<10; i++) {
        setTimeout(createConsole(i), 10);
    }
```
