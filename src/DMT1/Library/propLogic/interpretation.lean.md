```lean
import DMT1.Library.propLogic.utilities
import DMT1.Library.propLogic.semantics

namespace DMT1.Library.propLogic.interpretation

open propLogic.syntax
open utilities
open semantics
```


#### Boolean Interpretation


```lean
-- From interpretation, variable, and new Bool, override that interpretation to assign that new value to that variable
private def overrideVarValInInterp : Interp → Var → Bool → Interp
| i, v, b =>
  λ (v' : Var) =>
    if (v'.index == v.index)  -- if index is variables overridden
        then b                -- return new value
        else (i v')           -- else value under old interp
```


Helper for mk_row_interp function that follows
Convert a given list of Bool to an Interp function
```lean
def interpFromBools : List Bool → Interp
  | l => bools_to_interp_helper l.length l
where bools_to_interp_helper : (vars : Nat) → (vals : List Bool) → Interp
  | _, [] => (λ _ => false)
  | vars, h::t =>
    let len := (h::t).length
    overrideVarValInInterp
      (bools_to_interp_helper vars t)
      (Var.mk (vars - len)) h

def InterpFromRowCols : (row: Nat) → (cols: Nat) → Interp
| index, cols =>
  interpFromBools
    (list_bool_from_row_index_and_cols index cols)
```

Make an interpretation *function* for row index "row"
a truth table with "vars" variables/columns.
```lean
-- DEPENDENCY: mk_row_bools is from utilities
def interpFromRowCols : (row: Nat) → (cols: Nat) → Interp
| index, cols =>
  interpFromBools
    (list_bool_from_row_index_and_cols index cols)
```

Given the number, n, of variables, return a list of its 2^n interpretations.
Watch out, as the size of the constsructed lists grows very quickly.
```lean
def interpsFromNumVars (vars : Nat) : List Interp :=
  mk_interps_helper (2^vars) vars
where mk_interps_helper : (rows : Nat) → (numvars : Nat) → List Interp
  | 0,        _  => []
  | (n' + 1), v  => (InterpFromRowCols n' v)::(mk_interps_helper n' v)
```

Given an expression, e, return the number of distinct variable expressions
it contains.
```lean
def numVarsFromExpr : Expr → Nat := (fun e => max_variable_index e + 1) where
max_variable_index : Expr → Nat
| Expr.lit_expr _ => 0
| Expr.var_expr (Var.mk i) => i
| Expr.un_op_expr _ e => max_variable_index e
| Expr.bin_op_expr _ e1 e2 => max (max_variable_index e1) (max_variable_index e2)
```


Given a Expr, e, reduces to a list of its 2^n interpretations,
where n is the number of semantically distinct variable expressions
in e.
```lean
def listInterpsFromExpr : Expr → List Interp
| e => interpsFromNumVars (numVarsFromExpr e)
```

Given interp, i, and window parameter, w, return list of "0"/"1"
strings, in order of variable indices, from zero to the number, w,
reflecting the values "assigned/mapped" to these variables by i.
```lean
def bitStringsFromInterp : (i : Interp) → (w : Nat) → List String
| _, 0 => []
| i, (w' + 1) =>
  let b := (if (i ⟨w'⟩ ) then "1" else "0")
  bitStringsFromInterp i w' ++ [b]  -- ++ here is binary List.append
```

Given a list of Bool interps and a width w, output them as a list of list of Bool
```lean
def interpStringsFromInterps : List Interp → Nat → List (List String)
| [], _ => []
| h::t, n => bitStringsFromInterp h n::interpStringsFromInterps t n

end DMT1.Library.propLogic.interpretation
```
