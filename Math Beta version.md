""" Math Solver v5 — Pro GUI Edition (Tkinter + SymPy) by: tknohamza

Requirements (install from Command Prompt): pip install sympy numpy

Run in VS Code (recommended). This app supports: • Auto-detect (equation vs expression) • Symbolic & numeric solving (solve / nsolve) • Derivative, Integral, Limit • Systems of equations (comma/newline separated or Eq(...)) • Matrices (det, inverse, eigenvalues, eigenvectors) • Pretty printed output + save results to file

Tips:

Variables auto-defined: x, y, z, t

For systems: x + y = 10, 2x - y = 3 or: Eq(x + y, 10), Eq(2x - y, 3)

For nsolve (numeric): provide a comma-separated initial guess in the field (e.g., 1 or 1,1)

For limit: set variable and point (e.g., x and 0)

Matrices: use Matrix([[1,2],[3,4]]) """


import tkinter as tk from tkinter import ttk, messagebox from tkinter.scrolledtext import ScrolledText from datetime import datetime import sympy as sp

------------------------------------------------------------

Symbols and safe namespace

------------------------------------------------------------

x, y, z, t = sp.symbols("x y z t") SAFE_GLOBALS = { # Core SymPy constructors / functions allowed in sympify 'Symbol': sp.Symbol, 'symbols': sp.symbols, 'Eq': sp.Eq, 'Matrix': sp.Matrix, 'sin': sp.sin, 'cos': sp.cos, 'tan': sp.tan, 'asin': sp.asin, 'acos': sp.acos, 'atan': sp.atan, 'sinh': sp.sinh, 'cosh': sp.cosh, 'tanh': sp.tanh, 'exp': sp.exp, 'log': sp.log, 'ln': sp.log, 'sqrt': sp.sqrt, 'pi': sp.pi, 'E': sp.E, 'oo': sp.oo, 'Abs': sp.Abs, 'diff': sp.diff, 'integrate': sp.integrate, 'limit': sp.limit, 'det': sp.Matrix.det, 'inv': sp.Matrix.inv, 'eigenvals': sp.Matrix.eigenvals, 'eigenvects': sp.Matrix.eigenvects, # Common symbols for convenience 'x': x, 'y': y, 'z': z, 't': t, 'sp': sp, }

------------------------------------------------------------

Utility functions

------------------------------------------------------------

def pretty(obj) -> str: """Return a nicely formatted string for SymPy objects.""" try: return sp.pretty(obj, use_unicode=True) except Exception: return str(obj)

def save_result_to_file(content: str, filename: str = "math_solver_results.txt"): with open(filename, "a", encoding="utf-8") as f: f.write(f"[{datetime.now()}]\n{content}\n{'-'*70}\n")

def parse_multiple_expressions(text: str): """Split input into multiple parts by comma or newline, ignoring empty pieces.""" parts = [] for raw in text.replace("\n", ",").split(','): s = raw.strip() if s: parts.append(s) return parts

def sympify_safe(expr: str): """Safely parse a string into a SymPy expression with a limited namespace.""" return sp.sympify(expr, locals=SAFE_GLOBALS)

------------------------------------------------------------

Core computation

------------------------------------------------------------

def auto_detect_and_compute(inp: str): """ Auto detect whether input is an equation, a system, or an expression and compute accordingly. Returns (title, result_obj) """ inp = inp.strip() # System via Eq(...) if 'Eq(' in inp: parts = parse_multiple_expressions(inp) eqs = [sympify_safe(p) for p in parts] if len(eqs) == 1: # Could be a single Eq or a list literal like [Eq(...), Eq(...)] single = eqs[0] if isinstance(single, (list, tuple)): eqs = list(single) else: eqs = [single] vars_set = set() for e in eqs: if not isinstance(e, sp.Equality): raise ValueError("Expected Eq(...) expressions for a system.") vars_set |= e.free_symbols sol = sp.solve(eqs, *vars_set, dict=True) return ("System Solve (symbolic)", sol)

# Equation with '=' (not assignment)
if '=' in inp:
    # Support multiple equations separated by comma/newline (system)
    parts = parse_multiple_expressions(inp)
    if len(parts) > 1:
        eqs = []
        for p in parts:
            if '=' not in p:
                raise ValueError("All parts in a system must contain '='.")
            lhs, rhs = p.split('=', 1)
            eqs.append(sp.Eq(sympify_safe(lhs), sympify_safe(rhs)))
        vars_set = set()
        for e in eqs:
            vars_set |= e.free_symbols
        sol = sp.solve(eqs, *vars_set, dict=True)
        return ("System Solve (symbolic)", sol)
    else:
        lhs, rhs = inp.split('=', 1)
        lhs_expr = sympify_safe(lhs)
        rhs_expr = sympify_safe(rhs)
        eq = sp.Eq(lhs_expr, rhs_expr)
        vars_ = list(lhs_expr.free_symbols.union(rhs_expr.free_symbols))
        sol = sp.solve(eq, *vars_)
        return ("Solve (symbolic)", sol)

# Otherwise treat as expression (simplify)
expr = sympify_safe(inp)
return ("Simplify", sp.simplify(expr))

def compute_derivative(expr_text: str, var_text: str): expr = sympify_safe(expr_text) var = sympify_safe(var_text) if var_text else x return ("Derivative", sp.diff(expr, var))

def compute_integral(expr_text: str, var_text: str): expr = sympify_safe(expr_text) var = sympify_safe(var_text) if var_text else x return ("Integral", sp.integrate(expr, var))

def compute_limit(expr_text: str, var_text: str, point_text: str): expr = sympify_safe(expr_text) var = sympify_safe(var_text) if var_text else x point = sympify_safe(point_text) if point_text else 0 return ("Limit", sp.limit(expr, var, point))

def compute_nsolve(inp: str, guess_text: str): # nsolve requires an equation (or system) and initial guesses parts = parse_multiple_expressions(inp) equations = [] all_vars = set()

# Allow Eq(...) or '='
for p in parts:
    if 'Eq(' in p:
        e = sympify_safe(p)
        if isinstance(e, (list, tuple)):
            equations.extend(e)
        else:
            equations.append(e)
    elif '=' in p:
        L, R = p.split('=', 1)
        le, re = sympify_safe(L), sympify_safe(R)
        equations.append(sp.Eq(le, re))
    else:
        raise ValueError("Each equation must be Eq(...) or contain '='.")

for e in equations:
    if not isinstance(e, sp.Equality):
        raise ValueError("Invalid element in system; expected Eq(...)")
    all_vars |= e.free_symbols

# Guesses
guesses = [g.strip() for g in guess_text.split(',') if g.strip()] if guess_text else []
if len(equations) == 1:
    if len(guesses) != 1:
        raise ValueError("Provide exactly one initial guess for single-equation nsolve.")
    var_list = list(all_vars)
    if len(var_list) != 1:
        raise ValueError("Single equation must have exactly one variable for nsolve.")
    guess_val = sympify_safe(guesses[0])
    sol = sp.nsolve(equations[0], var_list[0], guess_val)
    return ("nsolve (numeric)", sol)
else:
    # System: need as many guesses as variables
    var_list = sorted(list(all_vars), key=lambda s: s.name)
    if len(guesses) != len(var_list):
        raise ValueError(f"Provide {len(var_list)} comma-separated guesses for variables {var_list}.")
    guess_vec = sp.Matrix([sympify_safe(g) for g in guesses])
    sol = sp.nsolve(equations, var_list, guess_vec)
    return ("nsolve (numeric, system)", {str(v): sol[i] for i, v in enumerate(var_list)})

def compute_matrix(inp: str, matrix_action: str): M = sympify_safe(inp) if not isinstance(M, sp.MatrixBase): raise ValueError("Input must be a Matrix(...)") if matrix_action == 'Determinant': return ("Matrix determinant", M.det()) if matrix_action == 'Inverse': return ("Matrix inverse", M.inv()) if matrix_action == 'Eigenvalues': return ("Matrix eigenvalues", M.eigenvals()) if matrix_action == 'Eigenvectors': return ("Matrix eigenvectors", M.eigenvects()) return ("Matrix (no-op)", M)

------------------------------------------------------------

Tkinter GUI

------------------------------------------------------------

class MathSolverApp(tk.Tk): def init(self): super().init() self.title("Math Solver v5 — Pro GUI (SymPy)") self.geometry("980x700") self.minsize(900, 640) self._build_ui()

def _build_ui(self):
    # Top banner
    banner = ttk.Label(self, text="📐 Math Solver v5 — Pro GUI", font=("Segoe UI", 16, "bold"))
    banner.pack(pady=8)

    container = ttk.Frame(self)
    container.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

    # Left panel (inputs)
    left = ttk.Frame(container)
    left.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)

    ttk.Label(left, text="Input (expression / equation / system):").pack(anchor=tk.W)
    self.input_text = ScrolledText(left, height=10, font=("Consolas", 11))
    self.input_text.pack(fill=tk.BOTH, expand=True, pady=4)

    # Options frame
    opts = ttk.LabelFrame(left, text="Options")
    opts.pack(fill=tk.X, pady=6)

    # Operation selector
    ttk.Label(opts, text="Operation:").grid(row=0, column=0, sticky=tk.W, padx=6, pady=4)
    self.op_var = tk.StringVar(value="Auto detect")
    self.op_combo = ttk.Combobox(opts, textvariable=self.op_var, state="readonly",
                                 values=[
                                     "Auto detect",
                                     "Simplify",
                                     "Solve",
                                     "System Solve",
                                     "Derivative",
                                     "Integral",
                                     "Limit",
                                     "nsolve (numeric)",
                                     "Matrix"
                                 ])
    self.op_combo.grid(row=0, column=1, sticky=tk.W, padx=6, pady=4)

    # Variable / point / guess
    ttk.Label(opts, text="Variable (for diff/integral/limit):").grid(row=1, column=0, sticky=tk.W, padx=6)
    self.var_entry = ttk.Entry(opts)
    self.var_entry.grid(row=1, column=1, sticky=tk.W, padx=6)

    ttk.Label(opts, text="Point (for limit):").grid(row=1, column=2, sticky=tk.W, padx=6)
    self.point_entry = ttk.Entry(opts, width=12)
    self.point_entry.grid(row=1, column=3, sticky=tk.W, padx=6)

    ttk.Label(opts, text="Initial guess(es) (for nsolve):").grid(row=2, column=0, sticky=tk.W, padx=6)
    self.guess_entry = ttk.Entry(opts)
    self.guess_entry.grid(row=2, column=1, columnspan=3, sticky=tk.W+tk.E, padx=6)

    # Matrix action
    ttk.Label(opts, text="Matrix action:").grid(row=3, column=0, sticky=tk.W, padx=6)
    self.mx_var = tk.StringVar(value="Determinant")
    self.mx_combo = ttk.Combobox(opts, textvariable=self.mx_var, state="readonly",
                                 values=["Determinant", "Inverse", "Eigenvalues", "Eigenvectors"])
    self.mx_combo.grid(row=3, column=1, sticky=tk.W, padx=6)

    # Buttons
    btns = ttk.Frame(left)
    btns.pack(fill=tk.X, pady=6)

    run_btn = ttk.Button(btns, text="▶ Run", command=self.on_run)
    run_btn.pack(side=tk.LEFT, padx=4)

    clear_btn = ttk.Button(btns, text="🧹 Clear", command=self.on_clear)
    clear_btn.pack(side=tk.LEFT, padx=4)

    save_btn = ttk.Button(btns, text="💾 Save Output", command=self.on_save)
    save_btn.pack(side=tk.LEFT, padx=4)

    # Examples
    examples = ttk.LabelFrame(left, text="Examples (double-click to insert)")
    examples.pack(fill=tk.BOTH, expand=False, pady=6)

    self.examples_list = tk.Listbox(examples, height=8, font=("Consolas", 11))
    self.examples_list.pack(fill=tk.BOTH, expand=True)
    for ex in [
        "x**2 - 5*x + 6 = 0",
        "2*(x + 1) - x",
        "diff(x**3 + 2*x, x)",
        "integrate(sin(x)*x, x)",
        "limit(sin(x)/x, x, 0)",
        "Eq(x + y, 10), Eq(2*x - y, 3)",
        "x + y = 10, 2*x - y = 3",
        "Matrix([[1, 2], [3, 4]])",
        "det(Matrix([[1,2],[3,4]]))",
        "eigenvals(Matrix([[2,1],[1,2]]))",
        "nsolve(x - cos(x) = 0, guess: 1)"  # hint line
    ]:
        self.examples_list.insert(tk.END, ex)
    self.examples_list.bind('<Double-Button-1>', self.on_example)

    # Right panel (output)
    right = ttk.Frame(container)
    right.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(10, 0))

    ttk.Label(right, text="Output:").pack(anchor=tk.W)
    self.output = ScrolledText(right, height=24, font=("Consolas", 11))
    self.output.pack(fill=tk.BOTH, expand=True)

    # Footer
    footer = ttk.Label(self, text="© 2025 — Math Solver v5 • SymPy + Tkinter", foreground="#555")
    footer.pack(pady=4)

# --------------------------------------------------------
# Event handlers
# --------------------------------------------------------
def on_example(self, event):
    sel = self.examples_list.curselection()
    if not sel:
        return
    text = self.examples_list.get(sel[0])
    if 'guess:' in text:
        # Special nsolve hint example
        self.input_text.delete('1.0', tk.END)
        self.input_text.insert(tk.END, "x - cos(x) = 0")
        self.op_var.set("nsolve (numeric)")
        self.var_entry.delete(0, tk.END)
        self.point_entry.delete(0, tk.END)
        self.guess_entry.delete(0, tk.END)
        self.guess_entry.insert(0, "1")
    else:
        self.input_text.delete('1.0', tk.END)
        self.input_text.insert(tk.END, text)

def on_clear(self):
    self.input_text.delete('1.0', tk.END)
    self.output.delete('1.0', tk.END)
    self.var_entry.delete(0, tk.END)
    self.point_entry.delete(0, tk.END)
    self.guess_entry.delete(0, tk.END)

def on_save(self):
    content = self.output.get('1.0', tk.END).strip()
    if not content:
        messagebox.showinfo("Save Output", "Nothing to save yet.")
        return
    save_result_to_file(content)
    messagebox.showinfo("Save Output", "Output appended to math_solver_results.txt")

def write_output(self, header: str, obj):
    self.output.insert(tk.END, f"\n=== {header} ===\n")
    self.output.insert(tk.END, pretty(obj) + "\n")
    self.output.see(tk.END)

def on_run(self):
    inp = self.input_text.get('1.0', tk.END).strip()
    if not inp:
        messagebox.showwarning("Input Required", "Please enter an expression or equation.")
        return
    op = self.op_var.get()

    try:
        if op == "Auto detect":
            header, res = auto_detect_and_compute(inp)
        elif op == "Simplify":
            expr = sympify_safe(inp)
            header, res = ("Simplify", sp.simplify(expr))
        elif op == "Solve":
            header, res = auto_detect_and_compute(inp)  # will treat '=' or expression
            if not header.startswith("Solve"):
                header = "Solve (symbolic)"
        elif op == "System Solve":
            # Force system handling
            parts = parse_multiple_expressions(inp)
            eqs = []
            for p in parts:
                if 'Eq(' in p:
                    e = sympify_safe(p)
                    if isinstance(e, (list, tuple)):
                        eqs.extend(e)
                    else:
                        eqs.append(e)
                else:
                    if '=' not in p:
                        raise ValueError("System Solve expects Eq(...) or 'a=b' per line/part.")
                    L, R = p.split('=', 1)
                    eqs.append(sp.Eq(sympify_safe(L), sympify_safe(R)))
            vars_set = set()
            for e in eqs:
                vars_set |= e.free_symbols
            res = sp.solve(eqs, *vars_set, dict=True)
            header = "System Solve (symbolic)"
        elif op == "Derivative":
            header, res = compute_derivative(inp, self.var_entry.get().strip())
        elif op == "Integral":
            header, res = compute_integral(inp, self.var_entry.get().strip())
        elif op == "Limit":
            header, res = compute_limit(inp, self.var_entry.get().strip(), self.point_entry.get().strip())
        elif op == "nsolve (numeric)":
            header, res = compute_nsolve(inp, self.guess_entry.get().strip())
        elif op == "Matrix":
            header, res = compute_matrix(inp, self.mx_var.get())
        else:
            header, res = auto_detect_and_compute(inp)

        # Write GUI output
        self.write_output(header, res)
        # Save to file
        out_block = f"Input:\n{inp}\nOperation: {op}\nResult:\n{pretty(res)}\n"
        save_result_to_file(out_block)
    except Exception as e:
        messagebox.showerror("Error", str(e))

if name == "main": app = MathSolverApp() app.mainloop()

