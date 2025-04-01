---
layout: post
title: 🗂 【Exploration】Concatenate Strings In Golang
date: 2025-04-01 00:00
---

When I was tackling [this](https://leetcode.com/problems/group-anagrams/description/) leetcode question by Golang, I found there are actually way many ways to concatenate string in Golang. That's why I want to write this post to demonstrate some of them and how I compare them to figure out which one is faster or more efficient.

```go
package concatenatestr

import (
	"bytes"
	"fmt"
	"strings"
	"testing"
)

var strs = []string{
	"The quick brown fox jumps over the lazy dog.",
	"Lorem ipsum dolor sit amet, consectetur adipiscing elit.",
	"Go is an amazing programming language for concurrency.",
	"This benchmark will test different string concatenation methods.",
	"Optimizing memory allocation can improve performance.",
	"Strings in Go are immutable, making efficient concatenation tricky.",
	"The builder pattern helps in reducing memory allocations.",
	"Testing with real-world data provides better insights.",
	"Performance benchmarks should be reproducible and fair.",
	"Using a buffer reduces the overhead of repeated allocations.",
	"Code readability is just as important as performance.",
	"Profiling helps in identfying bottlenecks in code.",
	"Choosing the right algorithm is crucial for efficiency.",
	"A well-structured program is easier to maintain.",
	"Concurrency in Go is powered by goroutines and channels.",
	"Using `b.ReportAllocs()` helps measure memory usage in benchmarks.",
	"Refactoring code can lead to better performance and clarity.",
	"Immutable data structures have their pros and cons.",
	"Memory leaks can be avoided with proper resource management.",
	"Effective debugging saves a lot of development time.",
}

func BenchmarkPlusOperator(b *testing.B) {
	for b.Loop() {
		res := ""
		for i := range strs {
			res += strs[i]
		}
	}
	b.ReportAllocs()
}

func BenchmarkSprintf(b *testing.B) {
	for b.Loop() {
		res := ""
		for i := range strs {
			res = fmt.Sprintf("%s%s", res, strs[i])
		}
	}
	b.ReportAllocs()
}

func BenchmarkStringsJoin(b *testing.B) {
	for b.Loop() {
		res := ""
		for i := range strs {
			res = strings.Join([]string{res, strs[i]}, "")
		}
	}
	b.ReportAllocs()
}

func BenchmarkStringsBuilder(b *testing.B) {
	for b.Loop() {
		var sb strings.Builder
		for i := range strs {
			sb.WriteString(strs[i])
		}
		sb.String()
	}
	b.ReportAllocs()
}

func BenchmarkBytesWriteString(b *testing.B) {
	for b.Loop() {
		var buf bytes.Buffer
		for i := range strs {
			buf.WriteString(strs[i])
		}
		buf.String()
	}
	b.ReportAllocs()
}

```

In summary of the above code, I'm comparing 5 different ways to combine strings:

1. Simple plus operator.
2. Using `fmt.Sprintf()`, which is used to format a string with placeholders and values.
3. String join, joining a slice of strings to concatenate multiple strings.
4. Using `strings.Builder`, introduced in Go 1.10, which is used to manipulate UTF-8 encoded strings.
5. Using `bytes.Buffer`, which maintains a list of bytes and has the `WriteString` function to add strings to the byte slice.

```
goos: darwin
goarch: arm64
pkg: concatenate-str
cpu: Apple M4 Pro
BenchmarkPlusOperator-2          1071540              1104 ns/op           12272 B/op         19 allocs/op
BenchmarkSprintf-2                633754              1908 ns/op           12946 B/op         59 allocs/op
BenchmarkStringsJoin-2            911908              1257 ns/op           12320 B/op         20 allocs/op
BenchmarkStringsBuilder-2        4109606               295.8 ns/op          2752 B/op          6 allocs/op
BenchmarkBytesWriteString-2      2287969               524.8 ns/op          5184 B/op          7 allocs/op
PASS
ok      concatenate-str 6.789s
```

As you can see, using some packages from Golang like `strings.Builder` and `bytes.Buffer` is much faster than other approaches. The fastest one is `strings.Builder`, which is almost 4 times faster than the plus operator and `strings.Join`, and 6 times faster than `fmt.Sprintf()`. Now I know what my go-to choice is when trying to concatenate strings.