---
name: latex-jupyter
description: Best practices for writing LaTeX math in Jupyter notebook markdown cells — error-free rendering, consistent style, and MathJax compatibility.
---

# LaTeX in Jupyter Notebooks — Best Practices Skill

You are an expert at writing LaTeX math notation inside Jupyter notebook markdown cells. Jupyter uses **MathJax** to render LaTeX — NOT a full LaTeX engine. This distinction is critical. Follow every rule below to produce error-free, beautiful mathematical notation.

---

## 1. MathJax vs Full LaTeX — Key Differences

MathJax is a subset of LaTeX. Many things that work in a `.tex` file will **break** in Jupyter. Internalize these differences:

| Feature | Full LaTeX | MathJax (Jupyter) |
|---|---|---|
| `\usepackage{}` | ✅ Works | ❌ Not supported |
| `\begin{equation}` | ✅ Works | ⚠️ Use `$$...$$` instead |
| `\begin{align}` | ✅ Works | ⚠️ Use `\begin{aligned}` inside `$$` |
| `\newcommand` | ✅ Works | ✅ Works (per-notebook scope) |
| `\text{}` | ✅ Works | ✅ Works |
| `\mathbb{}` | Requires `amssymb` | ✅ Built-in |
| `\cancel{}` | Requires `cancel` | ⚠️ May not render in all environments |
| `\reals` | ❌ | ✅ MathJax shortcut for `\mathbb{R}` |
| `\operatorname{}` | ✅ Works | ✅ Works |
| Equation numbering | ✅ Automatic | ❌ Not supported |

---

## 2. Math Delimiters — The Golden Rules

### Inline Math
Use **single dollar signs** `$...$` for math within a line of text:
```markdown
The determinant is $\det(A) = ad - bc$ where $A$ is a 2×2 matrix.
```

### Display Math
Use **double dollar signs** `$$...$$` for centered, standalone equations:
```markdown
$$
\det(A) = \begin{vmatrix} a & b \\ c & d \end{vmatrix} = ad - bc
$$
```

### Critical Rules
1. **Never leave a space** between `$` and the start/end of the math content:
   - ✅ `$x^2$`
   - ❌ `$ x^2 $` (may fail to render in some Jupyter versions)

2. **Always put `$$` on its own line** for display math in multi-line equations:
   ```markdown
   $$
   \vec{v} = \begin{bmatrix} 1 \\ 2 \\ 3 \end{bmatrix}
   $$
   ```

3. **Never nest** `$...$` inside `$$...$$` or vice versa.

4. **Don't use** `\[...\]` or `\(...\)` — while MathJax supports them, `$` and `$$` are the standard in Jupyter and avoid JSON escaping issues.

---

## 3. JSON String Escaping for `.ipynb` Files

Jupyter notebooks are JSON files. When writing LaTeX inside `.ipynb` source arrays, you must handle escaping:

### Backslash Escaping
Every `\` in LaTeX becomes `\\` inside a JSON string:

| LaTeX (rendered) | JSON string (in `.ipynb`) |
|---|---|
| `\vec{x}` | `\\vec{x}` |
| `\frac{a}{b}` | `\\frac{a}{b}` |
| `\begin{bmatrix}` | `\\begin{bmatrix}` |
| `\\` (line break in matrix) | `\\\\` |
| `\quad` | `\\quad` |

### Newlines in Source Arrays
Each element in the `"source"` array is a string. Use `\n` at the end of each line (except the last):

```json
"source": [
    "$$\n",
    "\\vec{v} = \\begin{bmatrix} 1 \\\\ 2 \\\\ 3 \\end{bmatrix}\n",
    "$$"
]
```

### Double Backslash for Matrix Row Breaks
The LaTeX `\\` (row separator) becomes `\\\\` in JSON:

```json
"source": [
    "$$A = \\begin{bmatrix} 1 & 2 \\\\\\\\ 3 & 4 \\end{bmatrix}$$"
]
```

**Why four backslashes (`\\\\`)?**
- The JSON parser reads `\\\\` as the literal string `\\`.
- MathJax then interprets `\\` as a row break.

### Ampersand in Matrices
The `&` character (column separator in matrices) does **not** need escaping in JSON. Use it directly:

```json
"source": [
    "$$\\begin{bmatrix} 1 & 2 \\\\\\\\ 3 & 4 \\end{bmatrix}$$"
]
```

---

## 4. Common Environments

### Matrices
| Type | Environment | Delimiters |
|---|---|---|
| Square brackets | `bmatrix` | $\begin{bmatrix} \end{bmatrix}$ |
| Parentheses | `pmatrix` | $\begin{pmatrix} \end{pmatrix}$ |
| Determinant bars | `vmatrix` | $\begin{vmatrix} \end{vmatrix}$ |
| Double bars (norm) | `Vmatrix` | $\begin{Vmatrix} \end{Vmatrix}$ |
| Plain (no delimiters) | `matrix` | $\begin{matrix} \end{matrix}$ |

**Preferred default**: Use `bmatrix` for matrices and vectors (matches this project's convention).

### Aligned Equations
Use `aligned` inside `$$...$$` (do NOT use `align` — it doesn't work reliably in MathJax):

```markdown
$$
\begin{aligned}
2x + 3y &= 7 \\
4x - y  &= 1
\end{aligned}
$$
```

### Cases (Piecewise)
```markdown
$$
f(x) = \begin{cases}
x^2 & \text{if } x \geq 0 \\
-x  & \text{if } x < 0
\end{cases}
$$
```

### Augmented Matrices
Use `array` with a column specification that includes `|`:
```markdown
$$
\left[\begin{array}{cc|c}
1 & 2 & 5 \\
3 & 4 & 6
\end{array}\right]
$$
```

---

## 5. Spacing in LaTeX

MathJax handles spacing automatically in most cases, but you'll sometimes need manual control:

| Command | Width | When to Use |
|---|---|---|
| `\,` | Thin | Before `dx` in integrals: `\int f(x)\,dx` |
| `\;` | Thick | Between conditions: `x > 0 \; \forall x` |
| `\quad` | 1 em | Between separate expressions |
| `\qquad` | 2 em | Wide separation |
| `\space` | Normal | General spacing (preferred in this project) |
| `\text{ }` | Text space | Inside `\text{}` blocks |
| `\!` | Negative thin | Tighten spacing |

**Project convention**: This workspace primarily uses `\space` and `\quad` for spacing. Be consistent with that pattern.

---

## 6. Text Inside Math Mode

Use `\text{}` for any natural-language text inside math:
```markdown
$$\det(A) = 0 \quad \text{if and only if A is singular}$$
```

Common patterns:
- `\text{if }` — note the trailing space inside the braces
- `\text{for }` — same trailing space
- `\text{where }` — same
- `\text{and }` — same
- `\text{(colinear)}` — parenthetical notes

**Never use** plain unescaped text inside `$...$` — it will render as italicized variable names.

---

## 7. Vectors & Matrices — Project Conventions

This project follows specific conventions. Always match them:

### Vectors
- **Arrow notation**: `\vec{v}` → $\vec{v}$ (primary style in this project)
- **Column vectors**: Always use `\begin{bmatrix}` with `\\` row separators
- **Row vectors**: Use `\vec{v}^\top` with `\begin{bmatrix}` and `&` separators
- **Unit vectors**: Use `\hat{i}`, `\hat{j}`, `\hat{k}`
- **Zero vector**: `\vec{0}`

### Matrices
- Name matrices with capital letters: $A$, $B$, $C$
- Always specify dimensions when introducing: "$A$ is a $3 \times 3$ matrix"
- Use `bmatrix` (not `pmatrix`) as the default

### Number Sets
- Use `\reals` for $\mathbb{R}$ (this is the convention in the existing notebooks)
- Use `\reals^n` for n-dimensional real space

---

## 8. Common Errors & How to Avoid Them

### Error 1: Missing Braces in Subscripts / Superscripts
- ❌ `$a_ij$` → renders as $a_i j$ (only `i` is subscripted)
- ✅ `$a_{ij}$` → renders as $a_{ij}$

### Error 2: Using `\\` Directly in JSON
- ❌ `"$\\begin{bmatrix} 1 \\ 2 \\end{bmatrix}$"` → broken
- ✅ `"$\\begin{bmatrix} 1 \\\\\\\\ 2 \\end{bmatrix}$"` → works

### Error 3: Forgetting `\left` and `\right` for Tall Delimiters
- ❌ `$(\frac{a}{b})$` → parentheses don't scale
- ✅ `$\left(\frac{a}{b}\right)$` → parentheses scale to content

### Error 4: Using `align` Instead of `aligned`
- ❌ `\begin{align}` → may silently fail in MathJax
- ✅ `\begin{aligned}` inside `$$...$$` → reliable

### Error 5: Broken `\text{}` with Special Characters
- ❌ `\text{x > 0}` → the `>` may cause issues
- ✅ Use math mode for math symbols even inside text annotations

### Error 6: Unescaped Curly Braces for Sets
- ❌ `${1, 2, 3}$` → renders nothing or errors
- ✅ `$\{1, 2, 3\}$` → renders properly as $\{1, 2, 3\}$

### Error 7: Missing `&` Alignment in Matrices
- Every row of a matrix must have the same number of `&` separators (one fewer than the number of columns).

### Error 8: Using `\mathbb` Without Uppercase
- ❌ `$\mathbb{r}$` → lowercase doesn't render in blackboard bold
- ✅ `$\mathbb{R}$` → renders properly

### Error 9: Line Breaks in Inline Math
- ❌ Putting `\\` inside `$...$` for line breaks — this doesn't work
- ✅ Use display math `$$...$$` with `\begin{aligned}` for multi-line

### Error 10: Mixing Markdown and LaTeX Improperly
- ❌ `**$\vec{v}$**` → may render inconsistently
- ✅ `$\mathbf{\vec{v}}$` for bold in math, or keep bold outside: `**vector** $\vec{v}$`

---

## 9. Rendering Verification Checklist

Before finalizing any LaTeX in a notebook, mentally verify:

- [ ] All `\begin{...}` have a matching `\end{...}`.
- [ ] All `{` have a matching `}`.
- [ ] All `\left` have a matching `\right` (use `\left.` or `\right.` for invisible delimiters).
- [ ] All matrix rows have the same number of `&` separators.
- [ ] Display math uses `$$` on separate lines (for multi-line).
- [ ] JSON strings use `\\\\` for LaTeX's `\\` row breaks.
- [ ] No stray whitespace between `$` and math content.
- [ ] `\text{}` is used for all natural language inside math mode.
- [ ] Subscripts/superscripts with multiple characters use `{}`.
- [ ] The notebook was referenced against `latex_cheatsheet.ipynb` for any unfamiliar symbols.

---

## 10. Quick Reference — Most Used Patterns

### Column Vector
```markdown
$$\vec{v} = \begin{bmatrix} v_1 \\ v_2 \\ v_3 \end{bmatrix}$$
```

### Matrix
```markdown
$$A = \begin{bmatrix} a & b \\ c & d \end{bmatrix}$$
```

### System of Equations
```markdown
$$
\begin{aligned}
2x + 3y &= 7 \\
4x - y  &= 1
\end{aligned}
$$
```

### Row Operation Label
```markdown
$$R_2 \leftarrow R_2 - 3R_1$$
```

### Augmented Matrix with Row Ops
```markdown
$$
\left[\begin{array}{ccc|c}
1 & 0 & 0 & 3 \\
0 & 1 & 0 & -1 \\
0 & 0 & 1 & 7
\end{array}\right]
$$
```

### Boxed Final Answer
```markdown
$$\boxed{x = 42}$$
```

### Determinant Expansion
```markdown
$$
\det(A) = \begin{vmatrix} a & b \\ c & d \end{vmatrix} = ad - bc
$$
```

### Eigenvalue Equation
```markdown
$$\det(A - \lambda I) = 0$$
```

### Projection
```markdown
$$\text{proj}_{\vec{v}} \vec{u} = \frac{\langle \vec{u}, \vec{v} \rangle}{\langle \vec{v}, \vec{v} \rangle} \vec{v}$$
```

---

## 11. `\newcommand` Shortcuts

Place these in the first markdown cell of a notebook to define reusable shortcuts:

```markdown
$$
\newcommand{\R}{\mathbb{R}}
\newcommand{\N}{\mathbb{N}}
\newcommand{\Z}{\mathbb{Z}}
\newcommand{\norm}[1]{\left\| #1 \right\|}
\newcommand{\abs}[1]{\left| #1 \right|}
\newcommand{\inner}[2]{\langle #1, #2 \rangle}
$$
```

**Important**: `\newcommand` definitions are **notebook-scoped**. They persist across cells in the same notebook but reset when the notebook is closed and reopened (they need to be re-rendered by running the cell).

---

## 12. Performance & Readability Tips

1. **One idea per cell** — Don't cram multiple equations into one markdown cell.
2. **Use display math for important results** — Inline math for context, display math for key formulas.
3. **Consistent notation** — Once you use `\vec{v}` for vectors, don't switch to `\mathbf{v}` mid-notebook.
4. **Label your equations** — Use surrounding markdown text to explain what each equation represents.
5. **Align multi-step derivations** — Use `\begin{aligned}` with `&=` alignment for clean step-by-step work.
6. **Separate concerns** — Theory in markdown cells, computation in code cells. Never embed LaTeX in Python strings for display (use markdown cells).
