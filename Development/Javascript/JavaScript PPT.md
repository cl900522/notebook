# 第一张
## 前言
>JavaScript 是唯一一门可以先用后学的编程语言(也许还包括 CSS 和 HTML)。

## 基本语法
### 内置类型
> 除了null,可以通过typeof运算符识别

类型        | 名称           | 备注
--------- | ------------ | -------------------------------------
string    | 字符串          |
number    | 数字           |
boolean   | 布尔           |
undefined | 未定义          | 使用var声明变量但未对其加以初始化时，这个变量的值就是undefined
null      | 空            |
object    | 对象,是一种复杂数据类型,Date Function. RegExp都是继承自Object |

1. **数组和函数不是不是**
2. **null值表示一个空对象指针，而这也正是使用typeof操作符检测null时会返回"object"**正确的返回结果应该是 "null"，但这个 bug 由来已久，在 JavaScript 中已经存在了将近二十年，也许永远也不会修复，因为这牵涉到太多的 Web 系统，“修复”它会产生更多的 bug，令许多系统无法正常工作。
3. **undefined值是派生自null值的，因此ECMA-262规定对它们的相等性测试要返回true,管null和undefined有这样的关系，但它们的用途完全不同**(undefined == null)
4. typeof function a(){} === "function"; 这样看来，function（函数）也是 JavaScript 的一个内置类型。然而查阅规范就会知道，它实际上是 object 的一个“子类型”。

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
        var globe = 12;
        console.log(globe);
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

## 面向对象
### 类和继承
javascript没有类的定义，而是通过函数以及原型链来实现类和继承的概念
```javascript
    function Person() {
        this.name;
        this.birthday;
    };
    function Engineer() {
        this.major;
    };
    Engineer.prototype = new Person();
    var xiaoming = new Engineer();
    console.log(xiaoming instanceof Engineer);
    console.log(xiaoming instanceof Person);
    xiaoming.name = "xm";
    xiaoming.age = 22;
    xiaoming.major = "Android";
    console.log(xiaoming);
```

### 其他继承实现方式
1. 对象冒充
2. call()方法
3. apply()方法
**可实现多继承**
**都无法通过instanceof来判断继承的上级类型**
```javascript
    function Base1() {
        this.name;
        this.say = function() {
            console.log("name=" + this.name);
        }
    }
    function Base2() {
        this.age;
        this.say = function() {
            console.log("age=" + this.age);
        }
    }
    function ClassZ() {
        this.newMethod = Base1;
        this.newMethod();
        delete this.newMethod;

        this.newMethod = Base2;
        this.newMethod();
        delete this.newMethod;
    }
    var a = new ClassZ();
    a.name = "a";
    a.age = 12;
    a.say(); //"a"
    console.log(a instanceof ClassZ); //true
    console.log(a instanceof Base1); //false
    console.log(a instanceof Base2); //false

    // call and apply
    var obj = new Object();
    Base1.call(obj);
    //Base1.apply(obj);
    obj.name = "obj";
    obj.age = 12;
    obj.say(); //"a"
    console.log(obj instanceof Base1); //false
    //console.log(obj instanceof Base2); //false
```
