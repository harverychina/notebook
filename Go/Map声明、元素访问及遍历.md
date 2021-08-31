## Map声明、元素访问及遍历
---
* Map声明
```go
func TestInitMap(t *testing.T) {
	m1 := map[int]int{1: 1, 2: 4, 3: 9}
	t.Log(m1[2])
	t.Logf("len m1=%d", len(m1))
	m2 := map[int]int{}
	m2[4] = 16
	t.Logf("len m2=%d", len(m2))
	// 引入了长度和capacity
	m3 := make(map[int]int, 10)
	t.Logf("len m3=%d", len(m3))
}
```
* Map元素访问及遍历
```go
/**
Map元素访问
*/
func TestAccessNotExistingKey(t *testing.T) {
	m1 := map[int]int{}
	t.Log(m1[1])
	m1[2] = 0
	t.Log(m1[2])
	// 判断值是否存在，还是值为0呢？
	//m1[3] = 0
	if v, ok := m1[3]; ok {
		t.Logf("Key 3's value is %d", v)
	} else {
		t.Log("key is not existing")
	}
}

/**
map 遍历
*/

func TestTravelMap(t *testing.T) {
	m1 := map[int]int{1: 1, 2: 4, 3: 9}
	for k, v := range m1 {
		t.Log(k, v)
	}
}

```