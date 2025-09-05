# TTT

TTT (Text Tree/Table) is a simple format for nested lists, maps, and tables of text data, merging the best ideas from CSV, JSON, YAML, and others.

## Example

TODO: create a concise example showcasing main features from the rules below.

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

* TTT format supports Unicode as is, without escape sequences like `\uFFFF`, default encoding is UTF-8.
* TTT file is parsed line by line with a simple strict state machine.
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
* Unquoted text is returned as a literal string value, never a number, a boolean, or anything else.

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
* Quoted text is returned as a literal string value after removing the opening and closing `"` and replacing each `""` with a single `"`.
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
    "level2: {
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
* Indented text is returned as a literal string value after deletion of:
  * 2 opening and 2 closing lines,
  * exactly N+2 first spaces from each non-empty line, even if some lines start with more than N+2 spaces.

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
* Inline list item is:
  * zero or more spaces,
  * a value,
  * zero or more spaces.

```
a,b, c, d , ,,  g

# JSON:
["a","b","c","d","","","g"]
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
  text(), list[], map{}

  # JSON:
  {
    "text": "",
    "list": [],
    "map": {}
  }
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
  
  inline, {inline: inline}, inline
  
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
id,name,description
:
1,Alice,mad girl
2,Bob,friend of Alice

# List of maps:
id: 1, name: Alice, description: mad girl
id: 2, name: Bob, description: friend of Alice
```

* Table is a multiline list where:
  * first item is inline list of keys,
  * second item is `:` character,
  * zero or more following items are inline lists of values.
* Table is returned as a native table type if it is supported by the target language.
* Otherwise it is returned as a list of maps, zipping the same keys to each list of values as shown in the example above.
* Unlike CSV, the header presence is explicit in TTT table thanks to the second item being `:` character.
* This `:` character is not allowed to be the only character in the line by other rules.
* See [Nested list](#nested-list) for TTT version of a CSV table without a header.
* Nested table is always explicit, just like a multiline list it is:
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

* Apply TODO in Example.
* Add valuable spec parts from [txtt](https://github.com/whyolet/txtt#txtt).
