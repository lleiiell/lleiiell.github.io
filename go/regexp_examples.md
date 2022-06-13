## Go Regexp 示例

### 表达式

选择和分组

正则表达式 | 意义
--- | ---
xy | x跟着y
x\|y | x或y
xy\|z | 同 (xy)\|z
xy* | 同 x(y*)


重复（贪婪和非贪婪）

正则表达式 | 意义
--- | ---
x* | 零个或多个 x，越多越好
x*? | 零个或多个 x，越少越好
x+	| 一个或多个 x，越多越好
x+?	| 一个或多个 x，越少越好
x?	| 零或一个 x，更喜欢一
x??	| 喜欢零
x{n} | 正好 n 个 x


字符类

正则表达式 | 意义
--- | ---
.	| 任何字符
[ab] |	字符 a 或 b
[^ab] |	除 a 或 b 以外的任何字符
[a-z] |	从 a 到 z 的任何字符
[a-z0-9] |	从 a 到 z 或 0 到 9 的任何字符
\d	| 一个数字：[0-9]
\D	| 非数字：[^0-9]
\s	| 一个空格字符：[\t\n\f\r ]
\S	| 非空白字符：[^\t\n\f\r ]
\w	| 一个单词字符：[0-9A-Za-z_]
\W	| 非单词字符：[^0-9A-Za-z_]


特殊字符

正则表达式 | 意义
--- | ---
\t |	水平制表符 =\011
\n |	换行=\012
\f |	换页 =\014
\r |	回车=\015
\v |	垂直制表符 =\013
\123 |	八进制字符代码（最多三位）
\x7F |	十六进制字符代码（正好两位数）


文本边界锚点

正则表达式 | 意义
--- | ---
\A |	在文本的开头
^ |	在文本或行的开头
$ |	在文本末尾
\b |	在 ASCII 字边界
\B |	不在 ASCII 字边界


不区分大小写和多行匹配

正则表达式 | 意义
--- | ---
i |	不区分大小写
m |	^ $ 匹配开始/结束文本以及开始/结束行（多行模式）
s |	让.匹配\n（单行模式）

### hello regexp

```go
func TestHello(t *testing.T) {
	fmt.Println(regexp.MatchString("^hello", "Hello world")) // false nil
	fmt.Println(regexp.MatchString("^Hello", "Hello world")) // true nil

	fmt.Println(regexp.MatchString("p([a-z]+)ch", "peach")) // true nil
}
```

### Compile

```go
func TestCompile(t *testing.T) {
	r, _ := regexp.Compile("p([a-z]+)ch")
	fmt.Println(r.MatchString("peach"))                   // true
	fmt.Println(r.FindString("peach punch"))              // peach
	fmt.Println("idx:", r.FindStringIndex("peach punch")) // idx: [0 5]

	// 返回包括：整个模式匹配和这些匹配中的子匹配的信息
	fmt.Println(r.FindStringSubmatch("peach punch"))      // [peach ea]
	fmt.Println(r.FindStringSubmatchIndex("peach punch")) // [0 5 1 3]

	fmt.Println(r.FindAllString("peach punch pinch", -1)) // [peach punch pinch]
	// all: [[0 5 1 3] [6 11 7 9] [12 17 13 15]]
	fmt.Println("all:", r.FindAllStringSubmatchIndex("peach punch pinch", -1))
	fmt.Println(r.FindAllString("peach punch pinch", 2)) // [peach punch]

	fmt.Println(r.Match([]byte("peach"))) // true

	// MustCompile：panic 而不是返回错误，从而使用全局变量更安全。
	r = regexp.MustCompile("p([a-z]+)ch")
	fmt.Println("regexp:", r) // regexp: p([a-z]+)ch

	fmt.Println(r.ReplaceAllString("a peach", "<fruit>")) // a <fruit>

	in := []byte("a peach")
	out := r.ReplaceAllFunc(in, bytes.ToUpper)
	fmt.Println(string(out)) // a PEACH

}
```

### 提取所有非字母数字字符

```go
func TestFindAllString(t *testing.T) {
	str1 := "We @@@Love@@@@ #Go!$! ****Programming****Language^^^"

	re := regexp.MustCompile(`[^a-zA-Z0-9]+`)

	fmt.Printf("Pattern: %v\n", re.String()) // Pattern: [^a-zA-Z0-9]+
	fmt.Println(re.MatchString(str1))        // true

	submatchall := re.FindAllString(str1, -1)
	for _, element := range submatchall {
		fmt.Println(element)
	}
}


// Pattern: [^a-zA-Z0-9]+
// true
//  @@@
// @@@@ #
// !$! ****
// ****
// ^^^
```

### 提取日期（YYYY-MM-DD）

```go
func TestFindDate(t *testing.T) {
	str1 := "If I am 20 years 10 months and 14 days old as of August 17,2016 then my DOB would be 1995-10-03"

	re := regexp.MustCompile(`\d{4}-\d{2}-\d{2}`)

	fmt.Printf("Pattern: %v\n", re.String()) 

	fmt.Println(re.MatchString(str1))  

	submatchall := re.FindAllString(str1, -1)
	for _, element := range submatchall {
		fmt.Println(element)
	}
}


// Pattern: \d{4}-\d{2}-\d{2}
// true
// 1995-10-03
```

### 提取 IP

```go
func TestIp(t *testing.T) {
	str1 := `Proxy Port Last Check Proxy Speed Proxy Country Anonymity 118.99.81.204
	118.99.81.204 8080 34 sec Indonesia - Tangerang Transparent 2.184.31.2 8080 58 sec 
	Iran Transparent 93.126.11.189 8080 1 min Iran - Esfahan Transparent 202.118.236.130 
	7777 1 min China - Harbin Transparent 62.201.207.9 8080 1 min Iraq Transparent`

	re := regexp.MustCompile(`(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)(\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)){3}`)

	fmt.Printf("Pattern: %v\n", re.String()) 
	fmt.Println(re.MatchString(str1))        // true

	submatchall := re.FindAllString(str1, -1)
	for _, element := range submatchall {
		fmt.Println(element)
	}
}


// Pattern: (25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)(\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)){3}
// true
// 118.99.81.204
// 118.99.81.204
// 2.184.31.2
// 93.126.11.189
// 202.118.236.130
// 62.201.207.9
```

### 提取域名

```go
func TestDomain(t *testing.T) {
	str1 := `http://www.suon.co.uk/product/1/7/3/`

	re := regexp.MustCompile(`^(?:https?:\/\/)?(?:[^@\/\n]+@)?(?:www\.)?([^:\/\n]+)`)
	fmt.Printf("Pattern: %v\n", re.String()) // print pattern
	fmt.Println(re.MatchString(str1))        // true

	submatchall := re.FindAllString(str1, -1)
	for _, element := range submatchall {
		fmt.Println(element)
	}
}


// Pattern: ^(?:https?:\/\/)?(?:[^@\/\n]+@)?(?:www\.)?([^:\/\n]+)
// true
// http://www.suon.co.uk
```

### 替换非字母数字

```go
func TestReplace(t *testing.T) {
	reg := regexp.MustCompile("[^A-Za-z0-9]+")

	newStr := reg.ReplaceAllString("#Golang#Python$Php&Kotlin@@", "-")
	fmt.Println(newStr)
}


// -Golang-Python-Php-Kotlin-
```

### 替换第一次出现

```go
func TestReplaceFirst(t *testing.T) {
	strEx := "Php-Golang-Php-Python-Php-Kotlin"
	reStr := regexp.MustCompile("^(.*?)Php(.*)$")
	repStr := "${1}Java$2"
	output := reStr.ReplaceAllString(strEx, repStr)
	fmt.Println(output)
}


// Java-Golang-Php-Python-Php-Kotlin
```

### 拆分字符串

```go
func TestSplitSpace(t *testing.T) {
	str1 := "Split   String on \nwhite    \tspaces."

	re := regexp.MustCompile(`\S+`)

	fmt.Printf("Pattern: %v\n", re.String()) // Print Pattern

	fmt.Printf("String contains any match: %v\n", re.MatchString(str1)) // True

	submatchall := re.FindAllString(str1, -1)
	for _, element := range submatchall {
		fmt.Println(element)
	}
}


// Pattern: \S+
// String contains any match: true
// Split
// String
// on
// white
// spaces.
```

## 参考

- [regular expressions - gobyexample.com](https://gobyexample.com/regular-expressions)
- [regular expressions - golangprograms.com](https://www.golangprograms.com/regular-expressions.html)
- [regexp cheat sheet - yourbasic.org](https://yourbasic.org/golang/regexp-cheat-sheet/)