## Jump to file containing error 

In this short tutorial, we will learn how to quickly jump the last file which contained errors within our log file via vim/vi editor.  
Everything in this tutorial is Ruby / Ruby on rails specific, therefore, users using a different language or framework, please be wary.  

In any case, I will try to make the tutorial as language and system independent as possible. 


###### All the commands used are MAC OS specific. so, it may vary on linux 

*Commands along with explanation:*
```shell
   vim $(tail -n 5000 log/development.log | sed -n '/Error/,/ /p' | egrep -o app/.*.(erb|html|rb|scss|css):d+ | tail -1 | awk '{split($0, a,":"); print +a[2] a[1]} )'
```

###### Breaking it down

1. Grab the last 5000 lines
2. Get whole Paragraph containg Error message  
3. Grep file path matching given pattern with line number 
4. Get last file path containg an error with line number 
5. Tail again.
6. Split the file path and line number string  
7. Generate a desired string for vim 
8. Opening up vim 

#### Get last 5000 lines 

```shell 
 tail -n 5000 log/development.log (filename) 
```

 Using the command `tail` with the flag `-n`. First and foremost, the tail command by default displays the last part of a file. In this particular case, we are using the flag 
in order to display more lines (we're using 5000). 

#### Grab whole error message logged to log file

```shell
sed -n '/Error/,/^$/p'
```

Upon grabbing the last 5000 lines, we will begin filtering our results. First stage of fitlering, is to get all the full Error messages being logged. 
Using sed, we read the standard by specifying output of the following patterns. So, in our Pattern matching following the `-n` flag, we are matching everything from an Error 
word all the way up to an empty space. Lastly, the `p` following the match patterns signifies everything matching the pattern written to the standard output. 

#### Grepping for filepaths 

```shell
 egrep -o app/.*.(erb|html|rb|scss|css):d+ 
```

Using the the commands `grep` with the flag `-E`, which sieves through the file and find files **matching only the given pattern**.
This is very rails specific. Most of the code will be in `app/` folder, therefore, we are looking for anything within this folder and a wild card tag (may it be a   
controller, model, view). 

Since we are more worried about the specific file type, we have given a `OR` regex pattern matching possible file extension along with a 
line number (d+) separated by **:**

#### Tail agian.  

```shell
 tail -n 1 
```

This is the same command as the first command. In our case, we are just getting the last file path which would be our most recent error logged. 
Additionally, if you wanted to get last `2,3,..n` filepath (to check other errors). You could easily add a `| head -1` after the `tail -n 1` and alter 
the number on the tail to desired (second last - `tail -n 2`, third last - `tail -n 3`). 


####  Split the file path and line number string 
 
```shell 
awk '{split($0, a,":"); print +a[2] a[1]} 
```

Here we are using the awk command to split the `file path` and the `line number` as separate entity to be used on vim.
A typical file path with error logged to the log file looks like: **app/views/index.html.erb:20**. Using the `split()` method provided
in awk we split on the semi-colon (:) into an array(a) to get **[app/views/index.html.erb, 20]**. Side note, the *$0* is the whole line argument.

After the split, our values will be stored in an array **a**, specified in our split method. We can go ahead and print the desired result the way we want. 

The desired result would be ` + linenumber filepath`.
Specifically, our exmaple would look like: `+20 app/views/index.html.erb`.

The reasoning behind this will be explained in the next section.

#### Opening up vim 

```shell 
vim $(...)
```

Based on the previous section, we've generated the string `+20 app/views/index.html.erb`. Using command substitution we generate that string on the fly and
print out the result to be run with the `vim` command. The +20 signifies jumping to line 20 in vim. 

``` shell 
 vim +20 app/views/index.html.erb
``` 
