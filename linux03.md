# grep, awk, find, sed — Practical Reference

## grep — search text for patterns

| Task | Command |
|---|---|
| Basic search | `grep "error" file.log` |
| Case-insensitive search | `grep -i "error" file.log` |
| Search recursively through a directory | `grep -r "TODO" ./src` |
| Show line numbers | `grep -n "error" file.log` |
| Invert match (lines that DON'T match) | `grep -v "debug" file.log` |
| Count matching lines | `grep -c "error" file.log` |
| Show only the matched text, not the whole line | `grep -o "error[0-9]*" file.log` |
| Match whole word only | `grep -w "cat" file.log` (won't match "concatenate") |
| Use regex (extended) | `grep -E "error|warning" file.log` |
| Show N lines of context after match | `grep -A 3 "error" file.log` |
| Show N lines of context before match | `grep -B 3 "error" file.log` |
| Show N lines before AND after | `grep -C 3 "error" file.log` |
| List only filenames with a match (not the lines) | `grep -l "error" *.log` |
| Search multiple files | `grep "error" *.log` |

**Common real use:** `grep -ri "connection refused" /var/log/nginx/` — hunting for an issue across logs, case-insensitive, recursive.

## awk — pattern scanning & text processing (column-based)

| Task | Command |
|---|---|
| Print a specific column (field) | `awk '{print $1}' file.txt` (1st column) |
| Print multiple columns | `awk '{print $1, $3}' file.txt` |
| Use a custom delimiter (e.g. CSV) | `awk -F, '{print $2}' file.csv` |
| Print lines matching a pattern | `awk '/error/ {print}' file.log` |
| Filter by column value | `awk '$3 > 100 {print}' file.txt` |
| Print line number + line | `awk '{print NR, $0}' file.txt` |
| Sum a column | `awk '{sum += $2} END {print sum}' file.txt` |
| Print last column regardless of column count | `awk '{print $NF}' file.txt` |
| Print total number of lines | `awk 'END {print NR}' file.txt` |

**Common real use:** `awk -F: '{print $1}' /etc/passwd` — extracts just usernames from `/etc/passwd` (colon-delimited file).

## find — locate files/directories

| Task | Command |
|---|---|
| Find by name | `find /var/www -name "*.log"` |
| Case-insensitive name search | `find /var/www -iname "*.LOG"` |
| Find by type (file / directory) | `find . -type f` / `find . -type d` |
| Find files modified in the last N days | `find . -mtime -7` |
| Find files older than N days | `find . -mtime +30` |
| Find files over a certain size | `find . -size +100M` |
| Find AND delete matching files | `find . -name "*.tmp" -delete` |
| Find AND run a command on each result | `find . -name "*.log" -exec rm {} \;` |
| Find files owned by a specific user | `find / -user prajjwal` |
| Limit search depth | `find . -maxdepth 2 -name "*.conf"` |

**Common real use:** `find /var/log -name "*.log" -mtime +30 -delete` — classic log cleanup script (delete logs older than 30 days).

## sed — stream editor (find & replace, text transformation)

| Task | Command |
|---|---|
| Replace first match per line | `sed 's/old/new/' file.txt` |
| Replace ALL matches per line (global) | `sed 's/old/new/g' file.txt` |
| Edit the file in place | `sed -i 's/old/new/g' file.txt` |
| Case-insensitive replace | `sed 's/old/new/gi' file.txt` |
| Delete lines matching a pattern | `sed '/pattern/d' file.txt` |
| Print only specific line numbers | `sed -n '5,10p' file.txt` |
| Delete a specific line number | `sed '3d' file.txt` |
| Replace only on a specific line number | `sed '2s/old/new/' file.txt` |
| Insert a line before a match | `sed '/pattern/i\new line' file.txt` |
| Append a line after a match | `sed '/pattern/a\new line' file.txt` |

**Common real use:** `sed -i 's/DEBUG/INFO/g' app.conf` — quickly swap a config value across a file without opening an editor.

## Quick rule of thumb — which tool for what

* **grep** → *finding* lines that match a pattern
* **awk** → working with *columns/fields* in structured text (logs, CSVs, `/etc/passwd`, etc.)
* **find** → locating *files* on disk by name/type/size/date, not text inside files
* **sed** → *editing/transforming* text (replace, delete, insert) in a stream or file

These four are commonly chained together with pipes, e.g.:
```
grep "error" app.log | awk '{print $1, $4}' | sort | uniq -c
```
