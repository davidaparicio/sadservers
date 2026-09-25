# "Tokelau": Delete from history

## Description

Inspired by this <a href="https://www.cyberciti.biz/faq/delete-command-from-history-linux-unix-osx-bash-shell/" target="_blank">nixCraft article</a>.
<br><br>
The objective of this exercise is to delete all the Bash <i>history</i> lines that contain the term <i>foo</i>.
<br><br>Clearing out or deleting the history file <i>/home/admin/.bash_history</i> is not allowed. Note that in our case, new commands (including the ones to try and delete "foo" from history) are also appended to the history file.

## Test

Running <kbd>history |grep "foo"</kbd> returns nothing.
<br><br>
The "Check My Solution" button runs the script <i>/home/admin/agent/check.sh</i>, which you can see and execute.


**check.sh**

```bash
#!/bin/bash
# DO NOT MODIFY THIS FILE ("Check My Solution" will fail)

# file must exist and be non-empty
if [ -s /home/admin/.bash_history ]; then
    history -r /home/admin/.bash_history
    history |grep -q "foo"

    if [ $? -eq 1 ]; then
        echo -n "OK"
    else
        echo -n "NO"
    fi
else
    echo -n "NO"
fi
```
