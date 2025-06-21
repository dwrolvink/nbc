# nbc
next-block-code

Toy encoding spec.


# Context 
In this spec we will refer to the `encoder` and the `reader`. The encoder makes the document intended to be read by the reader.
The encoder builds a document by stringing blocks together in a long chain.

Generally the first blocks are dedicated to setting the spec version, though this is not required and when absent version `1` is assumed.
There is no specification on what a set value actually means. The reader and encoder should agree between themselves. 
The spec in this page is intended as the default spec for the readers and encoders to at least negotiate which other version to use, if desired.

# Spec
## Core
- A document consists of messages, and messages consist of blocks
- Each block has 8 bits (default; can be overwritten).
- A message is either a `control message` or a `value message`
- First (maj; left) bit eq 1 for a control block.
  - So we can use 0-126 for single-block asci values (0-9,a-Z,:,/,etc)
- Most control messages are functions that expect one or more value messages as input
- When a control message expects a value message as input, further control messages may be prepended to the value message to further specify the value message.
- Control tables define what each control block does (incl defaults)
- Any block, or string of blocks, in a document before the first control message has no meaning in this spec and can be safely discarded.

| Asci repr. | Bits       | Name     | Description                                                       | Default value |
| :--------- | :--------- | :------- | :---------------------------------------------------------------- | :------------ |
| v          | 1 111 0110 | version  | Determines what the control bits do, length of blocks, etc        | 0 000 0001    |
| l          | 1 110 1100 | length   | Length of blocks                                                  | 0 000 1000    |
| n          | 1 110 1110 | next     | Next value message takes n blocks                                 | 0 000 0001    |
| e          | 1 110 0101 | extended | Next control messages takes n blocks                              | 0 000 0001    |
| a          | 1 110 0001 | asci     | Next value message should be interpreted as asci                  | -             |
| f          | 1 110 0110 | format   | Next value message should be interpreted as <?>                   | literal int   |

- Each control block can be used at any point in the document (e.g. version, length)
- Absence of version control block at position 1 assumes v=1 (this spec).
- We can write an assembly-like readable code. Parentheses may be used to improve legibility, but should not be required.

### Note on shortcode
- Shortcode is a way intended to quickly write nbc without having to type binary blocks. 
- Control blocks are represented by their asci representation
- Literal integers can be typed as is
- Asci values are typed as `"<value>"` or `'<value>'`
- In this document, shortcode will be surrounded by forward slashes

### Examples
Some examples to illustrate what we have asserted until now.
- The first line is the message to be encode
- The second line is the shortcode to achieve this
- The following lines are the generated blocks, where:
  - The first column is the binary code for the block
  - The second column is the shortcode representation
  - The third column is the resultant value as calculated by the parser

```
"version: 1"
$ parse 'v 1'
1 111 0110  v
0 000 0001    1

"version: v1"
$ parse 'v a n 2 "v1"'
1 111 0110  v           "v1"
1 110 0001    a         
1 110 1110      n       "v1"
0 000 0010        2
0 111 0110        "v"
0 011 0001        "1"

