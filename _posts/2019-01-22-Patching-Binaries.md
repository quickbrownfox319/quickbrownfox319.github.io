### Background
I've been playing around with IDA as an exercise for both work and leisure. IDA is a powerful disassembler by HexRays, which can basically turn any binary of almost any architecture into its assembly instructions. This makes debugging and reversing much easier, though can still be tricky to get familiar with. In this exercise, I'll show you how to reverse a simple crackme and patch it so we'll always get to the right answer with IDA. The binary I'm using is "Easy_ELF" from [reversing.kr](http://reversing.kr).


### Reconnaisance
As how all good reversing starts, we need to perform a bit of recon to get an idea of what we're dealing with. Let's try running it.

![Running the binary](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/easy-elf-run.png)

When we typed in "test", it produced a message telling us it was wrong. Clearly, this is looking for a password of some sort. Checking the file type, we get the output:

![file command](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/file-easy-elf.png)


This tells us some important information such whether it's 32-bit or 64-bit, the kind of architecture, and whether it's stripped among other things. Using the strings command, can see some interesting ones within the binary.

![strings on binary](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/strings-easy-elf.png)

Looks like when we enter the right password, we get "Correct!" as our output, and we have already seen that we get "Wrong" when we don't enter the right password. So our plan of attack will be to look for cross-references to these strings and see how they're being called.

### Following the Program Flow
Let's pull up IDA and get to the meat and potatoes of this. Here, we can use the default load settings, but if you have more information about your binary it is advised you give IDA as much information as possible to work with.

![Loading up IDA](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/ida-load-binary.png)

Let's revisit those strings and see if we can find any cross-references to that "Correct" result and work backwards. You can get to the strings subview by hitting `Shift+F12`, or by going to View>Open subviews>Strings.

![Strings in IDA](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/ida-strings.png)

We can follow cross-references by following the string to its location in memory and hitting `x` to get a list of all other locations referencing it. As we can see, it leads us to this auto-named function, `sub_80484F7`.

![X-refs to IDA strings](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/ida-strings-xref.png)

Since `sub_80484F7` is not a very clear name for our purposes, let's rename that to something easy like "correct_func". If we wanted to study this function a bit more, we will find out that it is simply passing "Correct!' to `write()`, which is detailed in this [man page](http://man7.org/linux/man-pages/man2/write.2.html).

![Correct function](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/ida-correct-func.png)

![Flow graph](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/ida-main-func.png)

Following this function's cross-reference, we can see that it's called by `main`. What should catch your eye is the comparator instruction `cmp eax, 1`, followed shortly by the jump-if-not-zero instruction `jnz short loc_804855B`. The jump as you will notice goes to the section that prints "Wrong" to STDOUT before returning. This is something we do not want. What we can guess is that the previous functions starting with `sub_` do some processing on the input we've provided to see if it matches what it expects. If it does, `cmp` sets the zero status flag and the `jnz` instruction will never be triggered. Otherwise, it will jump to the "Wrong" output and return. We could go and look at the function and try to reverse what it's expecting to provide the correct password, but I will leave this as an exercise to for the reader.

### Patching
Our goal now is to patch the binary so that it will always give us the result from `correct_func`. In this case, if we patch the `jnz` function so it never jumps to the "Wrong" output, we'll always continue directly to the correct output.

![IDA View](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/ida-view.png)

In this case, we can use a `NOP` instruction, or a "no-operation", to replace our `jnz`. To see where this is in the binary, we can go to our hex subview and enable synchronizing so we can more easily see which instruction corresponds to which bytes.

![Hex view before patching](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/pre-patch.png)

Here, IDA helpfully shows us that the `jnz` instruction matches the two bytes, `0x75` and `0x0C`. A `NOP` instruction in x86 is `0x90`, so let's replace those two bytes with that by right-clicking on the hex view and going to "Edit" (or use F2), and then typing in our desired bytes. When we're done, we can right click again (or F2), and "Apply Changes" to update our IDA database.

![Hex view after patching](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/post-patch.png)

As you can see, our instruction has changed from `jnz` to two `NOP` instructions! One thing to note is that IDA doesn't directly work on the binary file itself until you apply the patches. It is also important to note that you should always make a copy of the original binary before patching, and apply patches to copies of that original binary. In my case, I already made a copy, so I can go ahead and apply the patch directly to the binary by going to Edit>Patch program>Apply patches to input file.

![Patch binary menu](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/apply-patch-screen.png)

If all went well, you should see in the output window how many bytes were patched. Let's try the patched binary again and see if our patch worked.

![Running patched binary](https://github.com/quickbrownfox319/quickbrownfox319.github.io/raw/master/images/20190122/run-patched-binary.png)

It worked! As you can see, even though we haven't changed our input from "test", it gives us the correct output anyways. So there you have it, how to patch your simple crackme binary using IDA.
