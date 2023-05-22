# Exercise 0 - Bash commands

Use windows for linux (WSL), ubuntu bash, git bash or terminal in mac to solve all these exercises, **don’t use GUI**. A lot of the training in these exercises are both to search up commands and also to understand the computer science terminology.

**Tips**: work with exercise X alongside with other exercises.

---

## 0. Setup a git repository (\*)

a) Make a directory called Linux-training.

b) Create the files called

```bash
“note1.txt”, “note2.txt”, “note3.txt”, “note4"
```

---

## 1. Navigating and moving (\*)

a) Make a subdirectory called cool_notes

b) Move all .txt files into cool_notes

c) Delete note4

d) Change directory to cool_notes and list all the files there

e) Move note3.txt out from cool_notes into parent directory

f) Navigate to parent directory

g) From here list all files including hidden files

h) Change name on note3.txt to note_home.txt

---

## 2. Printing, variables and manipulating text (\*)

a) Print out “hello from note_home” into bash

b) Write this text string into note_home.txt

c) Print out the “current path is: <your current directory path>” in bash

d) Write this string into note_home in a new line so it should contain

```bash
hello from note_home
current_directory is: <your current directory path>
```

e) Print out the content of the note_home.txt

f) Count the number of words in this file

g) Count the number of lines in this file

h) Count the number of files in cool_notes

i) Check the disk usage in your directory and make the format human readable

---

## 3. Pokeventure (\*)

a) Create a folder called data with a subfolder called pokemons

b) Create a file called pokemon_list.txt

c) Type in a random list of pokemons, using echo and the bitshift operator

for example:

```bash
pikachu
voltorb
bulbasaur
mew
zapdos
mewtwo
```

d) Loop through your file and print out the following

```bash
pokemon: pikachu
pokemon: voltorb
pokemon: bulbasaur
pokemon: mew
pokemon: zapdos
pokemon: mewtwo
```

e) Now test out the following api manually in your browser [https://pokeapi.co/api/v2/pokemon-species/voltorb](https://pokeapi.co/api/v2/pokemon-species/voltorb)

f) Now test it out using bash, and see that it prints out the same results

g) Do a for loop on pokemon_list.txt, pick the pokemons on the file and request the api. Save each pokemon into their respective json file. Important: add a pause of 2 seconds after each iteration. Your structure should look something like this now.

```bash
└── data
    └── pokemons
        ├── bulbasaur.json
        ├── mew.json
        ├── mewtwo.json
        ├── pikachu.json
        ├── pokemon_list.txt
        ├── voltorb.json
        └── zapdos.json
```

h) Remove all files ending with .json using one command

i) Now move yourself to the same level, i.e. sibling to data directory. Create a bash script file called download_pokemons.sh. Put in bash logic for downloading the pokemons specified in data/pokemon/pokemon_list.txt file and saving it into data/pokemons/ and run it. Your file structure might look like this now. (\*\*)

```bash
├── data
│   └── pokemons
│       ├── bulbasaur.json
│       ├── mew.json
│       ├── mewtwo.json
│       ├── pikachu.json
│       ├── pokemon_list.txt
│       ├── voltorb.json
│       └── zapdos.json
└── download_pokemons.sh
```

---

## X. Commands glossary (\*)

Fill in this table. You can do this in any application, it might be too hardcore to do this with terminal only. Tips you can use man command to check documentation. Also try out the different commands to see them in action.

| Command | What does the command do                                                                                                                                                                   | Some useful options |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------- |
| cd      | current directory                                                                                                                                                                          |                     |
| ls      | list files in directory                                                                                                                                                                    |                     |
| touch   | create a new file or update the access and modification timestamps of an existing file                                                                                                     |                     |
| wc      | used to count the number of lines, words, and characters in a file or input from the standard input                                                                                        |                     |
| grep    | used for searching and filtering text based on patterns                                                                                                                                    |                     |
| mkdir   | creates directory                                                                                                                                                                          |                     |
| mv      | used to move or rename files and directories.                                                                                                                                              |                     |
| rm      | used to remove/delete files and directories.                                                                                                                                               |                     |
| rmdir   | used to remove/delete empty directories.                                                                                                                                                   |                     |
| ssh     | used to establish a secure shell (SSH) connection to a remote server or device.                                                                                                            |                     |
| curl    | is a command-line tool used to transfer data to or from a server using various protocols                                                                                                   |                     |
| sudo    | used to execute a command with elevated privileges or as the superuser (root user)                                                                                                         |                     |
| apt-get | It is used to install, upgrade, or remove software packages from the system                                                                                                                |                     |
| ps      | a command used to display information about currently running processes on a system                                                                                                        |                     |
| cp      | a command used to copy files and directories                                                                                                                                               |                     |
| less    | a command-line pager that allows you to view and navigate through the contents of a file.                                                                                                  |                     |
| top     | command-line utility that provides real-time monitoring of system processes and resource usage                                                                                             |                     |
| head    | is a command used to display the beginning (head) portion of a file                                                                                                                        |                     |
| echo    | used to display text or variables as output                                                                                                                                                |                     |
| cat     | used to concatenate and display the contents of files                                                                                                                                      |                     |
| chmod   | used to change the permissions of files and directories in Linux and Unix-based systems                                                                                                    |                     |
| chown   | used to change the ownership of files and directories in Linux and Unix-based systems                                                                                                      |                     |
| kill    | terminates or sends a signal to a process.                                                                                                                                                 |                     |
| &&      | logical operator in Bash that allows you to execute a command only if the preceding command succeeds                                                                                       |                     |
| wget    | command-line utility in Bash used for downloading files from the web                                                                                                                       |                     |
| pwd     | print working directory.                                                                                                                                                                   |                     |
| >>      | a redirection operator in Bash used to append the output of a command or the contents of a file to the end of another file, without overwriting its existing contents.                     |                     |
| >       | is a redirection operator in Bash used to redirect the output of a command or the contents of a file to a file, overwriting its existing contents.                                         |                     |
| \*      | is a wildcard character in Bash used to represent any sequence of characters. It can match multiple characters, including letters, numbers, and symbols, in a filename or command argument |                     |
| ./      | used in Bash to refer to the current directory. It is used to specify the current directory as the location for executing a command or running a script                                    |                     |
| diff    | used to compare and highlight the differences between two files or directories                                                                                                             |                     |
| find    | used to search for files and directories within a specified directory hierarchy                                                                                                            |                     |
