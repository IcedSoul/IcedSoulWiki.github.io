# Java面试知识点

## Java基础知识

- String是final不可被继承，“String不可变”

- 如果一个String已经被创建过了，那么会直接从String Pool中获得引用。
  - 只有String是不可变的，才可能使用String Pool（为什么？）
  - 假如String可以继承，AString和BString都继承了String，这个时候使用StringPool在比较是否存在的时候可能存在问题
- 泛型主要目的是为了参数化类型，Java1.5开始引入，Java的泛型是伪泛型，在编译的时候会确定具体的类型。