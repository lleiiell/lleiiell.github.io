## Go Reflect 示例

### 介绍

Reflect 提供在程序运行期对程序本身进行访问和修改的能力。


特点

- 反射主要与Golang的interface类型相关，只有interface类型才有反射一说
- interface变量包括 type, value 两部分，value 是实际变量值，type 是实际变量的类型
  - interface变量包含2个指针，一个指向值的类型，另外一个指向实际的值

### 术语

- `Value` 变量值
- `Type` 变量类型
- `Kind` 类型的种类，`[]int{}` 种类是 `slice`，类型是 `[]int`

### hello reflect

ValueOf 和 TypeOf

```go
func TestValueOfTypeOf(t *testing.T) {
	i := 123
	f := 123.456

	fmt.Println("reflect.ValueOf(i)", reflect.ValueOf(i)) // 123
	fmt.Println("reflect.TypeOf(i)", reflect.TypeOf(i))   // int
	fmt.Println("reflect.ValueOf(f)", reflect.ValueOf(f)) // 123.456
	fmt.Println("reflect.TypeOf(f)", reflect.TypeOf(f))   // float64
}
```

Kind 和 Type

```go
func TestKindType(t *testing.T) {
	fmt.Println("1 kind", reflect.ValueOf(1).Kind()) // int
	fmt.Println("1 type", reflect.ValueOf(1).Type()) // int

	fmt.Println("[]int{} kind", reflect.ValueOf([]int{}).Kind()) // slice
	fmt.Println("[]int{} type", reflect.ValueOf([]int{}).Type()) // []int

    // struct
	fmt.Println("struct kind", reflect.ValueOf(struct {
		Name string
	}{}).Kind()) 
    // struct { Name string }
	fmt.Println("struct type", reflect.ValueOf(struct {
		Name string
	}{}).Type())

	sp := "abc"
	fmt.Println("指针 kind", reflect.ValueOf(&sp).Kind()) // ptr
	fmt.Println("指针 type", reflect.ValueOf(&sp).Type()) // *string
}
```


### structs

获取 struct tags

- `Kind() Kind`，返回反射类型

```go
func TestStruct(t *testing.T) {
	type st struct {
		Name string `json:"name,omitempty"`
		Age  int    `json:"age,omitempty" form:"age"`
	}

	var u interface{} = st{}

	tp := reflect.TypeOf(u)
	if tp.Kind() != reflect.Struct {
		t.Error("tp.Kind()", tp.Kind())
		return
	}

	for i := 0; i < tp.NumField(); i++ {
		f := tp.Field(i)
		fmt.Println("f.Tag.Get(\"json\")", f.Tag.Get("json"),
			"f.Tag.Get(\"form\")", f.Tag.Get("form"))
	}
}


// f.Tag.Get("json") name,omitempty f.Tag.Get("form") 
// f.Tag.Get("json") age,omitempty f.Tag.Get("form") age
```

获取、设置 fields

- `Elem() Value`，返回反射 Value，值为 interface 包含或指针指向的具体值
- `SetString(s string)`，设置 Value 底层值为 s

```go
func TestStructField(t *testing.T) {
	type st struct {
		Name string `json:"name,omitempty"`
		Age  int    `json:"age,omitempty" form:"age"`
	}

	s1 := &st{"s1", 18}

	vs1 := reflect.ValueOf(s1).Elem()
	fs1Name := vs1.FieldByName("Name")
	fs1None := vs1.FieldByName("None")
	fmt.Println("fs1Name", fs1Name) // s1
	fmt.Println("fs1None", fs1None) // <invalid reflect.Value>
	if !fs1Name.IsValid() && !fs1Name.CanSet() {
		t.Error(fs1Name.IsValid(), fs1Name.CanSet())
		return
	}

	fs1Name.SetString("sn1")
	fmt.Println("s1", fmt.Sprintf("%+v", *s1)) // {Name:sn1 Age:18}
}
```

### slice

填充

```go
func TestSlice(t *testing.T) {
	c1 := make(chan interface{})
	go func() {
		for {
			select {
			case c := <-c1:
				vc := reflect.ValueOf(c)
				if vc.Kind() != reflect.Ptr {
					log.Fatal("vc.Kind()", vc.Kind())
				}

				vce := vc.Elem()
				fmt.Println("vce.Kind()", vce.Kind())
				fmt.Println("vce.Type()", vce.Type())

				fmt.Println("begin: c", c, vce.Len(), vce.Cap())
				vce.Set(reflect.MakeSlice(vce.Type(), 2, 2))

				for k, v := range []string{"abc", "xyz"} {
					vce.Index(k).Set(reflect.ValueOf(v))

					// panic: reflect: call of reflect.Value.SetString on interface Value
					//vce.Index(k).SetString(v)
				}
				fmt.Println("end: c", c, vce.Len(), vce.Cap())

			}
		}
	}()

	sl1 := &[]string{}
	sl2 := &[]interface{}{}
	c1 <- sl1
	c1 <- sl2
}


// vce.Kind() slice
// vce.Type() []string
// begin: c &[] 0 0
// end: c &[abc xyz] 2 2
// vce.Kind() slice
// vce.Type() []interface {}
// begin: c &[] 0 0
// end: c &[abc xyz] 2 2
```

### map

删除、设置

```go

func TestMap(t *testing.T) {
	m := map[int]string{2: "b"}

	c1 := make(chan interface{})
	wg := sync.WaitGroup{}
	wg.Add(1)

	go func() {
		defer wg.Done()
		for {
			select {
			case c := <-c1:
				if reflect.TypeOf(c).Kind() != reflect.Ptr {
					log.Fatal("reflect.TypeOf(c).Kind()", reflect.TypeOf(c).Kind())
				}

				vce := reflect.ValueOf(c).Elem()
				if vce.Type().Kind() != reflect.Map {
					log.Fatal("vce.Type().Kind()", vce.Type().Kind())

				}
				for _, k := range vce.MapKeys() {
					// del k
					fmt.Println("del k", k)
					vce.SetMapIndex(k, reflect.Value{})
				}

				// set
				fmt.Println("set map[3]c")
				vce.SetMapIndex(reflect.ValueOf(3), reflect.ValueOf("c"))
				return

			}
		}
	}()

	fmt.Println("begin m", m)
	c1 <- &m
	wg.Wait()

	fmt.Println("end m", m)
}


// begin m map[2:b]
// del k 2
// set map[3]c
// end m map[3:c]
```

### DeepEqual

```go
func TestDeepEqual2(t *testing.T) {
	a := []int{4, 5, 6}
	b := []int{4, 5, 6}
	c := []int{4, 6, 5}

	fmt.Println(reflect.DeepEqual(a, b))
	fmt.Println(reflect.DeepEqual(a, c))
}


// true
// false
```

struct

```go
func TestDeepEqual3(t *testing.T) {
	type st struct {
		Name    string
		Age     int
		Address *int
		Data    []int
	}

	a := st{
		Name:    "abc",
		Age:     88,
		Address: new(int),
		Data:    []int{1, 2, 3},
	}
	b := st{
		Name:    "abc",
		Age:     88,
		Address: new(int),
		Data:    []int{1, 2, 3},
	}

	fmt.Println(reflect.DeepEqual(a, b))
}


// true
```

## 参考

- [reflect examples - github.com/a8m](https://github.com/a8m/reflect-examples)
- [Golang的反射reflect深入理解和示例](https://juejin.cn/post/6844903559335526407)