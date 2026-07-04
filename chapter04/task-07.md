# Task 07: The Missing Space Mystery

**Mode:** Production Outage Translation (Mode B)
**Calibrated Difficulty:** Junior/Mid SRE
**Target Concepts:** LVM Resizing, `resize2fs`, Filesystem Boundaries

## Scenario
A developer panicked because their `/var/www/html` partition was 100% full. They submitted an emergency ticket for you to add 50GB of space. 

You log in and execute the following command:
`sudo lvextend -L +50G /dev/mapper/web_vg-html_lv`

The command completes successfully, outputting:
`Logical volume web_vg/html_lv successfully resized.`

You tell the developer the job is done. Two minutes later, they message you back angrily: "My script is still crashing with 'No space left on device'! I ran `df -h` and it still shows the old size!"

## Your Task
1. Why does `df -h` still show the old size even though `lvextend` reported success? What did you forget to do?
2. What is the single command you need to run to fix this and make the 50GB visible to the applications?
3. What flag could you have passed to `lvextend` originally to do both steps automatically in one command?

---
### Your Solution:
1. Because there is no space. We extended the logical volume but forgot to extend the Filesystem
2. We run `sudo resize2fs /dev/mapper/web_vg-html_lv` to resize the filesystem, by default it will extend to full capacity
3. We can do `sudo lvextend -r -L +50G /dev/mapper/web_vg-html_lv` to get both the steps done in one step

**Correction:**
100% correct! This is a very common trap for junior admins. Expanding the logical volume just expands the "container". You always have to use `resize2fs` (for ext4) or `xfs_growfs` (for XFS) to tell the filesystem inside to expand into the new empty space. And as you noted, the `-r` flag is the ultimate quality-of-life shortcut!
