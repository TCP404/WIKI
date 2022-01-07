# 6-strings

## 包含 contains

```go
func Contains(s, substr string) bool
func ContainsAny(s, chars string) bool
func ContainsRune(s string, r rune) bool
```

包含：判断主串 s 中是否包含子串 substr。例如 “abcde” 中就包含 “bcd”。

-   Contains 是字符串匹配；
-   ContainsAny 是非严格匹配，即不需要 substr 整个都匹配到，只要 substr 中有一个字符匹配成功就行；
-   ContainsRune 是 Unicode 字符匹配，即 rune 匹配。

```go
strings.Contains("seafood", "foo")	// ture
strings.Contains("seafood", "bar") 	// false
strings.Contains("seafood", "")		// ture
strings.Contains("", "")			// ture

strings.ContainsAny("team", "i")		// false
strings.ContainsAny("fail", "ui")		// true
strings.ContainsAny("ure", "ui")		// true
strings.ContainsAny("failure", "ui")	// true
strings.ContainsAny("foo", "")			// false
strings.ContainsAny("", "")				// false

strings.ContainsRune("aardvark", 97)	// true
strings.ContainsRune("timeout", 'a')	// false
```



## 拼接 Join

```go
func Join(elems []string, sep string) string
```

拼接字符串：将字符串切片 elems 中的字符串，夹带着分隔符 sep 拼接起来。

```go
s := []string{"foo", "bar", "baz"}
fmt.Println(strings.Join(s, "# "))	// foo# bar# baz#
```

## 下标 Index

```go
func Index(s, substr string) int
func IndexAny(s, chars string) int
func IndexByte(s string, c byte) int
func IndexRune(s string, r rune) int
func IndexFunc(s string, f func(rune) bool) int

func LastIndex(s, substr string) int
func LastIndexAny(s, chars string) int
func LastIndexByte(s string, c byte) int
func LastIndexFunc(s string, f func(rune) bool) int
```

位置：

Index 系列 返回主串 s 中 **第一个** 子串 substr 的下标，匹配不到返回 -1。

LastIndex 系列返回主串 s 中 **最后一个** 子串 substr 的下标，匹配不到返回 -1。



- Index、LastIndex：严格匹配子串，必须子串 substr 每个字符都匹配上才匹配成功；
- IdexAny、LastIndexAny：非严格匹配子串，只要子串 substr 有一个能匹配上的就匹配成功；
- IndexByte、LastIndexByte：匹配 ASCII 字符；
- IndexRune：匹配 Unicode 字符；
- IndexFunc、LastIndexFunc：匹配主串中满足 f(c) 条件的字符。也就是把子串换成某个字符群。

```go
strings.Index("chicken", "ken")			// 4
strings.Index("chicken", "dmr")			// -1

strings.Index("go gopher", "go")		// 0
strings.LastIndex("go gopher", "go")	// 3
strings.LastIndex("go gopher", "rodent")// -1


strings.IndexAny("chcicken", "aiouy")	// 3
strings.IndexAny("crwth", "aeiouy")		// -1

strings.LastIndexAny("go gopher", "go")		// 4
strings.LastIndexAny("go gopher", "rodent") // 8
strings.LastIndexAny("go gopher", "fail")	// -1


strings.IndexByte("golang", 'g')		// 0
strings.IndexByte("gophers", 'h')		// 3
strings.IndexByte("golang", 'x')		// -1

strings.LastIndexByte("Hello, world", 'l')	// 10
strings.LastIndexByte("Hello, world", 'o')	// 8
strings.LastIndexByte("Hello, world", 'x')	// -1


strings.IndexRune("chicken", 'k')		// 4
strings.IndexRune("chicken", 'd')		// -1


f := func(c rune) bool {
    return unicode.Is(unicode.Han, c)
}
strings.IndexFunc("Hello, 世界", f)	   // 7
strings.IndexFunc("Hello, world", f)	// -1

strings.LastIndexFunc("go 123", unicode.IsNumber)	// 5
strings.LastIndexFunc("123 go", unicode.IsNumber)	// 2
strings.LastIndexFunc("go", unicode.IsNumber)		// -1
```

## 重复 Repeat

```go
func Repeat(s string, count int) string
```

重复：将字符串 s 重复 count 次，相当于 Python 中的 `s * count`。

```go
strings.Repeat("nal", 2)	// nalnal
```

## 替换 Replace

```go
func Replace(s, old, new string, n int) string
func ReplaceAll(s, old, new string) string
```

替换：将字符串 s 中的 old 子串替换成 new 子串，替换 n 次；n 为 -1 时表示全部替换。

-   `ReplaceAll(s, old, new)` 等同与 `Replace(s, old, new, -1)`。

```go
strings.Replace("oink oink oink", "k", "yt", 2)		// oinyt oinyt oink
strings.Replace("oink oink oink", "in", "moo", -1)	// omook omook omook

strings.ReplaceAll("oink oink oink", "in", "moo")	// omook omook omook
```

## 分割 Split

```go
func Split(s, sep string) []string
func SplitAfter(s, sep string) []string

func SplitN(s, sep string, n int) []string
func SplitAfterN(s, sep string, n int) []string

func Fields(s string) []string
```

分割：将字符串 s 按照分割符 sep 分割成多个子串。

-   After：分割后会结果中会带着分割符；
-   N：分割成 n 个子串；
    -   n > 0：最多n个子串；最后一个子字符串将是未拆分的剩余字符；
    -   n == 0：返回 nil；
    -   n < 0：`SplitN(s, sep, -5)` 等同于 `Split(s, sep)`；
-   Fields：以连续的空白字符为分隔符，将 s 切分成多个子串，结果中不包含空白字符本身；相当于 `Split(s, " ")`，但是割出来的空白字符串不包含在返回结果中。
    -   空白字符有：\t, \n, \v, \f, \r, ' ', U+0085 (NEL), U+00A0 (NBSP)
    -   如果 s 中只包含空白字符，则返回一个空列表

```go
strings.Split("a,b,c", ",")							// ["a" "b" "c"]
strings.Split("a man a plan a canal panama", "a ")	// ["" "man " "plan " "canal panama"]
strings.Split(" xyz ", "")				// [" " "x" "y" "z" " "]
strings.Split("", "Bernardo O'Higgins")	// [""]

strings.SplitN	   ("a,b,c,d,e,f", ",", 3)	// ["a" "b" "c,d,e,f"]
strings.SplitAfter ("a,b,c,d,e,f", ",")		// ["a," "b," "c," "d," "e," "f"]
strings.SplitAfterN("a,b,c,d,e,f", ",", 3)	// ["a," "b," "c,d,e,f"]

strings.Fields("  foo b	ar  baz   "))		// ["foo" "b" "ar" "baz"]
strings.Split("  foo b	ar  baz   ", ""))	// ["" "" "foo" "b\tar" "" "baz" "" "" ""]
```

## 修剪 Trim

```go
func Trim	  (s, cutset string) string
func TrimLeft (s, cutset string) string
func TrimRight(s, cutset string) string
func TrimSpace(s string) string

func TrimPrefix(s, prefix string) string
func TrimSuffix(s, suffix string) string

func TrimFunc	  (s string, f func(rune) bool) string
func TrimLeftFunc (s string, f func(rune) bool) string
func TrimRightFunc(s string, f func(rune) bool) string
```

修剪：将字符集 cutset 从字符串 s 中删除。cutset 任何一个字符都可以匹配，即修剪完不会有 cutset 中任何一个字符。

-   TrimLeft，从左边开始删除所有匹配的字符，匹配规则为从左开始匹配字符，直到第一个不匹配的字符停止。例如 cutset 为 abc，则 abcaaE、abcabdE、abcEabc 的匹配结果为 abcaa、abcab、abc。

    ```go
    strings.TrimLeft("abcaaE", "abc")		// E
    strings.TrimLeft("abcabdE", "abc")		// dE
    strings.TrimLeft("abcEabc", "abc")		// Eabc
    strings.TrimLeft("abcabcdEabc", "abc")	// dEabc
    ```

-   Trim = TrimLeft + TrimRight

    ```go
    strings.Trim("abcEabc", "abc")		// E
    strings.Trim("abcabcdEabc", "abc")	// dE
    strings.Trim("abcabcdEbcc", "abc")  // dE
    ```

-   TrimPrefix =  TrimLeft，但是只删除一次匹配

-   TrimSuffix = TrimRight，但是只删除一次匹配

    ```go
    strings.TrimPrefix("abcaaE", "abc")			// aaE
    strings.TrimPrefix("abcabdE", "abc")		// abdE
    strings.TrimPrefix("abcEabc", "abc")		// Eabc
    strings.TrimPrefix("abcabcdEabc", "abc")	// abcdEabc
    ```

-   TrimSpace：

-   TrimFunc：要删除的字符按照 f 匹配

    ```go
    func f (r rune) bool {
        return !unicode.IsLetter(r) && !unicode.IsNumber(r)
    }
    
    strings.TrimFunc("¡¡¡Hello, Gophers!!!", f)	// Hello, Gophers
    ```

    

