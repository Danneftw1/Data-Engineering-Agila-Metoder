# Exercise 0 - Bash commands

Use windows for linux (WSL), ubuntu bash, git bash or terminal in mac to solve all these exercises, **don’t use GUI**. A lot of the training in these exercises are both to search up commands and also to understand the computer science terminology.

**Tips**: work with exercise X alongside with other exercises.

---

## 0. Setup a git repository (\*)

a) Make a directory called Linux-training.
```bash
mkdir Linux-training
```


b) Create the files called

```bash
“note1.txt”, “note2.txt”, “note3.txt”, “note4"
```
```bash
touch note{1..4}.txt
```

---

## 1. Navigating and moving (\*)

a) Make a subdirectory called cool_notes
```bash
mkdir  cool_notes
```


b) Move all .txt files into cool_notes

```bash
$ mv note{1..4}.txt cool_notes
```

c) Delete note4

```bash
$ rm note4.txt
```

d) Change directory to cool_notes and list all the files there

```bash
$ cd cool_notes
```

e) Move note3.txt out from cool_notes into parent directory

```bash
$ mv note3.txt ../
```

f) Navigate to parent directory

```bash
$ cd ../
```

g) From here list all files including hidden files

```bash
$ ls -a
```

h) Change name on note3.txt to note_home.txt

```bash
$ mv note3.txt note_home.txt
```

---

## 2. Printing, variables and manipulating text (\*)

a) Print out “hello from note_home” into bash

```bash
echo "hello from note_home"
```

b) Write this text string into note_home.txt

```bash
$ echo -e  "hello from note_home" note_home.txt
```

c) Print out the “current path is: <your current directory path>” in bash

```bash
$ echo "current path is: $PWD"
```

d) Write this string into note_home in a new line so it should contain

```bash
hello from note_home
current_directory is: <your current directory path>
```
```bash
echo -e  "\ncurrent path is: $PWD\n" >> note_home.txt
```


e) Print out the content of the note_home.txt

```bash
$ cat note_home.txt
```

f) Count the number of words in this file

```bash
$ wc -w  note_home.txt
```

g) Count the number of lines in this file

```bash
$ wc -l  note_home.txt
```

h) Count the number of files in cool_notes

```bash
$ ls cool_notes | wc -l
```

i) Check the disk usage in your directory and make the format human readable

```bash
$ du -sh ../
```

---

## 3. Pokeventure (\*)

a) Create a folder called data with a subfolder called pokemons
```bash
$ mkdir -p data/pokemons
```


b) Create a file called pokemon_list.txt

```bash
touch pokemon_list 
```

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

```bash
$ echo -e  "pikachu\ncharizard\nmewtwo\nchandelure\nlopunny\nvanillish\ngyarados" > pokemon_list.txt
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

```bash
while read -r line
do
    echo -e "pokemon: $line"
done < pokemon_list.txt
```

e) Now test out the following api manually in your browser [https://pokeapi.co/api/v2/pokemon-species/voltorb](https://pokeapi.co/api/v2/pokemon-species/voltorb)

```bash
CTRL + Click the link 
```

f) Now test it out using bash, and see that it prints out the same results

```bash
curl https://pokeapi.co/api/v2/pokemon-species/voltorb
```

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

```bash
while read -r line
do
     curl https://pokeapi.co/api/v2/pokemon-species/$line > "${line}.json"
     sleep 2
done < pokemon_list.txt
```

h) Remove all files ending with .json using one command

```bash
$ rm -r *.json
```

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

```bash

$ nano download_pokemons.sh


#!/bin/bash

cd pokemons

while read -r line
do
     echo -e "Downloading data for pokemon $line"
     curl https://pokeapi.co/api/v2/pokemon-species/$line > "${line}.json"
     sleep 2
     echo -e "Download complete for pokemon $line"
     sleep 1
done < pokemon_list.txt

CTRL + X, Y, ENTER

$ chmod +x download_pokemons.sh

$ ./download_pokemons.sh

```

---

## X. Commands glossary (\*)

Fill in this table. You can do this in any application, it might be too hardcore to do this with terminal only. Tips you can use man command to check documentation. Also try out the different commands to see them in action.

Absolutely, here's your updated table:

| Command | What does the command do | Some useful options | Options Explained |
| ------- | ------------------------ | ------------------- | ----------------- |
| cd      | Change directory | `-P`, `-L` | `-P`: Follows the physical directory structure without following symbolic links. `-L`: Follows symbolic links. |
| ls      | List directory contents | `-l`, `-a`, `-h` | `-l`: Provides a long listing format. `-a`: Shows all files, including hidden ones. `-h`: Displays file size in human-readable format. |
| touch   | Creates or updates a file | | |
| wc      | Print newline, word, and byte counts for a file | `-l`, `-w`, `-c` | `-l`: Counts the number of lines. `-w`: Counts the number of words. `-c`: Counts the number of bytes. |
| grep    | Search text using patterns | `-i`, `-r`, `-v` | `-i`: Makes the search case-insensitive. `-r`: Makes grep search recursively. `-v`: Inverts the search, showing lines that do not match. |
| mkdir   | Make directories | `-p`, `-v` | `-p`: Creates necessary parent directories. `-v`: Makes mkdir verbose, i.e., tells the user every directory it creates. |
| mv      | Move (rename) files | `-i`, `-n`, `-v` | `-i`: Prompts before overwriting files. `-n`: Does not overwrite an existing file. `-v`: Prints the name of each file before moving. |
| rm      | Remove files or directories | `-i`, `-r`, `-f` | `-i`: Prompts before every removal. `-r`: Removes directories and their contents recursively. `-f`: Forces removal without prompting. |
| rmdir   | Remove empty directories | | |
| ssh     | Remote system login/execute commands | `-p`, `-i` | `-p`: Specifies the port to connect to on the remote host. `-i`: Selects a file from which the identity for RSA or DSA authentication is read. |
| curl    | Transfer data from or to a server | `-I`, `-X`, `-d` | `-I`: Fetches only the HTTP-header. `-X`: Specifies a custom request method. `-d`: Sends the specified data in a POST request to the HTTP server. |
| sudo    | Execute command as another user (superuser by default) | | |
| apt-get | APT package handling utility (Debian based systems) | `update`, `install`, `remove` | `update`: Updates the list of available packages. `install`: Installs a new package. `remove`: Removes a package. |
| ps      | Report a snapshot of the current processes | `-e`, `-f`, `-l` | `-e`: Selects all processes. `-f`: Provides a full-format listing. `-l`: Provides a long-format listing. |
| cp      | Copy files and directories | `-i`, `-n`, `-v` | `-i`: Prompts before overwrite. `-n`: Does not overwrite an existing file. `-v`: Prints the name of each file before copying. |
| less    | Page through text one screenful at a time | | |
| top     | Display Linux processes | | |
| head    | Output the first part of files | `-n` | `-n`: Outputs the first 'n' lines. |
| echo    | Display a line of text | `-n`, `-e` | `-n`: Does not output the trailing newline. `-e`: Enables interpretation of backslash escapes. |
| cat     | Concatenate files and print on the standard output | `-n`, `-b` | `-n`: Numbers all output lines. `-b`: Numbers nonempty output lines. |
| chmod   | Change file mode bits | `-R`, `-f`, `-v` | `-R`: Changes files and directories recursively. `-f`: Suppresses most error messages. `-v`: Outputs a diagnostic for every file processed. |
| chown   | Change file owner and group | `-R`, `-f`, `-v` | `-R`: Changes files and directories recursively. `-f`: Suppresses most error messages. `-v`: Outputs a diagnostic for every file processed. |
| kill    | Send a signal to a process | `-l`, `-s` | `-l`: Lists all signal names. `-s`: Sends a specific signal. |
| wget    | Network downloader | `-q`, `-O`, `-c` | `-q`: Turns off wget's output. `-O`: Directs the output to the file specified. `-c`: Continues getting a partially downloaded file. |
| pwd     | Print name of current/working directory | | |
| >>      | Append output to a file | | |
| >       | Redirect output to a file | | |
| *      | Wildcard | | Any number of characters. Example *.json selects all files ending in json |
| ./      | Execute a file in the current directory | | |
| diff    | Compare files line by line | | |
| find    | Search for files in a directory hierarchy | | |
