+++
date = '2026-05-24'
draft = true
title = 'Syntax Test'
tags = ['test']
description = 'A page to test visual effects of various syntax.'
showToc = true
TocOpen = true
showReadingTime = true
showWordCount = true
math = true
+++

## Headings

# Heading 1
## Heading 2
### Heading 3
#### Heading 4
##### Heading 5
###### Heading 6

---

## Text Formatting

**Bold text**

*Italic text*

***Bold and italic***

~~Strikethrough~~

`Inline code`

Regular paragraph with a [link](https://example.com) and some text after it.

---

## Blockquotes

> This is a simple blockquote.

> Nested blockquote level 1
>
> > Nested blockquote level 2

---

## Lists

### Unordered List

- Item one
- Item two
  - Nested item
  - Another nested item
- Item three

### Ordered List

1. First item
2. Second item
   1. Nested ordered item
   2. Another nested ordered item
3. Third item

### Task List

- [x] Completed task
- [ ] Incomplete task
- [x] Another completed task

---

## Tables

| Name       | Type    | Default | Description          |
|------------|---------|---------|----------------------|
| `showToc`  | boolean | false   | Show table of contents |
| `draft`    | boolean | true    | Hide from production |
| `tags`     | array   | []      | Post tags            |

---

## Code Blocks

### With Filename

```bash {title="greet.sh"}
#!/bin/bash
echo "Hello, World!"
for i in {1..5}; do
  echo "Count: $i"
done
```

```python {title="greet.py"}
def greet(name: str) -> str:
    return f"Hello, {name}!"

if __name__ == "__main__":
    print(greet("World"))
```

### Bash

```bash
#!/bin/bash
echo "Hello, World!"
for i in {1..5}; do
  echo "Count: $i"
done
```

### Python

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"

if __name__ == "__main__":
    print(greet("World"))
```

### Go

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

### JavaScript

```javascript
const greet = (name) => `Hello, ${name}!`;
console.log(greet("World"));
```

### JSON

```json
{
  "name": "syntax-test",
  "version": "1.0.0",
  "tags": ["test", "demo"]
}
```

### TOML

```toml
[params]
  author = "ShiShiun Chen"
  showToc = true
```

---

## Images

![Placeholder image](https://placehold.co/800x400)

---

## Math

Inline math: \(E = mc^2\) and \(a^2 + b^2 = c^2\).

Block math (Gaussian integral):

\[
\int_{-\infty}^{\infty} e^{-x^2} \, dx = \sqrt{\pi}
\]

Block math (matrix multiplication):

\[
\begin{pmatrix} a & b \\ c & d \end{pmatrix} \begin{pmatrix} x \\ y \end{pmatrix} = \begin{pmatrix} ax + by \\ cx + dy \end{pmatrix}
\]

You can also use `$...$` and `$$...$$` for short expressions: $\pi \approx 3.14159$.

---

## Hugo Shortcodes

### YouTube

{{< youtube dQw4w9WgXcQ >}}

### Figure

{{< figure src="https://placehold.co/600x300" title="A placeholder figure" caption="This is the caption." >}}
