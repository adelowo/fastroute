[![Build Status](https://travis-ci.org/DATA-DOG/fastroute.svg?branch=master)](https://travis-ci.org/DATA-DOG/fastroute)
[![GoDoc](https://godoc.org/github.com/DATA-DOG/fastroute?status.svg)](https://godoc.org/github.com/DATA-DOG/fastroute)
[![codecov.io](https://codecov.io/github/DATA-DOG/fastroute/branch/master/graph/badge.svg)](https://codecov.io/github/DATA-DOG/fastroute)

# FastRoute

Insanely **fast** and **robust** http router for golang. Only **200**
lines of code. Uses standard **http.Handler** and has no limitations
to path matching compared to routers derived from **HttpRouter**.

``` go
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/DATA-DOG/fastroute"
)

func Index(w http.ResponseWriter, r *http.Request) {
	fmt.Fprint(w, "Welcome!\n")
}

func Hello(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "hello, %s!\n", fastroute.Parameters(r).ByName("name"))
}

func main() {
	log.Fatal(http.ListenAndServe(":8080", fastroute.New(
		fastroute.Route("/", Index),
		fastroute.Route("/hello/:name", Hello),
	)))
}
```

In overall, it is **not all in one** router, it is the same **http.Handler**
with do it yourself style, but with free **0 allocations** path pattern matching.
Feel free to just copy it and adapt to your needs.

This deserves a quote from **Rob Pike**:

> Fancy algorithms are slow when n is small, and n is usually small. Fancy
> algorithms have big constants. Until you know that n is frequently going
> to be big, don't get fancy.

The trade off this router makes is the size of **n**. But it provides
orthogonal building blocks, just like **http.Handler** does, in order to build
customized routers.

While all **HttpRouter** based implementations suffer from limitations such as
disallowing routes like **/user/new** and **/user/:user** together, due to
their tree structure. Or having custom **Handles** not compatible with
**http.Handler**. This router has none of these limitations.

By default this router does not provide:

1. Route based on HTTP method. Because it is simple to manage that in
   handler, middleware or custom router implementation.
2. Trailing slash redirects. Because that is simple to add a middleware
   for your preferred choice, before any route is looked up. You know
   and usually follow the same trailing slash strategy.
3. Fixed path redirects, you rarely need this for internal APIs or micro
   services. But if you do, again you follow your rules like: only lowercase,
   camel case. Then you may add a simple middleware to detect such anomalies
   and make a fixed path redirect.
4. All the panic recovery, not found, method not found, options or other middleware.
   That is up to your imagination, if such features are needed at all.

## Benchmarks

The benchmarks can be [found here](https://github.com/l3pp4rd/go-http-routing-benchmark/tree/fastroute).

Benchmark type            | repeats   | cpu time op    | mem op      | mem allocs op    |
--------------------------|----------:|---------------:|------------:|-----------------:|
Gin_Param                 |20000000   |    70.9 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_Param          |  500000   |    3116 ns/op  |   1056 B/op |     11 allocs/op |
HttpRouter_Param          |20000000   |     117 ns/op  |     32 B/op |      1 allocs/op |
**FastRoute_Param**       |20000000   |     105 ns/op  |      0 B/op |      0 allocs/op |
Pat_Param                 | 1000000   |    1910 ns/op  |    648 B/op |     12 allocs/op |
Gin_Param5                |20000000   |     117 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_Param5         |  300000   |    4597 ns/op  |   1184 B/op |     11 allocs/op |
HttpRouter_Param5         | 3000000   |     487 ns/op  |    160 B/op |      1 allocs/op |
**FastRoute_Param5**      |20000000   |     117 ns/op  |      0 B/op |      0 allocs/op |
Pat_Param5                |  300000   |    4717 ns/op  |    964 B/op |     32 allocs/op |
Gin_Param20               | 5000000   |     280 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_Param20        |  200000   |   11062 ns/op  |   3547 B/op |     13 allocs/op |
HttpRouter_Param20        | 1000000   |    1670 ns/op  |    640 B/op |      1 allocs/op |
**FastRoute_Param20**     |10000000   |     197 ns/op  |      0 B/op |      0 allocs/op |
Pat_Param20               |  100000   |   20735 ns/op  |   4687 B/op |    111 allocs/op |
Gin_ParamWrite            |10000000   |     170 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_ParamWrite     |  500000   |    3136 ns/op  |   1064 B/op |     12 allocs/op |
HttpRouter_ParamWrite     |10000000   |     156 ns/op  |     32 B/op |      1 allocs/op |
**FastRoute_ParamWrite**  |10000000   |     162 ns/op  |      0 B/op |      0 allocs/op |
Pat_ParamWrite            |  500000   |    3196 ns/op  |   1072 B/op |     17 allocs/op |
Gin_GithubStatic          |20000000   |    87.3 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_GithubStatic   |  100000   |   15215 ns/op  |    736 B/op |     10 allocs/op |
HttpRouter_GithubStatic   |30000000   |    49.7 ns/op  |      0 B/op |      0 allocs/op |
**FastRoute_GithubStatic**|20000000   |    60.3 ns/op  |      0 B/op |      0 allocs/op |
Pat_GithubStatic          |  200000   |   10970 ns/op  |   3648 B/op |     76 allocs/op |
Gin_GithubParam           |10000000   |     142 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_GithubParam    |  200000   |    9998 ns/op  |   1088 B/op |     11 allocs/op |
HttpRouter_GithubParam    | 5000000   |     301 ns/op  |     96 B/op |      1 allocs/op |
**FastRoute_GithubParam** | 5000000   |     387 ns/op  |      0 B/op |      0 allocs/op |
Pat_GithubParam           |  200000   |    7083 ns/op  |   2464 B/op |     48 allocs/op |
Gin_GithubAll             |   50000   |   28162 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_GithubAll      |     300   | 5731742 ns/op  | 211840 B/op |   2272 allocs/op |
HttpRouter_GithubAll      |   30000   |   49198 ns/op  |  13792 B/op |    167 allocs/op |
**FastRoute_GithubAll**   |   10000   |  179753 ns/op  |      5 B/op |      0 allocs/op |
Pat_GithubAll             |     300   | 4388066 ns/op  |1499571 B/op |  27435 allocs/op |
Gin_GPlusStatic           |20000000   |    73.3 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_GPlusStatic    | 1000000   |    2015 ns/op  |    736 B/op |     10 allocs/op |
HttpRouter_GPlusStatic    |30000000   |    34.0 ns/op  |      0 B/op |      0 allocs/op |
**FastRoute_GPlusStatic** |50000000   |    37.3 ns/op  |      0 B/op |      0 allocs/op |
Pat_GPlusStatic           | 5000000   |     330 ns/op  |     96 B/op |      2 allocs/op |
Gin_GPlusParam            |20000000   |    96.9 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_GPlusParam     |  300000   |    4334 ns/op  |   1056 B/op |     11 allocs/op |
HttpRouter_GPlusParam     |10000000   |     212 ns/op  |     64 B/op |      1 allocs/op |
**FastRoute_GPlusParam**  |10000000   |     145 ns/op  |      0 B/op |      0 allocs/op |
Pat_GPlusParam            | 1000000   |    2142 ns/op  |    688 B/op |     12 allocs/op |
Gin_GPlus2Params          |10000000   |     121 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_GPlus2Params   |  200000   |    8264 ns/op  |   1088 B/op |     11 allocs/op |
HttpRouter_GPlus2Params   |10000000   |     232 ns/op  |     64 B/op |      1 allocs/op |
**FastRoute_GPlus2Params**| 5000000   |     351 ns/op  |      0 B/op |      0 allocs/op |
Pat_GPlus2Params          |  200000   |    6557 ns/op  |   2256 B/op |     34 allocs/op |
Gin_GPlusAll              | 1000000   |    1279 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_GPlusAll       |   20000   |   66580 ns/op  |  13296 B/op |    142 allocs/op |
HttpRouter_GPlusAll       | 1000000   |    2358 ns/op  |    640 B/op |     11 allocs/op |
**FastRoute_GPlusAll**    |  500000   |    2546 ns/op  |      0 B/op |      0 allocs/op |
Pat_GPlusAll              |   30000   |   47673 ns/op  |  16576 B/op |    298 allocs/op |
Gin_ParseStatic           |20000000   |    71.2 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_ParseStatic    |  500000   |    2971 ns/op  |    752 B/op |     11 allocs/op |
HttpRouter_ParseStatic    |50000000   |    32.1 ns/op  |      0 B/op |      0 allocs/op |
**FastRoute_ParseStatic** |30000000   |    42.3 ns/op  |      0 B/op |      0 allocs/op |
Pat_ParseStatic           | 2000000   |     781 ns/op  |    240 B/op |      5 allocs/op |
Gin_ParseParam            |20000000   |    79.2 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_ParseParam     |  500000   |    3710 ns/op  |   1088 B/op |     12 allocs/op |
HttpRouter_ParseParam     |10000000   |     181 ns/op  |     64 B/op |      1 allocs/op |
**FastRoute_ParseParam**  |10000000   |     184 ns/op  |      0 B/op |      0 allocs/op |
Pat_ParseParam            |  500000   |    3165 ns/op  |   1120 B/op |     17 allocs/op |
Gin_Parse2Params          |20000000   |    91.5 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_Parse2Params   |  500000   |    3916 ns/op  |   1088 B/op |     11 allocs/op |
HttpRouter_Parse2Params   |10000000   |     212 ns/op  |     64 B/op |      1 allocs/op |
**FastRoute_Parse2Params**|10000000   |     147 ns/op  |      0 B/op |      0 allocs/op |
Pat_Parse2Params          |  500000   |    2980 ns/op  |    832 B/op |     17 allocs/op |
Gin_ParseAll              | 1000000   |    2264 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_ParseAll       |   10000   |  125569 ns/op  |  24864 B/op |    292 allocs/op |
HttpRouter_ParseAll       |  500000   |    3124 ns/op  |    640 B/op |     16 allocs/op |
**FastRoute_ParseAll**    |  500000   |    3324 ns/op  |      0 B/op |      0 allocs/op |
Pat_ParseAll              |   30000   |   56328 ns/op  |  17264 B/op |    343 allocs/op |
Gin_StaticAll             |  100000   |   19064 ns/op  |      0 B/op |      0 allocs/op |
GorillaMux_StaticAll      |    1000   | 1536755 ns/op  | 115648 B/op |   1578 allocs/op |
**FastRoute_StaticAll**   |  200000   |    9149 ns/op  |      0 B/op |      0 allocs/op |
HttpRouter_StaticAll      |  200000   |   10824 ns/op  |      0 B/op |      0 allocs/op |
Pat_StaticAll             |    1000   | 1597577 ns/op  | 533904 B/op |  11123 allocs/op |

We can see that **FastRoute** outperforms routers in most of the cases. In general it always boils
down to targeted case implementation.

**FastRoute** was easily adapted for this benchmark. Where static routes are served, nothing
is better or faster than a **hashmap**. **FastRoute** allows to build any kind of router,
depending on an use case. By default it targets smaller number of routes and the weakest
link is a lot of dynamic routes, because these are matched one by one.

## Contributions

Feel free to open a pull request. Note, if you wish to contribute an extension to public (exported methods or types) -
please open an issue before to discuss whether these changes can be accepted. All backward incompatible changes are
and will be treated cautiously.

## License

**FastRoute** is licensed under the [three clause BSD license][license]

[license]: http://en.wikipedia.org/wiki/BSD_licenses "The three clause BSD license"