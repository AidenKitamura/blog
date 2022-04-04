---
title: "Interview | My Interview with Shopee"
date: 2022-04-03T20:31:40+08:00
draft: false
categories: ["Programming"]
tags: ["Golang", "Computer Science"]
# weight: 1
cover:
    image: "https://www.hudson.cn/wp-content/cache/bb-plugin/cache/7-most-common-mistakes-1024x731-landscape.png"
    # can also paste direct link from external site
    # ex. https://i.ibb.co/K0HVPBd/paper-mod-profilemode.png
    alt: "Cover Image"
    # caption: ""
    relative: false # To use relative path for cover image, used in hugo Page-bundles
summary: "Sometimes being nervous could ruin things."
author: "AidenCLX"
translationKey: "interviewwithshopee"
showToc: false
---

I have just done my online assessment with shopee. In fact, I don't think I did very well in the online assessment. I was very nervous at that moment, and I felt like my brain just stopped working. It is not good because it indicated that I still need to manage my emotions better.

But that is it. Whether good or not, I have already done the online assessment and I want to refine my solution here a bit, as a practice. Due to the respect to that company, I will not share the exact same questions, but only a changed version with some code I wrote afterwards.

---

## Conditional Select using SQL

Assume we have two tables A and B, where A has two columns: ID and NAMES; and B has three columns: ID, CREATED_TIME, EVENT.

We need to write a SQL statement to select distinct NAMES which appeared more than 50 times in table B. Here we go:

```sql
SELECT DISTINCT names FROM A
INNER JOIN B
ON A.ID = B.ID
GROUP BY names
WHERE count(names) > 50
```

I must say it is not hard after the interview.

---

## String Operation

It took me about 15 minutes to write them. In fact, if I wasn't too nervous, I believe I can handle this question. It is not as hard as it seems. I know I need to use a finite state machine, but it took me some time to figure out the edge cases and I really don't have enough time to refine everything because I wasted some time doing the MCQs.

Anyways, the question asks you to parse a random string to its camelcase. That's it.

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

## Acknowlegdement

Although I think I won't be passing the online test (which is a pity in my opinion **_sigh_**), I still would like to thank Shopee for giving me the opportunity to try. It is a valuable experience to me. Thank you!