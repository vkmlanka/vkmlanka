### Hi there 👋

<!--
**vkmlanka/vkmlanka** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
@@ -94,4 +94,7 @@ $1
s
$7
aaaaaaa
```
```

# Customize data usage

60
bytefmt/bytefmt.go
@@ -10,16 +10,16 @@ import (
)

const (
	BYTE = 1 << (10 * iota)
	KILOBYTE
	MEGABYTE
	GIGABYTE
	TERABYTE
	PETABYTE
	EXABYTE
	sizeByte = 1 << (10 * iota)
	sizeKilo
	sizeMega
	sizeGiga
	sizeTera
	sizePeta
	sizeExa
)

var invalidByteQuantityError = errors.New("byte quantity must be a positive integer with a unit of measurement like M, MB, MiB, G, GiB, or GB")
var errInvalidByteQuantity = errors.New("byte quantity must be a positive integer with a unit of measurement like M, MB, MiB, G, GiB, or GB")

// FormatSize returns a human-readable byte string of the form 10M, 12.5K, and so forth.  The following units are available:
//  E: Exabyte
@@ -35,25 +35,25 @@ func FormatSize(bytes uint64) string {
	value := float64(bytes)

	switch {
	case bytes >= EXABYTE:
	case bytes >= sizeExa:
		unit = "E"
		value = value / EXABYTE
	case bytes >= PETABYTE:
		value = value / sizeExa
	case bytes >= sizePeta:
		unit = "P"
		value = value / PETABYTE
	case bytes >= TERABYTE:
		value = value / sizePeta
	case bytes >= sizeTera:
		unit = "T"
		value = value / TERABYTE
	case bytes >= GIGABYTE:
		value = value / sizeTera
	case bytes >= sizeGiga:
		unit = "G"
		value = value / GIGABYTE
	case bytes >= MEGABYTE:
		value = value / sizeGiga
	case bytes >= sizeMega:
		unit = "M"
		value = value / MEGABYTE
	case bytes >= KILOBYTE:
		value = value / sizeMega
	case bytes >= sizeKilo:
		unit = "K"
		value = value / KILOBYTE
	case bytes >= BYTE:
		value = value / sizeKilo
	case bytes >= sizeByte:
		unit = "B"
	case bytes == 0:
		return "0"
@@ -78,31 +78,31 @@ func ParseSize(s string) (uint64, error) {
	i := strings.IndexFunc(s, unicode.IsLetter)

	if i == -1 {
		return 0, invalidByteQuantityError
		return 0, errInvalidByteQuantity
	}

	bytesString, multiple := s[:i], s[i:]
	bytes, err := strconv.ParseFloat(bytesString, 64)
	if err != nil || bytes <= 0 {
		return 0, invalidByteQuantityError
		return 0, errInvalidByteQuantity
	}

	switch multiple {
	case "E", "EB", "EIB":
		return uint64(bytes * EXABYTE), nil
		return uint64(bytes * sizeExa), nil
	case "P", "PB", "PIB":
		return uint64(bytes * PETABYTE), nil
		return uint64(bytes * sizePeta), nil
	case "T", "TB", "TIB":
		return uint64(bytes * TERABYTE), nil
		return uint64(bytes * sizeTera), nil
	case "G", "GB", "GIB":
		return uint64(bytes * GIGABYTE), nil
		return uint64(bytes * sizeGiga), nil
	case "M", "MB", "MIB":
		return uint64(bytes * MEGABYTE), nil
		return uint64(bytes * sizeMega), nil
	case "K", "KB", "KIB":
		return uint64(bytes * KILOBYTE), nil
		return uint64(bytes * sizeKilo), nil
	case "B":
		return uint64(bytes), nil
	default:
		return 0, invalidByteQuantityError
		return 0, errInvalidByteQuantity
	}
}
6
bytefmt/bytefmt_test.go
@@ -33,13 +33,13 @@ func TestFormatSize(t *testing.T) {
}

func TestParseSize(t *testing.T) {
	if _, err := ParseSize("0"); err != invalidByteQuantityError {
	if _, err := ParseSize("0"); err != errInvalidByteQuantity {
		t.Error("parse error")
	}
	if _, err := ParseSize("0B"); err != invalidByteQuantityError {
	if _, err := ParseSize("0B"); err != errInvalidByteQuantity {
		t.Error("parse error")
	}
	if _, err := ParseSize("1A"); err != invalidByteQuantityError {
	if _, err := ParseSize("1A"); err != errInvalidByteQuantity {
		t.Error("parse error")
	}
	if b, err := ParseSize("123B"); err != nil || b != 123 {
8
helper/resp.go
@@ -6,20 +6,20 @@ import (
	"strconv"
)

const CRLF = "\r\n"
const crlf = "\r\n"

// CmdLine is alias for [][]byte, represents a command line
type CmdLine = [][]byte

func makeMultiBulkResp(args [][]byte) []byte {
	argLen := len(args)
	var buf bytes.Buffer
	buf.WriteString("*" + strconv.Itoa(argLen) + CRLF)
	buf.WriteString("*" + strconv.Itoa(argLen) + crlf)
	for _, arg := range args {
		if arg == nil {
			buf.WriteString("$-1" + CRLF)
			buf.WriteString("$-1" + crlf)
		} else {
			buf.WriteString("$" + strconv.Itoa(len(arg)) + CRLF + string(arg) + CRLF)
			buf.WriteString("$" + strconv.Itoa(len(arg)) + crlf + string(arg) + crlf)
		}
	}
	return buf.Bytes()
3
shortcut.go → parser/portal.go
@@ -1,4 +1,5 @@
package main
// Package parser is interface for parser
package parser

import (
	"github.com/hdt3213/rdb/core"
@@ -34,64 +34,67 @@
```
You can get some rdb examples in [cases](https://github.com/HDT3213/rdb/tree/master/cases)
The examples for json result:
```json
[
    {"db":0,"key":"hash","size":64,"type":"hash","hash":{"ca32mbn2k3tp41iu":"ca32mbn2k3tp41iu","mddbhxnzsbklyp8c":"mddbhxnzsbklyp8c"}},
    {"db":0,"key":"string","size":10,"type":"string","value":"aaaaaaa"},
    {"db":0,"key":"expiration","expiration":"2022-02-18T06:15:29.18+08:00","size":8,"type":"string","value":"zxcvb"},
    {"db":0,"key":"list","expiration":"2022-02-18T06:15:29.18+08:00","size":66,"type":"list","values":["7fbn7xhcnu","lmproj6c2e","e5lom29act","yy3ux925do"]},
    {"db":0,"key":"zset","expiration":"2022-02-18T06:15:29.18+08:00","size":57,"type":"zset","entries":[{"member":"zn4ejjo4ths63irg","score":1},{"member":"1ik4jifkg6olxf5n","score":2}]},
    {"db":0,"key":"set","expiration":"2022-02-18T06:15:29.18+08:00","size":39,"type":"set","members":["2hzm5rnmkmwb3zqd","tdje6bk22c6ddlrw"]}
]
```
# Generate Memory Report
RDB uses rdb encoded size to estimate redis memory usage.
```
rdb -c memory -o <output_path> <source_path>
```
Example:
```
rdb -c memory -o mem.csv cases/memory.rdb
```
The examples for json result:
```csv
database,key,type,size,size_readable,element_count
0,hash,hash,64,64B,2
0,s,string,10,10B,0
0,e,string,8,8B,0
0,list,list,66,66B,4
0,zset,zset,57,57B,2
0,large,string,2056,2K,0
0,set,set,39,39B,2
```
# Convert to AOF
Usage:
```
rdb -c aof -o <output_path> <source_path>
```
Example:
```
rdb -c aof -o mem.aof cases/memory.rdb
```
The examples for aof result:
```
*3
$3
SET
$1
s
$7
aaaaaaa
```
```

# Customize data usage

60
bytefmt/bytefmt.go
@@ -10,16 +10,16 @@ import (
)

const (
	BYTE = 1 << (10 * iota)
	KILOBYTE
	MEGABYTE
	GIGABYTE
	TERABYTE
	PETABYTE
	EXABYTE
	sizeByte = 1 << (10 * iota)
	sizeKilo
	sizeMega
	sizeGiga
	sizeTera
	sizePeta
	sizeExa
)

var invalidByteQuantityError = errors.New("byte quantity must be a positive integer with a unit of measurement like M, MB, MiB, G, GiB, or GB")
var errInvalidByteQuantity = errors.New("byte quantity must be a positive integer with a unit of measurement like M, MB, MiB, G, GiB, or GB")

// FormatSize returns a human-readable byte string of the form 10M, 12.5K, and so forth.  The following units are available:
//  E: Exabyte
@@ -35,25 +35,25 @@ func FormatSize(bytes uint64) string {
	value := float64(bytes)

	switch {
	case bytes >= EXABYTE:
	case bytes >= sizeExa:
		unit = "E"
		value = value / EXABYTE
	case bytes >= PETABYTE:
		value = value / sizeExa
	case bytes >= sizePeta:
		unit = "P"
		value = value / PETABYTE
	case bytes >= TERABYTE:
		value = value / sizePeta
	case bytes >= sizeTera:
		unit = "T"
		value = value / TERABYTE
	case bytes >= GIGABYTE:
		value = value / sizeTera
	case bytes >= sizeGiga:
		unit = "G"
		value = value / GIGABYTE
	case bytes >= MEGABYTE:
		value = value / sizeGiga
	case bytes >= sizeMega:
		unit = "M"
		value = value / MEGABYTE
	case bytes >= KILOBYTE:
		value = value / sizeMega
	case bytes >= sizeKilo:
		unit = "K"
		value = value / KILOBYTE
	case bytes >= BYTE:
		value = value / sizeKilo
	case bytes >= sizeByte:
		unit = "B"
	case bytes == 0:
		return "0"
@@ -78,31 +78,31 @@ func ParseSize(s string) (uint64, error) {
	i := strings.IndexFunc(s, unicode.IsLetter)

	if i == -1 {
		return 0, invalidByteQuantityError
		return 0, errInvalidByteQuantity
	}

	bytesString, multiple := s[:i], s[i:]
	bytes, err := strconv.ParseFloat(bytesString, 64)
	if err != nil || bytes <= 0 {
		return 0, invalidByteQuantityError
		return 0, errInvalidByteQuantity
	}

	switch multiple {
	case "E", "EB", "EIB":
		return uint64(bytes * EXABYTE), nil
		return uint64(bytes * sizeExa), nil
	case "P", "PB", "PIB":
		return uint64(bytes * PETABYTE), nil
		return uint64(bytes * sizePeta), nil
	case "T", "TB", "TIB":
		return uint64(bytes * TERABYTE), nil
		return uint64(bytes * sizeTera), nil
	case "G", "GB", "GIB":
		return uint64(bytes * GIGABYTE), nil
		return uint64(bytes * sizeGiga), nil
	case "M", "MB", "MIB":
		return uint64(bytes * MEGABYTE), nil
		return uint64(bytes * sizeMega), nil
	case "K", "KB", "KIB":
		return uint64(bytes * KILOBYTE), nil
		return uint64(bytes * sizeKilo), nil
	case "B":
		return uint64(bytes), nil
	default:
		return 0, invalidByteQuantityError
		return 0, errInvalidByteQuantity
	}
}
6
bytefmt/bytefmt_test.go
@@ -13,33 +13,33 @@
		t.Error("format error")
	}
	if FormatSize(123*(1<<10)) != "123K" {
		t.Error("format error")
	}
	if FormatSize(123*(1<<20)) != "123M" {
		t.Error("format error")
	}
	if FormatSize(123*(1<<30)) != "123G" {
		t.Error("format error")
	}
	if FormatSize(123*(1<<40)) != "123T" {
		t.Error("format error")
	}
	if FormatSize(123*(1<<50)) != "123P" {
		t.Error("format error")
	}
	if FormatSize(math.MaxUint64) != "16E" {
		t.Error("format error")
	}
}

func TestParseSize(t *testing.T) {
	if _, err := ParseSize("0"); err != invalidByteQuantityError {
	if _, err := ParseSize("0"); err != errInvalidByteQuantity {
		t.Error("parse error")
	}
	if _, err := ParseSize("0B"); err != invalidByteQuantityError {
	if _, err := ParseSize("0B"); err != errInvalidByteQuantity {
		t.Error("parse error")
	}
	if _, err := ParseSize("1A"); err != invalidByteQuantityError {
	if _, err := ParseSize("1A"); err != errInvalidByteQuantity {
		t.Error("parse error")
	}
	if b, err := ParseSize("123B"); err != nil || b != 123 {
8
helper/resp.go
@@ -6,20 +6,20 @@ import (
	"strconv"
)

const CRLF = "\r\n"
const crlf = "\r\n"

// CmdLine is alias for [][]byte, represents a command line
type CmdLine = [][]byte

func makeMultiBulkResp(args [][]byte) []byte {
	argLen := len(args)
	var buf bytes.Buffer
	buf.WriteString("*" + strconv.Itoa(argLen) + CRLF)
	buf.WriteString("*" + strconv.Itoa(argLen) + crlf)
	for _, arg := range args {
		if arg == nil {
			buf.WriteString("$-1" + CRLF)
			buf.WriteString("$-1" + crlf)
		} else {
			buf.WriteString("$" + strconv.Itoa(len(arg)) + CRLF + string(arg) + CRLF)
			buf.WriteString("$" + strconv.Itoa(len(arg)) + crlf + string(arg) + crlf)
		}
	}
	return buf.Bytes()
3
shortcut.go → parser/portal.go
@@ -1,4 +1,5 @@
package main
// Package parser is interface for parser
package parser

import (
	"github.com/hdt3213/rdb/core"
  
