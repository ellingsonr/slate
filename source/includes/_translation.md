# Translation

Translation and substitution allows Algebruh to leverage SymPy's symbolic solver power. To do so we need to translate either the entire Algebruh AST into tree a SymPy AST (translation), or translate the specific sub-tree/s of the Algrebruh AST into a SymPy AST. Then we perform some set of SymPy operation on the tranlated tree, and finally convert it back in a Algebruh AST.

## Why go through the effort?

If SymPy as an AST structure and all the maths operations we could ever want to perform on those ASTs, why evern bother with Algebruh ASTs? Why not keep the trees in SymPy? There are two issues:
1. Expressiveness
2. Eager Evaluation

### Expressiveness

Allowing a user to express themselves as freely as possible on of the goals of Algebruh. SymPy is not concerned with that. The structure of it's ASTs is concerned with solving expressions. For instance, SymPy has a `Mul` node that expresses the multiplication of two [term factors](#term-factor). But it doesn't have a `Div` node. If you want to express `y/x`, you need to use negative exponents:  
`Mul(y, Pow(x, -1))`

One could argue 