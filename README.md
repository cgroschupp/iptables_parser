# iptables-parser

Parse lines generated by `iptables-save`.
This parser is inspired by [Ben Johnson's SQL Parser](https://github.com/benbjohnson/sql-parser).

## Description

This parser parses lines returned from `iptables-save` or `iptables -S` and returns a Line or an Error.
A Line can be a Rule, Comment, Policy (default rule) or Header,
all of them being structs.

### Match Extensions

`iptables` has a lot of match extensions.
Only a few are implemented.
If one is not implemented, the parses returns an error for that line.

### Target Extensions

Just like in [Match Extensions](#Match-Extension), not all of the target extensions are implemented. 

## Example

```go
package main

import (
	"fmt"
	"log"

	ipt "github.com/coreos/go-iptables/iptables"
	iptp "github.com/kilo-io/iptables_parser"
)

func main() {
	t, err := ipt.NewWithProtocol(ipt.ProtocolIPv4)
	if err != nil {
		log.Fatal(err.Error())
	}
	rs, err := t.List("filter", "DOCKER")
	if err != nil {
		log.Fatal(err.Error())
	}
	for _, r := range rs {
		fmt.Println(r)
		tr, err := iptp.NewFromString(r)
		if err != nil {
			fmt.Printf("Error: %v", err)
			continue
		}
		switch r := tr.(type) {
		case iptp.Rule:
			fmt.Printf("rule parsed: %v\n", r)
		case iptp.Policy:
			fmt.Printf("policy parsed: %v\n", r)
		default:
			fmt.Printf("something else happend: %v\n", r)
		}

	}
}
```
