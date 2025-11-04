# Commands
- ##### pwd
	To check current directory

- #### cd
	Change directory
	cd .. => go to previous directory
	cd../.. => get to grand parent folder
	cd --path-- -> go to this directory

- #### ls
	list all contents of directory
	- ls  -> list current directory
	- ls dir -> list all directories in dir
	- -l  -> details of all directories
	- -R -> details of all sub directories
	- -lt -> sorts based on last modified
	- -lr -> sorts based on first modified
	- -la -> gets hidden files
	- ##### grep
		- ls -lR | grep .json  
		  -lR flag recurcively checks, grep searches for all .json files
		- ls *.json    -> all .json files in current directory
	- * => wildcard  => ls * voo *  searches for all files with voo

- #### mkdir
	create a new folder
	- mkdir dir_name
	- mkdir -p dir_name/subdir => recursively create directory

- #### touch 
	create new file

- #### cat
	view the file contents in the command prompt
	- cat filename
	- cat > filename   -> overwrite file
	- cat >> filename   -> append to file

- #### vi
	edit the file in command prompt

- #### mv
	mv files from one folder to another
	    mv _-filepath_  _-folderpath_
	rename a file
		mv -filepath/oldname -filepath/newname

- #### cp
    copy files/folders from one folder to another
		cp _-filepath_  _-folderpath_
	when copying folders you need to add a flag -r
		cp -r _-folder1path_  _-folder2path_

- ### rm
	remove file
		rm path

- ### pipe or |
	op1 | op2
	- whatever ouput comes from op1 flows to op2
	

- ### chmod
	change permissions for files/folders
	-R flag required for folders
	- chmod ugo+-rwx { filepath / -R folderpath}
	- u/g/o => whom to change permissions for user{owns file}, group{members of the user's group} or others.
	-  +/-  => add or remove permissions
	- r/w/x => read, write, execute permissions
	-  permission notation => -r-xrw-r-x => first 3 spaces for user, ther grp and then others. - indicates no permission
	- another way is by using numbers. => r=1,w=2,x=4 => differnt permissions will have differnt numeric values.
	  752 => rwxr-x-w-
	  chmod 752 dir_name

- ### head tail
	- head file => get first 10 lines
	- tail file => get first 10 lines
	- head -20 file => get first 10 lines

- ### wc
	- wc filename -> returns {#lines ,#words ,#chars}

- ### grep
	returns lines of occurances of a particular word/regex
	- grep "word" filename
	- grep -c "word" filename => returns  number of lines
	- grep -h "word" filename => returns all the lines
	- grep -hi "word" filename => i ignores all the cases
	- -n => returns line numbers as well
	- -w => matches the whole word=> ignores occurances where word appears in between
	- -o => returns only matched parts
	- -v => select everything apart from word
	- -A 5 => get 5 lines before selected line
	- -B 5 => get 5 lines after selected line
	- -C 5 => get 5 lines before and after selected line

- ### sed
	allows to substitute words in file


- ### awk
	is a scripting language in itself. Can be used to perform multiple actions. 
	Similar to SQL for text files.


# bash script

```bash

echo hello world
wc index.js
```