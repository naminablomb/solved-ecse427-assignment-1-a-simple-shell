Download Link: https://assignmentchef.com/product/solved-ecse427-assignment-1-a-simple-shell
<br>
In this assignment you are required to create a C program that implements a shell interface that accepts user commands and executes each command in a separate process. Your shell program provides a command prompt, where the user inputs a line of command. The shell is responsible for executing the command. The shell program assumes that the first string of the line gives the name of the executable file. The remaining strings in the line are considered as arguments for the command. Consider the following example.

<strong>sh &gt; cat prog.c </strong>

The <strong>cat </strong>is the command that is executed with <strong>prog.c</strong> as its argument. Using this command, the user displays the contents of the file <strong>prog.c</strong> on the display terminal. If the file <strong>prog.c</strong> is not present, the <strong>cat</strong> program displays an appropriate error message. The shell is not responsible for such error checking. In this case, the shell is relying on <strong>cat</strong> to report the error.

One technique for implementing a shell interface is to have the parent process first read what the user enters on the command line (i.e., <strong>cat prog.c</strong>), and then create a separate child process that performs the command. Unless specified otherwise, the parent process waits for the child to exit before continuing. However, UNIX shells typically also allow the child process to run in the background – or concurrently – as well by specifying the ampersand (<strong>&amp;</strong>) at the end of the command. By re-entering the above command as follows the parent and child processes can run concurrently.  <strong>sh &gt; cat prog.c &amp;</strong>

Remember that parent is the shell process and child is the process that is running <strong>cat</strong>. Therefore, when the parent and child run concurrently because the command line ends with an &amp;, we have the shell running before the cat completes. So, the shell can take the next input command from the user while <strong>cat</strong> is still running. As discussed in the lectures, the child process is created using the <strong>fork()</strong> system call and the user’s command is executed by using one of the system calls in the <strong>exec()</strong> family (see <strong>man exec</strong> for more information).

<h1>Simple Shell</h1>

A C program that provides the basic operations of a command line shell is supplied below for illustration purposes. This program is composed of two functions: <strong>main()</strong> and  <strong>getcmd().</strong> The <strong>getcmd()</strong> function reads in the user’s next command, and then parses it into separate tokens that are used to fill the argument vector for the command to be executed. If the command is to be run in the background, it will end with ‘<strong>&amp;</strong>’, and <strong>getcmd()</strong> will update the background parameter so the <strong>main()</strong> function can act accordingly. The program terminates when the user <strong>enters &lt;Control&gt;&lt;D&gt; </strong>because <strong>getcmd()</strong> invokes <strong>exit().</strong>

The <strong>main()</strong> calls <strong>getcmd()</strong>, which waits for the user to enter a command. The contents of the command entered by the user are loaded into the <strong>args</strong> array. For example, if the user enters <strong>ls –l</strong> at the command prompt, <strong>args[0]</strong> is set equal to the string “<strong>ls</strong>” and <strong>args[1]</strong> is set to the string to “<strong>– l</strong>”. (By “string,” we mean a null-terminated C-style string variable.)

This programming assignment is organized into three parts: (1) creating the child process and executing the command in the child, (2) signal handling feature, and (3) additional features.




<h1>Creating a Child Process</h1>

The first part of this programming assignment is to modify the <strong>main()</strong> function in the figure below so that upon returning from <strong>getcmd()</strong> a child process is forked and it executes the command specified by the user.

<strong> </strong>

<strong>// DISCLAIMER: This code is given for illustration purposes only. It can contain bugs! // You are given the permission to reuse portions of this code in your assignment. </strong>

<strong>// </strong>

<strong>#include &lt;stdio.h&gt; </strong>

<strong>#include &lt;unistd.h&gt; </strong>

<strong>#include &lt;string.h&gt; </strong>

<strong>#include &lt;stdlib.h&gt; </strong>

<strong>// </strong>

<strong>// This code is given for illustration purposes. You need not include or follow this  </strong>

<strong>// strictly. Feel free to write better or bug free code. This example code block does not </strong>

<strong>// worry about deallocating memory. You need to ensure memory is allocated and deallocated // properly so that your shell works without leaking memory. </strong>

<strong>// </strong>

<strong>int getcmd(char *prompt, char *args[], int *background) </strong>

<strong>{ </strong>

<strong>    int length, i = 0;     char *token, *loc;     char *line = NULL;     size_t linecap = 0; </strong>

<strong> </strong>

<strong>    printf(“%s”, prompt); </strong>

<strong>    length = getline(&amp;line, &amp;linecap, stdin); </strong>

<strong> </strong>

<strong>    if (length &lt;= 0) {         exit(-1); </strong>

<strong>    } </strong>

<strong> </strong>

<strong>    // Check if background is specified..     if ((loc = index(line, ‘&amp;’)) != NULL) { </strong>

<strong>        *background = 1; </strong>

<strong>        *loc = ‘ ‘; </strong>

<strong>    } else </strong>

<strong>        *background = 0; </strong>

<strong> </strong>

<strong>    while ((token = strsep(&amp;line, ” t
”)) != NULL) {         for (int j = 0; j &lt; strlen(token); j++)             if (token[j] &lt;= 32)                 token[j] = ‘ ’;         if (strlen(token) &gt; 0)             args[i++] = token; </strong>

<strong>    }  </strong>

<strong>    return i; </strong>

<strong>}  </strong>

<strong>int main(void) </strong>

<strong>{ </strong>

<strong>    char *args[20];     int bg;     while(1) {         bg = 0; </strong>

<strong>        int cnt = getcmd(“
&gt;&gt;  “, args, &amp;bg); </strong>

<strong> </strong>

<strong>        /* the steps can be..: </strong>

<ul>

 <li><strong>fork a child process using fork() </strong></li>

 <li><strong>the child process will invoke execvp() </strong></li>

 <li><strong>if background is not specified, the parent will wait,               otherwise parent starts the next command… */ </strong></li>

</ul>

<strong>    } </strong>

<strong>} </strong>

<strong> </strong>

<strong> </strong>

As noted above, the <strong>getcmd()</strong> function loads the contents of the args array with the command line given by the user. This args array will be passed to the <strong>execvp()</strong> function, which has the following interface:

<strong>execvp(char *command, char *params[]); </strong>

Where command represents the file to be executed and params store the parameters to be supplied to this command. Be sure to check the value of background to determine if the parent process should wait for the child to exit or not. You can use the <strong>waitpid()</strong> function to make the parent wait on the newly created child process. Check the man page for the actual usage of the <strong>waitpid()</strong> or similar functions that you can use.

<h1>Signal Handling Feature</h1>

The next task is to modify the above program so that it provides a <em>signal </em>handling feature. You can think of the signals as software interrupts. The signals are sent to a process because of external events such as keyboard presses (pressing the Ctrl + C key), external programs sending signals with specific values, or abnormal conditions created by the program (segmentation faults). A process is wired to act in a default way on the receipt of a given signal. You can change this default behaviour using a system call. The figure below shows how a program reacts to a delivery of a signal assuming the program is programmed to anticipate the arrival of the signal.




You need to develop a signal feature that would do the following: (a) kill a program running inside the shell when Ctrl+C (SIGINT) is pressed in the keyboard and (b) ignore the Ctrl+Z (SIGTSTP) signal. That is for SIGINT you kill the process that is running within the shell. For Ctrl+Z, you just ignore it so that your shell would not react to it using the default action.

There are two ways to setup signal handlers. We would use the legacy interface because it is easy to understand. For portable programs, the approach is not recommended because it works differently on different Unix/Linux implementations. The general approach is very simple. You create a function for handling a particular signal and hookup the function to act as the signal handler using the <strong>signal()</strong> system call. The following example program should give you the general idea.







<strong> </strong>

<strong> </strong>

<h1>Built-in Commands</h1>

A command is considered built-in, when all the functionality is completely built into the shell (i.e., without relying on an external program). The main feature of the shell discussed in the first two sections of this handout is about forking a child process to run external commands. Now, we want to implement the following built-in commands: <strong>cd</strong> (change directory), <strong>pwd</strong> (present working directory), <strong>exit</strong> (leave shell), <strong>fg</strong> (foreground a background job), and <strong>jobs</strong> (list background jobs). The <strong>cd</strong> command could be implemented using the <strong>chdir()</strong> system call, the <strong>pwd</strong> could be implemented using the <strong>getcwd()</strong> library routine, and the <strong>exit</strong> command is necessary to quit the shell. The <strong>fg</strong> command should be called with a number (e.g., <strong>fg 3</strong>) and it will pick the job numbered 3 (assuming it exists) and put it into the foreground. The command <strong>jobs</strong> should list all the jobs that are running in the background at any given time. These are jobs that are put into the background by giving the command with <strong>&amp;</strong> as the last one in the command line. Each line in the list provided by the jobs should have a number identifier that can be used by the <strong>fg</strong> command to bring the job to the foreground (as explained above).

<strong>cd</strong> command: This command takes a single argument that is string. It changes into that directory. You are basically calling the chdir() system call with the string that is passed in as the argument. You can optionally print the current directory (present working directory) if there are no arguments. You don’t need to support additional features (e.g., those found in production shells like bash).

<strong>pwd</strong> command: This command takes no argument. It prints the present working directory.

<strong>exit</strong> command: This command takes no argument. It terminates the shell. It will terminate any jobs that are running in the background before terminating the shell.

<strong>fg</strong> command: This command takes an optional integer parameter as argument. It brings the background job to the foreground. The background can contain many jobs you could have started with the <strong>&amp;</strong> at the end.

<strong>jobs</strong> command: This command takes no argument. It lists all the jobs that are running in the background.

<h1>Simple Output Redirection</h1>

The next feature to implement is the output redirection. If you type <strong>ls &gt; out.txt</strong>

The output of the <strong>ls</strong> command should be sent to the <strong>out.txt</strong> file. See the class notes on how to implement this feature.

<h1>Simple Command Piping</h1>

The last feature to implement is a simplified command piping. If you type <strong>ls | wc -l</strong>

The output of the <strong>ls</strong> command should be sent to the <strong>wc –l</strong> command. One easy way of implementing this command piping is to write the output of ls to the disk and ask the second command to read the input from the disk. However, in this assignment and in the real system, you don’t implement it through the file system. You implement command piping using an inter-process communication mechanism called pipes (anonymous). You can follow our discussion in the class and it should implement a command piping that should work with the above example.