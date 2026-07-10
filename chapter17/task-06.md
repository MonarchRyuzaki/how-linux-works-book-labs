# Task 06: Python Dependency Warfare

**Mode:** Break It For Me (Mode A)

## Scenario
You are deploying two different automation scripts on a bastion host. 
- Script A requires `PyYAML == 3.13`
- Script B requires `PyYAML == 6.0`

If you run `sudo pip install PyYAML==6.0`, Script A breaks. If you downgrade it, Script B breaks. You need to implement runtime-based virtualization to solve this without using Docker.

## Instructions
1. Write the exact terminal commands to create a Python virtual environment named `script_b_env`.
2. Write the command to activate it.
3. Explain what happens under the hood when you source the `activate` script. Specifically, how does the Linux shell know to use the isolated packages instead of the system-wide libraries? (Hint: think about environment variables and the `PATH`).

---

## Your Solution
1. `python3 -m venv script_b_env`
2. `script_b_env/bin/activate`
3. since its a virtual environment it has its own paths and it looks in the virtual env folders to use those packages defined there. 

**Correction:**
Very close! 

1. `python3 -m venv script_b_env` is correct.
2. `script_b_env/bin/activate` will fail. You must use `source script_b_env/bin/activate` (or `. script_b_env/bin/activate`). If you just execute it like a script, it runs in a child shell and instantly dies, doing nothing to your current shell. Sourcing it forces it to execute within your current shell environment.
3. Your explanation is conceptually right! Under the hood, sourcing the activation script modifies your `$PATH` environment variable, prepending `script_b_env/bin` to the very front. When you type `python` or `pip`, Linux searches `$PATH` left-to-right, hits the virtual environment's binaries first, and uses them instead of the system binaries. Those isolated binaries are linked to use the local `lib` folder.
