## Typescript使用笔记

[Typescript文档](https://www.tslang.cn/docs/home.html)

### 基本类型

```typescript
// 字符串
let name: string = "bob";

// 数组
let list: number[] = [1, 2, 3];

//元组
let x: [string, number] = ["hello" , 10]

// 枚举 
enum Color { Red = 1, Green = 2, Blue = 4 }
let c: Color = Color.Green;
// 转换后
var Color;
(function (Color) {
    Color[Color["Red"] = 1] = "Red";
    Color[Color["Green"] = 2] = "Green";
    Color[Color["Blue"] = 4] = "Blue";
})(Color || (Color = {}));
var c = Color.Green;

// Any
let notSure: any = 4;
notSure.ifItExists(); 
notSure.toFixed(); 

let prettySure: Object = 4;
prettySure.toFixed();   // error
// Any类型可以赋予值并且调用任何方法,而Object类型只能调用Object上的方法
```

### 接口
> Typescript中的接口可用于对值所具有的结构进行类型检查

```typescript
// demo1
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}
let myObj = { size: 10, label: "Size 10 Object" };  
printLabel(myObj);

// demo2
interface LabelledValue {
    label: string;
}
function printLabel(labelledObj: LabelledValue) {
    console.log(labelledObj.label);
}
let myObj = { size: 10, label: "Size 10 Object" }; 
printLabel(myObj);

// demo1,demo2中都是对myObj进行赋值,myObj并没有定义类型。Typescript中当对象赋值给一个另一个变量时, myObj不会经过额外属性检查，所以编译器不会报错!!!
// demo3会报错
interface LabelledValue {
    label: string;
}
let myObj: LabelledValue = { size: 10, label: "Size 10 Object" };
/*
 Type '{ size: number; label: string; }' is not assignable to type 'LabelledValue'.
  Object literal may only specify known properties, and 'size' does not exist in type 'LabelledValue'.
*/ 

// 其他
interface SquareConfig {
    color?: string;            // 可选属性
    readonly x: number;        // 只读属性
    age: string | number       // 联合类型
    [propName: string]: any;   // 任意属性
}

// 函数类型
interface SearchFunc {
    (source: string, subString: string): boolean;  // 定义输入输出
}
let mySearch: SearchFunc = () => { return true };
mySearch(); //   error TS2554: Expected 2 arguments, but got 0.

// 可索引类型 
interface NumberDictionary {  // 当对象key为number类型,val必须为string类型
    [index: number]: string;
}
var obj: NumberDictionary = {
    1: '22',
}

// 存在索引属性时,索引属性的返回结果必须包含或者等于其他属性的返回结果
interface NumberDictionary {
  [index: string]: number; 
  length: number;    // 可以，length是number类型
  name: string       // 错误，`name`的类型与索引类型返回值的类型不匹配
}

// Javascript取值obj[key]或者obj.key,都会将key转换为string对象再去索引对象,所以数字索引的返回值必须是字符串索引返回值类型的子类型
interface NumberDictionary {
    [index: string]: number;
    [index: number]: any;   // any是number的子类型,没问题
}
```

### 类
> 类中默认的修饰符为public,外部实例可访问。protected和private在外部实例无法访问,protected在继承的子类中可以访问,private则不可以。

```typescript
// 举个栗子说明: fullName为成员变量可通过this访问,constrcutor中参数有修饰符时表示是成员变量,introduceSelf为原型方法
class Student {
    fullName: string;  
    constructor(public firstName, private middleInitial, public lastName, age:string) {
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
    introduceSelf() {
        console.log(`My name is ${this.fullName}`)
    }
}
```
转换后
```typescript
var Student = /** @class */ (function () {
    function Student(firstName, middleInitial, lastName) {
        this.firstName = firstName;
        this.middleInitial = middleInitial;
        this.lastName = lastName;
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
    Student.prototype.introduceSelf = function () {
        console.log("My name is " + this.fullName);
    };
    return Student;
}());
```

### 函数
> 定义函数的输入输出类型,参数个数不确定时需要进行重载声明

函数声明
```typescript

// 函数声明
function add1(x: number, y: number): number {
    return x + y;
}

// add2的类型会通过类型推论确定
let add2 = function(x: number, y: number): number { return x + y; };

// 指定add3的类型
let add3: (x: number, y: number) => number = function(x: number, y: number): number { return x + y; };

```

函数重载
```typescript
// 以下是func的2个重载声明,`func(x):any`并不是重载列表的一部分
function func(x:string):string;
function func(x:number):number;
function func(x):any{
   
}
```

### 泛型

```typescript
// 变量类型T,输入输出都是T类型
function identity<T>(arg: T): T {
    return arg;
}
let output = identity<string>("myString") // 显示声明
let output = identity("myString");      // 类型推论,编译器自动推断类型
```

```typescript
// 定义泛型函数
function identity<T>(arg: T): T {
    return arg;
}

// 函数类型中直接使用泛型
let myIdentity1: <U>(arg: U) => U = identity;

// 接口中使用泛型
interface GenericIdentityFn {
    <T>(arg: T): T;
}
let myIdentity2: GenericIdentityFn = identity;

// 泛型参数当作接口的一个参数
interface GenericIdentityFn<T> {
    (arg: T): T;
}
let myIdentity3: GenericIdentityFn<number> = identity;
```

```typescript
// 泛型继承接口
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  
    return arg;
}

```

### 类型推论
> Typescript中的类型推论发生在初始化变量和成员，设置默认参数值和决定函数返回值时

```js
// 

let x = 3;  // x推断为数字
x = "34";   // Type '"34"' is not assignable to type 'number'

// 推断为联合类型
let x = [0, "string"];        // number | string
let obj = [new A(), new B()]; // A | B

```

### 类型兼容性

```js

// 对象字面量  小 = 大
let x: { name: string };
let y = { name: 'Bob', age: "22" };
x = y;

// 函数返回结果  大 = 小
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'Seattle'});
x = y; // OK
y = x; // Error because x() lacks a location property

```

```js
// 参数个数 小 = 大
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```
### 模块

```js
// JQuery.d.ts
declare let $: JQuery;
export default $;

// 引入JQuery
import $ from "JQuery";

// 可以使用如下指令打包成各个模块加载系统使用的代码
tsc --module [commonjs | amd | umd | es6 | ...] index.ts
```


