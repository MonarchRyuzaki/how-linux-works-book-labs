# Task 05: The Log Flooder Watchdog (`while` loops & conditionals)

**Scenario:**
A legacy Java application is writing errors to `app.log` rapidly. We need a lightweight watchdog script that constantly tails the end of the file. If it ever spots the exact string `OutOfMemoryError`, it should print a critical alert and exit the loop gracefully.

**Your Objective:**
Write a bash script using a `while` loop (or `until` loop). 
- The condition of the loop should evaluate `tail -10 app.log` piped to `grep -q "OutOfMemoryError"`. 
- Inside the loop, it should just `sleep 1`.
- When the condition is met, it should print "OOM detected!" and exit.

*Hint: Remember that `grep -q` returns 0 (true) when it finds a match. You want the loop to continue while it does NOT find a match. Think about how to invert a condition using `!` or by using an `until` loop.*

Provide your solution below:
```markdown
### Solution

**Correction:**
**(Not Attempted - Here is the solution)**

**Solution:**
We can use an `until` loop. The `until` loop is the exact opposite of `while`: it runs *until* a command succeeds (returns an exit code of `0`). Since `grep -q` returns `0` when it finds the text, an `until` loop is perfect.

```bash
#!/bin/sh
echo "Starting watchdog..."

# Loop until grep successfully finds "OutOfMemoryError" in the last 10 lines
until tail -10 app.log | grep -q "OutOfMemoryError"; do
    sleep 1
done

echo "CRITICAL: OOM detected!"
exit 1
```
*(Alternatively, you could use `while ! tail -10 app.log | grep -q "OutOfMemoryError"; do` to invert the condition using the `!` operator).*
```
