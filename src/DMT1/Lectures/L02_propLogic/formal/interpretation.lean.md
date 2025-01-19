```lean
import DMT1.Lectures.L02_propLogic.formal.utilities
import DMT1.Lectures.L02_propLogic.formal.semantics

/-!
#### Boolean Interpretation
-/

namespace DMT1.lecture.propLogic.semantics
open propLogic.syntax

-- From interpretation, variable, and new Bool, override that interpretation to assign that new value to that variable
def overrideVarValInInterp : Interp → Var → Bool → Interp
| i, v, b =>
  λ (v' : Var) =>
    if (v'.index == v.index)  -- if index is variables overridden
        then b                -- return new value
        else (i v')           -- else value under old interp


/-!
Helper for mk_row_interp function that follows
Convert a given list of Bool to an Interp function
-/
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

/-!
Make an interpretation *function* for row index "row"
a truth table with "vars" variables/columns.
-/
-- DEPENDENCY: mk_row_bools is from utilities
def interpFromRowCols : (row: Nat) → (cols: Nat) → Interp
| index, cols =>
  interpFromBools
    (list_bool_from_row_index_and_cols index cols)

/-!
Given the number, n, of variables, return a list of its 2^n interpretations.
Watch out, as the size of the constsructed lists grows very quickly.
-/
def interpsFromNumVars (vars : Nat) : List Interp :=
  mk_interps_helper (2^vars) vars
where mk_interps_helper : (rows : Nat) → (numvars : Nat) → List Interp
  | 0,        _  => []
  | (n' + 1), v  => (InterpFromRowCols n' v)::(mk_interps_helper n' v)

/-!
Given an expression, e, return the number of distinct variable expressions
it contains. That in turn is one more than the highest variable index value
starting from 0. The answer for P ∧ P would thus be 1 (not 2), for example.
For ⊤ it'd be 0: there are no variable expressions in the structure of ⊤.

This function definition provides a beautiful example of case analysis: here
on the structure of a propositional logic expression argument. If it's literal,
the result is computed one way; a variable expression, another way, applying an
interpretation function to compute the result; or compound operator expressions,
unary or binary. The meaning of a compound expression is computed in turn by
applying the appropriate combing/transforming operation to the *meanings of
its subexpressions, computed by recursive evaluation, now of a subexpression,
under the same fixed interpretation.
-/
def numVarsFromExpr : Expr → Nat := (fun e => max_variable_index e + 1) where
max_variable_index : Expr → Nat
| Expr.lit_expr _ => 0
| Expr.var_expr (Var.mk i) => i
| Expr.un_op_expr _ e => max_variable_index e
| Expr.bin_op_expr _ e1 e2 => max (max_variable_index e1) (max_variable_index e2)


/-!
Given a Expr, e, reduces to a list of its 2^n interpretations,
where n is the number of semantically distinct variable expressions
in e.
-/
def listInterpsFromExpr : Expr → List Interp
| e => interpsFromNumVars (numVarsFromExpr e)

/-!
Given interp, i, and window parameter, w, return list of "0"/"1"
strings, in order of variable indices, from zero to the number, w,
reflecting the values "assigned/mapped" to these variables by i.
-/
def bitStringsFromInterp : (i : Interp) → (w : Nat) → List String
| _, 0 => []
| i, (w' + 1) =>
  let b := (if (i ⟨w'⟩ ) then "1" else "0")
  bitStringsFromInterp i w' ++ [b]  -- ++ here is binary List.append

/-!
Given a list of Bool interps and a width w, output them as a list of list of Bool
-/
def interpStringsFromInterps : List Interp → Nat → List (List String)
| [], _ => []
| h::t, n => bitStringsFromInterp h n::interpStringsFromInterps t n

end DMT1.lecture.propLogic.semantics
```