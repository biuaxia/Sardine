title: Golang输出中国象棋
date: 2022-05-18 17:18:00
toc: false
index_img: https://b3logfile.com/file/2022/05/image-40bc4e8c.png
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
)

var (
	boardInfo = [9]rune{
		'车', '马', '相', '士', '帅', '士', '相', '马', '车',
	}
)

func main() {
	var board [10][9]rune

	board[2][1] = '炮'
	board[2][7] = '炮'
	board[7][1] = '炮'
	board[7][7] = '炮'

	for i := 0; i < len(board[1]); i++ {
		board[4][i] = '卒'
	}
	for i := 0; i < len(board[1]); i++ {
		board[5][i] = '兵'
	}

	for i := 0; i < len(boardInfo); i++ {
		board[0][i] = boardInfo[i]
	}
	for column := range board[len(board)-1] {
		board[len(board)-1][column] = boardInfo[column]
		if 4 == column {
			board[len(board)-1][column] = '将'
		}
	}

	for i := range board {
		for j := range board[i] {
			if board[i][j] == 0 {
				board[i][j] = '·'
			}
		}
	}

	for i := 0; i < len(board); i++ {
		for j := 0; j < len(board[i]); j++ {
			fmt.Printf("%c\t", board[i][j])
		}
		fmt.Println()
	}
}
```

运行结果：

```text
车      马      相      士      帅      士      相      马      车
·       ·       ·       ·       ·       ·       ·       ·       ·
·       炮      ·       ·       ·       ·       ·       炮      ·
·       ·       ·       ·       ·       ·       ·       ·       ·
卒      卒      卒      卒      卒      卒      卒      卒      卒
兵      兵      兵      兵      兵      兵      兵      兵      兵
·       ·       ·       ·       ·       ·       ·       ·       ·
·       炮      ·       ·       ·       ·       ·       炮      ·
·       ·       ·       ·       ·       ·       ·       ·       ·
车      马      相      士      将      士      相      马      车
```
