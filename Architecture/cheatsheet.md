# Markdown Cheat Sheet

Thanks for visiting [The Markdown Guide](https://www.markdownguide.org)!

This Markdown cheat sheet provides a quick overview of all the Markdown syntax elements. It can’t cover every edge case, so if you need more information about any of these elements, refer to the reference guides for [basic syntax](https://www.markdownguide.org/basic-syntax) and [extended syntax](https://www.markdownguide.org/extended-syntax).

$$\begin{CD} A @>a>> B \\ @VbVV @AdAA \\ C @>f^2>> D \end{CD}$$

## Basic Syntax

These are the elements outlined in John Gruber’s original design document. All Markdown applications support these elements.

### Heading

# H1
## H2
### H3

### Bold

**bold text**

### Italic

*italicized text*

### Blockquote

> blockquote

### Ordered List

1. First item
2. Second item
3. Third item

### Unordered List

- First item
- Second item
- Third item

### Code

`code`

### Horizontal Rule

---

### Link

[Markdown Guide](https://www.markdownguide.org)

### Image

![alt text](https://www.markdownguide.org/assets/images/tux.png)

## Extended Syntax

These elements extend the basic syntax by adding additional features. Not all Markdown applications support these elements.

### Table

| Syntax | Description |
| ----------- | ----------- |
| Header | Title |
| Paragraph | Text |

### Fenced Code Block

```
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```

### Footnote

Here's a sentence with a footnote. [^1]

[^1]: This is the footnote.

### Heading ID

### My Great Heading {#custom-id}

### Definition List

term
: definition

### Strikethrough

~~The world is flat.~~

### Task List

- [x] Write the press release
- [ ] Update the website
- [ ] Contact the media

### Emoji

That is so funny! :joy:

(See also [Copying and Pasting Emoji](https://www.markdownguide.org/extended-syntax/#copying-and-pasting-emoji))

### Highlight

I need to highlight these ==very important words==.

### Subscript

H~2~O

### Superscript

$X^2$

### LaTex
$$x^2 \sin(x) \cup \notin \dot{a} \ddot{a} \Sigma^{*} \bar{x} \bar{\boldsymbol{\Gamma^t \Delta, \epsilon, \leq, }}$$


$$
E_{CFG} = \{ \langle G \rangle | G \text{ is a CFG and } L(G) = \emptyset \}
$$



$$
D^2_N = \left\\{\begin{array}{c}
D^2_{N-1} + 2D_{N-1} + 1 = 10\\\\
or\\\\
D^2_{N-1} - 2D_{N-1} + 1 = 20\\
\end{array}\right.
\tag{6.7}
$$

$$\begin{CD} A @>a>> B \\ @VbVV \\ F @= D \end{CD}$$

$$\begin{CD} A @>a>> B \\ @VbVV @AdAA \\ C @>f^2>> D \end{CD}$$