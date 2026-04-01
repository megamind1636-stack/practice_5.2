# Shell Scripting — Session Notes
> Problems: BPT3_4 (find & delete empty files) | Practice_5.2 (touch files in dirs)

---

## Problem 1 — BPT3 Problem 4: Delete Empty Files

### What the problem asked
- Write `script.sh` that takes a directory path as `$1`
- Find all empty files recursively using `find`
- Delete those files
- Print the names of deleted files to stdout

---

### How we approached it

#### Step 1: Read the man page for `find`
First thing — instead of guessing, opened the manual to understand available flags.

```bash
man find
```

**Searching inside man page in terminal:**
- Press `/` → enter search mode
- Type keyword (e.g. `empty`) → press Enter
- Press `n` → next match
- Press `N` → previous match
- Press `q` → quit

---

#### Step 2: Found `-empty` flag

In the **TESTS** section of `man find`:

> `-empty` → File is empty and is either a regular file or a directory.

So to find only empty regular files (not directories):

```bash
find "$1" -type f -empty
```

---

#### Step 3: Understanding `-print` and `-delete`

In the man page, actions like `-print` are described as:

> `-print   True;   print the full file name on the standard output...`

> ⚠️ **Important:** `"True;"` here is just describing the **return value** of the action — it is **NOT** part of the command!

---

#### ❌ Mistake Made — Adding 'True' in command

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| `find "$1" -type f -empty -print True -delete` | `find "$1" -type f -empty -print -delete` |

**Why:** `"True;"` in the man page is documentation describing what the action returns. It is not a flag or argument.

---

#### Step 4: Final Script

```bash
#!/bin/bash
find "$1" -type f -empty -print -delete
```

> 💡 `-print` comes **before** `-delete` so the filename is printed BEFORE the file is deleted. Order matters!

> 💡 `-delete` automatically implies `-depth` (depth-first traversal) — files inside subdirectories are processed before the parent directory.

---

## Problem 2 — Practice 5.2: Touch Files in Directories

### What the problem asked
- School has classrooms named `1A` to `12E` (12 standards × 5 sections)
- Each classroom = one directory (already created by `synchro init`)
- Create files named `1` to `40` inside **each** classroom directory

---

### How we approached it

#### First Idea (correct direction, wrong execution)

```bash
for i in '$pwd'; do
    touch {1..40}
done
```

**Two problems here:**
1. `'$pwd'` with single quotes → variable NOT expanded, treated as literal string
2. `touch {1..40}` without a path → creates files in current directory, not inside each classroom

---

#### Understanding `$(ls)` vs `$ls`

> Can we use `$ls` to loop over directories?

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| `for i in "$ls"; do` | `for i in $(ls); do` |

- `$ls` → looks for a **variable** named `ls` — doesn't exist!
- `$(ls)` → **command substitution** — runs `ls` and uses its output

---

#### Touching files inside a directory — no nested loop needed

Instead of `cd`-ing into each directory, we can pass the path directly to `touch`:

| ❌ Overcomplicated | ✅ Simple |
|---------|-----------|
| `cd "$i"` then `touch {1..40}` then `cd ..` | `touch "$i"/{1..40}` |

> 💡 `"$i"/{1..40}` → bash brace expansion creates paths like `1A/1  1A/2  ...  1A/40` in one shot!

---

#### The Bug — `script.sh` included in loop

When we ran `for i in *; do`, it failed with:

```
touch: cannot touch 'script.sh/1': Not a directory
```

**Why:** `*` expands to **ALL** files and directories including `script.sh` itself! So the loop tried to treat `script.sh` as a directory.

---

#### Fix — Loop only over directories

| ❌ Wrong | ✅ Correct |
|---------|-----------|
| `for i in *; do` | `for i in $(ls -d */); do` |

Two clean options:
- `for i in */` → trailing slash glob matches **only directories**
- `for i in $(ls -d */)` → `ls -d */` lists only directories

---

#### Final Script

```bash
#!/bin/bash
for i in $(ls -d */); do
    touch "$i"/{1..40}
done
```

---

## Key Concepts Learned

### 1. Reading `man` pages in terminal
| Key | Action |
|-----|--------|
| `man <cmd>` | Open manual |
| `/keyword` | Search forward |
| `n` / `N` | Next / previous match |
| `q` | Quit |

> ⚠️ `"True;"` and `"False;"` in man pages = **return value description**, NOT part of the command!

---

### 2. `find` command key flags

| Flag | Meaning |
|------|---------|
| `-type f` | Only regular files |
| `-type d` | Only directories |
| `-empty` | Only empty files/directories |
| `-print` | Print full path to stdout |
| `-delete` | Delete matched files (also implies `-depth`!) |
| `-exec cmd {} \;` | Run command on each matched file |

---

### 3. Bash — Variables vs Commands

| Syntax | Meaning |
|--------|---------|
| `$variable` | Expand a variable |
| `$(command)` | Command substitution — run command, use output |
| `'single quotes'` | NO expansion at all |
| `"double quotes"` | Variables expand, commands do NOT |

---

### 4. Bash — Globbing & Brace Expansion

| Syntax | Meaning |
|--------|---------|
| `*` | All files and directories |
| `*/` | Only directories (trailing slash) |
| `{1..40}` | Brace expansion: generates 1 2 3 ... 40 |
| `"$i"/{1..40}` | Creates paths: `dir/1` `dir/2` ... `dir/40` |

---

### 5. `touch` command

```bash
touch filename          # create empty file or update timestamp
touch dir/{1..40}       # create 40 files inside dir in one command
```
