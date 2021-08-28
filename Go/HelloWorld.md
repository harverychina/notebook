## 1、注意事项
包必须是main

## 2、helloworld程序
```go
package main

import "fmt"

    
func main() {
    // 测试main函数输出参数值
    fmt.a
    
    fmt.Print("Hello World")
}
```

## 3、程序结构， main函数参数， 返回值
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	// 测试输出 main 函数参数的值
	//fmt.Println(os.Args)
	if len(os.Args) > 1 {
		fmt.Println("Hello World", os.Args[1])
	}
	// 退出
	//os.Exit(0)
	os.Exit(-1)
}
```