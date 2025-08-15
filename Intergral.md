#Mathematical Integral Solver 🧮

<a name="objectifs"></a> objectifs

> A Python program that focuses on solving integrals (both definite and indefinite) using sympy, while still supporting derivatives, limits, simplification, and equations.




---

Installation

pip install sympy numpy

or

!pip install sympy numpy

> [!IMPORTANT]
Use Command Prompt to install resources.
Make sure numpy is installed.




---

<h3 align="center">Math Integral Solver Code</h3>  
---

version 3 — Integration-Focused 🧡

# math_integral_solver_v3.py
# by : tknohamza

import re
import sympy as sp
from sympy import Eq
from sympy.parsing.sympy_parser import parse_expr

# رموز شائعة جاهزة للاستخدام
x, y, z, t = sp.symbols('x y z t', real=True)
allowed_symbols = {
    'x': x, 'y': y, 'z': z, 't': t,
    'sin': sp.sin, 'cos': sp.cos, 'tan': sp.tan, 'sec': sp.sec, 'csc': sp.csc, 'cot': sp.cot,
    'asin': sp.asin, 'acos': sp.acos, 'atan': sp.atan, 'log': sp.log, 'ln': sp.log, 'exp': sp.exp,
    'sqrt': sp.sqrt, 'E': sp.E, 'pi': sp.pi, 'oo': sp.oo,
    'diff': sp.diff, 'integrate': sp.integrate, 'limit': sp.limit, 'Abs': sp.Abs
}

def parse_expr_safe(s: str):
    return parse_expr(s, local_dict=allowed_symbols, global_dict=allowed_symbols, evaluate=False)

def try_sympify(s: str):
    return sp.sympify(s, locals=allowed_symbols)

def pretty_out(title, obj):
    print(f"\n[{title}]")
    sp.pprint(obj, use_unicode=True)
    try:
        print("\nLaTeX:", sp.latex(obj))
    except Exception:
        pass

def handle_integral_from_compact(user_input: str):
    pat_def1 = r'^\s*int\[(?P<a>.+?),(?P<b>.+?)\]\s*(?P<fx>.+?)\s*d(?P<var>[a-zA-Z])\s*$'
    pat_ind1 = r'^\s*int\[(?P<fx>.+?),(?P<var>[a-zA-Z])\]\s*$'
    pat_def2 = r'^\s*∫\s*_(?P<a>.+?)\s*\^\s*(?P<b>.+?)\s*(?P<fx>.+?)\s*d(?P<var>[a-zA-Z])\s*$'
    pat_ind2 = r'^\s*∫\s*(?P<fx>.+?)\s*d(?P<var>[a-zA-Z])\s*$'

    for pat in (pat_def1, pat_def2):
        m = re.match(pat, user_input)
        if m:
            a = try_sympify(m.group('a'))
            b = try_sympify(m.group('b'))
            var = sp.Symbol(m.group('var'), real=True)
            fx = parse_expr_safe(m.group('fx'))
            res = sp.integrate(fx, (var, a, b))
            return res, var, fx, (a, b)

    for pat in (pat_ind1, pat_ind2):
        m = re.match(pat, user_input)
        if m:
            var = sp.Symbol(m.group('var'), real=True)
            fx = parse_expr_safe(m.group('fx'))
            res = sp.integrate(fx, var)
            return res, var, fx, None

    return None

def solve_line(user_input: str):
    user_input = user_input.strip()
    compact = handle_integral_from_compact(user_input)
    if compact is not None:
        res, var, fx, bounds = compact
        pretty_out("Input f(x)", fx)
        if bounds is None:
            pretty_out("Indefinite Integral", res)
        else:
            a, b = bounds
            pretty_out(f"Definite Integral over {var} from {a} to {b}", res)
            try:
                num = sp.N(res)
                print("Numeric (evalf):", num)
            except Exception:
                pass
        return

    try:
        if '=' in user_input and '==' not in user_input:
            lhs, rhs = user_input.split('=')
            lhs_expr = parse_expr_safe(lhs)
            rhs_expr = parse_expr_safe(rhs)
            eq = Eq(lhs_expr, rhs_expr)
            vars_in = list(lhs_expr.free_symbols.union(rhs_expr.free_symbols))
            sol = sp.solve(eq, *vars_in)
            pretty_out("Equation", eq)
            pretty_out("Solution", sol)
            return

        expr = parse_expr_safe(user_input)

        if expr.has(sp.Integral) or user_input.startswith("integrate"):
            val = sp.simplify(sp.integrate(*expr.args) if expr.func is sp.Integral else sp.simplify(expr))
            pretty_out("Integral Result", val)
            if isinstance(val, (sp.Float, sp.Integer)) or (not val.free_symbols):
                print("Numeric (evalf):", sp.N(val))
            return

        if expr.has(sp.Derivative) or user_input.startswith("diff"):
            val = sp.simplify(sp.diff(*expr.args) if expr.func is sp.Derivative else sp.simplify(expr))
            pretty_out("Derivative Result", val)
            return

        if user_input.startswith("limit"):
            val = sp.simplify(expr)
            pretty_out("Limit", val)
            return

        simp = sp.simplify(expr)
        pretty_out("Simplified", simp)

    except Exception as e:
        print("Error: Invalid input. Please ensure the expression is correct.")
        print("Details:", e)

def main():
    print("=== Mathematical Integral Solver (v3) ===")
    print("Examples:")
    print("  int[0, pi] sin(x) dx")
    print("  int[x**2 * exp(x), x]")
    print("  ∫_0^1 x**2 dx")
    print("  ∫ sin(x)/x dx")
    print("  integrate(sin(x)*x, (x, 0, pi))")
    print("  diff(x**3 + 2*x, x)")
    print("  limit(sin(x)/x, x, 0)")
    print("  x**2 - 5*x + 6 = 0")
    print("-----------------------------------------")

    while True:
        user_input = input("Your input (or 'exit'): ").strip()
        if user_input.lower() in {"exit", "quit", "q"}:
            print("Goodbye!")
            break
        solve_line(user_input)

if __name__ == "__main__":
    main()


---

Supported Operations

Input	Meaning

int[0, pi] sin(x) dx	Definite integral
int[x*log(x), x]	Indefinite integral
∫_0^1 x**2 dx	Symbolic ∫ notation
integrate(exp(-x**2), (x, -oo, oo))	Gaussian integral
diff(x**3 + 2*x, x)	Derivative
limit(sin(x)/x, x, 0)	Limit
x**2 - 5*x + 6 = 0	Equation solving



---

Example Output

=== Mathematical Integral Solver (v3) ===
Examples:
  int[0, pi] sin(x) dx
  int[x**2 * exp(x), x]
  ∫_0^1 x**2 dx
  integrate(sin(x)*x, (x, 0, pi))
-----------------------------------------
Your input (or 'exit'): int[0, pi] sin(x) dx

[Input f(x)]
sin(x)

[Definite Integral over x from 0 to pi]
2

Numeric (evalf): 2.00000000000000


---

<h3 align="center">Copyright © 2025</h3>  > Please feel free to contact us if you have any comments or questions :
Email: tknohamzacontact@gmail.com
Socials: Facebook, Instagram, Twitter, Telegram