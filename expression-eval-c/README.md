# 🧮 Expression Evaluation using Stack (C)

This mini-project demonstrates **expression evaluation** in C using the **stack data structure**.  
It converts **infix expressions** (like `3 + 4 * (2 - 1)`) into **postfix form** and then evaluates them.

---

## 🚀 How It Works
1. **Infix → Postfix** conversion using the **Shunting Yard algorithm**.
2. **Postfix Evaluation** using stack operations.
3. Handles operators: `+`, `-`, `*`, `/`, `^` and parentheses.

---

## 🧱 Build the Program

```bash
gcc -o expression_eval expression_eval.c -lm
```

---

## ▶️ Run the Program

### Option 1 — Command-line Expression
```bash
./expression_eval "3 + 4 * (2 - 1)"
```

### Option 2 — Interactive Mode
```bash
./expression_eval
```
Then type your expression manually.

---

## 🧮 Example

```
Input: 3 + 4 * (2 - 1)
Postfix: 3 4 2 1 - * +
Result: 7
```

---

## 📂 Files

| File | Description |
|------|--------------|
| `expression_eval.c` | Main C source code implementing stack evaluation |
| `README.md` | Project details and usage instructions |

---

## 🏷️ Author
**Nanmugil (814724104102)**  
📧 [814724104102@trp.srmtrichy.edu.in](mailto:814724104102@trp.srmtrichy.edu.in)
**Mohammed Faheem madhar  (814724104094)**
📧 [814724104094@trp.srmtrichy.edu.in]

---


