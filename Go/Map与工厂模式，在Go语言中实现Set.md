## Map与工厂模式，在Go语言中实现Set
---
```
Go的内置集合中没有Set实现，可以使用map[type]bool
1. 元素的唯一性
2. 基本操作：添加、判断元素是否存在、删除元素、元素个数
```

```go
// 使用map[type]bool结构，实现Set功能，目前是数字乘方、平方
func TestMapWithFunValue(t *testing.T) {
	m := map[int]func(op int) int{}
	m[1] = func(op int) int { return op }
	m[2] = func(op int) int { return op * op }
	m[3] = func(op int) int { return op * op * op }
	t.Log(m[1](2), m[2](2), m[3](2))
}
```

```go
/**
元素的唯一性
*/

func TestMapForSet(t *testing.T) {
	mySet := map[int]bool{}
	mySet[1] = true
	// 判断元素是否存在
	n := 3
	if mySet[n] {
		t.Logf("%d is existing", n)
	} else {
		t.Logf("%d is not existing", n)
	}
	// 获取集合长度
	mySet[3] = true
	t.Log(len(mySet))

	// 删除元素
	//delete(mySet, 1)
	n = 1
	if mySet[n] {
		t.Logf("%d is existing", n)
	} else {
		t.Logf("%d is not existing", n)
	}
}
```
