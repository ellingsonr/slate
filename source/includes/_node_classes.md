# Node Classes

## Common Attributes

These attributes are common to all node classes.

### nid 
```python
node.nid
> '7115e193-9924-4273-bd31-432dee6fe0ce'
```
Unique id. If none is provided, one will be generated using `uuid.uuid4`.


### comparator
```python
node.comparator
> '0d8f0e97b031024bb5fba9304f5d6108cc827700'
```
A hash of the subset of attributes that identify a node as unique. This allows two nodes with the same sub tree to equated regardless of position in the same (or different) expressions.


### parent
```python
node.parent
> '95f1f233-d3bd-4ff6-9fef-4aae84cbfa02'
```
Unique id of parent expression.


### ntype
```python
node.ntype
> 'CONSTANT'
```
`Enum` like string that's anolgous to `str(node.__class__)` Perhaps we should just use `ininstance()`. This seems to be cleaner?


### to_dict
```python
node.to_dict()
> {'id': '7115e193-9924-4273-bd31-432dee6fe0ce', 'parent': None, 'type': 'CONSTANT', 'value': 1, 'exponent': None}
```
Used for serialization. Returns a python dict of the node class attributes.


### from_dict
```python
nodes = {}
node_dict = {'id': '7115e193-9924-4273-bd31-432dee6fe0ce', 'parent': None, 'type': 'CONSTANT', 'value': 1, 'exponent': None}
Constant.from_dict(node_dict, nodes)
> <tree_handler.node_mod.node_classes.Constant object at 0x107781408>
```
Used for deserialization. Returns a node class for a given dict of attributes.


### generate_comparator
```python
> node.generate_comparator()

```
Generates a nodes comparator. For the set of attributes used in comparator generation for a given node class, see the section node clase subsection. 

<aside class="notice">
Comparators are dependant on their child nodes' comaprators. Therefore, if a node is mutated, it's comparator and parent comparator must be updated. This causes a recursive update that traverses up the expression tree, to the root node. All node attribute setter methods must invoke <code/>generate_comparator</code>
</aside>

<aside class="warning">this function should never need to be called from outside a node class.</aside>

## Shared Attributes

These attirbutes aren't common across all node classes, but are found in several classes:


### children
```python
node.children
> ['123', '456']
node.children = ['123', '456', '789']
node.children
> ['123', '456', '789']
```
Found in: [brackets](#brackets), [exponent](#exponent), [line](#line), [term](#term)

A list of child node ids. Can be mutated through setter. Accessor returns a clone of children list to prevent unintended mutation. The types of children any particalar node can hold depends on the node, eg. a term node's children should hold only [term factor](#term-factor) node ids. A line should only hold [term](#term) node ids.


### add_node_child
```python
node.children
> ['123', '456']
node.add_node_child('789', 1)
node.children
> ['123', '789', '456']
```
Found in: [brackets](#brackets), [exponent](#exponent), [line](#line), [term](#term)

Allows for a single node id to be added at an index. If index is `None` or not provided, the new child id will be added at the end of the children list. 


### remove_node_child
```python
node.children
> ['123', '456', '789']
node.remove_node_child('456')
node.children
> ['123', '789']
```
Found in: [brackets](#brackets), [equals](#equals), [exponent](#exponent), [fraction](#fraction), [line](#line), [term](#term)

Allows for a single node id to be removed.


### exponent
```python
node.exponent
> '123'
node.exponent = '456'
node.exponent
> '456'
node.exponent = None
node.exponent
> None
```
Found in: [brackets](#brackets), [constant](#constant), [parameter](#parameter)

A single node id of an [exponent](#exponent) node. `None` should be used to indicate the lack of an exponent.


### value
```python
constant.value
> 5
constant.value = 10
constant.exponent
> 10
parameter.value
> 'x'
parameter.value = 'y'
parameter.value
> 'y'
```
Found in: [constant](#constant), [parameter](#parameter)

A single value, numerical if node type is [constant](#constant), string if node type is [parameter](#parameter)






## Brackets

Brackets are similar to a line in that it is a collect of terms. The key differences between it and lines are viusal and that brackets are also [term factors](#term-factor)

### Characteristics
```python
root_node, nodes = generate_tree("(2*x)^3")
print_all_trees(nodes)
"
LINE TERM(+) BRACKETS EXPONENT TERM(+) CONSTANT(3)
                      TERM(+) CONSTANT(2)
                              PARAMETER(x)
"   
```
Characteristic | Description
-------------- | -----------
visual                | one or more [terms](#term) surrounded by `(` and `)` characters
parent types          | [root](#root), [equals](#equals), [fraction](#fraction), [term](#term)
child types           | [term](#term)
comparator attributes | [ntype](#ntype), [exponent](#exponent), [children](#children)

### Constructor
```python
nodes = {}
constant_1 = Constant(None, 3, None, nodes)
term_1 = Term(None, [constant_1.nid], "+", nodes)
exponent_1 = Exponent(None, [term_1.nid], nodes)
constant_2 = Constant(None, 2, None, nodes)
parameter = Paramter(None, 'x', None, nodes)
term_2 = Term(None, [constant_2.nid, parameter.nid], "+", nodes)
node = Brackets(None, [term_2.nid], exponent_1.nid, nodes)
node
> <tree_handler.node_mod.node_classes.Brackets object at 0x107887b28>
print_all_trees(nodes)
"
BRACKETS EXPONENT TERM(+) CONSTANT(3)
         TERM(+) CONSTANT(2)
                 PARAMETER(x)
"   
```
Parameter | Type | Default Value 
--------- | ---- | -------------
[parent](#parent)     | string          | `None`
[children](#children) | list of strings | `None`
[exponent](#exponent) | string          | `None`
[nodes](#nodes)       | dict            | required 
[_nid](#nid)          | string          | not required, should only be used if new node is replacing root, and id needs to be preserved. 






## Constant

Constants are nodes that contain numerical values. They can also have an [exponent](#exponent), like other [term factors](#term-factor)

### Characteristics
```python
root_node, nodes = generate_tree("3")
print_all_trees(nodes)
"
LINE TERM(+) CONSTANT(3)
"   
```
Characteristic | Description
-------------- | -----------
visual                | numerical value such as `42` or `3.142`
parent types          | [term](#term)
comparator attributes | [ntype](#ntype), [exponent](#exponent), [value](#value)

### Constructor
```python
nodes = {}
constant_1 = Constant(None, 3, None, nodes)
node
> <tree_handler.node_mod.node_classes.Constant object at 0x107887b28>
print_all_trees(nodes)
"
CONSTANT(3)
"   
```
Parameter | Type | Default Value 
--------- | ---- | -------------
[parent](#parent)     | string          | `None`
[value](#value)       | int or float    | required
[exponent](#exponent) | string          | `None`
[nodes](#nodes)       | dict            | required 
[_nid](#nid)          | string          | not required, should only be used if new node is replacing root, and id needs to be preserved. 





## Equals

An Equals node is one that splits an expression in two, forming an equation. An equals node is always the [root node](#root) of the expression tree

### Characteristics
```python
root_node, nodes = generate_tree("x=1")
print_all_trees(nodes)
"
EQUALS LINE TERM(+) PARAMETER(x)
       LINE TERM(+) CONSTANT(1)
"   
```
Characteristic | Description
-------------- | -----------
visual                | an `=` character with an expression on each side
parent types          | [root](#root)
child types           | [line](#line), [brackets](#brackets)
comparator attributes | [ntype](#ntype), [lhs](#lhs), [rhs](#rhs)

### Constructor
```python
nodes = {}
parameter = Paramter(None, 'x', None, nodes)
term_1 = Term(None, [parameter.nid], "+", nodes)
line_1 = Line(None, [term_1.nid], nodes)
constant = Constant(None, 1, None, nodes)
term_2 = Term(None, [constant.nid], "+", nodes)
line_2 = Line(None, [term_2.nid], nodes)
node = Equals(None, line_1.nid, line_2.nid, nodes)
node
> <tree_handler.node_mod.node_classes.Equals object at 0x107887b28>
print_all_trees(nodes)
"
EQUALS LINE TERM(+) PARAMETER(x)
       LINE TERM(+) CONSTANT(1)
"   
```
Parameter | Type | Default Value 
--------- | ---- | -------------
[parent](#parent)     | string          | `None`
[lhs](#lhs)           | string          | `None`
[rhs](#rhs)           | string          | `None`
[nodes](#nodes)       | dict            | required 
[_nid](#nid)          | string          | not required, should only be used if new node is replacing root, and id needs to be preserved. 

### lhs
```python
node.lhs
> '123'
node.lhs = '456'
node.lhs
> '456'
```
Lhs is a node id for a [line](#line) or [brackets](#brackets)

### rhs
```python
node.rhs
> '123'
node.rhs = '456'
node.rhs
> '456'
```
Rhs is a node id for a [line](#line) or [brackets](#brackets)





## Exponent

Exponents are similar to [line](#line) except their parents are always [term factors](#term-factor). 

### Characteristics
```python
root_node, nodes = generate_tree("x^3")
print_all_trees(nodes)
"
LINE TERM(+) PARAMETER(x) EXPONENT TERM(+) CONSTANT(3)
"   
```
Characteristic | Description
-------------- | -----------
visual                | one or more [terms](#term) raised above it's [base](#base)
parent types          | [brackets](#brackets), [constant](#constant), [parameter](#parameter)
child types           | [term](#term)
comparator attributes | [ntype](#ntype), [children](#children)

### Constructor
```python
nodes = {}
constant = Constant(None, 3, None, nodes)
term_1 = Term(None, [constant.nid], "+", nodes)
exponent = Exponent(None, [term_1.nid], nodes)
node = Paramter(None, 'x', exponent.nid, nodes)
> <tree_handler.node_mod.node_classes.Parameter object at 0x107887b28>
print_all_trees(nodes)
"
PARAMETER(x) EXPONENT TERM(+) CONSTANT(3)
"   
```
Parameter | Type | Default Value 
--------- | ---- | -------------
[parent](#parent)     | string          | `None`
[children](#children) | list of strings | `None`
[nodes](#nodes)       | dict            | required 
[_nid](#nid)          | string          | not required, should only be used if new node is replacing root, and id needs to be preserved. 




## Fraction

A Fraction node has a two children nodes, a [numerator](#numerator) and a [denominator](#denominator). While a fraction is technically a term factor, it can't have an [exponent](#exponent) and it is always the single child of a [term](#term)

### Characteristics
```python
root_node, nodes = generate_tree("1/x")
print_all_trees(nodes)
"
LINE TERM(+) FRACTION LINE TERM(+) CONSTANT(1)
                      LINE TERM(+) PARAMETER(x)
"   
```
Characteristic | Description
-------------- | -----------
visual                | a horizontal bar with an [line](#line) or [brackets](#brackets) above and a [line](#line) or [brackets](#brackets) below it.
parent types          | [term](#term)
child types           | [line](#line), [brackets](#brackets)
comparator attributes | [ntype](#ntype), [numerator](#numerator), [denominator](#denominator)

### Constructor
```python
nodes = {}
constant = Constant(None, 1, None, nodes)
term_1 = Term(None, [constant.nid], "+", nodes)
line_1 = Line(None, [term_1.nid], nodes)
parameter = Paramter(None, 'x', None, nodes)
term_2 = Term(None, [parameter.nid], "+", nodes)
line_2 = Line(None, [term_2.nid], nodes)
node = Fraction(None, line_1.nid, line_2.nid, nodes)
node
> <tree_handler.node_mod.node_classes.Equals object at 0x107887b28>
print_all_trees(nodes)
"
FRACTION LINE TERM(+) CONSTANT(1)
         LINE TERM(+) PARAMETER(x)
"   
```
Parameter | Type | Default Value 
--------- | ---- | -------------
[parent](#parent)           | string | `None`
[numerator](#numerator)     | string | `None`
[denominator](#denominator) | string | `None`
[nodes](#nodes)             | dict   | required 
[_nid](#nid)                | string | not required, should only be used if new node is replacing root, and id needs to be preserved. 

### numerator
```python
node.numerator
> '123'
node.numerator = '456'
node.numerator
> '456'
```
Numerator is a node id for a [line](#line) or [brackets](#brackets)

### denominator
```python
node.denominator
> '123'
node.denominator = '456'
node.denominator
> '456'
```
Denominator is a node id for a [line](#line) or [brackets](#brackets)




## Line

Line nodes are similar to a brackets in that it is a collect of terms. The key differences between it and brackets are viusal are that lines are not [term factors](#term-factor)

### Characteristics
```python
root_node, nodes = generate_tree("3 - x")
print_all_trees(nodes)
"
LINE TERM(+) CONSTANT(3)
     TERM(-) PARAMETER(x)
"   
```
Characteristic | Description
-------------- | -----------
visual                | one or more [terms](#term)
parent types          | [equals](#equals), [fraction](#fraction), [term](#term)
child types           | [term](#term)
comparator attributes | [ntype](#ntype), [children](#children)

### Constructor
```python
nodes = {}
constant= Constant(None, 3, None, nodes)
term_1 = Term(None, [constant.nid], "+", nodes)
parameter = Paramter(None, 'x', None, nodes)
term_2 = Term(None, [parameter.nid], "-", nodes)
node = Line(None, [term_1.nid, term_2.nid], None, nodes)
node
> <tree_handler.node_mod.node_classes.Line object at 0x107887b28>
print_all_trees(nodes)
"
LINE TERM(+) CONSTANT(3)
     TERM(-) PARAMETER(x)
"   
```
Parameter | Type | Default Value 
--------- | ---- | -------------
[parent](#parent)     | string          | `None`
[children](#children) | list of strings | `None`
[nodes](#nodes)       | dict            | required 
[_nid](#nid)          | string          | not required, should only be used if new node is replacing root, and id needs to be preserved. 





## Parameter

Parameters are nodes that contain string values that represent some variable. They can also have an [exponent](#exponent), like other [term factors](#term-factor)

### Characteristics
```python
root_node, nodes = generate_tree("x")
print_all_trees(nodes)
"
LINE TERM(+) PARAMETER(x)
"   
```
Characteristic | Description
-------------- | -----------
visual                | string value such as `x` or `y_1`
parent types          | [term](#term)
comparator attributes | [ntype](#ntype), [exponent](#exponent), [value](#value)

### Constructor
```python
nodes = {}
constant_1 = Parameter(None, 'x', None, nodes)
node
> <tree_handler.node_mod.node_classes.Parameter object at 0x107887b28>
print_all_trees(nodes)
"
PARAMETER(x)
"   
```
Parameter | Type | Default Value 
--------- | ---- | -------------
[parent](#parent)     | string          | `None`
[value](#value)       | string          | required
[exponent](#exponent) | string          | `None`
[nodes](#nodes)       | dict            | required 
[_nid](#nid)          | string          | not required, should only be used if new node is replacing root, and id needs to be preserved.





## Term

Term node hold have two key attributes, a list of [#children](#children) called the [term factors](#term-factor) and a [sign](#sign). If the [sign](#sign) is positive and the term is first child of it parent, then the sign will not be displayed.

### Characteristics
```python
root_node, nodes = generate_tree("2*x")
print_all_trees(nodes)
"
LINE TERM(+) CONSTANT(2)
             PARAMETER(x)
"   
```
Characteristic | Description
-------------- | -----------
visual                | [sign](#sign) followed by one or more [term factors](#term-factor) separated by `*` characters
parent types          | [brackets](#brackets), [exponent](#exponent), [line](#line)
child types           | [brackets](#brackets), [constant](#constant), [fraction](#fraction), [parameter](#parameter)
comparator attributes | [ntype](#ntype), [children](#children), [sign](#sign)

### Constructor
```python
nodes = {}
constant = Constant(None, 2, None, nodes)
parameter = Paramter(None, 'x', None, nodes)
node = Term(None, [constant.nid, parameter.nid], "+", nodes)
node
> <tree_handler.node_mod.node_classes.Term object at 0x107887b28>
print_all_trees(nodes)
"
TERM(+) CONSTANT(2)
        PARAMETER(x)
"   
```
Parameter | Type | Default Value 
--------- | ---- | -------------
[parent](#parent)     | string          | `None`
[children](#children) | list of strings | `None`
[sign](#sign)         | string          | required
[nodes](#nodes)       | dict            | required 
[_nid](#nid)          | string          | not required, should only be used if new node is replacing root, and id needs to be preserved. 

### sign
```python
node.sign
> '+'
node.sign = '-'
node.sign
> '-'
```
The sign of a term is a string `+` or `-`. It can be mutated via the mutator.