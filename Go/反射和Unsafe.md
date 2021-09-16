# 反射和Unsafe

一、反射编程
```go
import (
	"fmt"
	"reflect"
	"testing"
)

func TestTypeAndValue(t *testing.T) {
	var f int64 = 10
	t.Log(reflect.TypeOf(f), reflect.ValueOf(f))
	t.Log(reflect.ValueOf(f).Type())
}

func CheckType(v interface{}) {
	t := reflect.TypeOf(v)
	switch t.Kind() {
	case reflect.Float32, reflect.Float64:
		fmt.Println("Float")
	case reflect.Int, reflect.Int32, reflect.Int64:
		fmt.Println("Integer")
	default:
		fmt.Println("Unknown", t)
	}
}

func TestBasicType(t *testing.T) {
	var f float64 = 12
	CheckType(&f)
}

func TestDeepEqual(t *testing.T) {
	a := map[int]string{1: "one", 2: "two", 3: "three"}
	b := map[int]string{1: "one", 2: "two", 3: "three"}
	//t.Log(a == b)
	t.Log("a==b?", reflect.DeepEqual(a, b))

	s1 := []int{1, 2, 3}
	s2 := []int{1, 2, 3}
	s3 := []int{3, 2, 1}

	t.Log("s1 == s2?", reflect.DeepEqual(s1, s2))
	t.Log("s1 == s3?", reflect.DeepEqual(s1, s3))

	c1 := Customer{"1", "Mike", 40}
	c2 := Customer{"1", "Mike", 40}
	fmt.Println(c1 == c2)
	fmt.Println(reflect.DeepEqual(c1, c2))
}

// 定义一些使用数据结构

type Employee struct {
	EmployeeID string
	Name       string `format:"normal"`
	Age        int
}

func (e *Employee) UpdateAge(newVal int) {
	e.Age = newVal
}

type Customer struct {
	CookeID string
	Name    string
	Age     int
}

func TestInvokeByName(t *testing.T) {
	e := &Employee{"1", "Mike", 30}
	// 按名字获取成员
	t.Logf("name:value(%[1]v, Type(%[1]T)", reflect.ValueOf(*e).FieldByName("name"))
	if nameField, ok := reflect.TypeOf(*e).FieldByName("Name"); !ok {
		t.Error("Failed to get 'Name' field.")
	} else {
		t.Log("Tag:format", nameField.Tag.Get("format"))
	}
	reflect.ValueOf(e).MethodByName("UpdateAge").Call([]reflect.Value{reflect.ValueOf(1)})
	t.Log("Updated Age:", e)
}

```

- 按名字访问结构的成员
```go
    reflect.ValueOf(*e).FieldByName("Name")
```
- 按名字访问结构的方法
```go
    reflect.ValueOf(e).MethodByName("UpdateAge").Call([]reflect.Value{reflect.ValueOF(1)})
```

二、万能程序
DeepEqual 比较切片和map

```go

import (
	"errors"
	"reflect"
	"testing"
)

func TestDeepEqual(t *testing.T) {
	a := map[int]string{1: "one", 2: "two", 3: "three"}
	b := map[int]string{1: "one", 2: "two", 4: "three"}

	//t.Log(a == b)
	t.Log(reflect.DeepEqual(a, b))

	s1 := []int{1, 2, 3}
	s2 := []int{1, 2, 3}
	s3 := []int{1, 2, 3}
	t.Log("s1==s2?", reflect.DeepEqual(s1, s2))
	t.Log("s1==s2?", reflect.DeepEqual(s1, s3))
}

type Employee struct {
	EmployeeID string
	Name       string
	Age        int
}

type Customer struct {
	CookeID string
	Name    string
	Age     int
}

func (e *Employee) UpdateAge(newVal int) {
	e.Age = newVal
}

func fillBySettings(st interface{}, settings map[string]interface{}) error {
	// func (v Value) Elem() Value
	//
	//
	//
	//

	if reflect.TypeOf(st).Kind() != reflect.Ptr {
		return errors.New("The first param should be a pointer to the struct type.")
	}
	// Elem() 获取指针指向的值
	if reflect.TypeOf(st).Elem().Kind() != reflect.Ptr {
		return errors.New("the first param should be a pointer to the struct type.")
	}
	if settings == nil {
		return errors.New("settings is nil.")
	}

	var (
		field reflect.StructField
		ok    bool
	)
	for k, v := range settings {
		if field, ok = (reflect.ValueOf(st)).Elem().Type().FieldByName(k); !ok {
			continue
		}
		if field.Type == reflect.TypeOf(v) {
			vstr := reflect.ValueOf(st)
			vstr = vstr.Elem()
			vstr.FieldByName(k).Set(reflect.ValueOf(v))
		}
	}
	return nil
}

func TestFillNameAndAge(t *testing.T) {
	settings := map[string]interface{}{"Name": "Mike", "Age": 40}
	e := Employee{}
	if err := fillBySettings(&e, settings); err != nil {
		t.Fatal(err)
	}

	t.Log(e)
	c := new(Customer)
	if err := fillBySettings(c, settings); err != nil {
		t.Fatal(err)
	}
	t.Log(*c)
}
```

三、不安全编程
1. “不安全”行为的危险性
```go
   i:=10
   f:=*(*float64)(unsafe.Pointer(&i))
```

```go

type Customer struct {
	Name string
	Age  int
}

func TestUnsafe(t *testing.T) {
	i := 10
	f := *(*float64)(unsafe.Pointer(&i))
	t.Log(unsafe.Pointer(&i))
	t.Log(f)
}

// The cases is suitable for unsafe
type MyInt int

// 合理的类型转换
func TestConvert(t *testing.T) {
	a := []int{1, 2, 3, 4}
	b := *(*[]MyInt)(unsafe.Pointer(&a))
	t.Log(b)
}

// 原子类型操作
func TestAtomic(t *testing.T) {
	var shareBufferPtr unsafe.Pointer
	writeDataFn := func() {
		data := []int{}
		for i := 0; i < 100; i++ {
			data = append(data, i)
		}
		atomic.StorePointer(&shareBufferPtr, unsafe.Pointer(&data))
	}
	readDataFn := func() {
		data := atomic.LoadPointer(&shareBufferPtr)
		fmt.Println(data, *(*[]int)(data))
	}
	var wg sync.WaitGroup
	writeDataFn()
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func() {
			for i := 0; i < 10; i++ {
				writeDataFn()
				time.Sleep(time.Microsecond * 100)
			}
			wg.Done()
		}()
		wg.Add(1)
		go func() {
			for i := 0; i < 10; i++ {
				readDataFn()
				time.Sleep(time.Microsecond * 100)
			}
			wg.Done()
		}()
	}
	wg.Wait()
}
```