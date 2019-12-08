# bash commands

## tips 

* To search command history in bash type 'Ctrl + r' 
 This will open reverse search prompt. You can type in search word and it will show commands
 If press enter and command will execute
 If press right or left arrow you can edit the command

* Ctrl + a => Return to the start of the command you’re typing
* Ctrl + e => Go to the end of the command you’re typing
* Ctrl + w => Delete the word / argument left of the cursor
* Ctrl + l => Clear the screen

## bash one liners

* Write a date every 5 seconds in a file

```
while true; do date >> /var/log/date.txt;sleep 5; done;
```
* Write a bash one-liner that writes the current date to the file ~/tmp/date.txt every 5 seconds. Create the directory if it doesn't exist yet. A new date does not overwrite the existing date in the file but appends it.

```
if [ ! -d ./tmp ]; then mkdir -p ./tmp; fi; while true; do echo $(date) >> ./tmp/date.txt; sleep 5; done;
```
* Write a bash one-liner that defines a variable counter with the initial value 0. Increment the variable every second and print out its value to the console.

```
counter=0; while true; do counter=$((counter+1)); echo "$counter"; sleep 1; done;
```

* Write a bash one-liner that defines a variable with a value between 1 and 100. In a loop, print the value of the variable if it is less than 50. Break the loop if the value is more or equal to 50 and print out the message "END: $value".

```
while true; do random=$(((RANDOM % 100) + 1)); if [ $random -le 50 ]; then echo "$random"; else echo "END: $random"; break; fi; sleep 1; done;
```