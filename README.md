Download Link: https://assignmentchef.com/product/solved-cs107-assignment6-binary-bomb
<br>
Completing this assignment will give you:

solid practice in reading and tracing assembly code

an understanding of how data access, control structures, and function calls translate between C and asm experience reverse-engineering a compelling reason to invest in mastering the gdb debugger!

Overview

For the code-study exercises, you will examine a security vulnerability in a uncarefully-written program, exploit it using a stack bu er over ow attack, and work through how stack protector defends against such attacks. Then, it’s on to the pièce de résistance: defusing a nefarious binary bomb! This is fun stu and we hope you enjoy solving our puzzles as much as we enjoy creating them!

<h1>Getting started</h1>

Check out a copy of your cs107 repository with the command:     git clone /afs/ir/class/cs107/repos/assign6/$USER assign6

<strong>Do not start by running the bomb to “see what it will do…”</strong> You will quickly learn that “what it does” is explode &#x1f642; When started, it immediately goes into waiting for input and when you enter the wrong response, it will explode and deduct points. Thoroughly read the bomb information in this writeup and the advice page before attempting do anything with bomb!

<h1>1.    Code study: bu er over ow</h1>

The program in atm.c simulates the operation of a simpli ed ATM. The code makes a good faith attempt at keeping your money safe, but its use of the deprecated gets function introduces a signi cant security vulnerability. This vulnerability is the focus on this exercise.

Skim the code in atm.c to understand its basic operation. This ATM o ers only one option, a “fast cash” withdrawal of $200. A text le containing sunets and balances serves as the bank’s database. Your login name is your sunet and the password for everyone is their sunet reversed.

Run samples/atm_soln to try to make a withdrawal. With a paltry $107 in your account, your request is denied— sad times! Never fear, just impersonate Michael Chang with

samples/atm_soln mchang91 to get the hookup. You know his password and that guy has so

much coin, he’ll never miss it. (If you run the program again, you’ll note all balances return to their original levels. No money actually changes hands in this ATM; which is a blessing given its lack of security.) With a password scheme that lame, sure, anyone can get rich, but the more sophisticated exploit is getting the bank to hand you money, but without taking it from anyone else (and least of all from Michael!).

The program does take pains to try to maintain bank security by rejecting unknown users and invalid passwords and denying withdrawals in excess of the current balance. Yet a bit of hacking can score $200 free and clear with nary a complaint about insu           cient funds. Your CS107 knowledge of assembly and the runtime stack is nally going to pay o !

The chink in the armor of this ATM is in its use of gets to read the user’s password. We rst looked at gets in assign3 (/class/cs107/assign3/) and recently revisited it in lab7 (/class/cs107/lab7). Answer the following questions in your readme.txt le.

<ol>

 <li>It’s not just this call to gets that is dangerous, any such call is. Explain why it is</li>

</ol>

<strong>impossible</strong> to use gets safely.

The exploit you are to pull o is to login as yourself, view your account balance, and despite its lack of funds, convince the ATM to dispense $200 to you without decrementing your balance. Start your attack by studying the C code and disassembly for fast_cash to work out a precise understanding of the stack layout.

<ol>

 <li>On a piece of scratch paper, diagram the stack frame for fast_cash and work out where</li>

</ol>

the local variables, saved registers, and stack housekeeping are placed. In your readme, summarize your diagram: indicate the total size of the stack frame for fast_cash and identify the order/position of the data within the frame. What data will be overwritten if you enter a password that is one character too long? What about a password that is 30 characters too long or 60 too long?

Your tactic will be to enter a password with excess characters, but not just any old letters, instead bytes carefully chosen to overwrite the stack housekeeping in such a way to take control of the execution. Look carefully at the disassembly for main . Where is the code supposed to return after a call to fast_cash ? Where would you rather it return instead? There is a “password” you can give to fast_cash that will cause it to return to that more desirable location. Work out what this password is on paper using the stack diagram you sketched previously.

<ol>

 <li>In your readme, identify the precise length and contents of the password to enter for your</li>

</ol>

bu er over ow attack to reroute control so as to dispense money to you.

Now it’s time to put your plan into action. You are going to create a le password.txt that contains the input you worked out above and feed that le to the ATM program. The characters in your “password” are a bit odd and your regular editor may frown upon them, so we provide a tiny program that lets you specify raw bytes as an array and writes to a le. Edit the raw bytes in create_password.c , then compile and run that program to write your password.txt le.

<ol>

 <li>Verify that your password.txt works by piping it as input to the program like this:</li>

</ol>

samples/atm_soln &lt;password.txt . (You can also use sanity check to perform this test) It

should log you in, report your too-small balance, and dispense $200 in cash to you, without changing your existing balance. (No question to answer in readme.txt for this subpart, just con rm that you ran sanitycheck and successfully absconded with the cash!)

Pro-tip: If you enjoyed this little taste of security, you’ll love CS155 (http://cs155.stanford.edu)!

<h1>2.    Code study: stack protector</h1>

The samples/atm_soln was compiled from atm.c without stack protection (gcc ag fnostack-protector ); the samples/protected_atm_soln was compiled from the same atm.c le but with protection ( fstack-protector ). We rst introduced stack protector in lab3 (/class/cs107/lab3), but that was before we had the assembly know-how that we now do. Let’s dig into how this stack protection works. Answer the following questions in your readme.txt le.

<ol>

 <li>What happens when you try to exploit the protected version with your password.txt input,</li>

</ol>

e.g. samples/protected_atm_soln &lt;password.txt ?

<ol>

 <li>Compare the disassembly for both fast_cash functions (one with and one without stack</li>

</ol>

protection). The bulk of the instructions will be the same (although addresses will have shifted and operations may be slightly shu     ed) but a few additional instructions have been inserted into the function prolog and epilog in the protected version. Identify those instructions.

<ol>

 <li>Upon entry to a function, the protector code writes a value to the stack in a particular location. When leaving the function, it reads back the value from that location and veri es it was not perturbed. This value is called the canary (as in “canary in a coalmine”). How many bytes is the canary? What location on the stack is the canary written to? Why was that particular location chosen?</li>

 <li>The canary is not a static value. Instead, its value varies from run to run. Run the program under gdb and set a breakpoint within fast_cash and use your mad gdb skills to dig out the value of the canary. Run the program again and observe the canary has a di erent value. Why is important that the canary value not be predictable? (Or phrased di erently, how could you defeat stack protector if you knew the xed value of the canary?)</li>

</ol>

<h1>3.    Binary bomb</h1>

Those nefarious Cal students have broken into our myth machines and planted some mysterious executables we are calling “binary bombs.” These programs are believed to be armed and dangerous. Without the original source, we don’t have much to go on, but we have observed that the programs seem to operate in a sequence of levels. There are 4 levels in total. Each level challenges the user to enter a string. If the user enter the correct string, it defuses the level and the program proceeds on. But given the wrong input, the bomb explodes by printing an earthshattering KABOOM! and terminating. To deactivate the entire bomb, one needs to successfully defuse each of its levels.

The Cal students have littered our systems with these landmines and we need your help. Each of you is given a bomb to disable. Your mission is to apply your best asm detective skills to work out the input required to pass each level and render the entire bomb harmless.

Your bomb is given to you as an executable, i.e. as compiled object code. From the assembly, you will work backwards to construct a picture of the original C source in a process known as reverse-

engineering. Once you understand what makes your bomb “tick”, you can supply each level with the input it requires and defuse it. The levels get progressively more complex, but the expertise you gain as you move up from each level should o set this di    culty. One confounding factor is that the bomb has a hair trigger, prone to exploding at the least provocation. Each time your bomb explodes, it noti es the sta , which deducts from your score. Thus, there are consequences to detonating the bomb– you must tread carefully!

Reverse-engineering requires a mix of di erent approaches and techniques and will give you an opportunity to practice with a variety of tools. The most powerful weapon in your arsenal will be the debugger and an important learning goal of the assignment is to expand your gdb prowess.

Building a well-developed gdb repertoire can pay big dividends the rest of your career!

<h2>Bomb logistics</h2>

Our counter-intelligence e orts been able to con rm a few things about how the bombs operate:

If you start the bomb with no command-line argument, it reads input typed at the console. If you give an argument to the bomb:

the bomb will read lines from input.txt until it reaches EOF (end of le), and then switch over to reading from the console. This feature allows you to store inputs for solved levels in input.txt and avoid retyping them each time.

Explosions can be triggered when executing at the shell or within gdb. However, gdb o ers you tools you can use to intercept explosions, so your safest choice is to work under gdb and employ protective measures.

The bomb in your repository was lovingly created just for you and is unique to your id. It is said that the bomb can detect if an impostor attempts to execute your bomb and won’t play along.

The bombs are designed for the myth computers (running on the console or logged in remotely). There is a rumor that the bomb will refuse to run anywhere else.

The bombs were compiled from C code using gcc. Apparently Cal students don’t know how to edit a Make le to change the ags to achieve much obfuscation of the object code. The Cal students also weren’t aware the function names would be visible in the object code, so they didn’t take pains to disguise them. Thus, a function name of

initialize_bomb or read_five_numbers can be a clue. Similarly, they played it straight

with use of the standard C library functions, so if you encounter a call to qsort or sscanf , it is the real deal.

Direct modi cation of the binary bomb executable can change its behavior, but be forewarned that we will test your submission against your original unmodi ed binary, so while hacking the executable is great fun, it won’t be of much use as a strategy for solving the levels. (Although it can be an entertaining and educational exercise in suppressing explosions…)

There is one important restriction: Do not use brute force! You could write a program to try every possible input to nd a solution. But this is trouble for several reasons:

You lose points on every incorrect guess which explodes the bomb. A noti cation is sent on each bomb explosion. Wild guessing will saturate the network, creating ill will among other users and attracting the ire of the system administrators who have the authority to revoke your privileges because you are abusing shared resources.

We haven’t told you how long the strings are, nor have we told you what characters they can contain. Even if you made the (wrong) assumptions that they all are less than 80 characters long and only contain lowercase letters, you will have 26<sup>80 </sup>guesses for each level. Trying them all will take an eternity, and you will not have an answer before you graduate.

Part of your submission requires answering questions that show your understanding of the assembly code, which guessing will not provide. &#x1f642;

<h2>Bomb readme</h2>

The bulk of your e ort on bomb goes into defusing the levels, but we have a few follow-up questions. Answer these questions in your readme.txt le.

<ol>

 <li>What tactics did you use to suppress/avoid/disable explosions?</li>

 <li>level_1 contains an instruction of the form mov $&lt;hex&gt;,%edi . Explain how this instruction ts into the operation of level_1 . What is this hex value and for what purpose is it being moved? Why can this instruction reference %edi instead of the full %rdi register?</li>

 <li>level_2 contains a jle that is not immediately preceded by a cmp or test Explain how a branch instruction operates in such a context. Under what conditions is this particular jle branch taken?</li>

 <li>Explain how the loop in the winky function of level_3 is exited.</li>

 <li>The read_array function used in level_4 declares a local variable that is stored on the</li>

</ol>

stack at 0x8(%rsp) . What is the type/size of this variable? Explain how can you discern its type from following along in the assembly, even though there is no explicit type information in the assembly instructions. Within read_array there is no instruction that writes to this variable. Explain how the variable is initialized (what value it is set to and when/where does that happen?).

<ol>

 <li>Explain how the cmp function is used in level_4 . What type of data is being compared</li>

</ol>

and what ordering does it apply?

<h1>Advice/FAQ</h1>

Don’t miss out on the good stu in our companion document!

Go to advice/FAQ page (advice.html)


