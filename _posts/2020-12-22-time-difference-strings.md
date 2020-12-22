---
layout: post
title: Passing Build Time in Go
author: Fahad Siddiqui
authorUrl: https://github.com/fahadsiddiqui
date: "2020-12-22 14:55:18"
---

To pass build time stamp in Go, you can use `-ldflags` when running `go build`.

For example you have a variable residing in the package `abc/build.go`

As

```go
package abc

var (
	BuildTime = "Tue Dec 22 13:22:49 PKT 2020"
)
```

You will do the following and pass value of `$(date)` (through shell)

> By default the `date` will format string as Unix timestamp

```bash
go build -ldflags="-X 'abc.BuildTime=$(date)'"
```

We can now use `time`'s function `.Parse(layout, str)` to process this string as a time stamp in `go`.

```
t1, err := time.Parse(time.UnixDate, BuildTime)
```

Here you can see the full example + a bonus function which calculates the elapsed time also (`GetStringForBuildTime()`).

```go
package abc

import (
	"fmt"
	"math"
	"strings"
	"time"
)

var (
	BuildTime = "Tue Dec 22 13:22:49 PKT 2020"
)

func BuildTimeElapsed() (hs float64, ms float64, ss float64) {
	t1, err := time.Parse(time.UnixDate, BuildTime)
	if err != nil {
		return 0, 0, 0
	}

	t2 := time.Now()

	var mf, sf float64

	hs = t2.Sub(t1).Hours()

	hs, mf = math.Modf(hs)
	ms = mf * 60

	ms, sf = math.Modf(ms)
	ss = sf * 60

	return
}

func GetStringForBuildTime() string {
	h, m, s := BuildTimeElapsed()

	var tsList []string

	if h > 0 {
		tsList = append(tsList, fmt.Sprintf("%.f hour(s)", h))
	}

	if m > 0 {
		tsList = append(tsList, fmt.Sprintf("%.f minute(s)", m))
	}

	if s > 0 {
		tsList = append(tsList, fmt.Sprintf("%.2f second(s)", s))
	}

	return strings.Join(tsList, ", ")
}
```

Have fun, coding!
