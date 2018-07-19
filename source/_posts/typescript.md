---
title: TypeScript
date: 2018-06-18 15:34:21
categories:
- Node.js
tags:
- TypeScript
---
# Installation and IDE support
`npm install -g typescript`

`tsc -v`

`tsc main.ts`

`tsc *.ts --watch`

`ts-node server.ts`

VSC has built-in support of TypeScript.

[Online tool](http://www.typescriptlang.org/play/index.html)

[Template Project](https://github.com/superliuwr/api_template_typescript)

<!-- more -->

# Features
## Static Typing

* number – All numeric values are represented by the number type, there aren’t separate definitions for integers, floats or others.
* string – The text type, just like in vanilla JS strings can be surrounded by ‘single quotes’ or “double quotes”.
* boolean – true or false, using 0 and 1 will cause a compilation error.
* any – A variable with this type can have it’s value set to a string, number, or anything else.
* Arrays – Has two possible syntaxes: my_arr: number[]; or my_arr: Array<number>.
* void – Used on function that don’t return anything.
* tuple - `let x: [ string, number ] = [ "hello", 10 ]`
* enum - `enum Color { Red, Green, Blue }; let c: Color = Color.Green`
* null and undefined
* never
* union - `number | string`

## Interfaces

``` typescript
interface Food {
    name: string;
    calories: number;
    readonly x: number;
    color?: string;
}

interface LoggerInterface{
    log(arg:any):void;
}

// 函数接口
interface SearchFunc {
  (source: string, subString: string): boolean;
}

interface StringArray{
    [index: number]: string;
}
let MyArray: StringArray;
MyArray = ["是","云","随","风"];
console.log(MyArray[2]); // 随
```

## Functions

``` typescript
// 普通函数
function add(a: number, b?: number): number {
  return a + b;
}

// 函数参数
function readFile(file: string, callback: (err: Error | null, data: Buffer) => void) {
  fs.readFile(file, callback);
}

// 通过 type 语句定义类型
type CallbackFunction = (err: Error | null, data: Buffer) => void;
function readFile(file: string, callback: CallbackFunction) {
  fs.readFile(file, callback);
}

// 通过 interface 语句来定义类型
interface CallbackFunction {
  (err: Error | null, data: Buffer): void;
}
function readFile(file: string, callback: CallbackFunction) {
  fs.readFile(file, callback);
}
```

## Classes

``` typescript
class Menu {
  // Our properties:
  // By default they are public, but can also be private or protected.
  private items: Array<string>;  // The items in the menu, an array of strings.
  protected pages: number;         // How many pages will the menu be, a number.
  readonly other: string;

  get numOfPages(): number {
    return pages.length;
  }

  // A straightforward constructor. 
  constructor(item_list: Array<string>, total_pages: number) {
    // The this keyword is mandatory.
    this.items = item_list;    
    this.pages = total_pages;
  }

  // Methods
  public list(): void {
    console.log("Our menu for today:");
    for(var i=0; i<this.items.length; i++) {
      console.log(this.items[i]);
    }
  }

} 

// Create a new instance of the Menu class.
var sundayMenu = new Menu(["pancakes","waffles","orange juice"], 1);

// Call the list method.
sundayMenu.list();
```

Inheritance

``` typescript
class HappyMeal extends Menu {
  // Properties are inherited

  // A new constructor has to be defined.
  constructor(item_list: Array<string>, total_pages: number) {
    // In this case we want the exact same constructor as the parent class (Menu), 
    // To automatically copy it we can call super() - a reference to the parent's constructor.
    super(item_list, total_pages);
  }

  // Just like the properties, methods are inherited from the parent.
  // However, we want to override the list() function so we redefine it.
  list(): void{
    console.log("Our special menu for children:");
    for(var i=0; i<this.items.length; i++) {
      console.log(this.items[i]);
    }

  }
}

// Create a new instance of the HappyMeal class.
var menu_for_children = new HappyMeal(["candy","drink","toy"], 1);

// This time the log message will begin with the special introduction.
menu_for_children.list();
```

## Generics

Generics are templates that allow the same function to accept arguments of various different types. Creating reusable components using generics is better than using the any data type, as generics preserve the types of the variables that go in and out of them.

``` typescript
// The <T> after the function name symbolizes that it's a generic function.
// When we call the function, every instance of T will be replaced with the actual provided type.

// Receives one argument of type T,
// Returns an array of type T.

function genericFunc<T>(argument: T): T[] {    
  var arrayOfT: T[] = [];    // Create empty array of type T.
  arrayOfT.push(argument);   // Push, now arrayOfT = [argument].
  return arrayOfT;
}

var arrayFromString = genericFunc<string>("beep");
console.log(arrayFromString[0]);         // "beep"
console.log(typeof arrayFromString[0])   // String

var arrayFromNumber = genericFunc(42);
console.log(arrayFromNumber[0]);         // 42
console.log(typeof arrayFromNumber[0])   // number
```

## Modules

External modules as comparing to `Namespaces`.

## Third-party Declaration Files

When using a library that was originally designed for regular JavaScript, we need to apply a declaration file to make that library compatible with TypeScript. A declaration file has the extension .d.ts and contains various information about the library and its API.

TypeScript declaration files are usually written by hand, but there’s a high chance that the library you need already has a .d.ts. file created by somebody else. DefinitelyTyped is the biggest public repository, containing files for over a thousand libraries. There is also a popular Node.js module for managing TypeScript definitions called Typings.

当遇到缺少模块声明文件的情况，开发者可以尝试通过 npm install @types/xxx 来安装模块声明文件即可。

如果我们使用的第三方模块在 DefinitelyTyped 找不到对应声明文件，也可以尝试使用require()这个终极的解决方法，它会将模块解析成 any 类型，不好的地方就是没有静态类型检查了。

## Advanced Types and Type Guards

## Namespaces

模块是用来组织一些具有某种内在联系的特性和对象。模块能够使代码的结构更加清晰，可以使用 namespace 和 export 关键字在 TypeScript 中声明模块。

Internal modules as comparing to `Modules`.

``` typescript
namespace mygame{
    interface Test1Interface{
        /*...*/
    }

    export interface Test2Interface{
        /*...*/
    }
    export interface Test3Interface{
        /*...*/
    }

    export class Rect{
        /*...*/
    }

    export class Circle implements Test2Interface{
        /*...*/
    }
}
```

# Others
## tsconfig.json 配置文件

``` json
//tsconfig.json 指定了用来编译这个项目的根文件和编译选项。
{
  "compilerOptions": {     //compilerOptions:编译选项,可以被忽略，这时编译器会使用默认值
    "allowSyntheticDefaultImports": true,//允许从没有设置默认导出的模块中默认导入。这并不影响代码的显示，仅为了类型检查。
    "baseUrl": "./src",//解析非相对模块名的基准目录
    "emitDecoratorMetadata": true, //给源码里的装饰器声明加上设计类型元数据
    "experimentalDecorators": true,//启用实验性的ES装饰器
    "module": "commonjs",        //指定生成哪个模块系统代码
    "moduleResolution": "node",  //决定如何处理模块。或者是"Node"对于Node.js/io.js，或者是"Classic"（默认）
    "noEmitHelpers": true,//不再输出文件中生成用户自定义的帮助函数代码，如__extends。
    "noImplicitAny": false,     //在表达式和声明上有隐含的any类型时报错
    "sourceMap": true,          //用于debug ,生成相应的.map文件
    "strictNullChecks": false,//在严格的null检查模式下，null和undefined值不包含在任何类型里，只允许用它们自己和any来赋值（有个例外，undefined可以赋值到void）。
    "target": "es5",             //目标代码类型
    "paths": {  //模块名到基于baseUrl的路径映射的列表
    },
    "lib": [  //编译过程中需要引入的库文件的列表
        "dom",
        "es6"
      ],
    "types": [  //要包含的类型声明文件名列表；如果指定了types，只有被列出来的包才会被包含进来
      "hammerjs",
      "node",
      "source-map",
      "uglify-js",
      "webpack"
    ]
  },
  "exclude": [  //如果"files"和"include"都没有被指定，编译器默认包含当前目录和子目录下所有的TypeScript文件（.ts, .d.ts 和 .tsx），排除在"exclude"里指定的文件。
    "node_modules",
    "dist"
  ],
  "awesomeTypescriptLoaderOptions": {  //Typescript加载选项
    "forkChecker": true,
    "useWebpackText": true
  },
  "compileOnSave": false,          
  "buildOnSave": false
}
```

## TSLint 代码规范检查

## 发布模块

相比直接使用 JavaScript 编写的 npm 模块，使用 TypeScript 编写的模块需要增加以下几个额外的工作：
* 发布前将 TypeScript 源码编译成 JavaScript.
* 需要修改 tsconfig.json 的配置，使得编译时生成模块对应的 .d.ts 文件。
* 在 package.json 文件增加 types 属性。

``` json
{
  "compilerOptions": {
    "module": "commonjs",
    "moduleResolution": "node",
    "target": "es6",
    "rootDir": "src",
    "outDir": "dist",
    "sourceMap": true,
    "declaration": true,
    "noImplicitAny": true,
    "noImplicitReturns": true,
    "noImplicitThis": true
  }
}
```

``` json
{
  "main": "dist/math.js",
  "typings": "dist/math.d.ts",
  "scripts": {
    "compile": "rm -rf dist && tsc",
    "prepublish": "npm run compile"
  }
}
```

## 单元测试

要执行使用 TypeScript 编写的单元测试程序，可以有两种方法：
* 先通过 tsc 编译成 JavaScript 代码后，再执行。
* 直接执行 .ts 源文件。

``` json
{
  "scripts": {
    "test": "mocha --compilers ts:ts-node/register src/test.ts"
  }
}
```

# References
1. https://zhongsp.gitbooks.io/typescript-handbook/content/index.html
2. http://www.typescriptlang.org/docs/tutorial.html
3. https://legacy.gitbook.com/book/basarat/typescript/details

