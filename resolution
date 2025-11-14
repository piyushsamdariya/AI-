import re
from itertools import combinations

def is_variable(x):
    return isinstance(x, str) and x[0].islower()

def parse_literal(literal):
    neg = literal.startswith("~")
    if neg:
        literal = literal[1:]
    pred, args = literal.split("(")
    args = tuple(a.strip() for a in args[:-1].split(","))
    return (pred.strip(), args, neg)

def negate_literal(literal):
    pred, args, neg = literal
    return (pred, args, not neg)

def unify(x, y, subst=None):
    if subst is None:
        subst = {}
    if x == y:
        return subst
    if is_variable(x):
        return unify_var(x, y, subst)
    if is_variable(y):
        return unify_var(y, x, subst)
    if isinstance(x, tuple) and isinstance(y, tuple):
        if len(x) != len(y):
            return None
        for a, b in zip(x, y):
            subst = unify(a, b, subst)
            if subst is None:
                return None
        return subst
    return None

def unify_var(var, x, subst):
    if var in subst:
        return unify(subst[var], x, subst)
    elif x in subst:
        return unify(var, subst[x], subst)
    elif occurs_check(var, x, subst):
        return None
    subst[var] = x
    return subst

def occurs_check(var, x, subst):
    if var == x:
        return True
    if isinstance(x, tuple):
        return any(occurs_check(var, xi, subst) for xi in x)
    if x in subst:
        return occurs_check(var, subst[x], subst)
    return False

def substitute(literal, subst):
    pred, args, neg = literal
    new_args = []
    for a in args:
        new_args.append(subst.get(a, a))
    return (pred, tuple(new_args), neg)

def apply_substitution(clause, subst):
    return [substitute(l, subst) for l in clause]

def resolve(ci, cj):
    resolvents = []
    for li in ci:
        for lj in cj:
            if li[0] == lj[0] and li[2] != lj[2]:  # same predicate, opposite signs
                subst = unify(li[1], lj[1])
                if subst is not None:
                    new_clause = [l for l in ci if l != li] + [l for l in cj if l != lj]
                    new_clause = apply_substitution(new_clause, subst)
                    new_clause = list(set(new_clause))
                    if not new_clause:
                        return [[]]
                    resolvents.append(new_clause)
    return resolvents

def resolution(KB, query):
    neg_query = [negate_literal(l) for l in query]
    clauses = KB + [neg_query]
    new = set()

    while True:
        pairs = list(combinations(clauses, 2))
        for (ci, cj) in pairs:
            resolvents = resolve(ci, cj)
            if [] in resolvents:
                print("Derived empty clause (contradiction). ")
                return True
            for res in resolvents:
                res_tuple = tuple(res)
                if res_tuple not in new:
                    new.add(res_tuple)
        if all(r in clauses for r in new):
            print("No new clauses found. ")
            return False
        for r in new:
            if list(r) not in clauses:
                clauses.append(list(r))

def parse_clause(s):
    literals = [parse_literal(lit.strip()) for lit in s.split(" v ")]
    return literals

def main():
    print("=== Resolution in First-Order Logic ===\n")
    n = int(input("Enter number of clauses in KB: "))
    KB = []
    for i in range(n):
        clause_str = input(f"Clause {i+1}: ")
        KB.append(parse_clause(clause_str))
    query_str = input("\nEnter query: ")
    query = parse_clause(query_str)
    print("\n--- RESOLUTION STEPS ---")
    result = resolution(KB, query)
    print("\n--- RESULT ---")
    if result:
        print("The query CAN be inferred from the KB ")
    else:
        print("The query CANNOT be inferred from the KB ")

if __name__ == "__main__":
    main()
