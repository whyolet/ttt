# TTT

Let's try to merge the best parts of formats like CSV, JSON, YAML to one simple format - TTT (Text Tree/Table).

## Example

```
# comment

unquoted key or value

"quoted key or value
can include newlines,

empty lines,
self-escaped "",
[,]{:}(#) characters,
leading/trailing whitespace "

# list
val1, val2, [nested, list]

# multiline list
val1
val2
[
  nested
  list
]

# indentation is optional
val1
val2
[
nested
list
]

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
```

## Roadmap

* Add rules and other like in [txtt](https://github.com/whyolet/txtt#txtt).
