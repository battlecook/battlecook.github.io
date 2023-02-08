---
layout: post
title:  golang 에서 파일이 unzip 으로 압축 되어있는지 확인하는 방법
comments: true
---



```go
func IsGzipCompressed(data []byte) bool {
    gzipHeaderSize := 10

    if len(data) < gzipHeaderSize {
        return false
    }

    gzipHeaderMagicNumber := []byte{0x1f, 0x8b}

    if bytes.Equal(data[:2], gzipHeaderMagicNumber) {
        return true
    }

    return false
}

```
