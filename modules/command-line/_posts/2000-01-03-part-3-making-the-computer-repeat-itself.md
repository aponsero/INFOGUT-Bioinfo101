---
title: "Part 3: Making the computer repeat itself"
---

# Part 3: Making the computer repeat itself
{: .no_toc}

<div class="panel" markdown="1">
**Estimated time:** 60 to 75 minutes.<br>
**You need:** the practice terminal at [webterm.app/en/free-play](https://webterm.app/en/free-play) for the first half, and the online editor at [linuxvox.com/online-bash-compiler](https://linuxvox.com/online-bash-compiler/) for the second half.<br>
**Before you start:** this page builds on [Part 1]({{ site.baseurl }}/modules/command-line/part-1-finding-your-way-around/) and [Part 2]({{ site.baseurl }}/modules/command-line/part-2-asking-questions-of-your-files/), in particular wildcards.
</div>

**On this page**

* TOC
{:toc}

---

## How to use this page

Same as Parts 1 and 2. Grey boxes are commands to type. Predict before you press Enter.

**This page uses two different tools.** The first half continues in WebTerm. The second half moves to a different site. Section 5 explains the switch: don't skip it, or the change will be confusing.

---

## 1. The problem worth solving

You have eight files in `samples/`. If you wanted to do the same thing to each one, you could type eight commands. Tedious, but survivable.

Now imagine a 96-well plate. Or ten plates.

Nobody types 960 commands. And nobody should, because typing the same thing 960 times is exactly how a typo gets into your results on plate seven and nobody notices for a month.

**This page is about telling the computer "do this to each of them" once.**

It's the same instinct as reaching for a multichannel pipette. You already have this idea; you're just about to learn how to say it.

### Getting set up

Open [webterm.app/en/free-play](https://webterm.app/en/free-play).

If your project from Part 2 is still there, check it:

```bash
tree ~/rna_project
```

If not, paste the setup line from [Part 2, section 1]({{ site.baseurl }}/modules/command-line/part-2-asking-questions-of-your-files/#restarting-and-topping-up) and press Enter, then check again. You need less of it than in Part 2, but you do need the `samples` folder.

---

## 2. Variables: a box with a name

Before loops, one small idea.

```bash
sample="sample_01"
```

Nothing happened, visibly. You put a piece of text into a box called `sample`.

Now open the box:

```bash
echo $sample
```

The `$` means **give me what's in the box**. Without it, you'd just get the word `sample`. Try it:

```bash
echo sample
```

```bash
echo $sample
```

That difference is the whole idea. `sample` is a word. `$sample` is *whatever is currently inside* `sample`.

You can use it anywhere:

```bash
echo "Now processing $sample"
```

```bash
ls ~/rna_project/samples/${sample}_R1.fastq
```

The curly brackets in that last one just say where the name ends, so the shell doesn't go looking for a box called `sample_R1`.

### One trap

```bash
sample = "sample_01"
```

That fails. **No spaces around the `=`.** The shell reads spaces as separators, so it thinks you're trying to run a command called `sample`.

You will do this at least once. Now you'll recognise the error when you do.

---

## 3. Loops at the prompt

Now the real thing.

```bash
cd ~/rna_project/samples
ls
```

Start with the simplest possible loop:

```bash
for f in *_R1.fastq; do echo $f; done
```

Four filenames printed, one per line.

**Read it as a sentence.** *For each thing, call it `f`, in the list `*_R1.fastq`, do this: print `f`. Done.*

| Part | Means |
|---|---|
| `for f` | Call each item `f` while we work on it |
| `in *_R1.fastq` | Here's the list of things to work through |
| `do` | Here's what to do to each one |
| `done` | That's the end of the instructions |

`f` is just a name you chose. This does exactly the same thing:

```bash
for banana in *_R1.fastq; do echo $banana; done
```

It's worth running once, because it proves the name means nothing to the computer. Choose names that mean something to *you*:

```bash
for sample in *_R1.fastq; do echo $sample; done
```

### Making it say something useful

```bash
for sample in *_R1.fastq; do echo "Processing $sample"; done
```

```bash
for sample in *_R1.fastq; do echo "Now starting $sample, please wait"; done
```

### Where the list comes from

Notice that the list is a **wildcard from Part 2**. That's the connection worth seeing: you already knew how to describe a set of files. Now you can do something to each of them.

Try a different set:

```bash
for sample in control_*; do echo "Control file: $sample"; done
```

Or a list you write by hand:

```bash
for sample in sample_01 sample_02 sample_03; do echo "Processing $sample"; done
```

Both work. A wildcard finds real files; a written list is just words you typed.

**Try it:** write a loop that prints a message for every `_R2` file.

<details markdown="1">
<summary>Answer</summary>

```bash
for sample in *_R2.fastq; do echo "Reverse read: $sample"; done
```
</details>

---

## 4. Everything so far has been temporary

Look at what you just wrote. It works. Now consider:

- It's gone the moment you close this tab.
- Nobody else can see what you did.
- In three months, you won't remember it.
- If someone asks how you processed those samples, you have no answer.

That's the case for writing it down in a file: a **script**.

A script isn't a different kind of programming. It's the same commands you've been typing, saved so that they can be run again, read by someone else, and pointed at as a record of what you did.

That's the whole idea, and it's the same argument the [FAIR data module]({{ site.baseurl }}/modules/fair-data/programme/) makes about your data.

---

## 5. Switching tools

You now move to a different site for the rest of this page.

Open [linuxvox.com/online-bash-compiler](https://linuxvox.com/online-bash-compiler/).

It looks different from WebTerm, and it works differently:

- **There's no prompt.** You write in an editor on the left and press **Run Code** (or Ctrl+Enter / Cmd+Enter).
- **Output appears in panes on the right**, not underneath what you typed.
- **Your project isn't here.** This is a separate sandbox. `rna_project` doesn't exist in it.

That last point matters. It's why the scripts below use a written-out list of sample names instead of a wildcard: there are no files here to match. You haven't broken anything; the files are just somewhere else.

**Nothing is saved here either.** If you write something you want to keep, copy it into your notes before you leave.

---

## 6. Script 1: Hello

Clear the editor and type:

```bash
echo "Hello from my first script"
```

Press **Run Code**.

That's a script. Genuinely, one line, saved in an editor, run on demand. There's no threshold you have to cross to call it one.

Add a second line and run it again:

```bash
echo "Hello from my first script"
echo "It has two lines now"
```

Both ran, in order, top to bottom. That's all a script does: your commands, in the order you wrote them.

---

## 7. Script 2: Two kinds of output

Replace everything in the editor with this:

```bash
echo "This goes to the output pane"
echo "This goes to the error pane" >&2
echo "And the script carries on"
```

Run it, and look at the two panes on the right.

**Two lines went green. One went red.**

The `>&2` did that. It's the same `>` you used in Part 2 to send output into a file, just pointed at somewhere that isn't a file.

### Why two panes exist

Every command can produce two separate streams:

- **stdout**: the results. What you asked for.
- **stderr**: the complaints. Warnings and errors.

They're kept apart so you can separate them. On a real computer:

```
some_tool > results.txt
```

sends the *results* into the file, while any errors still appear on your screen, which is exactly what you want. You don't want error messages buried in the middle of your data.

The first time you see that happen it's baffling. Now it won't be.

### The exit code

Look at the **Exit Code** in the execution details.

**Zero means it finished fine. Anything else means something went wrong.**

That's the whole convention. It exists so that one script can check whether another one succeeded before carrying on. You won't use it directly here, but you'll see the number, and now you know what it's telling you.

---

## 8. Script 3: Breaking it on purpose

Replace the editor contents with this. There's a deliberate typo: leave it in.

```bash
echo "Starting the analysis"
ecoh "Processing the samples"
echo "Analysis finished"
```

**Before you run it: what do you think will happen?**

Now run it.

### What actually happened

- The output pane says **Starting the analysis** and **Analysis finished**.
- The error pane complains about `ecoh`.
- The exit code is **not zero**.

Look carefully at that first point.

**The middle line did nothing, and the script carried on anyway.**

It didn't stop. It didn't warn you in the output. It printed a cheerful "Analysis finished" and handed you a result that was missing a step.

This is the most important thing on this page. **Bash does not stop when something fails, unless you tell it to.** A script can look like it worked, say it finished, and have skipped the part that mattered.

It's the same error as the `pdw` typo in Part 1, but now it's buried in a script, where it's easy to miss.

Which means: **check the error pane even when it says it finished.** Especially when it says it finished.

### Now fix it

Correct `ecoh` to `echo` and run again.

Three green lines, empty error pane, exit code zero.

You just debugged something. That's the actual job: writing code that doesn't work yet, reading what it tells you, and fixing it. Everyone does this, constantly, forever.

---

## 9. Script 4: The loop, written down

Last one. Replace the editor contents:

```bash
# Report on each sample in the study
# Written by: your name here

for sample in sample_01 sample_02 sample_03; do
    echo "Processing $sample"
done
```

Run it.

### Three things to notice

**The `#` lines are comments.** Bash ignores them completely. They're there for humans: for you in three months, and for whoever else opens this file. A script that explains itself is the difference between a record and a mystery.

**The list is written out by hand,** because your `.fastq` files aren't in this sandbox. On a real computer this would be a wildcard, exactly like the loops you wrote in WebTerm.

**The `do` block is indented.** Bash doesn't care, but it makes the shape of the loop visible at a glance. Do it anyway.

### Make it yours

Try these, running after each change:

- Add a fourth sample to the list.
- Change the message.
- Add a line before the loop, `echo "Starting run"`, and one after it, `echo "All done"`.
- Add a variable at the top, `study="RNA pilot"`, and use `$study` inside the loop message.

<details markdown="1">
<summary>One version of all of it</summary>

```bash
# Report on each sample in the study
# Written by: your name here

study="RNA pilot"

echo "Starting run for $study"

for sample in sample_01 sample_02 sample_03 sample_04; do
    echo "Processing $sample from $study"
done

echo "All done"
```
</details>

**Copy your final script into your notes before you leave.** Nothing here is saved.

---

## Command summary

| Thing | What it does |
|---|---|
| `name="value"` | Put something in a box (no spaces around `=`) |
| `$name` | Give me what's in the box |
| `${name}_R1` | Curly brackets show where the name ends |
| `echo "text"` | Print something |
| `for x in list; do ... ; done` | Do something to each item in a list |
| `#` | A comment: ignored by bash, read by humans |
| `>&2` | Send this to the error stream instead of the output |

| Concept | Note |
|---|---|
| stdout | The results: what you asked for |
| stderr | The complaints: warnings and errors |
| Exit code 0 | Finished fine |
| Exit code not 0 | Something went wrong |
| **Bash doesn't stop on errors** | Check the error pane even when it says it finished |

---

## If something goes wrong

**`sample = "x"` fails**: remove the spaces around the `=`.

**`$sample` prints nothing**: the box is empty, or the name is misspelled. Check with `echo $sample`.

**The loop prints the wildcard instead of filenames**: nothing matched. Run `ls` with the same wildcard to check, and check you're in the right folder with `pwd`.

**Your files aren't in LinuxVox**: correct. It's a separate sandbox. Use a written-out list instead of a wildcard.

**The script says it finished but something's missing**: read the error pane. Bash carries on after a failed line.

**Everything is broken**: in WebTerm, reload and paste the setup line from Part 2. In LinuxVox, clear the editor and start again.

---

## What's next

The [Summary]({{ site.baseurl }}/modules/command-line/summary/) page recaps everything from the three parts in one table, with links to go further.
