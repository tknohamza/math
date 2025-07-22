# Mathematical Equation Solver

### <a name="objectifs"></a> objectifs
> the Python program that can solve a wide range of mathematical expressions and equations using `sympy`.


first download this library :

```shell
pip install sympy
```
or
```shell
!pip install sympy
```

> [!IMPORTANT]
> use `command prompt` to install resources.
> Make sure `numpy` is installed.

</p>
<h3 align="center">Math Solver Code</h3>
<p align="center">
</p>

> [!TIP]
> It is mandatory to use VS Code to save and run the project.

version 1 :

```shell
import sympy as sp

def main():
    print("Mathematical Equation Solver")
    print("Enter a mathematical expression or equation (e.g., x**2 + 2*x - 3 = 0)")
    user_input = input("Your input: ")

    try:
        if '=' in user_input:
            lhs, rhs = user_input.split('=')
            equation = sp.Eq(sp.sympify(lhs), sp.sympify(rhs))
            solution = sp.solve(equation)
        else:
            expr = sp.sympify(user_input)
            solution = sp.simplify(expr)

        print("Result:", solution)

    except Exception as e:
        print("Error: Invalid input. Please ensure the expression is correct.")
        print("Details:", e)

if __name__ == "__main__":
    main()

# by : tknohamza
```

version 2 :

```shell
import sympy as sp

def main():
    print("Mathematical Equation Solver")
    print("Enter a mathematical expression or equation (e.g., x**2 + 2*x - 3 = 0)")
    user_input = input("Your input: ")

    try:
        # Check if it's an equation
        if '=' in user_input:
            lhs, rhs = user_input.split('=')
            lhs_expr = sp.sympify(lhs)
            rhs_expr = sp.sympify(rhs)
            equation = sp.Eq(lhs_expr, rhs_expr)

            # Detect variables in the expression
            variables = list(lhs_expr.free_symbols.union(rhs_expr.free_symbols))
            solution = sp.solve(equation, *variables)
        else:
            # Simplify an expression
            expr = sp.sympify(user_input)
            solution = sp.simplify(expr)

        print("Result:", solution)

    except Exception as e:
        print("Error: Invalid input. Please ensure the expression is correct.")
        print("Details:", e)

if __name__ == "__main__":
    main()

# by : tknohamza
```

> [!NOTE]
> Supported Operations :
You can input:
- Algebraic equations like `x**2 - 4 = 0`
- Expressions for simplification: `2*(x + 1) - x`
- Derivatives: `diff(x**2 + x, x)`
- Integrals: `integrate(sin(x)*x, x)`
- Limits: `limit(sin(x)/x, x, 0)`
- Solving linear systems (with a small extension)

```
# exemple :

This means the equation x**2 - 5*x + 6 = 0 has two solutions:
x = 2
x = 3
```

You can now type any of these examples:

| Input                      | Meaning                     |
| -------------------------- | --------------------------- |
| `x**2 - 5*x + 6 = 0`       | Solves a quadratic equation |
| `2*(x + 1) - x`            | Simplifies the expression   |
| `diff(x**3 + 2*x, x)`      | Derivative                  |
| `integrate(sin(x) * x, x)` | Indefinite integral         |
| `limit(sin(x)/x, x, 0)`    | Computes the limit at x=0   |


> [!WARNING]
> It is possible to do so today, but there's no need to worry about it.


</p>
<h3 align="center">Copyright © 2025</h3>
<p align="center">
</p>

> Please feel free to contact us if you have any comments or questions :
tknohamzacontact@gmail.com
Please let us know about it :
<a href="https://facebook.com/tknohamza">Facebook</a>, <a href="https://instagram.com/r/tknohamza">Instagram</a>, <a href="https://twitter.com/tknohamza">Twitter</a>, <a href="https://t.me/tknohamzachannel">Telegram</a>
