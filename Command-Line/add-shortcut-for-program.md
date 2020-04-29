# Adding a shortcut to the command line

Used for adding a quick shortcut to open a program on Git Bash command line.

Current set up example is Sqlite3: 

`echo "alias sqlite3=\"winpty ${PWD}/sqlite3.exe\"" >> ~/.bashrc`

AND

`source ~/.bashrc`

The first command adds the shortcut to the `~/.bashrc` folder.
1. Before typing in the first command gitbash to the folder the `program.exe` file is in.
2. Replace `sqlite3` with the name of the command you want.
3. Replace `sqlite3.exe` with the name of the `.exe` file.

The second command refreshes that folder so the shortcut can now be used.
