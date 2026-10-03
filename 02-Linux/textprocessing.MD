# Linux Text Processing — awk, sed & grep

A practical cheat sheet for the most important Linux text-processing commands used in DevOps, log analysis, troubleshooting, and shell scripting.

---

## Table of Contents

- [Quick Comparison](#quick-comparison)
- [awk](#awk)
- [sed](#sed)
- [grep](#grep)
- [Combining awk, sed and grep](#combining-awk-sed-and-grep)
- [Quick Cheat Sheet](#quick-cheat-sheet)
- [Important Symbols](#important-symbols)
- [Common Workflow](#common-workflow)
- [Interview Quick Revision](#interview-quick-revision)

---

## Quick Comparison

| Command | Main Purpose |
|---|---|
| `awk` | Extract, filter, calculate, and process columns |
| `sed` | Search, replace, delete, and modify text |
| `grep` | Search and filter text |

### Simple Mental Model

```text
awk  → Process
sed  → Modify
grep → Find
```

---

# awk

`awk` is mainly used to **process columns, filter data, perform calculations, and format output**.

## Basic Syntax

```bash
awk 'pattern { action }' file
```

---

## awk Fields

Suppose `users.txt` contains:

```
John 25 Chennai
Mike 30 Bangalore
Sam 28 Mumbai
```

Fields:

```
$0  → Entire line
$1  → First field
$2  → Second field
$3  → Third field
$NF → Last field
NF  → Number of fields
NR  → Current line number
```

---

## 1. Print First Column

```bash
awk '{print $1}' file
```

### Example

`users.txt`:

```
John 25 Chennai
Mike 30 Bangalore
Sam 28 Mumbai
```

Output:

```
John
Mike
Sam
```

---

## 2. Print Multiple Columns

```bash
awk '{print $1,$2}' file
```

Output:

```
John 25
Mike 30
Sam 28
```

---

## 3. Print Last Column

```bash
awk '{print $NF}' file
```

`$NF` → Last field.

Output (for `users.txt`):

```
Chennai
Bangalore
Mumbai
```

---

## 4. Print Number of Fields

```bash
awk '{print NF}' file
```

Output:

```
3
3
3
```

---

## 5. Print Line Number Along With the Line

```bash
awk '{print NR,$0}' file
```

`NR` → Current line number.

Output:

```
1 John 25 Chennai
2 Mike 30 Bangalore
3 Sam 28 Mumbai
```

---

## 6. Print a Specific Line

```bash
awk 'NR==5 {print}' file
```

Prints line 5.

---

## 7. Print a Range of Lines

```bash
awk 'NR>=5 && NR<=10 {print}' file
```

Prints lines 5 through 10.

---

## 8. Filter Using a Numeric Condition

```bash
awk '$2 > 50 {print}' file
```

Prints lines where column 2 is greater than 50.

### Example

`data.txt`:

```
app1 40
app2 75
app3 90
```

Output:

```
app2 75
app3 90
```

---

## 9. String Comparison

```bash
awk '$3 == "ERROR" {print}' file
```

Prints lines where the third column equals `ERROR`.

### Example

`app.log`:

```
2026-10-03 09:20:15 ERROR
2026-10-03 09:25:10 INFO
```

Output:

```
2026-10-03 09:20:15 ERROR
```

---

## 10. Search Using Regex

```bash
awk '/ERROR/ {print}' file
```

Prints lines containing `ERROR`.

---

## 11. Regex on a Specific Field

```bash
awk '$4 ~ /server/ {print}' file
```

`~` → Matches a regular expression.

### Example

`app.log`:

```
2026-10-03 09:20:15 app server-1
2026-10-03 09:21:10 app db-1
```

Output:

```
2026-10-03 09:20:15 app server-1
```

---

## 12. NOT Regex Match

```bash
awk '$4 !~ /server/ {print}' file
```

`!~` → Does not match regex.

Output (for the file above):

```
2026-10-03 09:21:10 app db-1
```

---

## 13. Sum a Column

Suppose `sales.txt`:

```
John 100
Mike 200
Sam 300
```

Command:

```bash
awk '{sum += $2} END {print sum}' file
```

Output:

```
600
```

---

## 14. Calculate Average

```bash
awk '{sum += $2} END {print sum/NR}' file
```

Output:

```
200
```

---

## 15. Count Lines

```bash
awk 'END {print NR}' file
```

---

## 16. Custom Field Separator (CSV)

For a CSV file:

```
John,25,Chennai
Mike,30,Bangalore
```

Use:

```bash
awk -F',' '{print $1,$2}' file.csv
```

Output:

```
John 25
Mike 30
```

`-F` → Field separator.

---

## 17. Colon Separator

Useful with `/etc/passwd`:

```bash
awk -F':' '{print $1}' /etc/passwd
```

Output:

```
root
daemon
bin
```

---

# sed

`sed` is mainly used to **search, replace, delete, print, and modify text**.

## Basic Syntax

```bash
sed 'command' file
```

---

## 1. Replace First Occurrence

```bash
sed 's/old/new/' file
```

`s` → Substitute.

Replaces the first occurrence on each line.

### Example

Input:

```
error here and error there
```

Output:

```
new here and error there
```

---

## 2. Replace All Occurrences

```bash
sed 's/old/new/g' file
```

`g` → Global replacement.

### Important Pattern

```bash
sed 's/FIND/REPLACE/g' file
```

---

## 3. Case-Insensitive Replacement

```bash
sed 's/old/new/gi' file
```

- `g` → Global
- `i` → Ignore case

---

## 4. Print a Specific Line

```bash
sed -n '5p' file
```

- `-n` → Suppress automatic output
- `p` → Print

---

## 5. Print a Range of Lines

```bash
sed -n '1,5p' file
```

Prints lines 1 through 5.

---

## 6. Print Lines Matching a Pattern

```bash
sed -n '/ERROR/p' file
```

Prints lines containing `ERROR`.

---

## 7. Delete a Specific Line

```bash
sed '5d' file
```

`d` → Delete.

---

## 8. Delete a Range

```bash
sed '2,5d' file
```

Deletes lines 2 through 5 from the output.

---

## 9. Delete Matching Lines

```bash
sed '/ERROR/d' file
```

Deletes lines containing `ERROR` from the output.

---

## 10. Delete First Line

```bash
sed '1d' file
```

---

## 11. Delete Last Line

```bash
sed '$d' file
```

`$` → Last line.

---

## 12. Delete Blank Lines

```bash
sed '/^$/d' file
```

---

## 13. Delete Comments

```bash
sed '/^#/d' file
```

Deletes lines starting with `#`.

---

## 14. Add Text at Beginning

```bash
sed 's/^/#/' file
```

`^` → Beginning of line.

### Example

Input:

```
server=1
port=8080
```

Output:

```
#server=1
#port=8080
```

---

## 15. Add Text at End

```bash
sed 's/$/END/' file
```

`$` → End of line.

Output (for the same input):

```
server=1END
port=8080END
```

---

## 16. Modify File Directly

### Linux / GNU sed

```bash
sed -i 's/old/new/g' file
```

### macOS

```bash
sed -i '' 's/old/new/g' file
```

`-i` → Modify the actual file.

---

# grep

`grep` is mainly used to **search and filter text**.

## Basic Syntax

```bash
grep [options] "pattern" file
```

---

## 1. Basic Search

```bash
grep "ERROR" file
```

Find lines containing `ERROR`.

### Example

`app.log`:

```
2026-10-03 09:20:15 ERROR [db-service] Database connection failed
2026-10-03 09:25:10 ERROR [api-service] Request failed
2026-10-03 09:26:00 INFO  [api-service] Request completed
```

Output:

```
2026-10-03 09:20:15 ERROR [db-service] Database connection failed
2026-10-03 09:25:10 ERROR [api-service] Request failed
```

---

## 2. Case-Insensitive Search

```bash
grep -i "error" file
```

`-i` → Ignore case.

Matches:

```
ERROR
Error
error
eRrOr
```

---

## 3. Show Line Numbers

```bash
grep -n "ERROR" file
```

`-n` → Show line number.

Example:

```
8:2026-10-03 09:20:15 ERROR [db-service] Database connection failed
```

---

## 4. Count Matching Lines

```bash
grep -c "ERROR" file
```

`-c` → Count matching lines.

Output:

```
2
```

---

## 5. Exclude Matching Lines

```bash
grep -v "ERROR" file
```

`-v` → Invert the match.

Shows lines that do not contain `ERROR`.

---

## 6. Recursive Search

```bash
grep -r "ERROR" directory
```

`-r` → Search recursively through directories.

---

## 7. Recursive Search + Line Numbers

```bash
grep -rn "ERROR" directory
```

- `-r` → Recursive
- `-n` → Line number

Very useful for troubleshooting.

### Example

Output:

```
var/log/app.log:8:2026-10-03 09:20:15 ERROR Database connection failed
var/log/db.log:3:2026-10-03 09:22:00 ERROR Connection timeout
```

---

## 8. Match Whole Word

```bash
grep -w "ERROR" file
```

`-w` → Match the complete word.

---

## 9. Multiple Patterns

```bash
grep -E "ERROR|WARN" file
```

Finds lines containing either `ERROR` or `WARN`.

`-E` → Extended regular expression.

---

## 10. Show Context Around a Match

### Lines After

```bash
grep -A 2 "ERROR" file
```

`-A 2` → Show 2 lines after the match.

### Lines Before

```bash
grep -B 2 "ERROR" file
```

`-B 2` → Show 2 lines before the match.

### Lines Before and After

```bash
grep -C 2 "ERROR" file
```

`-C 2` → Show 2 lines before and after.

---

## 11. Show Only Matching Text

```bash
grep -o "ERROR" file
```

`-o` → Print only the matched text.

Output:

```
ERROR
ERROR
```

---

## 12. Show Files Containing a Match

```bash
grep -rl "ERROR" directory
```

`-l` → Show filenames containing the match.

Output:

```
app.log
db.log
```

---

## 13. Show Files Without a Match

```bash
grep -rL "ERROR" directory
```

`-L` → Show filenames that do not contain the match.

---

## 14. Search Live Logs

```bash
tail -f app.log | grep "ERROR"
```

Continuously displays new log lines containing `ERROR`.

### Example

Terminal output while the app keeps logging:

```
2026-10-03 09:20:15 ERROR [db-service] Database connection failed
2026-10-03 09:31:02 ERROR [api-service] Request failed
2026-10-03 09:33:48 ERROR [cache] Redis unavailable
```

Each new `ERROR` line appears in real time as it is written to `app.log`.

---

# Combining awk, sed and grep

## grep → awk

```bash
grep "ERROR" file | awk '{print $1,$2,$4}'
```

Process:

```text
file
   ↓
grep → Find ERROR
   ↓
awk  → Extract columns
```

---

## grep → sed

```bash
grep "ERROR" file | sed 's/ERROR/PROBLEM/g'
```

---

## grep → awk → sort

```bash
grep "ERROR" file | awk '{print $4}' | sort
```

---

## tail → grep

```bash
tail -f app.log | grep "ERROR"
```

Useful for monitoring live application logs.

---

## ps → grep

```bash
ps aux | grep nginx
```

Find processes containing `nginx`.

---

## docker → grep

```bash
docker ps | grep nginx
```

Find Docker containers containing `nginx`.

---

# Quick Cheat Sheet

## awk

```bash
awk '{print $1}' file
awk '{print $1,$2}' file
awk '{print $NF}' file
awk '{print NF}' file
awk '{print NR,$0}' file
awk 'NR==5 {print}' file
awk 'NR>=5 && NR<=10 {print}' file
awk '$2 > 50 {print}' file
awk '$3 == "ERROR" {print}' file
awk '/ERROR/ {print}' file
awk '$4 ~ /server/ {print}' file
awk '$4 !~ /server/ {print}' file
awk '{sum += $2} END {print sum}' file
awk '{sum += $2} END {print sum/NR}' file
awk 'END {print NR}' file
awk -F',' '{print $1,$2}' file.csv
awk -F':' '{print $1}' /etc/passwd
```

## sed

```bash
sed 's/old/new/' file
sed 's/old/new/g' file
sed 's/old/new/gi' file
sed -n '5p' file
sed -n '1,5p' file
sed -n '/ERROR/p' file
sed '5d' file
sed '2,5d' file
sed '/ERROR/d' file
sed '1d' file
sed '$d' file
sed '/^$/d' file
sed '/^#/d' file
sed 's/^/#/' file
sed 's/$/END/' file

# Linux
sed -i 's/old/new/g' file

# macOS
sed -i '' 's/old/new/g' file
```

## grep

```bash
grep "ERROR" file
grep -i "error" file
grep -n "ERROR" file
grep -c "ERROR" file
grep -v "ERROR" file
grep -r "ERROR" directory
grep -rn "ERROR" directory
grep -w "ERROR" file
grep -E "ERROR|WARN" file
grep -A 2 "ERROR" file
grep -B 2 "ERROR" file
grep -C 2 "ERROR" file
grep -o "ERROR" file
grep -rl "ERROR" directory
grep -rL "ERROR" directory
tail -f app.log | grep "ERROR"
```

---

# Important Symbols

## awk

| Symbol | Meaning |
|---|---|
| `$0` | Entire line |
| `$1` | First field |
| `$2` | Second field |
| `$NF` | Last field |
| `NF` | Number of fields |
| `NR` | Current line number |
| `~` | Matches regex |
| `!~` | Does not match regex |
| `-F` | Field separator |

## sed

| Symbol / Command | Meaning |
|---|---|
| `s` | Substitute |
| `d` | Delete |
| `p` | Print |
| `g` | Global |
| `i` | Ignore case |
| `-n` | Suppress automatic output |
| `-i` | Edit file directly |
| `^` | Beginning of line |
| `$` | End of line |

## grep / Regular Expressions

| Symbol | Meaning |
|---|---|
| `^` | Beginning of line |
| `$` | End of line |
| `.` | Any single character |
| `[]` | Character set |
| `[^]` | NOT character set |
| `\|` | OR |
| `*` | Zero or more |
| `+` | One or more |
| `?` | Zero or one |

---

# Common Workflow

## Extract a Column With awk

```bash
awk '{print $4}' file
```

## Find Records Matching a Condition

```bash
awk '$2 > 500 {print}' file
```

## Calculate a Total

```bash
awk '{sum += $2} END {print sum}' file
```

## Replace Text With sed

```bash
sed 's/localhost/production-server/g' file
```

## Remove Comments

```bash
sed '/^#/d' file
```

## Remove Blank Lines

```bash
sed '/^$/d' file
```

## Find Errors in Logs

```bash
grep -i "error" file
```

## Watch Errors in Real Time

```bash
tail -f app.log | grep -i "error"
```

---

# Interview Quick Revision

## What is awk?

`awk` is used to **process structured text, especially columns and fields**.

## What is sed?

`sed` is a **stream editor** used to search, replace, delete, and transform text.

## What is grep?

`grep` is used to **search and filter text based on patterns**.

## Difference Between awk, sed and grep

```text
awk  → Process / Calculate / Extract
sed  → Modify / Replace / Delete
grep → Find / Filter
```

## What does `$1` mean in awk?

```text
$1 → First field
$2 → Second field
$NF → Last field
$0 → Entire line
```

## What does `NR` mean?

```text
NR → Current record/line number
```

## What does `NF` mean?

```text
NF → Number of fields
```

## What does `END` mean in awk?

```text
END → Execute after all input has been processed
```

## What does `s` mean in sed?

```text
s → Substitute
```

## What does `g` mean in sed?

```text
g → Global replacement
```

## What does `-i` mean in sed?

```text
-i → Modify the actual file
```

## What does `-r` mean in grep?

```text
-r → Recursive search
```

## What does `-i` mean in grep?

```text
-i → Ignore case
```

## What does `-n` mean in grep?

```text
-n → Show line numbers
```

---

# Final Cheat Sheet

```text
awk  → PROCESS
sed  → MODIFY
grep → FIND

awk '{print $1}' file
awk '{print $NF}' file
awk '{print NR,$0}' file
awk '$2 > 50 {print}' file
awk '{sum += $2} END {print sum}' file

sed 's/old/new/' file
sed 's/old/new/g' file
sed -n '1,5p' file
sed '/ERROR/d' file
sed -i 's/old/new/g' file

grep -i "error" file
grep -n "error" file
grep -rn "error" directory
grep -E "error|warn" file

command1 | command2
```

---

## Tip

Master these concepts first:

1. `awk`
2. `sed`
3. `grep`
4. Pipes `|`
5. Redirection `>`, `>>`
6. Regular expressions
7. Log analysis

Together, these commands form a strong foundation for **Linux and DevOps troubleshooting**.
