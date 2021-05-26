---
title: Dart语法自学笔记（二）
date: 2021-05-26 17:10:54
tags:
---

本篇介绍Dart的函数

<!-- more -->
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

## 把函数作为参数
```dart
fn1() => print('fn1');
fn2(fn){
  fn();
}

void main() {
  fn2(fn1); // fn1
}
```

## 