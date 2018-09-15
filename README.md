# Package consistent
A Golang implementation of Consistent Hashing and Consistent Hashing With Bounded Loads.

https://en.wikipedia.org/wiki/Consistent_hashing

https://research.googleblog.com/2017/04/consistent-hashing-with-bounded-loads.html


### Consistent Hashing Example

```go

package main

import (
	"log"
	"github.com/chenjinxuan/consistent"
)

func main() {
	c := consistent.New(255)

	// adds the hosts to the ring
	c.Add("127.0.0.1:8000")
	c.Add("92.0.0.1:8000")

	// Returns the host that owns `key`.
	//
	// As described in https://en.wikipedia.org/wiki/Consistent_hashing
	//
	// It returns ErrNoHosts if the ring has no hosts in it.
	host, err := c.Get("/app.html")
	if err != nil {
		log.Fatal(err)
	}

	log.Println(host)
}

```


### Consistent Hashing With Bounded Loads Example

```go

package main

import (
	"log"
	"github.com/chenjinxuan/consistent"
)

func main() {
	c := consistent.New(255)

	// adds the hosts to the ring
	c.Add("127.0.0.1:8000")
	c.Add("92.0.0.1:8000")

	// It uses Consistent Hashing With Bounded loads
	// https://research.googleblog.com/2017/04/consistent-hashing-with-bounded-loads.html
	// to pick the least loaded host that can serve the key
	//
	// It returns ErrNoHosts if the ring has no hosts in it.
	//
	host, err := c.GetLeast("/app.html")
	if err != nil {
		log.Fatal(err)
	}

	// increases the load of `host`, we have to call it before sending the request
	c.Inc(host)

	// send request or do whatever
	log.Println("send request to", host)

	// call it when the work is done, to update the load of `host`.
	c.Done(host)

}

```

### test
```
package main

import (
	"fmt"
	"github.com/chenjinxuan/consistent"
	"strconv"
)

func main() {
	a := consistent.New(255)
	a.Add("node1")
	a.Add("node2")
	a.Add("node3")
	a.Add("node4")
	var c = make(map[int]string)
	for i := 0; i < 100; i++ {
		c[i], _ = a.Get(strconv.Itoa(i))
	}
	a.Add("node5")
	var x = make(map[int]string)
	for i := 0; i < 100; i++ {
		x[i], _ = a.Get(strconv.Itoa(i))
	}
	a.Remove("node5")
	var w = make(map[int]string)
	for i := 0; i < 100; i++ {
		w[i], _ = a.Get(strconv.Itoa(i))
	}
	for i := 0; i < 100; i++ {
		fmt.Println(i, "==", c[i], "==", x[i], "==", w[i])
	}
}
```

## Docs

https://godoc.org/github.com/lafikl/consistent



# License

```
MIT License

Copyright (c) 2017 Khalid Lafi

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

```
