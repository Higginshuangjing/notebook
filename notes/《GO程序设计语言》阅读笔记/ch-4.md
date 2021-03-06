## 复合数据类型

### 数组

在GO中，数组的长度是固定的，所以更多的是使用`slice`，但是理解数组是理解`slice`的基础。  

数组的长度必须由常量来表示，必须要在编译期就可以知道数组的长度。  
如果不手动指定长度，使用`...`来填充，则会默认按照声明数组后侧的元素数量：
```go
arr := [...]int{1, 2, 3}
```

在函数调用中，数组作为参数也是按值传递，即使修改数组中的某一项，也不会影响到函数外的数组。  
如果需要修改原数组，可以使用指针传递地址。

因为在GO中，任何类型的变量都有默认值，所以下述代码会创建一个长度100，除了下标99为-1，其余item都为0的数组：
```go
arr := []int{99: -1}
```

类型相同的数组之间可以进行比较，`[2]int`与`[3]int`被认为是两种类型，所以无法比较。  
```go
a := [2]int{1, 2}
b := [...]int{1, 2}

a == b // true

c := [3]int{1, 2}

a == c // error 类型不匹配
```

### Map

Map在创建时需要指定Key和Value的类型，两者可以不同。  
但是Key的类型需要保证一定可以使用`==`，用来判断Key是否存在。  

无法获取Map元素的地址，因为Map所占用的内存可能会增长，分配新的内存，导致之前的地址失效。  

#### 创建一个map的几种方式

因为map是一个类似slice的不固定长度的结构。  
所以在创建时并不需要手动指定长度。  

```go
maps := make(map[string]int)

maps["Niko"] = 18
maps["Bellic"] = 20
```

```go
maps := map[string]int{
  "Niko": 18,
  "Bellic": 20, // #需要注意，在GO中，最后一个元素的逗号，一定要有#
}
```

```go
var maps map[string]int

Println(maps == nil)    // true
Println(len(maps) == 0) // true

// 但是如果针对一个零值map来进行赋值操作会宕机

maps["Niko"] = 18 // Error

// 必须要先初始化
maps = make(map[string]int)
maps["Niko"] = 18 // Success
```

#### 删除元素

删除元素可以通过内置的`delete`函数来进行操作：
```go
delete(maps, "key") // 即使 maps["key] 不存在也不会出错的
```

#### 操作元素

就像普通的变量一样，`map`对应的`value`都是有其类型对应的默认值的：
```go
maps["Roman"] += 1 // 即使不存在Roman，也会有int对应的默认值0
maps["Roman"]++    // ++ -- 简写都是支持的
```

#### 迭代

`map`可以使用`range`的方式来迭代，但是不能保证迭代的顺序。  
```go
ages := map[string]int{
  "Niko": 18,
  "Bellic": 20
}

for name, age range ages {
  // 有可能先输出的是Bellic
}
```

因为`map`可能会因为因为随着元素数量的增长而重新分配更大的内存空间，所以会导致之前的地址无效。  
因此，禁止对`map`元素进行取地址的操作。

> 在实践中，遍历 的顺序是随机的，每一次遍历的顺序都不相同。这是故意的，每次都使用随机的遍历顺序可以强制 要求程序不会依赖具体的哈希函数实现。 *很强势*  

#### 判断key值是否有效

在获取map的value时，有第二个参数返回，可以根据它来判断该key是否有效：
```go
name, ok := maps["Niko"]
if !ok {
  // Niko 不是一个有效的key
}

// 或者可以用简写变成一条语句
if name, ok := maps["Niko"]; !ok {
  // Niko 不是一个有效的key
}
```

##### 判断两个map是否相等

判断两个map是否相等可以简单的使用循环结合着上述的`ok`来进行判断：
```go
func equal(x, y map[string]int) bool {
	if len(x) != len(y) {
		return false
	}

	for k, xv := range x {
		if yx, ok := y[k]; !ok || yx != xv {
			return false // 不存在该key，或者该key对应的值不想等
		}
	}

	return true
}
```

### 结构体

如果想修改一个函数返回的结构体，那么函数必须要返回结构体的指针，否则对其进行赋值操作会报错。  
只是取值的话没有这个限制。