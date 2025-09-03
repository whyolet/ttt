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
* Initial state is an implicit newline-separated list.
  * This allows to support zero, one, or multiple root values in one file naturally without additional concepts like "document" and separators like `---` in YAML.
  * Empty file parses to an empty list:
    ```
    # TTT file:

    # JSON file:
    []
    ```

### List

* List is either comma-separated or newline-separated.

#### Comma-separated list

```
foo, bar baz, [nested, list here]

# JSON:
["foo", "bar baz", ["nested", "list here"]]
```

* Comma-separated list (CSL) is either explicit or implicit.
* Explicit CSL is:
  * `[` character,
  * implicit CSL,
  * `]` character.
* Implicit CSL is zero or more items separated by a comma.
* CSL item is:
  * zero or more spaces,
  * a value,
  * zero or more spaces.

```
a,b, c, d , ,,  g

# JSON:
["a","b","c","d","","","g"]
```

#### Newline-separated list

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

* Newline-separated list (NSL) is either explicit or implicit.
* Explicit NSL is:
  * `[` character,
  * a newline,
  * implicit NSL,
  * a newline,
  * `]` character.
* Implicit NSL is zero or more items separated by a newline.
* NSL item is:
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

* Comma-separated list can be implicit when it is nested in a newline-separated list, as in CSV:
  ```
  a,aa,aaa
  b, bb, bbb

  # JSON:
  [
    ["a", "aa", "aaa"],
    ["b", "bb", "bbb"]
  ]
  ```
* All other combinations require the nested list to be explicit:
  ```
  comma, [comma, comma], comma

  comma, [
    newline
    newline
  ], comma

  newline
  [
    newline
    newline
  ]
  newline
  ```
  vs implicit:
  ```
  newline
  comma, comma
  newline
  ```
* Empty nested list have to be explicit:
  ```
  # TTT file is a parent list,
  # with this nested list:
  []

  # JSON file:
  [[]]
  ```

## Old example

TODO: Create rules by moving and improving ideas from the example below.

```
# comment

unquoted key or value

"quoted key or value
can include newlines,

empty lines,
self-escaped "",
[,]{:}(#) characters,
leading/trailing whitespace "

# map
key1: val1, key2: [nested, list], key3: {nested: map}

# multiline map
{
  key1: val1
  key2: [nested, list]
  key3: {nested: map}
}

# whitespace around is optional
{
key1:val1
key2:[nested,list]
key3:{nested:map}
}

# {} distinguish map from
# list of maps
map1:here
map2:there,more:items

# list of lists
v11,v12,v13
v21,v22,v23

# table
k1,k2,k3
:
v11,v12,v13
v21,v22,v23

# is equal to list of maps
k1: v11, k2: v12, k3: v13
k1: v21, k2: v22, k3: v23

# nested tables
id,data
:
10,[
  k1,k2
  :
  v11,v12
  v21,v22
]
20,[]

# indented key or value
level1: {
  level2: {
    level3: (
      indented key or value
      can include newlines,

      empty lines,
      [,]{:}(#)" characters,
      leading/trailing whitespace 
    )
  }
}

# optional : before [{( in map
# and optional , in list and map

select[a,b,c],from[t],where{and[
  eq[a,b]
  ne[b,c]
]}

# equivalent JSON
{
  "select": ["a", "b", "c"],
  "from": ["t"],
  "where": {"and": [
    {"eq": ["a", "b"]},
    {"ne": ["b", "c"]}
  ]}
}

# equivalent YAML
select:
  - a
  - b
  - c
from:
  - t
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

* Apply TODOs above.
* Add valuable spec parts from [txtt](https://github.com/whyolet/txtt#txtt).
