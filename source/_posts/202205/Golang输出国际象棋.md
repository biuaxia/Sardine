title: Golang输出国际象棋
date: 2022-05-18 15:33:00
toc: false
index_img: https://b3logfile.com/file/2022/05/image-cdca4b06.png
category:
- Go
tags:
- Go
- go
- Golang
- golang
- 输出
---

```golang
package main

import (
	"fmt"
	"strings"
)

var (
	boardInfo = []string{
		"r", "k", "q", "r", "b", "n", "p", "r",
	}
)

func main() {
	var board [8][8]string
	for i := 0; i < len(boardInfo); i++ {
		board[0][i] = boardInfo[i]
	}
	for column := range board[len(board)-1] {
		board[len(board)-1][column] = strings.ToUpper(boardInfo[column])
	}

	for i := range board {
		for j := range board[i] {
			if board[i][j] == "" {
				board[i][j] = "·"
			}
		}
	}

	for i := 0; i < len(board); i++ {
		for j := 0; j < len(board[i]); j++ {
			fmt.Printf("%s\t", board[i][j])
		}
		fmt.Println()
	}
}
```

运行结果：

```text
r       k       q       r       b       n       p       r
·       ·       ·       ·       ·       ·       ·       ·
·       ·       ·       ·       ·       ·       ·       ·
·       ·       ·       ·       ·       ·       ·       ·
·       ·       ·       ·       ·       ·       ·       ·
·       ·       ·       ·       ·       ·       ·       ·
·       ·       ·       ·       ·       ·       ·       ·
R       K       Q       R       B       N       P       R
```
