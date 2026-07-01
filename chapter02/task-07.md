# Task 07: The Recursive Nightmare 

## Scenario (Junior/Mid SRE Level)
Security compliance has requested an archive of every single system configuration file in `/etc` that contains the text `password` (case-insensitive), so they can audit them for hardcoded secrets. 

You need to find these files, extract them, and bundle them into a single compressed archive named `audit.tar.gz`.

## The Challenge
1. **The Search**: What single `grep` command can recursively search all files in `/etc` for the word "password" (case-insensitive) and print ONLY the filenames? (Check the `grep` options).
Solution: 
```bash
grep -irl "passowrd" /etc/*
```

2. **The Archive Workflow**: Explain the two-step process of using `tar` and `gzip` to create `audit.tar.gz` from those files.
Solution: 
```bash
gzip audit.md 
tar cvf audit.md.gz 
```

3. **The Shortcut**: How can you combine the archiving and compressing steps into a single `tar` command? Write out the full command assuming you have a list of files.
Solution: 
```bash
tar zcvf audit.md 
```

**Correction:**
1. Almost perfect! `grep -irl "password" /etc/` is exactly what you need. (Just watch the "passowrd" typo!). The `-l` flag is the key here.
2. **Incorrect Workflow**: `tar` doesn't take a `.gz` file! `tar` is the "Tape Archive" tool used to bundle multiple files into one big file (`tar cvf audit.tar file1 file2`). Then, you compress that massive file with `gzip audit.tar`, which results in `audit.tar.gz`.
3. The shortcut is close, but your syntax is slightly off. The full command should be: `tar zcvf audit.tar.gz file1 file2`. The `z` flag tells `tar` to pass the archive through `gzip` automatically!

## Related Concepts (Chapter 2)
- Recursive `grep` and text processing
- Archiving (`tar`)
- Compression (`gzip`)
- Pipelining commands
