# Translation and Substitution

Translation and substitution allows Algebruh to leverage SymPy's symbolic solver power. To do so we need to translate either the entire Algebruh AST into tree a SymPy AST (translation), or translate the specific sub-tree/s of the Algrebruh AST into a SymPy AST. Then we perform some set of SymPy operation on the tranlated tree, and finally convert it back in a Algebruh AST.

## Why go through the effort?

If SymPy as an AST structure and all the maths operations we could ever want to perform on those ASTs, why evern bother with Algebruh ASTs? Why not keep the trees in SymPy? There are two issues:
1. Expressiveness
2. Eager Evaluation

Being familiar with SymPys AST or 'expression' structure is vital to understanding Algebruh's Translation and Substitution. [SymPy Expression Manipullation](https://docs.sympy.org/latest/tutorial/manipulation.html)

### Expressiveness

Allowing a user to express themselves as freely as possible on of the goals of Algebruh. SymPy is not concerned with that. The structure of it's ASTs is concerned with solving expressions. For instance, SymPy has a `Mul` node that expresses the multiplication of two [term factors](#term-factor). But it doesn't have a `Div` node. If you want to express `y/x`, you need to use negative exponents:  
`Mul(y, Pow(x, -1))`

Affording a high degree of expression would result in the divorce of the AST structure from the expression that the user sees. This means that pattern matching now needs to account for the shape the AST and then 'shape' of the expression that the user sees - substantially increasing complexity. 

Algebruh's AST mirror the user expression exactly. If a rewrite doesn't map to the desired user expression, then a set of [secondary rewrites](#secondary-rewrite) can be applied to reform the expression into the desired shape.

### Eager Evaluation

SymPy eagerly evaluates it's ASTs. This means that simply inspecting the AST can cause evulation such simplification without an explicit prompt to do so. This reduces expresssiveness as certain expressions are always modified by SymPy. This makes accurate sub tree rewrites very difficult as the entire tree is evaulated. SymPy does allow tree nodes to instantiated with `evaluate=False`, but this is cumbersome.

## Translation

Translation converts an entire sub tree into a SymPy expression. This means that the entire sub tree will be evaluated. At the moment, translation isn't as widely used as substitution, so the former is not as feature rich as the latter.

### container_to_sym
```python
container_id, nodes = tokenizer.generate_tree("-32 * a * b * c + 54 * x * y * z")
result = container_to_sym(container_id, nodes)
result
> -32*a*b*c + 54*x*y*z
```
Converts an Algebruh [container](#container) into a SymPy expression. If the container has multiple [terms](#term), the root node of the SymPy expression will be an Add node. The first parameter is the ID of the container.

Parameter | Type | Default Value 
--------- | ---- | -------------
[node_id](#nid)        | string          | required
[nodes](#nodes)        | dict            | required 



### term_to_sym
```python
container_id, nodes = tokenizer.generate_tree("-32 * a * b * c")
term_id = nodes[entry_node_id].children[0]
result = term_to_sym(term_id, nodes)
result
> -32*a*b*c
```
Converts an Algebruh [term](#term) into a SymPy expression. If the term has multiple term factors, the root node of the SymPy expression will be an Mul node. The first parameter is the ID of the term.

Parameter | Type | Default Value 
--------- | ---- | -------------
[node_id](#nid)        | string          | required
[nodes](#nodes)        | dict            | required 

### term_factor_to_sym
```python
nodes = {}
constant = Constant(None, 123, None, nodes)
result = term_factor_to_sym(constant.nid, nodes)
result
> 123
```
Converts an Algebruh [term factor](#term-factor) into a SymPy expression. The first parameter is the ID of the term.

Parameter | Type | Default Value 
--------- | ---- | -------------
[node_id](#nid)        | string          | required
[nodes](#nodes)        | dict            | required 


### sym_to_container
```python
container_id, nodes = tokenizer.generate_tree("-32 * a * b * c + 54 * x * y * z")
sym = container_to_sym(container_id, nodes)
sym
> -32*a*b*c + 54*x*y*z

sym_to_container(sym, False, nodes)
__output_tree__(nodes)
"""
LINE TERM(-) CONSTANT(32)
             PARAMETER(a)
             PARAMETER(b)
             PARAMETER(c)
     TERM(+) CONSTANT(54)
             PARAMETER(x)
             PARAMETER(y)
             PARAMETER(z)
LINE TERM(-) CONSTANT(32)
             PARAMETER(a)
             PARAMETER(b)
             PARAMETER(c)
     TERM(+) CONSTANT(54)
             PARAMETER(x)
             PARAMETER(y)
             PARAMETER(z)
"""
```

Converts a SymPy expression into an Algebruh [container](#container). The first parameter is a SymPy Add node.

Parameter | Type | Default Value 
--------- | ---- | -------------
sym_expression         | SypPy Expression | required
use_brackets           | boolean          | required
[nodes](#nodes)        | dict             | required 



### sym_to_term
```python
container_id, nodes = tokenizer.generate_tree("-32 * a * b * c")
term_id = nodes[entry_node_id].children[0]
sym = term_to_sym(term_id, nodes)
sym
> -32*a*b*c

sym_to_container(sym, False, nodes)
__output_tree__(nodes)
"""
LINE TERM(-) CONSTANT(32)
             PARAMETER(a)
             PARAMETER(b)
             PARAMETER(c)
TERM(-) CONSTANT(32)
        PARAMETER(a)
        PARAMETER(b)
        PARAMETER(c)
"""
```
Converts a SymPy expression into an Algebruh [term](#term). The first parameter is a SymPy Mul node.

Parameter | Type | Default Value 
--------- | ---- | -------------
sym_expression         | SypPy Expression | required
[nodes](#nodes)        | dict             | required

### sym_to_term_factor
```python
nodes = {}
constant = Constant(None, 123, None, nodes)
sym = term_factor_to_sym(constant.nid, nodes)
sym
> 123

sym_to_container(sym, False, nodes)
__output_tree__(nodes)
"""
CONSTANT(123)
CONSTANT(123)
"""
```
Converts a SymPy expression into an Algebruh [term factor](#term-factor).

Parameter | Type | Default Value 
--------- | ---- | -------------
sym_expression         | SypPy Expression | required
[nodes](#nodes)        | dict             | required