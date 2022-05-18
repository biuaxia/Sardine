title: Golang不同格式输出九九乘法表
date: 2022-05-18 14:33:00
toc: false
index_img: https://b3logfile.com/file/2022/05/image-bc047597.png
category:
- Go
tags:
- Go
- go
- Golang
- golang
- 格式
- 输出
---

```golang
package main

import (
	"fmt"
	"strings"
)

func style0() {
	m := make(map[int][]string, 9)
	for i := 1; i < 10; i++ {
		for j := i; j < 10; j++ {
			str := fmt.Sprintf("%d * %d = %2d\t", i, j, i*j)
			s, ok := m[i]
			if !ok {
				m[i] = []string{str}
			} else {
				m[i] = append(s, str)
			}
		}
	}

	for i := 0; i < 9; i++ {
		for j := 1; j < 10-i; j++ {
			fmt.Print(m[j][i])
		}
		fmt.Println()
	}
}

func style1() {
	m := make(map[int][]string, 9)
	for i := 1; i < 10; i++ {
		for j := i; j < 10; j++ {
			str := fmt.Sprintf("%d * %d = %2d\t", i, j, i*j)
			s, ok := m[i]
			if !ok {
				m[i] = []string{str}
			} else {
				m[i] = append(s, str)
			}
		}
	}

	for i := 0; i < 9; i++ {
		var placeholder []string
		for k := 0; k < i; k++ {
			placeholder = append(placeholder, fmt.Sprint("          \t"))
		}
		fmt.Print(strings.Join(placeholder, ""))

		for j := 9 - i; j > 0; j-- {
			fmt.Print(m[j][i])
		}
		fmt.Println()
	}
}

func style2() {
	for i := 1; i < 10; i++ {
		for j := i; j < 10; j++ {
			str := fmt.Sprintf("%d * %d = %2d\t", i, j, i*j)
			fmt.Print(str)
		}
		fmt.Println()
	}
}

func style3() {
	for i := 1; i < 10; i++ {
		var placeholder []string
		for k := 1; k < i; k++ {
			placeholder = append(placeholder, fmt.Sprint("          \t"))
		}
		fmt.Print(strings.Join(placeholder, ""))
		for j := 9; j >= i; j-- {
			str := fmt.Sprintf("%d * %d = %2d\t", i, j, i*j)
			fmt.Print(str)
		}
		fmt.Println()
	}
}

// 九九乘法表，4种输出格式
func main() {
	style0()
	style1()
	style2()
	style3()
}
```

运行结果：

```text
1 * 1 =  1	2 * 2 =  4	3 * 3 =  9	4 * 4 = 16	5 * 5 = 25	6 * 6 = 36	7 * 7 = 49	8 * 8 = 64	9 * 9 = 81	
1 * 2 =  2	2 * 3 =  6	3 * 4 = 12	4 * 5 = 20	5 * 6 = 30	6 * 7 = 42	7 * 8 = 56	8 * 9 = 72	
1 * 3 =  3	2 * 4 =  8	3 * 5 = 15	4 * 6 = 24	5 * 7 = 35	6 * 8 = 48	7 * 9 = 63	
1 * 4 =  4	2 * 5 = 10	3 * 6 = 18	4 * 7 = 28	5 * 8 = 40	6 * 9 = 54	
1 * 5 =  5	2 * 6 = 12	3 * 7 = 21	4 * 8 = 32	5 * 9 = 45	
1 * 6 =  6	2 * 7 = 14	3 * 8 = 24	4 * 9 = 36	
1 * 7 =  7	2 * 8 = 16	3 * 9 = 27	
1 * 8 =  8	2 * 9 = 18	
1 * 9 =  9	
9 * 9 = 81	8 * 8 = 64	7 * 7 = 49	6 * 6 = 36	5 * 5 = 25	4 * 4 = 16	3 * 3 =  9	2 * 2 =  4	1 * 1 =  1	
          	8 * 9 = 72	7 * 8 = 56	6 * 7 = 42	5 * 6 = 30	4 * 5 = 20	3 * 4 = 12	2 * 3 =  6	1 * 2 =  2	
          	          	7 * 9 = 63	6 * 8 = 48	5 * 7 = 35	4 * 6 = 24	3 * 5 = 15	2 * 4 =  8	1 * 3 =  3	
          	          	          	6 * 9 = 54	5 * 8 = 40	4 * 7 = 28	3 * 6 = 18	2 * 5 = 10	1 * 4 =  4	
          	          	          	          	5 * 9 = 45	4 * 8 = 32	3 * 7 = 21	2 * 6 = 12	1 * 5 =  5	
          	          	          	          	          	4 * 9 = 36	3 * 8 = 24	2 * 7 = 14	1 * 6 =  6	
          	          	          	          	          	          	3 * 9 = 27	2 * 8 = 16	1 * 7 =  7	
          	          	          	          	          	          	          	2 * 9 = 18	1 * 8 =  8	
          	          	          	          	          	          	          	          	1 * 9 =  9	
1 * 1 =  1	1 * 2 =  2	1 * 3 =  3	1 * 4 =  4	1 * 5 =  5	1 * 6 =  6	1 * 7 =  7	1 * 8 =  8	1 * 9 =  9	
2 * 2 =  4	2 * 3 =  6	2 * 4 =  8	2 * 5 = 10	2 * 6 = 12	2 * 7 = 14	2 * 8 = 16	2 * 9 = 18	
3 * 3 =  9	3 * 4 = 12	3 * 5 = 15	3 * 6 = 18	3 * 7 = 21	3 * 8 = 24	3 * 9 = 27	
4 * 4 = 16	4 * 5 = 20	4 * 6 = 24	4 * 7 = 28	4 * 8 = 32	4 * 9 = 36	
5 * 5 = 25	5 * 6 = 30	5 * 7 = 35	5 * 8 = 40	5 * 9 = 45	
6 * 6 = 36	6 * 7 = 42	6 * 8 = 48	6 * 9 = 54	
7 * 7 = 49	7 * 8 = 56	7 * 9 = 63	
8 * 8 = 64	8 * 9 = 72	
9 * 9 = 81	
1 * 9 =  9	1 * 8 =  8	1 * 7 =  7	1 * 6 =  6	1 * 5 =  5	1 * 4 =  4	1 * 3 =  3	1 * 2 =  2	1 * 1 =  1	
          	2 * 9 = 18	2 * 8 = 16	2 * 7 = 14	2 * 6 = 12	2 * 5 = 10	2 * 4 =  8	2 * 3 =  6	2 * 2 =  4	
          	          	3 * 9 = 27	3 * 8 = 24	3 * 7 = 21	3 * 6 = 18	3 * 5 = 15	3 * 4 = 12	3 * 3 =  9	
          	          	          	4 * 9 = 36	4 * 8 = 32	4 * 7 = 28	4 * 6 = 24	4 * 5 = 20	4 * 4 = 16	
          	          	          	          	5 * 9 = 45	5 * 8 = 40	5 * 7 = 35	5 * 6 = 30	5 * 5 = 25	
          	          	          	          	          	6 * 9 = 54	6 * 8 = 48	6 * 7 = 42	6 * 6 = 36	
          	          	          	          	          	          	7 * 9 = 63	7 * 8 = 56	7 * 7 = 49	
          	          	          	          	          	          	          	8 * 9 = 72	8 * 8 = 64	
          	          	          	          	          	          	          	          	9 * 9 = 81	
```
