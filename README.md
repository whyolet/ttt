![ttt icon](icons/ttt1280x640.png)

# TTT

TTT (Text Tree/Table) is a simple format for nested lists, maps, and tables of text data, merging the best ideas from CSV, JSON, YAML, and others.

## Example

```
feature, example
:
unquoted text, hello world! 👋

quoted text, "it can include
newlines, empty lines,
self-escaped "" quote,
[,]{:}(#) characters,
leading/trailing whitespace "

indented text, (
  like quoted text, but
  without need to escape "
  and ideal for indented
    nested structures
)

list, [
  multiline
  multiline
  inline, [inline], [
    csv-like,list,of,lists
    for,data,without,header
  ]
]

table, [
  id,name,email,notes
  :
  1,Alice,a@example.com,curious
  2,Bob,b@example.com,""
  # Whole big example
  # is a table too!
]

comment, is ignored  # comment

map, {
  key: text
  indented(
    text
    here
  )
  list[
    multiline: list
    of{inline: maps}, is[here]
  ]
  map{is: inline}
  # Big map here is multiline.
}

dsl, [
  select[a, b, c],from[
    table
  ],where{and[
    eq[a, b]
    ne[b, c]
  ]}
]

compact mode,{
level2{
level3{}
}
}
```

## Why

* [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) doesn't support nested lists, maps, and reliable header presence check.
* [JSON](https://json.org/) is too verbose, yet lacks comments and tables.
* [YAML](https://yaml.org/) has too complex [spec](https://yaml.org/spec/1.2.2/) and [implicit typing surprises](https://hitchdev.com/strictyaml/why/implicit-typing-removed/).
* Other alternatives have similar limitations, from A to Z:
  * [CSON](https://github.com/bevry/cson)
  * [edn](https://github.com/edn-format/edn)
  * [ELDF](https://eldf.org/)
  * [ENO](https://eno-lang.org/)
  * [HCL](https://github.com/hashicorp/hcl)
  * [HJSON](https://hjson.github.io/)
  * [HOCON](https://github.com/lightbend/config/blob/master/HOCON.md)
  * [HRON](https://github.com/mrange/hron)
  * [INI](https://en.wikipedia.org/wiki/INI_file)
  * [ION](https://amzn.github.io/ion-docs/)
  * [JONF](https://github.com/whyolet/jonf) - my attempt #1.
  * [JSON5](https://json5.org/)
  * [JSONC](https://jsonc.org/)
  * [Jsonnet](https://jsonnet.org/)
  * [NestedText](https://nestedtext.org/)
  * [OpenDDL](https://openddl.org/)
  * [OGDL](https://ogdl.org/spec/)
  * [Property list](https://en.wikipedia.org/wiki/Property_list)
  * [Query string](https://en.wikipedia.org/wiki/Query_string#Web_forms)
  * [SDLang](https://sdlang.org/)
  * [S-expression](https://en.wikipedia.org/wiki/S-expression)
  * [StrictYAML](https://hitchdev.com/strictyaml/)
  * [TOML](https://toml.io/en/)
  * [txtt](https://github.com/whyolet/txtt) - my attempt #2.
  * [XML](https://en.wikipedia.org/wiki/XML)
* Keeping it simple yet versatile is definitely a challenge.
  * TTT is my attempt #3 to solve it.
  * Third time's the charm?

## Rules

### File

* TTT format supports Unicode as is, without escape sequences like `\uFFFF`.
* Encoding is always UTF-8 to keep TTT parser and formatter simple. [BOM](https://en.wikipedia.org/wiki/Byte_order_mark) is not supported.
* TTT file has `.ttt` extension and `text/ttt` content type (to be registered).
* TTT file is parsed line by line with a simple strict state machine.
* Invalid input leads to explicit error and no output at all to fix the issues earlier.
* Initial state is an implicit multiline list.
  * This allows to support zero, one, or multiple root values in one file naturally without additional concepts like "document" and separators like `---` in YAML.
  * Empty file parses to an empty list:
    ```
    # TTT file:

    # JSON file:
    []
    ```

### Comment

```
# comment

value # inline comment
```

* Comment is:
  * `#` character,
  * zero or more characters
    until a newline or the end of file.
* Comment styles other than `#` are not supported to keep it standardized and minimalist.

### Value

* Value is either a text, a list, a map, or a table.

### Text

* Text is either unquoted, quoted, or indented.
* Escape sequences like `\n` are not supported:
  * almost all characters can be represented as is,
  * few cases unsupported by unquoted text are covered by quoted text and indented text.

#### Unquoted text

```
hello world! 👋
```

* Unquoted text is one or more characters except few unsupported:
  * newlines,
  * leading/trailing whitespace,
  * 10 special characters:  
    `[,]{:}(#)"`
* Unquoted text is parsed as a literal text string value, never a number, a boolean, or anything else.

#### Quoted text

```
"quoted text
can include newlines,

empty lines,
self-escaped "" quote,
[,]{:}(#) characters,
leading/trailing whitespace "

# JSON:
"quoted text\ncan include newlines,\n\nempty lines,\nself-escaped \" quote,\n[,]{:}(#) characters,\nleading/trailing whitespace "
```

* Quoted text is:
  * `"` character,
  * zero or more:
    * `""` pairs,
    * characters other than `"`
  * `"` character.
* Quoted text is parsed as a literal text string value after removing the opening and closing `"` and replacing each `""` with a single `"`.
* Multiline text in an indented structure is better represented with an indented text.

#### Indented text

```
level1{
  level2{
    level3(
      indented text
      can include newlines,

      empty lines,
      [,]{:}(#)" characters,
      leading/trailing whitespace

    )
  }
}

# JSON:
{
  "level1": {
    "level2": {
      "level3": "indented text\ncan include newlines,\n\nempty lines,\n[,]{:}(#)\" characters,\nleading/trailing whitespace\n"
    }
  }
}
```

* Indented text is:
  * `(` character in a line indented with N spaces,
  * a newline,
  * zero or more:
    * empty lines,
    * lines indented with N+2 or more spaces,
  * a newline,
  * N spaces,
  * `)` character.
* Indented text is parsed as a literal text string value after deletion of:
  * 2 opening and 2 closing lines,
  * the first N+2 spaces from each non-empty line, even if some lines start with more than N+2 spaces.

```
(
  indented
  text
)

# JSON:
"indented\ntext"
```

```
(

  indented
    text

)

# JSON:
"\nindented\n  text\n"
```

### Indentation

```
# Indented TTT:
quotes[
  {
    text(
      You can have
      any color you want,

        as long as it's black.
    )
    author: Henry Ford
  }
]
```

* One indentation level is exactly 2 spaces to keep it:
  * standardized,
  * compact enough,
  * very readable.
* However, sometimes a big file with deeply nested indentation may break a [size limit](https://kubernetes.io/docs/concepts/configuration/secret/#restriction-data-size), leading to workarounds such as converting indented YAML to a more compact one-line JSON [here](https://github.com/VictoriaMetrics/helm-charts/pull/1889/files#diff-394304da26181c13863c5f7658226a622e2c9326814639607313b06a36e78b32R17).
* For such edge cases indentation in TTT is optional.
* Indented TTT example above and compact TTT example below are equivalent.

```
# Compact TTT:
quotes[{
text:"You can have
any color you want,

  as long as it's black."
author:Henry Ford
}]
```

* This compact TTT (96 bytes) is:
  * 48% smaller than indented JSON (142 bytes),
  * 40% smaller than indented TTT (134 bytes),
  * 25% smaller than properly indented YAML (120 bytes),
  * 16% smaller than equally compact JSON (111 bytes),
  * 15% smaller than compact YAML (110 bytes),
  * 13% smaller than the most compact JSON (108 bytes).
* Compact TTT beats the main competitors even with such a small example.
* More deeply nested structure will make the difference even bigger.
* TTT formatter produces indented output by default, with explicit flag required to produce compact output.

```
# Properly indented YAML:
quotes:
  - text: |
      You can have
      any color you want,

        as long as it's black.
    author: Henry Ford

# Compact YAML:
quotes:
- text: |
    You can have
    any color you want,

      as long as it's black.
  author: Henry Ford

# Indented JSON:
{
  "quotes": [
    {
      "text": "You can have\nany color you want,\n\n  as long as it's black.",
      "author": "Henry Ford"
    }
  ]
}

# Equally compact JSON:
{"quotes":[{
"text":"You can have\nany color you want,\n\n  as long as it's black.",
"author":"Henry Ford"
}]}

# The most compact JSON:
{"quotes":[{"text":"You can have\nany color you want,\n\n  as long as it's black.","author":"Henry Ford"}]}
```

### Newline

* Both parser and formatter treat newline as `\n` only,
* because accepting `\r\n` and single `\r` to implement the [robustness principle](https://en.wikipedia.org/wiki/Robustness_principle) would actually lead to the [lack of robustness](https://en.wikipedia.org/wiki/Robustness_principle#Criticism),
* and normalization of `\r\n` and `\r` to `\n` would require a special way to preserve `\r` in the output,
* like a `"quoted text with escape sequences like \r\n"`,
* which would increase complexity without much value added.

### List

* List is either inline or multiline.

#### Inline list

```
foo, bar baz, [nested, list here]

# JSON:
["foo", "bar baz", ["nested", "list here"]]
```

* Inline list is either explicit or implicit.
* Explicit inline list is:
  * `[` character,
  * implicit inline list,
  * `]` character.
* Implicit inline list is zero or more items separated by a comma.
  * Trailing comma is invalid as not adding value here.
* Inline list item is:
  * zero or more spaces,
  * a value,
  * zero or more spaces.

```
a,b, c, d ,  e  

# JSON:
["a", "b", "c", "d", "e"]
```

#### Multiline list

```
foo
bar baz
[
  nested
  list here
]

# JSON:
[
  "foo",
  "bar baz",
  [
    "nested",
    "list here"
  ]
]
```

* Multiline list is either explicit or implicit.
* Explicit multiline list is:
  * `[` character in a line indented with N spaces,
  * a newline,
  * implicit multiline list indented with either zero or N+2 spaces,
  * a newline,
  * N spaces,
  * `]` character.
* Implicit multiline list is zero or more items separated by a newline.
* Multiline list item is:
  * zero or more:
    * spaces,
    * empty lines,
    * comment lines,
  * a value,
  * zero or more:
    * spaces,
    * empty lines,
    * comment lines.

```
a
b
# comment

  c
d
""
[
nested
list here
]

# JSON:
[
  "a", "b", "c", "d", "",
  ["nested", "list here"]
]
```

#### Nested list

* CSV table looks like TTT implicit multiline list with nested implicit inline lists:
  ```
  # CSV, TTT:
  a,aa,aaa
  b,bb,bbb

  # JSON:
  [
    ["a", "aa", "aaa"],
    ["b", "bb", "bbb"]
  ]
  ```
* See [Table](#table) for TTT version of a CSV table with a header.
* Inline list with two or more items can be implicit when it is nested in a multiline list:
  ```
  multiline
  inline, inline
  multiline
  ```
* It can be explicit too:
  ```
  multiline
  [inline, inline]
  multiline
  ```
* All other cases of nested lists have to be explicit:
  ```
  multiline
  []
  multiline

  multiline
  [inline]
  multiline

  multiline
  [
    multiline
    multiline
  ]
  multiline

  inline, [
    multiline
    multiline
  ], inline
  
  inline, [inline, inline], inline
  ```

### Map

* Map is either inline or multiline.

#### Inline map

```
foo: bar baz, indented(
  text
  here
), list[a, b], map{k: v}

# JSON:
{
  "foo": "bar baz",
  "indented": "text\nhere",
  "list": ["a", "b"],
  "map": {"k": "v"}
}
```

* Inline map is either explicit or implicit.
* Explicit inline map is:
  * `{` character,
  * implicit inline map,
  * `}` character.
* Implicit inline map is zero or more items separated by a comma.
* Inline map item is either basic or advanced.
* Basic map item is:
  * zero or more spaces,
  * a text key,
  * zero or more spaces,
  * `:` character,
  * zero or more spaces,
  * a quoted or unquoted text,
  * zero or more spaces.
  ```
  a:b, c: d , e : f

  # JSON:
  {"a": "b", "c": "d", "e": "f"}
  ```
* Advanced map item is:
  * zero or more spaces,
  * a text key,
  * either of:
    * indented text,
    * explicit list,
    * explicit map,
  * zero or more spaces.
  ```
  indented(
    text
  ), list[], map{}

  # JSON:
  {
    "indented": "text",
    "list": [],
    "map": {}
  }
  ```
* Duplicate keys are [invalid](https://hitchdev.com/strictyaml/why/duplicate-keys-disallowed/).
* Empty key is quoted:
  ```
  "": unquoted
  "": "quoted"
  ""(
    indented
  )
  ""[]
  ""{}
  ```

#### Multiline map

```
{
  foo: bar baz
  multiline(
    text
    here
  )
  list[
    a
    b
  ]
  map{
    k: v
    key: val
  }
}

# JSON:
{
  "foo": "bar baz",
  "multiline": "text\nhere",
  "list": [
    "a",
    "b",
  ],
  "map": {
    "k": "v",
    "key": "val"
  }
}
```

* Multiline map is always explicit to avoid confusion with a list of inline maps as in this [DSL example](#dsl-example).
* Multiline map is:
  * `{` character in a line indented with N spaces,
  * a newline,
  * zero or more items separated by a newline and indented with either zero or N+2 spaces,
  * a newline,
  * N spaces,
  * `}` character.
* Multiline map item is:
  * zero or more:
    * spaces,
    * empty lines,
    * comment lines,
  * either a basic
    or an advanced map item,
  * zero or more:
    * spaces,
    * empty lines,
    * comment lines.

```
{
# compact version
# of the previous example

foo: bar baz
multiline: "text
here"
list[a,b]
map{k:v,key:val}
}

# JSON is the same
# as in the previous example.
```

#### Nested map

* Inline map with one or more items can be implicit when it is nested in a multiline list:
  ```
  multiline
  inline: inline
  multiline
  ```
* It can be explicit too:
  ```
  multiline
  {inline: inline}
  multiline
  ```
* All other cases of nested maps have to be explicit:
  ```
  multiline
  {}
  multiline

  multiline
  {
    multiline: multiline
  }
  multiline

  inline, {
    multiline: multiline
  }, inline
  
  inline, {inline: inline}, {inline2: inline2}, inline

  inline, {inline: inline, inline2: inline2}, inline
  
  inline{inline: inline}

  inline{
    multiline: multiline
  }

  {
    multiline{inline: inline}
    multiline2{
      multiline: multiline
    }
  }
  ```

### Table

```
# Table:
id,name,email,notes
:
1,Alice,a@example.com,curious
2,Bob,b@example.com,""

# List of maps:
id: 1, name: Alice, email: a@example.com, notes: curious
id: 2, name: Bob, email: b@example.com, notes: ""
```

* Table is a multiline list where:
  * first item is inline list of keys,
  * second item is `:` character,
  * zero or more following items are inline lists of values.
* Table is parsed as a list of maps having the same keys.
* If TTT formatter detects a list of N or more maps having the same keys, it produces TTT table.
  * N equals 2 by default and may be configured.
* Table can also be parsed/formatted directly to/from optimized structures like [pandas.DataFrame](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.html) using custom callback functions of parser/formatter, optional by default.
* Unlike CSV, the header presence is explicit in TTT table thanks to the second item being `:` character.
* Table cannot be confused with a multiline list because `:` character is invalid as a multiline list item due to other rules.
* See [Nested list](#nested-list) for TTT version of a CSV table without a header.
* Nested table is always explicit, just like a multiline list:
  ```
  id,parts
  :
  1,[
    width,height
    :
    20,30
    40,50
  ]
  ```

## DSL example

```
select[a, b, c],from[
  table
],where{and[
  eq[a, b]
  ne[b, c]
]}

# JSON:
{
  "select": ["a", "b", "c"],
  "from": ["table"],
  "where": {"and": [
    {"eq": ["a", "b"]},
    {"ne": ["b", "c"]}
  ]}
}

# YAML:
select:
  - a
  - b
  - c
from:
  - table
where:
  and:
    - eq:
      - a
      - b
    - ne:
      - b
      - c
```

## Roadmap

* Strict grammar file:
  * for syntax highlighters and linters,
  * to generate parsers and formatters for multiple languages.
* Generate and test a VS Code/Cursor extension.
* Generate and test reference parser and formatter for JS.
* Use it.
