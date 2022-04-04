---
title: "面试 | 我与Shopee的面试"
date: 2022-04-03T20:31:40+08:00
draft: false
分类: ["编程"]
tags: ["Golang", "计算机科学"]
# weight: 1
cover:
    image: "https://www.hudson.cn/wp-content/cache/bb-plugin/cache/7-most-common-mistakes-1024x731-landscape.png"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "Cover Image"
    # caption: ""
    relative: false # To use relative path for cover image, used in hugo Page-bundles
summary: "有的时候太紧张会坏事。"
author: "AidenCLX"
translationKey: "interviewwithshopee"
showToc: false
---

我做完了Shopee给我的在线测试。事实上，我不认为我做得很好；在那个时候我非常的紧张，而且我感觉我的大脑几乎要停滞了。这并不好，因为这体现出来我对于自己的紧张情绪管理还有待加强。

但事情就是这样了。是好是坏，我做完了这份在线测试，而且我想要在这里把我的答案稍微修正一下。出于对公司的尊敬，我不会把原题在这里说出来，而只会做一个修改过的版本，并放上我对于修改后版本的解答。

---

## 用SQL做条件筛选

假设我们有两张表，分别是A和B，并且A拥有两列：ID和NAMES。B拥有三列：ID，CREATED_TIME，EVENT。

写出命令，找出所有的NAMES，它们对应的EVENT数量超过50。

```sql
SELECT DISTINCT names FROM A
INNER JOIN B
ON A.ID = B.ID
GROUP BY names
WHERE count(names) > 50
```

说实话在面试后再一想其实也不是很难。。

---

## 字符处理

我花了大约15分钟来写以下代码。事实上，如果我不是过于紧张，我相信这道题我是可以做出来的。他并不像看上去那么复杂；我知道我需要用一个有限状态机，但我在实际操作时花了一些时间来确定状态和边缘情况，外加我在做选择的时候也浪费了一些时间，这也导致我没有足够的时间来写完代码。

总之，这个问题是要求你写一个代码，将输入的字符串转化为他对应的camelcase。就是这样。

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strings"
	"unicode"
)

const (
	BEGIN = iota
	LOWERCASE
	UPPERCASE
)

func main() {
	var line string
	scanner := bufio.NewScanner(os.Stdin)
	if scanner.Scan() {
		line = scanner.Text()
		fmt.Println(process(line))
	}
}

func process(s string) string {
	var sb strings.Builder
	var STATE int
	for _, r := range s {
		switch STATE {
		case BEGIN:
			if unicode.IsLetter(r) {
				// Add lowercase first letter
				// Go to lowercase
				sb.WriteRune(unicode.ToLower(r))
				STATE = LOWERCASE
				continue
			}
			if unicode.IsDigit(r) {
				// Add digit, go to uppercase
				sb.WriteRune(r)
				STATE = UPPERCASE
				continue
			}
			// Ignore other situations

		case UPPERCASE:
			if unicode.IsLetter(r) {
				// Add uppercase letter
				// go to lowercase
				sb.WriteRune(unicode.ToUpper(r))
				STATE = LOWERCASE
				continue
			}
			if unicode.IsDigit(r) {
				// Add Digit
				// remain uppercase
				sb.WriteRune(r)
				continue
			}
			// ignore others
		case LOWERCASE:
			if unicode.IsLetter(r) {
				// Add as is
				// remain lowercase
				// If uppercase, just add
				sb.WriteRune(r)
				continue
			}
			if unicode.IsDigit(r) {
				// Add digit
				// Go to uppercase
				sb.WriteRune(r)
				STATE = UPPERCASE
			}
			// Go to uppercase for
			// any other situation
			STATE = UPPERCASE
		}
	}
	return sb.String()
}
```

---

## 致谢

虽然我想我可能没法进入到下一轮面试了（在我看来这真的很可惜 **_叹气_**），但我还是想感谢Shopee能够给我这个机会来参与他们的选拔。这对我来说是很宝贵的经历。谢谢你们！