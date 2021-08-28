---
title: Dart语法自学笔记（二）
date: 2021-05-26 17:10:54
tags:
    - Dart
    - flutter
cover: https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.33/articles/Dart语法自学笔记（一）/cover3.jpg
---

本篇介绍Dart的函数，类，抽象类，接口，泛型及库的使用
<!-- more -->
## 命名函数，匿名函数，箭头函数

**注意！！箭头函数只能写一个语句**

```dart
// void 可省略
void func () { 
  print('hello world');
};

// 匿名
() { print(123); };

// 箭头
func2 () => print(123);
```

## 函数的定义及调用

```dart
int sunNum(int n){
  var sum = 0;
  for (var i =1; i<=n; i++) {
    sum+=i;
  }
  return sum;
}

void main() {
  print(sumNum(10));
  // 55
}
```


## 可选参数， 默认参数，命名参数

- 可选参数，用 `[]` 括起来，且不能指定类型
- 默认参数，直接等于一个值就是默认参数，可指定类型
  ```dart
  String userInfo (String username, [age, String gender='m'] ) {
    if (age == null ) return '$username';
    return '$username, $age, $gender';
  }

  void main(){
    print(userInfo('没名字', 23));
    // 没名字, 23, m
  }
  ```

- 命名参数，和上面两种类似，如果要指定类型，则必须有默认值

```dart
yyy (String username, {age, String gender='male', job='programmer'}) {
  if (age == null || job == null) return '$username, 年龄保密, $gender';
  return '$username, 年龄保密, $gender, $job';
}

void main () {
  print(yyy('张三', age:23, gender: 'female'));
  // 张三, 年龄保密, male programmer
}
```

## 把函数作为参数传参
```dart
fn1() => print('fn1');
fn2(fn){
  fn();
}

void main() {
  fn2(fn1); // fn1
}
```

## 类和构造函数

- dart为单继承语言
- 类里的同名方法为构造函数，在构造函数生成之前还能执行其他的操作，写在括号和左花括号之间，如下：

```dart
class PersonY {
  String name;
  int age;

  // 且构造函数之前还能进行操作 :name='构造函数之前生成的name', age=999 .用的少
  PersonY(String name, int age): name='构造函数之前生成的name', age=999 {
    print('构造函数生成之前打印的内容:${this.name}--${this.age}');
    this.name=name;
    this.age=age;
  }
}

main() {
  PersonY p2 = new PersonY('李四', 45);
  print(p2.name); 
  // 构造函数生成之前打印的内容:构造函数之前生成的name--999
  // 李四
}
```

- 构造函数可以简写
- dart构造函数只有一个，但命名构造函数可以写多个：

```dart
class PersonZ {
  String name;
  int age;

  // 简写：
  PersonZ(this.name, this.age);
  // 命名构造函数：如果有属性，则一定要传入属性维护起来：例如这里的 this.name 和 this.age
  PersonZ.named(this.name, this.age) {
    print('这是命名构造函数');
  }
}

main() {
  // 调用命名构造函数仍要传参：
  PersonZ pp = new PersonZ.named('xxxx', 12);
  print(pp.name); 
  print(pp.age); 
}
```
以上执行输出如下：   

> 这是命名构造函数
> xxxx
> 12

## 私有属性

dart里并没有 `public` `private` `protected` 等关键字，定义私有属性时，必须：

- 以 `_` 为开头的属性
- 独立为一个文件，并且 `import` 到 `main` 方法里执行，才能达到私有属性的效果

```dart
class Pet{
  // 私有属性前加下划线,并且必须写在独立的文件里:
  String _name;
  int age;

  Pet(this._name, this.age);

  void printInfo() {
    // 类里可以直接访问私有属性_name, Pet外部则不能
    print('${this._name}====${this.age}');
  }

  // 私有方法_run
  void _run(){
    print('私有方法');
  }
  
  // 外部访问私有方法:
  execRun(){
    this._run();
  }
}
```

调用处：

```dart
// 1. 必须卸载独立的文件里,私有变量才生效:
import './08-私有属性/Pet.dart';

void main() {
  Pet a = new Pet('狗', 3);
  /*
  // print(a._name); 
  2. 报错,不能直接访问,_name为私有属性,须要定义方法访问
  */ 

  // 非私有属性,可以直接访问:
  print(a.age); // 3
  a.printInfo(); // 狗====3

  a.execRun(); //间接调用私有方法
}
```

## 连缀操作符

```dart
Person p2 = new Person('王五', 34);
p1..name = '李四'
  ..age = 30
  ..printInfo();

// 相当于
// p1.name = '李四';
// p1.age = 30;
// p1.printInfo();
```

## 继承

可利用 vs code 里的快捷方式进行操作，不用自己写：

![dart 继承的vs code操作](https://cdn.jsdelivr.net/gh/ys558/my-blog-imgs@0.34/articles/Dart语法自学笔记（二）/01.gif)

### `super` 关键字

```dart
class Person {
  String name;
  num age;
  Person(this.name, this.age);
  void printInfo() {
    print('父类的printInfo：${this.name}---${this.age}');
  }
}

class Web extends Person {
  String gender='m';
  Web(String name, num age, 
  // String gender 子类自己的属性： 
  String gender) : 
  // super 可用于继承父类的属性：
  super(name, age) {
    // 2. 属于该函数自由的构造函数写这里:
    this.gender = gender;
  }

  run(){
    // super 可用于继承父类的方法：
    super.printInfo();
    print('${this.name}--${this.age}--${this.gender}');
  }

  // 重写父类存在的方法: @override, 用于标示，可省略
  @override
  void printInfo() {
    print('子类重写的方法：${this.gender}-${this.name}');
  }
}
```

## `abstract` 抽象类

- 抽象类：
```dart
abstract class Pet {
  // 抽象方法，不写方法体：
  eat();
  run();

  // 抽象类里的普通方法：
  printInfo(){
    print('抽象类的普通方法');
  }
}
```

- 抽象方法被继承时均要重写：
```dart
class Dog extends Pet {
  @override
  eat() {
    // TODO: implement eat
    print('小狗吃骨头');
  }

  @override
  run() {
    // TODO: implement run
    print('小狗在跑');
  }
}
```

### `implements` 接口 

- dart里没有接口的概念，用抽象类替代
- 接口继承时，用关键字 `implements` 

下面的例子举连接数据库的抽象类和实例：
```dart
abstract class Db {
  String uri=''; //数据库连接地址
  add(String data);
  save();
  delete();
}

class Mysql implements Db {
  @override
  String uri;

  Mysql(this.uri);

  @override
  add(String data) {
    // TODO: implement add
    print('这是mysql的add方法:${data}');
  }

  @override
  delete() {
    // TODO: implement delete
    return null;
  }

  @override
  save() {
    // TODO: implement save
    return null;
  }
  
}
```

- 一个类可 `implements` 多个接口：

```dart
abstract class A { printA(); }
abstract class B { printB(); }
class C implements A, B {
  @override
  printA() {
    // TODO: implement printA
    return null;
  }

  @override
  printB() {
    // TODO: implement printB
    return null;
  }
}
```

## `mixins` 混合

- dart 特有的特性，因为dart是单继承，`mixins` 可让其实现实现类似多继承的特性，一个类可 `mixins` 多个类
- 实现该特性用 `with` 关键字
- 被 `mixins` 的父类不能有构造函数，如有构造函数，不能被 `mixins`

```dart
// D 和 E 没有构造函数：
class D { printD(){ print('D'); }}
class E { printE(){ print('E'); }}
class F with D, E {
}

main() {
  F f = new F();
  f.printD(); // D
  f.printE(); // E
}
```

- 有构造函数被 `mixins`的方法：`extends` 后再 `with`, 如下例子:

```dart
class Person {
  String name;
  int age;
  // 构造函数:
  Person(this.name, this.age);
  void printInfo(){ print('${this.name}-${this.age}');}
}
class G { void printG(){print('G');} }
// P 继承自Person, 且混合G: 因Person中有构造函数Person(this.name, this.age); 故不能直接with
class P extends Person with G {
  P(String name, int age) : super(name, age);
}

main() {
  P p = new P('呵呵',37);
  p.printInfo(); // 呵呵-37
  p.printG(); // G
}
```

## 泛型

泛型即定义传入和返回类型的校验的一种规范

```dart
// 1. 定义一种泛型，返回的是泛型 T，传入的是类型 <T>，
T getData <T> (T value){
  print(value);
}

main() {
  // 2. 调用泛型方法并指定类型：
  print(getData <String> ('你好')); // 你好
  print(getData <int> (123)); // 123
}
```

- 泛型用于类

```dart
// <T> 将检验类型作为参数传入
class PrintClass <T> {
  List list = <T>[];
  // T 将检验类型作为参数传入
  void add(T value) {
    this.list.add(value);
  }
  void printInfo() {
    for(var i=0; i<this.list.length; i++) {
      print(this.list[i]);
    }
  }
}

main() {
  // 调用时须传入检验的类型 <String>
  PrintClass p = new PrintClass <String> ();
  p.add('xxxxxxx');
  p.printInfo(); // xxxxxxx
}
```

- 泛型用于接口

```dart
abstract class Cache <T> {
  getByKey(String key);
  void setByKey(String key, T value);
}
// 3.1 实现:
class FileCache <T> implements Cache <T> {
  @override
  getByKey(String key) {
    return null;
  }

  @override
  void setByKey(String key, T value) {
    print('我是文件缓存, 把key=${key} value=${value}的数据写入内存中');
  }
}

main() {
  MemoryCache m = new MemoryCache <String> ();
  m.setByKey('index', '123'); // 我是内存缓存, 把key=index value=123的数据写入内存中
}
```

## 库的使用

- 系统内置库，如：

```dart
import 'dart:io';
import 'dart:convert';

_getDataFromZhihu() async {
  var httpClient = new HttpClient();
  var uri = new Uri.http('news-at.zhihu.com', '/api/3/stories/latest');
  var request = await httpClient.getUrl(uri);
  var response = await request.close();
  return await response.transform(utf8.decoder).join(' ');
}
main(List<String> args) async {
  var result = await _getDataFromZhihu();
  print(result);
}
```

- 第三方包管理系统pub，官网在[这里](https://pub.dev/packages)，功能类似[npm](https://npmjs.org/)的网站，这里举例 [`http` 库](https://pub.dev/packages/http)的使用

1. 引入，在根目录下写下一个`pubspec.yaml` 文件，类似 `package.json`
2. 写下你需要安装的库的版本，如下，可以在[这里](https://pub.dev/packages/http/install)拷贝其版本号
```yaml
name: xxx
description: A new flutter module.
environment:
  sdk: '>=2.10.0 <3.0.0'
dependencies:
  http: ^0.13.3
```

2. 执行安装

方法一，在`pubspec.yaml` 文件写好 `dependencies`，再执行如下：

```bash
dart pub get
```

方法二，直接执行以下命令，安装最新版本
```bash
dart pub add http_parser
```

3. 使用

pub 库是以 `package` 开头：
```dart
import 'package:http/http.dart' as http;
```