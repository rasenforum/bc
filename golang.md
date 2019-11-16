# Golang

### 실습(Internet)
https://golang.org/

<br>

### 실습(Local)
#### 1. hello-world
```
package main

import "fmt"

func main() {
	fmt.Println("hello world")
}
```

<br>

#### 2. variable
```
package main

import "fmt"

func main() {
	var a = "this is a"
	fmt.Println(a)

	b := "this is b"
	b = "this is c"
	fmt.Println(b)
}
```

<br>

#### 3. param
```
package main

import "fmt"

func main() {
	testparam("2")
}

func testparam(a string) {
	fmt.Println(a)
}
```

<br>

#### 4. return_1

```
package main

import "fmt"

func main() {
	fmt.Println(testReturn())
}

func testReturn() (string, string, int) {
	return "3", "1", 1
}
```

<br>


#### 5.return_2
```
package main

import "fmt"

func main() {
	a, _ := testReturn()
	fmt.Println(a)
}

func testReturn() (string, string) {
	return "3", "2"
}
```

<br>

#### 6.return_3

```
package main

import "fmt"

func main() {
	var1, _ := testReturn()
	fmt.Println(var1)
	//fmt.Println(var2)
}

func testReturn() (string, string) {
	return "3", "2"
}
```

#### 7. time
```
package main

import (
	"fmt"
	"time"
)

func main() {
	p := fmt.Println
	now := time.Now().Format("20060102150405")
	p(now)
}
```


#### 8. json
```
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	UserId       string `json:"userId"`
	UserName     string `json:"userName"`
	UserPassword string `json:"userPassword"`
	RegDate      string `json:"regDate"`
}

func main() {

	//1. create struct data
	user := User{
		UserId:       "user01",
		UserName:     "홍길동",
		UserPassword: "1q2w3e4r@",
		RegDate:      "20190101",
	}

	//2. convert struct to jsonBytes
	jsonBytes, err := json.Marshal(user)
	if err != nil {
		panic(err)
	}

	//3. convert jsonBytes to jsonString
	jsonString := string(jsonBytes)

	//4. print jsonString
	fmt.Println(jsonString)
}
```

