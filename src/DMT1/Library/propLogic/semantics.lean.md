```lean
import DMT1.Library.propLogic.syntax
import DMT1.Library.propLogic.domain

namespace DMT1.propLogic
```

# Semantics

```lean
def evalUnOp : UnOp → (Bool → Bool)
| (UnOp.not) => not

def evalBinOp : BinOp → (Bool → Bool → Bool)
| BinOp.and => and
| BinOp.or => or
| BinOp.imp => imp
| BinOp.iff => iff

open PLExpr

abbrev BoolInterp := BoolVar → Bool -- varInterp would be better name

def evalPLExpr : PLExpr → BoolInterp → Bool
| lit_expr b,             _ => b
| (var_expr v),           i => i v
| (un_op_expr op e),      i => (evalUnOp op) (evalPLExpr e i)
| (bin_op_expr op e1 e2), i => (evalBinOp op) (evalPLExpr e1 i) (evalPLExpr e2 i)


end DMT1.propLogic
```