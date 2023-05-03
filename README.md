Download Link: https://assignmentchef.com/product/solved-cs3224-homework-2-shell-implementation
<br>



<strong>Homework 2 Notes:</strong>

<strong> </strong>

This assignment does not involve modifying or using xv6 (although the code for shell.c is adapted from the xv6 shell).

You should write, compile, and test your code on OS X or Linux, rather than in QEMU. While it may be tempting to just copy xv6’s implementation, there are enough diff erences between the xv6 APIs and those in Linux/OS X that doing so would be a bad idea. You can look at how it works for inspiration though.

In this assignment, you will implement pieces of a UNIX shell, and get some familiarity with some UNIX library calls and the UNIX process model. By the end of the assignment, you will have a shell that can run complex pipelines of commands, such as:cat /usr/share/dict/words | grep cat | sed s/cat/dog/ &gt; doggerel.txt

The above pipeline takes /usr/share/dict/words (a file generally installed on UNIX systems that contains a list of English words), selects out the words containing the string “cat”, and then uses sed to replace “cat” with dog, so that, for example, “concatenate” becomes

“condogenate”. The results are output to “doggerel.txt”. (You can find detailed descriptions of each of the commands in the pipeline by consulting the manual page for the command; e.g.: “man grep” or “man sed”.)

Start by downloading the <u>shell.c skeleton</u> file attached to this homework.  You don’t have to understand how the parser works in detail, but you should have a general idea of how the flow of control works in the program. You will also see the “// your code here” comments, which is where you will implement the functionality to make the shell actually work.

Next, try to compile the source code to the shell:

$ gcc shell.c -o shell

You can then run and interact with the shell by typing ./shell :

<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="225751475062415111101016">[email protected]</a>:~$ ./shell cs3224&gt; ls exec not implemented cs3224&gt;

Note that the command prompt for our shell is set to cs3224&gt; to make it easy to tell the diff erence between it and the Linux/OS X shell. You can quit your shell by typing Control-C or Control-D.

Problem 1 – Command Execution

Implement basic command execution by filling in the code inside of the case ‘ ‘ block in the runcmd function. You will want to look at the manual page for the exec(3) function by typing “man 3 exec” (Note: throughout this course, when referring to commands that one can look up in the man pages, we will typically specify the section number in parentheses — thus, since exec is found in section 3, we will say exec(3)).

Once this is done, you should be able to use your shell to run single commands, such as







cs6233&gt; ls

cs6233&gt; grep cat /usr/share/dict/words







<strong>Hint</strong>:




<ol>

 <li>You will notice that there are many variants on exec(3). You should read through the diff erences between them, and then choose the one that allows you to run the commands above — in particular, pay attention to whether the version of exec you’re using requires you to enter in the full path to the program, or whether it will search the directories in the PATH environment variable.</li>

</ol>

Problem 2 – I/O Redirection

Now extend the shell to handle input and output redirection. Programs will be expecting their input on standard input and will write to standard output, so you will have to open the file and then replace standard input or output with that file. As before, the parser already recognizes the

‘&gt;’ and ‘&lt;‘ characters and builds a redircmd structure for you, so you just need to use the information in that redircmd to open a file and replace standard input or output with it.

<strong>Hints</strong>:

<ol>

 <li>Look at the dup2(2) and open(2) calls.</li>

 <li>The file descriptor the program is currently using for input or output is available in rcmd- &gt;fd .</li>

 <li>If you’re confused about where rcmd-&gt;fd is coming from, look at the redircmd function and remember that 0 is standard input, 1 is standard output.</li>

 <li>Be careful with the open call; in particular, make sure you read about the case when you pass the O_CREAT</li>

</ol>

When this is done, you should be able to redirect the input and output of commands:

cs6233&gt; ls &gt; a.txt cs6233&gt; sort -r &lt; a.txt

Problem 3 – Pipes

The final task is to add the ability to pipe the output of one command into the input of another. You will fill out the code for the ‘|’ case of the switch statement in runcmd to do this.

<strong>Hints</strong>:

<ol>

 <li>The parser provides the left command in pcmd-&gt;left and the right command in pcmd- &gt;right .</li>

 <li>Look at the fork(2), pipe(2), close(2), , and wait(2) calls.</li>

 <li>If your program just hangs, it may help to know that reads to pipes with no data will block until all file descriptors referencing the pipe are closed.</li>

 <li>Note that fork(2) creates an exact copy of the current process. The two process share any file descriptors that were open at the time the fork occurred. You can get a sense for this behavior by looking at a small test program like this one:</li>

</ol>

<table width="0">

 <tbody>

  <tr>

   <td width="553"> #include &lt;stdio.h&gt;#include &lt;unistd.h&gt;#include &lt;fcntl.h&gt;#include &lt;sys/stat.h&gt; int main() { int filedes; filedes = open(“myfile.txt”, O_RDWR | O_CREAT, S_IRUSR | S_IWUSR); int rv; rv = fork(); if (rv == 0) { char msg[] = “Process 1
”;printf(“Hello, I’m in the child, my process ID is %d
”,getpid()); write(filedes, msg, sizeof(msg));} else { char msg[] = “Process 2
”;printf(“This is the parent process, my process ID is %d and mychild is %d
”, getpid(), rv); write(filedes, msg, sizeof(msg));} close(filedes);}</td>

  </tr>

 </tbody>

</table>




If you put that code into a file, compile it, and then run the resulting program, you should see a result like:







<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="6e1b1d0b1c2e0d1d585c5d5d">[email protected]</a>:~$ ./a.out

This is the parent process, my process ID is 56968 and my child is 56969 Hello, I’m in the child, my process ID is 56969

<a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="5c292f392e1c3f2f6a6e6f6f">[email protected]</a>:~$ cat myfile.txt

Process 2

Process 1

You can see that both the parent and child process both got a copy of “filedes”, and that writes to it from each process went to the same underlying file.

<ol start="5">

 <li>You may find it helpful to re-read the first chapter of the xv6 book, which describes in detail how the xv6 shell works. Note that the code show there will <em>not </em>work as-is — you will have to adapt it for the Linux/OS X environment.</li>

</ol>

Once this is done, you should be able to run a full pipeline:




cs6233&gt; cat /usr/share/dict/words | grep cat | sed s/cat/dog/ &gt; doggerel.txt cs6233&gt; grep con &lt; doggerel.txt