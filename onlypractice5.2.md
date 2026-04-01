# Practice 5.2 — Touch Files in Directories

## What the problem asked
- School has classrooms named `1A` to `12E` (12 standards × 5 sections)
- Each classroom = one directory (already created by `synchro init`)
- Create files named `1` to `40` inside **each** classroom directory

---

## How we approached it

### First Idea (correct direction, wrong execution)

```bash
for i in '$pwd'; do
    touch {1..40}
done
```

**Two problems here:**
1. `'$pwd'` with single quotes → variable NOT expanded, treated as literal string
2. `touch {1..40}` without a path → creates files in current directory, not inside each classroom

---

### Understanding `$(ls)` vs `$ls`

> Can we use `$ls` to loop over directories?

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| `for i in "$ls"; do` | `for i in $(ls); do` |

- `$ls` → looks for a **variable** named `ls` — doesn't exist!
- `$(ls)` → **command substitution** — runs `ls` and uses its output

---

### Touching files inside a directory — no nested loop needed

Instead of `cd`-ing into each directory, pass the path directly to `touch`:

| ❌ Overcomplicated | ✅ Simple |
|---------|-----------|
| `cd "$i"` then `touch {1..40}` then `cd ..` | `touch "$i"/{1..40}` |

> 💡 `"$i"/{1..40}` → bash brace expansion creates paths like `1A/1  1A/2  ...  1A/40` in one shot!

---

### The Bug — `script.sh` included in loop

When we ran `for i in *; do`, it failed with:

```
touch: cannot touch 'script.sh/1': Not a directory
```

**Why:** `*` expands to **ALL** files and directories including `script.sh` itself!

---

### Fix — Loop only over directories

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| `for i in *; do` | `for i in $(ls -d */); do` |

Two clean options:
- `for i in */` → trailing slash glob matches **only directories**
- `for i in $(ls -d */)` → `ls -d */` lists only directories

---

## Final Script

```bash
#!/bin/bash
for i in $(ls -d */); do
    touch "$i"/{1..40}
done
```

---

## Key Concepts

| Syntax | Meaning |
|--------|---------|
| `$variable` | Expand a variable |
| `$(command)` | Command substitution — run command, use output |
| `'single quotes'` | NO expansion at all |
| `*` | All files AND directories |
| `*/` | Only directories |
| `{1..40}` | Brace expansion: 1 2 3 ... 40 |
| `"$i"/{1..40}` | Creates paths: `dir/1` `dir/2` ... `dir/40` |
