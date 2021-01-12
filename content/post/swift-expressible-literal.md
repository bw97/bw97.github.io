---
title: "Swift Expressible Literal"
date: 2021-01-12T10:33:52+08:00
draft: false
toc: false
tags: ["swift"]
---

> This is the English version of my [post at cnblogs.com](https://www.cnblogs.com/playground/p/swift-expressible-literal.html).

# A Question

Given the code bellow:

```swift
struct Book {
    let name: String
}

let book: Book = "Charlotte's Web"
```

Is there a way to make it work?

So, is there a way to implicitly create a struct from a string literal?

# Expressible Literal Protocol

The answer is yes, with the the magic of `expressible literal protocol`. It's a suite of protocols which includes:

| protocol | literal |
| :-- | :--: | 
| `ExpressibleByNilLiteral` | `nil` |
| `ExpressibleByBooleanLiteral` | boolean literal |
| `ExpressibleByIntegerLiteral` | interger literal |
| `ExpressibleByFloatLiteral` | float literal |
| `ExpressibleByUnicodeScalarLiteral` | unicode scalar literal |
| `ExpressibleByExtendedGraphemeClusterLiteral` | extended grapheme cluster literal |
| `ExpressibleByStringLiteral` | string literal|
| `ExpressibleByStringInterpolation` | interpolated string literal |
| `ExpressibleByArrayLiteral` | array literal |
| `ExpressibleByDictionaryLiteral` | dictionary literal |

In this case, adding `ExpressibleByStringLiteral` conformance can make the implicit instantiation possible.

```swift
extension Book: ExpressibleByStringLiteral {
    init(stringLiteral value: String) {
        self.init(name: value)
    }
}
```