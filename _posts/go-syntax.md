---
title: go语法学习
date: 2020-11-14 19:11:48
tags: 
---
go语言基础组成：包声明，引入包，函数，变量，语句&表达式，注释。
<!-- more -->
当标识符（常量/变量/类型/函数名/结构字段）以一个大写字母开头，如Group1，那么该标识符对象就可以被外部包的代码所使用，这被称为导出。如果标识符是小写字母开头，则对外部包是不可见的，但是在整个包的内部是可见并且可用的。
go语言数据类型：
1.布尔型 例：var b bool = true
2.数字类型 整形int和浮点型float32，float64
3.字符串类型
4.派生类型
	包括 1.指针类型（Pointer）
		2.数组类型
		3.结构化类型（struct）
		4.Channel类型
		5.函数类型
		6.切片类型
		7.接口类型（interface）
		8.Map类型

## 变量声明
var identifier type
例：var a string = "my" 也可以简写为a := "my"
多个变量声明var id1, id2 type

## 常量
显示类型定义：const b string = "abc"
隐式类型定义：const b = "abc"

## 循环语句
死循环写法 `for true {}`

## 函数
定义如下
```go
func function_name([paramter list]) [return_types] {
	body
}
例：
func max(num1, num2 int) int {
	var result int
	if (num1 > num2) {
		result = num1
	} else {
		result = num2
	}
	return result
}
```
函数返回多个值
Go函数可以返回多个值，例：
```go
func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("A", "B")
	fmt.Println(a, b)
}
```

## 变量作用域
### 局部变量
函数内定义的变量
### 全局变量
函数外定义的变量
### 形式参数
形式参数作为函数的局部变量使用

## 数组
### 声明
var variale_name [SIZE] variable_type
var arr [10] float32
### 初始化数组
var arr = [3]float32{10.0, 2.0, 1.1}
其中，[]中数字可以忽略
var arr = [...]float32{10.0, 2.0, 1.1}

## 指针
### 声明
var var_name *var_type
var ip *int
### 空指针
当一个指针定义以后没有分配到任何变量时，值为nil

## 结构体
定义结构体
type struct_variable_type struct {
	member definition
	...
}
变量声明
variable_name := struct_variable_type {value1, value2...valuen}
或
variable_name := struct_variable_type {key1: value1, key2: value2..., keyn:valuen}
访问结构体成员使用点号`.`操作符

## 切片（Slice）
动态扩容
var identifier []type
或使用make()函数创建切片
var slice1 []type = make([]type, len)
简写为 slice1 := make([]type, len)
### 切片初始化
s :=[] int {1, 2, 3}

使用len()函数获取长度
使用cap()函数获取容量
使用append()方式追加元素

## Range
range关键字用于for循环中迭代数组（array），切片（slice），通道（channel）或集合（map）的元素。在数组和切片中返回元素的索引和索引对应的值，在集合中返回key-value对。
```go
nums := []int{2, 3, 4}
sum := 0
// 在数组上使用range将传入index和值两个变量。如果不需要使用该元素的序号，使用空白符"_"省略。
for _,num := range nums {
	sum += num
}
for i, num := range nums {
	if num == 3 {
		fmt.Println("index:", i)
	}
}
```

## map
定义map
var map_variable map[key_data_type]value_data_type
使用make函数
map_variable := make(map[key_data_type]value_data_type)
```go
var cMap map[string]string
cMap = make(map[string]string)

cMap ["abc"] = "def"

for country := range cmap {
	fmt.Println(country, cMap[country])
}
```

delete()删除集合的元素
delete(cMap, "abc")

