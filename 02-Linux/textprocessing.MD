# Linux Text Processing — grep, awk & sed

A practical cheat sheet for the most important Linux text-processing commands used in DevOps, log analysis, troubleshooting, and shell scripting.

---

## Quick Comparison

| Command | Main Purpose |
|---|---|
| `grep` | Search and filter text |
| `awk` | Extract, filter, calculate, and process columns |
| `sed` | Search, replace, delete, and modify text |

### Simple Mental Model

\`\`\`text
grep → Find
awk  → Process
sed  → Modify
\`\`\`

---

# grep

`grep` is mainly used to **search and filter text**.

## Basic Syntax

\`\`\`bash
grep [options] "pattern" file
\`\`\`

---

## 1. Basic Search

\`\`\`bash
grep "ERROR" app.log
\`\`\`

Find lines containing `ERROR`.

### Example

\`\`\`
2026-10-03 09:20:15 ERROR [db-service] Database connection failed
2026-10-03 09:25:10 ERROR [api-service] Request failed
\`\`\`

---

## 2. Case-Insensitive Search

\`\`\`bash
grep -i "error" app.log
\`\`\`

`-i` → Ignore case.

Matches:

\`\`\`
ERROR
Error
error
eRrOr
\`\`\`

---

## 3. Show Line Numbers

\`\`\`bash
grep -n "ERROR" app.log
\`\`\`

`-n` → Show line number.

Example:

\`\`\`
8:2026-10-03 09:20:15 ERROR [db-service] Database connection failed
\`\`\`

---

## 4. Count Matching Lines

\`\`\`bash
grep -c "ERROR" app.log
\`\`\`

`-c` → Count matching lines.

---

## 5. Exclude Matching Lines

\`\`\`bash
grep -v "ERROR" app.log
\`\`\`

`-v` → Invert the match.

Shows lines that do not contain `ERROR`.

---

## 6. Recursive Search

\`\`\`bash
grep -r "ERROR" /var/log/
\`\`\`

`-r` → Search recursively through directories.

---

## 7. Recursive Search + Line Numbers

\`\`\`bash
grep -rn "ERROR" /var/log/
\`\`\`

- `-r` → Recursive
- `-n` → Line number

Very useful for troubleshooting.

---

## 8. Match Whole Word

\`\`\`bash
grep -w "ERROR" app.log
\`\`\`

`-w` → Match the complete word.

---

## 9. Multiple Patterns

\`\`\`bash
grep -E "ERROR|WARN" app.log
\`\`\`

Finds lines containing either `ERROR` or `WARN`.

`-E` → Extended regular expression.

---

## 10. Show Context Around a Match

### Lines After

\`\`\`bash
grep -A 2 "ERROR" app.log
\`\`\`

`-A 2` → Show 2 lines after the match.

### Lines Before

\`\`\`bash
grep -B 2 "ERROR" app.log
\`\`\`

`-B 2` → Show 2 lines before the match.

### Lines Before and After

\`\`\`bash
grep -C 2 "ERROR" app.log
\`\`\`

`-C 2` → Show 2 lines before and after.

---

## 11. Show Only Matching Text

\`\`\`bash
grep -o "ERROR" app.log
\`\`\`

`-o` → Print only the matched text.

---

## 12. Search Only Specific Files

\`\`\`bash
grep -rn --include="*.log" "ERROR" .
\`\`\`

Searches only `.log` files.

---

## 13. Exclude a Directory

\`\`\`bash
grep -rn --exclude-dir=node_modules "ERROR" .
\`\`\`

Useful when searching source-code projects.

---

## 14. Show Files Containing a Match

\`\`\`bash
grep -rl "ERROR" .
\`\`\`

`-l` → Show filenames containing the match.

---

## 15. Show Files Without a Match

\`\`\`bash
grep -rL "ERROR" .
\`\`\`

`-L` → Show filenames that do not contain the match.

---

## 16. Search Multiple Files

\`\`\`bash
grep "ERROR" app.log server.log database.log
\`\`\`

Example output:

\`\`\`
app.log:ERROR Database connection failed
server.log:ERROR Server crashed
database.log:ERROR Connection failed
\`\`\`

---

## 17. Search Live Logs

\`\`\`bash
tail -f app.log | grep "ERROR"
\`\`\`

Continuously displays new log lines containing `ERROR`.

---

# awk

`awk` is mainly used to **process columns, filter data, perform calculations, and format output**.

## Basic Syntax

\`\`\`bash
awk 'pattern { action }' file
\`\`\`

---

## awk Fields

Suppose `users.txt` contains:

\`\`\`
John 25 Chennai
Mike 30 Bangalore
Sam 28 Mumbai
\`\`\`

Fields:

\`\`\`
$0  → Entire line
$1  → First field
$2  → Second field
$3  → Third field
$NF → Last field
NF  → Number of fields
NR  → Current line number
\`\`\`

---

## 1. Print First Column

\`\`\`bash
awk '{print $1}' users.txt
\`\`\`

---

## 2. Print Multiple Columns

\`\`\`bash
awk '{print $1,$3}' users.txt
\`\`\`

Output:

\`\`\`
John Chennai
Mike Bangalore
Sam Mumbai
\`\`\`

---

## 3. Print Entire Line

\`\`\`bash
awk '{print $0}' users.txt
\`\`\`

---

## 4. Print Last Column

\`\`\`bash
awk '{print $NF}' users.txt
\`\`\`

`$NF` → Last field.

---

## 5. Print Number of Fields

\`\`\`bash
awk '{print NF}' users.txt
\`\`\`

`NF` → Number of fields.

---

## 6. Print Line Number

\`\`\`bash
awk '{print NR,$0}' users.txt
\`\`\`

`NR` → Current line number.

---

## 7. Print Specific Line

\`\`\`bash
awk 'NR==5 {print}' file.txt
\`\`\`

Prints line 5.

---

## 8. Print Range of Lines

\`\`\`bash
awk 'NR>=5 && NR<=10 {print}' file.txt
\`\`\`

Prints lines 5 through 10.

---

## 9. Filter Using a Condition

\`\`\`bash
awk '$2 > 25 {print $1,$2}' users.txt
\`\`\`

Prints records where column 2 is greater than 25.

---

## 10. String Comparison

\`\`\`bash
awk '$3 == "Chennai" {print $1}' users.txt
\`\`\`

Prints names where the third column is `Chennai`.

---

## 11. Search Using Regex

\`\`\`bash
awk '/ERROR/ {print}' app.log
\`\`\`

Prints lines containing `ERROR`.

---

## 12. Regex on a Specific Field

\`\`\`bash
awk '$4 ~ /server/ {print}' app.log
\`\`\`

`~` → Matches a regular expression.

---

## 13. NOT Regex Match

\`\`\`bash
awk '$4 !~ /server/ {print}' app.log
\`\`\`

`!~` → Does not match regex.

---

## 14. Sum a Column

Suppose:

\`\`\`
John 100
Mike 200
Sam 300
\`\`\`

Command:

\`\`\`bash
awk '{sum += $2} END {print sum}' file.txt
\`\`\`

Output:

\`\`\`
600
\`\`\`

---

## 15. Calculate Average

\`\`\`bash
awk '{sum += $2} END {print sum/NR}' file.txt
\`\`\`

---

## 16. Count Lines

\`\`\`bash
awk 'END {print NR}' file.txt
\`\`\`

---

## 17. Count Matching Records

\`\`\`bash
awk '$3 == "ERROR" {count++} END {print count}' app.log
\`\`\`

---

## 18. BEGIN and END

\`\`\`bash
awk 'BEGIN {print "Starting"} {print $1} END {print "Finished"}' file.txt
\`\`\`

Execution order:

\`\`\`
BEGIN
  ↓
Read input
  ↓
Process each line
  ↓
END
\`\`\`

### Important

\`\`\`
BEGIN → Before input
{ }   → Every input line
END   → After all input
\`\`\`

---

## 19. Custom Field Separator

For a CSV file:

\`\`\`
John,25,Chennai
Mike,30,Bangalore
\`\`\`

Use:

\`\`\`bash
awk -F',' '{print $1,$3}' users.csv
\`\`\`

`-F` → Field separator.

---

## 20. Colon Separator

Useful with `/etc/passwd`:

\`\`\`bash
awk -F':' '{print $1}' /etc/passwd
\`\`\`

---

## 21. Pipe Separator

\`\`\`bash
awk -F'|' '{print $1}' file.txt
\`\`\`

---

# sed

`sed` is mainly used to **search, replace, delete, print, and modify text**.

## Basic Syntax

\`\`\`bash
sed 'command' file
\`\`\`

---

## 1. Replace First Occurrence

\`\`\`bash
sed 's/error/warning/' app.log
\`\`\`

`s` → Substitute.

Replaces the first occurrence on each line.

---

## 2. Replace All Occurrences

\`\`\`bash
sed 's/error/warning/g' app.log
\`\`\`

`g` → Global replacement.

### Important Pattern

\`\`\`bash
sed 's/FIND/REPLACE/g' file
\`\`\`

---

## 3. Case-Insensitive Replacement

\`\`\`bash
sed 's/error/warning/gi' app.log
\`\`\`

- `g` → Global
- `i` → Ignore case

---

## 4. Replace Only on Matching Lines

\`\`\`bash
sed '/ERROR/s/failed/problem/' app.log
\`\`\`

Only lines containing `ERROR` are modified.

---

## 5. Print a Specific Line

\`\`\`bash
sed -n '5p' file.txt
\`\`\`

- `-n` → Suppress automatic output
- `p` → Print

---

## 6. Print a Range of Lines

\`\`\`bash
sed -n '1,5p' file.txt
\`\`\`

Prints lines 1 through 5.

---

## 7. Print Lines Matching a Pattern

\`\`\`bash
sed -n '/ERROR/p' app.log
\`\`\`

Prints lines containing `ERROR`.

---

## 8. Delete a Specific Line

\`\`\`bash
sed '5d' file.txt
\`\`\`

`d` → Delete.

---

## 9. Delete a Range

\`\`\`bash
sed '2,5d' file.txt
\`\`\`

Deletes lines 2 through 5 from the output.

---

## 10. Delete Matching Lines

\`\`\`bash
sed '/ERROR/d' app.log
\`\`\`

Deletes lines containing `ERROR` from the output.

---

## 11. Delete First Line

\`\`\`bash
sed '1d' file.txt
\`\`\`

---

## 12. Delete Last Line

\`\`\`bash
sed '$d' file.txt
\`\`\`

`$` → Last line.

---

## 13. Delete Blank Lines

\`\`\`bash
sed '/^$/d' file.txt
\`\`\`

---

## 14. Delete Comments

\`\`\`bash
sed '/^#/d' config.txt
\`\`\`

Deletes lines starting with `#`.

---

## 15. Add Text at Beginning

\`\`\`bash
sed 's/^/#/' file.txt
\`\`\`

`^` → Beginning of line.

---

## 16. Add Text at End

\`\`\`bash
sed 's/$/END/' file.txt
\`\`\`

`$` → End of line.

---

## 17. Modify File Directly

### Linux / GNU sed

\`\`\`bash
sed -i 's/old/new/g' file.txt
\`\`\`

### macOS

\`\`\`bash
sed -i '' 's/old/new/g' file.txt
\`\`\`

`-i` → Modify the actual file.

---

## 18. Multiple sed Commands

\`\`\`bash
sed -e 's/ERROR/WARN/g' -e '/DEBUG/d' app.log
\`\`\`

`-e` → Execute multiple sed expressions.

---

# Combining grep, awk and sed

## grep → awk

\`\`\`bash
grep "ERROR" app.log | awk '{print $1,$2,$4}'
\`\`\`

Process:

\`\`\`
app.log
   ↓
grep → Find ERROR
   ↓
awk  → Extract columns
\`\`\`

---

## grep → sed

\`\`\`bash
grep "ERROR" app.log | sed 's/ERROR/PROBLEM/g'
\`\`\`

---

## grep → awk → sort

\`\`\`bash
grep "ERROR" app.log | awk '{print $4}' | sort
\`\`\`

---

## tail → grep

\`\`\`bash
tail -f app.log | grep "ERROR"
\`\`\`

Useful for monitoring live application logs.

---

## ps → grep

\`\`\`bash
ps aux | grep nginx
\`\`\`

Find processes containing `nginx`.

---

## docker → grep

\`\`\`bash
docker ps | grep nginx
\`\`\`

Find Docker containers containing `nginx`.

---

# Quick Cheat Sheet

## grep

\`\`\`bash
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
\`\`\`

## awk

\`\`\`bash
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
\`\`\`

## sed

\`\`\`bash
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
\`\`\`

---

# Most Important for DevOps

## grep

\`\`\`bash
grep -i "error" app.log
grep -n "ERROR" app.log
grep -rn "ERROR" /var/log/
grep -E "ERROR|WARN" app.log
tail -f app.log | grep "ERROR"
\`\`\`

## awk

\`\`\`bash
awk '{print $1}' file
awk '{print $NF}' file
awk '{print NR,$0}' file
awk '$2 > 50 {print}' file
awk '{sum += $2} END {print sum}' file
awk -F',' '{print $1,$2}' file.csv
\`\`\`

## sed

\`\`\`bash
sed 's/old/new/' file
sed 's/old/new/g' file
sed -n '1,5p' file
sed '/ERROR/d' file
sed -i 's/old/new/g' file
\`\`\`

---

# Mental Model

\`\`\`text
                 Linux Text Processing
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
        grep            awk            sed
          │              │              │
         Find          Process        Modify
          │              │              │
       Search          Columns        Replace
       Filter          Conditions     Delete
       Count           Calculate      Print
       Regex           Sum/Average    Edit
          │              │              │
          └──────────────┼──────────────┘
                         ↓
                       Pipe |
\`\`\`

## One-Line Memory Trick

\`\`\`text
grep → FIND
awk  → PROCESS
sed  → MODIFY
\`\`\`

---

# Important Symbols

## grep / Regular Expressions

| Symbol | Meaning |
|---|---|
| `^` | Beginning of line |
| `$` | End of line |
| `.` | Any single character |
| `[]` | Character set |
| `[^]` | NOT character set |
| `|` | OR |
| `*` | Zero or more |
| `+` | One or more |
| `?` | Zero or one |

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

---

# DevOps Examples

## Find Application Errors

\`\`\`bash
grep -i "error" app.log
\`\`\`

## Find Errors with Line Numbers

\`\`\`bash
grep -in "error" app.log
\`\`\`

## Search All Log Files

\`\`\`bash
grep -rn --include="*.log" "ERROR" /var/log/
\`\`\`

## Watch Errors in Real Time

\`\`\`bash
tail -f app.log | grep -i "error"
\`\`\`

## Find Failed Requests

\`\`\`bash
grep "status=500" app.log
\`\`\`

## Extract a Specific Column

\`\`\`bash
awk '{print $4}' app.log
\`\`\`

## Find Records Matching a Condition

\`\`\`bash
awk '$2 > 500 {print}' data.txt
\`\`\`

## Calculate a Total

\`\`\`bash
awk '{sum += $2} END {print sum}' data.txt
\`\`\`

## Replace Text

\`\`\`bash
sed 's/localhost/production-server/g' config.txt
\`\`\`

## Remove Comments

\`\`\`bash
sed '/^#/d' config.txt
\`\`\`

## Remove Blank Lines

\`\`\`bash
sed '/^$/d' config.txt
\`\`\`

---

# Interview Quick Revision

## What is grep?

`grep` is used to **search and filter text based on patterns**.

## What is awk?

`awk` is used to **process structured text, especially columns and fields**.

## What is sed?

`sed` is a **stream editor** used to search, replace, delete, and transform text.

## Difference Between grep, awk and sed

\`\`\`text
grep → Find / Filter
awk  → Process / Calculate / Extract
sed  → Modify / Replace / Delete
\`\`\`

## What does `$1` mean in awk?

\`\`\`text
$1 → First field
$2 → Second field
$NF → Last field
$0 → Entire line
\`\`\`

## What does `NR` mean?

\`\`\`text
NR → Current record/line number
\`\`\`

## What does `NF` mean?

\`\`\`text
NF → Number of fields
\`\`\`

## What does `END` mean in awk?

\`\`\`text
END → Execute after all input has been processed
\`\`\`

## What does `s` mean in sed?

\`\`\`text
s → Substitute
\`\`\`

## What does `g` mean in sed?

\`\`\`text
g → Global replacement
\`\`\`

## What does `-i` mean in sed?

\`\`\`text
-i → Modify the actual file
\`\`\`

## What does `-r` mean in grep?

\`\`\`text
-r → Recursive search
\`\`\`

## What does `-i` mean in grep?

\`\`\`text
-i → Ignore case
\`\`\`

## What does `-n` mean in grep?

\`\`\`text
-n → Show line numbers
\`\`\`

---

# Final Cheat Sheet

\`\`\`text
grep → FIND
awk  → PROCESS
sed  → MODIFY

grep -i "error" file
grep -n "error" file
grep -rn "error" directory
grep -E "error|warn" file

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

command1 | command2
\`\`\`

---

## Tip

Master these concepts first:

1. `grep`
2. `awk`
3. `sed`
4. Pipes `|`
5. Redirection `>`, `>>`
6. Regular expressions
7. Log analysis

Together, these commands form a strong foundation for **Linux and DevOps troubleshooting**.