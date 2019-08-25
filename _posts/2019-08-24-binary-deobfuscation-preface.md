---
layout: post
tile: "Binary Deobfuscation: Preface"
date: 2019-08-24 16:00:06 +0000
categories: [reversing, obfuscation]
tags: [obfuscation, deobfuscation, reverse engineering]
---

<h1 id="binary-deobfuscation-preface">Binary Deobfuscation: Preface</h1>

Table of Contents
* toc
{:toc}

---

## A Short Introduction

Hello.

My name is Cal, and as you read initially, this is place for me to organize my thoughts. 

For a long time, I have been meaning to throw together some information regarding binary obfuscation. This post is going to be a preface to my journey exploring a variety of different protectors, the protections they provide (in terms of obfuscation), and how anybody with a little time and patience can overcome them.

It's my sincere hope that, by the end of my writing, both you and I may walk away with something remarkable: not just the tooling to break modern, existing binary obfuscations, but a more fundamental understanding behind the ideas that create them. The ability to manipulate obfuscated binary objects in a way not bound by time, or a specific implementation; that is what I'm hoping to forge.
{: #goals}

## Some Ground-Rules

I'm not interested in using paid, existing tools. This is something I feel alienates at-least a portion of readers, and I'm against it. So kiss your pirated version of IDA goodbye, and strap in.

Also, the contents of this series will largely revolve around samples of x64 binaries built for Windows, but the tooling may not. In the spirit of being fair to readers, and advancing my own knowledge of Linux binaries, I'll do my best to write some posts in that area as well; and mix the two when warranted.

# The Roadmap

As an initial exercise, I'd like to explore [Obfuscator-LLVM (OLLVM)][0]. This will allow us to build multiple, flexible examples of specific binary obfuscations, and understand what goes into both making and breaking them. This will also serve as a primer for [LLVM's][1] intermediate representation (IR), and how we can leverage it to better understand our programs.

Once we have the groundwork for the rest of our research put in place, we will begin analyzing larger, more developed commercial protectors. Therein, I hope to put the techniques obscuring program function into context with the aforementioned [OLLVM][0] work.

Lastly, I hope to demonstrate the results of my research by using live, in-the-wild samples of binary obfuscation. This research would mean nothing to me, should it stay within the confines of meticulously arranged examples; and it certainly wouldn't mean anything to you. Let's just hope I find a way to evade lawsuits as we go.

## Meet The Enemy

Speaking of lawsuits, below contains a list of software protectors that have caught my attention. As you may come to notice, there is quite a bit of overlap in terms of the protections they supply. This was intentional, as I hope to elaborate on certain obfuscation techniques in the context of multiple providers before focusing on one specifically.

### Commercial Protectors

* Obfuscator-LLVM (OLLVM) (*) - <https://github.com/obfuscator-llvm/obfuscator/wiki>
* The Tigress C Diversifier/Obfuscator (*) - <http://tigress.cs.arizona.edu/index.html>
* ReWolf's Virtualizer (*) - <https://github.com/rwfpl/rewolf-x86-virtualizer>
* ASProtect - <http://www.aspack.com/asprotect64.html>
* Obsidium - <https://www.obsidium.de/show/details/en>
* PELock - <https://www.pelock.com/products/pelock>
* Enigma Protector - <https://enigmaprotector.com/>
* CodeVirtualizer - <https://www.oreans.com/codevirtualizer.php>
* Themida - <https://www.oreans.com/themida.php>
* VMProtect - <https://vmpsoft.com/>
* GuardIT - <https://www.arxan.com/application-protection/desktop-server>

<p class="muted">(*) Non-commercial applications</p>

Now, this is a list of (primarily) commercial protectors. It goes without saying, but it's important to note: there are other examples of obfuscation out in the world of binaries, some of which have never been processed by commercial protection apps. Don't be mistaken: these examples are not out of the scope of our research by any means. Below I have listed some of such examples.

### The Real Enemy

* ZeusVM - <https://en.wikipedia.org/wiki/Zeus_(malware)>
* FinSpy VM - <https://en.wikipedia.org/wiki/FinFisher>
* Nymaim - <https://www.cyber.nj.gov/threat-profiles/trojan-variants/nymaim>
* Smoke Loader - <https://www.cyber.nj.gov/threat-profiles/trojan-variants/smoke-loader>
* Swizzor - <https://en.wikipedia.org/wiki/Swizzor>
* APT10 ANEL - <https://en.wikipedia.org/wiki/Red_Apollo>
* xTunnel - <https://en.wikipedia.org/wiki/Fancy_Bear>
* Uroburos/Turla - <https://en.wikipedia.org/wiki/Turla_(malware)>

When people mention binary obfuscation, malware is a very easy conclusion to arrive at. There is another side to that coin, though, and our list wouldn't be complete without it. Below are the examples of 'legitimate' obfuscation I've collected. These are companies looking to obscure the inner workings of their code for reasons that range from monetary incentive, to consumer protection (or so they say).

### 'Legitimate' Obfuscation

* Skype - <https://www.skype.com/en/>
* Spotify (See also [Widevine][5]) - <https://www.spotify.com/us/>
* Adobe Photoshop - <https://www.adobe.com/products/photoshop.html>
* BattlEye - <https://www.battleye.com/>
* EasyAntiCheat - <https://www.easy.ac/en-us/>
* Windows PatchGuard/Kernel Patch Protection - <https://en.wikipedia.org/wiki/Kernel_Patch_Protection>

Quick note: some of the above applications have been protected via entries in our [commercial protectors](#commercial-protectors) list. In the event that the obfuscation implemented by these applications too closely resembles output from our commercial protectors, it will be stated, and subsequently omitted from future posts.

## Techniques, Topics, and Prevalence

In this section, I am going to outline the obfuscation techniques used by the above applications, and arrange a table to represent the prevalence of said techniques.

#### Instruction Substitution

Also known as instruction mutation,

> "... this obfuscation technique simply consists in replacing standard binary operators (like addition, subtraction or boolean operators) by functionally equivalent, but more complicated sequences of instructions" [^cite12]

While I believe OLLVM's description of this technique gets the point across, I would like to generalize it a bit. However this technique is implemented, it carries the primary objective of changing the content of the code, while retaining the original semantics. This technique isn't limited to numerical operations. As long as the result of the transformation is functionally equivalent to the original, the implementation is entirely up to the creator.

Let's look at some examples of how this might work.

```x86asm
mov eax, 1337h
```

The above assembly snippet might be transformed into something like the following:

```x86asm
add eax, 266Ch
shr eax, 1h
inc eax
```

While the two snippets may look different, they are functionally equivalent. Below I have included the various phases of an example function as it transforms under OLLVM's instruction substitution pass.

<div class="img-cont">
	<img src="/assets/example_function_c_source.png" alt="example function c source code">
	<p>Original C Source Code</p>
</div>

<div class="img-cont">
	<img src="/assets/example_function_disasm.png" alt="example function disassembly">
	<p>Clean Function Disassembly</p>
</div>

<div class="img-cont">
	<img src="/assets/example_function_mutated_disasm.png" alt="example function obfuscated disassembly">
	<p>Obfuscated Function Disassembly</p>
</div>

#### Control Flow Flattening

Also known as CFG flattening or CFF,

> The purpose behind control flow flattening is "to completely flatten the control flow graph of a program" [^cite0]

The [control flow graph](https://en.wikipedia.org/wiki/Control-flow_graph) here refers to the various graphs generated by analysis tools which show a programs layout in the form of [basic blocks](https://en.wikipedia.org/wiki/Basic_block).

Further,
> "In a control-flow graph each node in the graph represents a basic block, i.e. a straight-line piece of code without any jumps or jump targets; jump targets start a block, and jumps end a block. Directed edges are used to represent jumps in the control flow. There are, in most presentations, two specially designated blocks: the entry block, through which control enters into the flow graph, and the exit block, through which all control flow leaves." [^cite1]

<div class="img-cont">
	<img src="/assets/dummy_program_source.png" alt="Dummy test program c source code">
	<p>The original source code</p>
</div>

<div class="img-cont">
	<img src="/assets/dummy_program_asm.png" alt="Dummy test program assembly code">
	<p>The disassembly for our test function</p>
</div>

<div class="img-cont">
	<img src="/assets/ollvm_control_flow_flattening_pass.png" alt="Test function after OLLVM obfuscation">
	<p>The disassembly for our test function after being ran through one of OLLVM's control flow flattening obfuscation passes</p>
</div>

#### Indirect Branches

An indirect branch, 

> "... rather than specifying the address of the next instruction to execute, as in a direct branch, the argument specifies where the address is located" [^cite2]

An easy way to think of this is an instruction sequence like the following:
```x86asm
mov eax, function_address
...
jmp eax
```
Here is an example of an indirect branch, taken out of one of the above programs. This program is riddled with branches that look just like this; and at the time, IDA was not able to follow them.

![indirect branch ida](/assets/ida-indirect-jump.png)

#### Opaque Predicates

> "an opaque predicate is a predicate—an expression that evaluates to either "true" or "false"—for which the outcome is known by the programmer ... but which, for a variety of reasons, still needs to be evaluated at run time" [^cite3]

Opaque predicates are a particularly nasty form of control flow obfuscation. They come in all shapes and sizes, and determining the opaque-*ness* of a predicate can sometimes prove to be quite a challenge.

To expand on the above explanation, in the context of an obfuscated binary this "true" or "false" expression is evaluated in the form of a conditional branch. Below I have included a small example of an *opaque* predicate.

```x86asm
mov eax, 4h
...
cmp eax, 4h
je label_1
mov eax, 224h ; * this will never execute
jmp label_187 ; *
label_1:
mov eax, 22h
...
```

Now, as you may expect, this is not a realistic example of an in-the-wild opaque predicate. However, this example does illustrate the idea of using conditional branches to obfuscate the control flow of a program. And while this example is clearly quite simple, most implementations of the technique are not. For instance, the form of opaque predicate above is sometimes referred to as an *invariant* opaque predicate. [^cite15] There are also *contextual* opaque predicates, whose value is not self-contained, but which relies on the value of an external variable (invariant). [^cite15] Which isn't to forget about *dynamic* opaque predicates, which are part of a grouping of predicates, arranged in such a way that all possible paths perform the same underlying functionality. [^cite15] [^cite16] This is all just the tip of the iceberg.

Opaque predicates also routinely cause problems within analysis platforms when certain assumptions are made. For instance, IDA Pro is one example of a tool that expects nice, proper code. [^cite17] Below I have included an example of what can happen when you violate the assumptions made during IDA's binary analysis.

<div class="img-cont">
	<img src="/assets/fasm_opaque_predicate.png" alt="opaque predicate assembly source code">
	<p>'Opaque' predicate assembly code</p>
</div>

<div class="img-cont">
	<img src="/assets/ida_disassembles_opaque_predicate.png" alt="IDA Pro disassembling opaque predicate">
	<p>IDA Pro continues to disassemble the code</p>
</div>

<div class="img-cont">
	<img src="/assets/fasm_opaque_predicate_confuse_analysis_2.png" alt="opaque predicate improved assembly source code">
	<p>Improved 'opaque' predicate assembly code</p>
</div>

<div class="img-cont">
	<img src="/assets/ida_confused_analysis_2.png" alt="IDA Pro confused analysis">
	<p>Confusing IDA's analysis hides proceeding instructions</p>
</div>

As you can see in the above demonstration, even simple attempts at disrupting analysis tools can greatly modify the output. Here, IDA made the assumption that the one dummy data byte in our opaque predicate was an [opcode expansion prefix](http://www.c-jump.com/CIS77/CPU/x86/lecture.html#X77_0040_opcode_sizes) forcing the instruction to require at-least one more byte. This next byte, of course, chipped into the real instruction that would be executed.

If one wasn't paying attention, you might miss that IDA signaled to this transformation in the instruction `jz short near ptr loc_40100E+1` where the `+1` at the end would tell an attentive reverse engineer that the initial branch skips one byte at it's target location. Below, I have shown a very simple fix for this transformation.

<div class="img-cont">
	<img src="/assets/ida_confused_analysis_2_fix.png" alt="IDA Pro confused analysis fix">
	<p>Simple fix for the above transformation</p>
</div>

### Function Inlining

> "In computing, inline expansion, or inlining, is a manual or compiler optimization that replaces a function call site with the body of the called function" [^cite4]
	
Function inlining is a well known compiler optimization technique. [^cite5] It shouldn't surprise you to learn, when used inappropriately, and interwoven with our other obfuscation techniques, inline expansion can become a rather effective means of obfuscation. Take the following example:

```C
int bar (int val)
{
	return val * 2;
}

int foo (int a, int b)
{
	int c = bar(b);
	return a + c;
}
```

In order to reduce overhead from that call, your compiler may perform inline expansion and allow the function `bar` to be inlined into `foo`, shown below:

```C
int foo (int a, int b)
{
	int c = b * 2;
	return a + c;
}
```

Which could, of course, further reduce by removing the dependency on variable `c`, but you get the point.

Now, imagine instead of multiplying a number by two, the function `bar` was response for logging runtime information to a file. In this hypothetical logging function, suppose we also decide to implement some menial logic regarding the type of information we're logging. If the function `foo` now were to call this logging function, even just a couple times, you can already begin to imagine how much our function would expand on the source level; not to mention the disassembly.

It's ironic: the most effective means to simplifying function inlining is also used as an obfuscation technique.

### Function Outlining
> "Outline pieces of a function into their own functions" [^cite6]

Synonymous with function splitting, the definition (and further implementation) of function outlining is broad. Some protectors will simply divvy up a function's basic blocks; which will then retain a direct one-to-one relationship within their parent function. Some other protectors create entirely new functions, utilized by multiple different callers. For instance, take the following diagram:

<div class="img-cont">
	<img src="/assets/function-splitting-tigress-asu.png" alt="function splitting asu tigress">
	<p>ASU's Tigress C Obfuscator "Function Splitting" Diagram (<a href="http://tigress.cs.arizona.edu/transformPage/docs/split/index.html" target="_blank">source</a>)</p>
</div>

The above image is to imply that the new functions, `f1` and `f2`, may contain code that isn't present within the original function `f` (such as a prologue and epilogue). This transformation would then enable other functions, which may rely on the functionality of `f1` or `f2`, to omit their own code and rely on an outlined function instead. 

Later down the road, we will investigate how exactly this feature is implemented by Tigress. In the meantime, we can make an interesting comparison to OLLVM. Unlike Tigress' implied function outlining implementation, OLLVM behaves in the exact opposite manner. OLLVM's disassembly would retain a direct, one-to-one relationship with it's parent, in the event of a splitting pass.

### Dead Code

> "...dead code is a section in the source code of a program which is executed but whose result is never used in any other computation" [^cite13]

Additionally, it may be important to note: the registers dead code uses throughout it's life cycle are sometimes referred to as *dead registers*.<sup class="fn-m">!</sup>

Here, I have included a small assembly snippet which includes only functional code.

```x86asm
mov eax, 2h
add eax, 2h
cmp eax, 4h
...
```

Below, I have reworked our above example with a few instructions that correspond to our above definition of dead code. These instructions are marked with asterisks.

```x86asm
mov ebx, 24h ; *
mov eax, 2h
add ebx, 12h ; *
add eax, 2h
mov ecx, ebx ; *
inc ecx ; *
cmp eax, 4h
...
```

Since the dead code above occupies the registers `ebx` and `ecx`, these registers can now be considered *dead*. However, that isn't to say these registers will always remain dead. The occupation of non-dead, *living* code, would indeed restore the status of `ebx` and `ecx`.

### Junk Code

Also known as unreachable code,

> "... is part of the source code of a program which can never be executed because there exists no control flow path to the code from the rest of the program" [^cite14]

One of the things the above snippet fails to recognize, is that in the event of an opaque predicate whose secondary condition never executes, there would exist a control flow path to it's location. I have created an example of this below.

```x86asm
mov eax, 4h
...
cmp eax, 4h
je label_1 		; (A)
mov ecx, 1h 		; * this will never execute
sub ebx, ecx 		; *
push ebx 		; *
call function_224	; *
label_1: 		; (B)
...
```

The instructions nested between the conditional jump (A) and our first label (B) are junk code; they'll never execute. However, this doesn't stop a control flow path from existing to their location.

### Constant Folding

In order to understand the idea of constant unfolding (seen below), you must first understand the idea of constant folding.

> "Constant folding is the process of recognizing and evaluating constant expressions at compile time rather than computing them at runtime." [^cite7]

Suppose we started with a piece of code like the following:

```C
int x = 220;
int y = (x / 2) + 1;
```

Constant folding, in this case, is just the reduction of the expression pointed to by our variable `y` at compile time:

```C
int x = 220;
int y = 111;
```

As stated above, constant folding is a form of optimization performed by compilers. Directly, this is not a technique that commonly corresponds to code obfuscation<sup class="fn-m">!</sup>; but given the context, constant folding could very much be used as a means to obscure code function; as is the case with many compiler optimizations.

### Constant Unfolding

While both constant folding and constant unfolding can be used as an obfuscation technique, constant unfolding is more commonly seen.<sup class="fn-m">!</sup>

Given the example of constant folding above, I have constructed a more intricate example of constant unfolding below. Something to keep in mind: although constant folding and unfolding seem easy to deal with, don't let this fool you. Remember, the strength usually lies in the implementation.

```C
int x = 220;
int y = (((x - 0xAA) | 0x58) ^ (((((x / 4) * 0x138759) >> 0x10) & 0x11) + 0x6)) + 0x3;
```

![constant unfolding python simplification](/assets/constant-unfolding-python-simplification.png)

So, for a minute now, imagine the above arithmetic is done in assembly, amid other operations, across a flattened function, all while being executed through a custom virtual machine.

Oh, right...

### Virtualization

> "In computing, a virtual machine (VM) is an emulation of a computer system" [^cite8]

That's It. 

The virtual machines used in code obfuscation aren't like [VMware](https://www.vmware.com/solutions/virtualization.html) or [VirtualBox](https://www.virtualbox.org/wiki/VirtualBox). They don't make use of technologies like [AMD-V](https://en.wikipedia.org/wiki/AMD-V) or [Intel VT-x](https://en.wikipedia.org/wiki/Intel_VT-x) (thankfully). Instead, the virtual machines found within obfuscated binaries are simply interpreters.<sup class="fn-m">!</sup>

The original, non-virtualized code of the underlying binary has been transformed into a custom bytecode. This custom bytecode is fed to an interpreter, whose job it is to mimic the actions of said underlying binary.

<div class="img-cont">
	<img src="/assets/codevirtualizer_vm_transformation.jpg" alt="codevirtualizer vm transformation">
	<p>Orean's CodeVirtualizer vm-based protection diagram (<a href="https://www.oreans.com/CodeVirtualizer.php" target="_blank">source</a>)</p>
</div>

Above, I have included a screenshot from the commercial protector, [CodeVirtualizer](https://www.oreans.com/CodeVirtualizer.php). As you can see, the simplistic code on the left is transformed into an entirely different instruction set on the right.

I understand the above explanation is lacking. The problem with going into detail on this obfuscation technique now is the potential for variability in it's implementation. So, let's put a pin in it. I will go into considerably more detail on this technique in the posts to come.

### Self Modification

> "In computer science, self-modifying code is code that alters its own instructions while it is executing ..." [^cite9]

There isn't any more I'd like to say about this technique now. Like the above, the implementation defines the strength of the obfuscation. 

This is a technique I am excited to explore, though, as it will involve us creating some examples to work off of.

### Jitting

Lastly, I want to briefly discuss the concept of *jitting*.

> "In computing, just-in-time (JIT) compilation (also dynamic translation or run-time compilations) is a way of executing computer code that involves compilation during execution of a program – at run time – rather than prior to execution." [^cite10]

There is a lot to be said about the potential for this technique. It's one that I believe is only used in the [Tigress C Diversifier][6]. This is also a technique we will be expanding on within examples that we create.

Interestingly, [Tigress][6] also includes the ability to implement a transformation called *JitDynamic*, which "... is similar to the Jit transformation, except the jitted code is continuously modified and updated at runtime" [^cite11]. This technique would then be better classified as self modifying code.

Now is a good time to state, I have not yet reversed all of the [above applications](#meet-the-enemy). The table below is based largely on information I have read from others; and cited accordingly. Data lacking of citations is largely speculative, or backed by research I have conducted that will not be made public. The applications, the techniques, and their ordering are all subject to change. In time, this snippet will be replaced, and citations to my own research will be used in place of the existing ones below. For now, take everything in the table below with a grain of salt, and read up on those citations (if there are any).

### Commercial Protectors

<span style="overflow-x: scroll;">

| Application               | Instruction Substitution | Control Flow Modification | Indirect Branches | Opaque Predicates | Function Inlining | Function Outlining | Dead Code | Junk Code | Constant Unfolding | Virtualization | Self Modification | Jitting |
| ------------------------- |:------------------------------------------------:|:-------------------------:|:-----------------:|:-----------------:|:-----------------:|:------------------:|:---------:|:---------:|:------------------:|:--------------:|:-----------------:|:-------:|
| Tigress                   |         ✅[^cite18]     |            ✅[^cite18]    |    ✅[^cite18]   |     ✅[^cite18]   |    ✅[^cite18]    |    ✅[^cite18]    |✅[^cite18]|✅[^cite18]|    ✅[^cite18]   |  ✅[^cite18]   |    ✅[^cite18]   |✅[^cite18]|
| Themida                   |         ✅[^cite19]     |✅<sup class="fn-m">!</sup>|✅<sup class="fn-m">!</sup>|✅<sup class="fn-m">!</sup>|         ❌<sup class="fn-m">!</sup>       |         ✅<sup class="fn-m">!</sup>       |     ✅<sup class="fn-m">!</sup>   |    ✅<sup class="fn-m">!</sup>   |         ✅<sup class="fn-m">!</sup>       |  ✅[^cite19]   |         ❌<sup class="fn-m">!</sup>       |  ❌<sup class="fn-m">!</sup>   |
| VMProtect                 |         ✅[^cite20]              |            ✅<sup class="fn-m">!</sup>            |          ✅<sup class="fn-m">!</sup>      |        ✅[^cite20]        |         ❔         |         ✅[^cite20]        |     ✅[^cite20]    |    ✅[^cite20]    |          ✅<sup class="fn-m">!</sup>       |      ✅[^cite20]       |         ❌<sup class="fn-m">!</sup>       |  ❌<sup class="fn-m">!</sup>   |
| GuardIT                   |         ❔              |             ❔             |           ❔       |         ❔        |         ❔         |          ❔        |     ❔    |     ❔    |           ❔        |      ❔        |         ❔         |  ❔    |
| PELock                    |         ✅[^cite21]              |            ✅[^cite21]             |          ✅[^cite21]       |        ✅[^cite21]        |         ❌<sup class="fn-m">!</sup>       |         ✅[^cite21]        |     ✅[^cite21]    |    ✅[^cite21]    |          ❌<sup class="fn-m">!</sup>       |      ❌<sup class="fn-m">!</sup>      |         ❌<sup class="fn-m">!</sup>       |  ❌<sup class="fn-m">!</sup>   |
| OLLVM                     |         ✅[^cite22]     |   ✅[^cite23]             |❌<sup class="fn-m">!</sup>       |✅[^cite24]       |         ❌<sup class="fn-m">!</sup>       |   ✅[^cite23]     |     ❌<sup class="fn-m">!</sup>   |✅[^cite24]|    ✅[^cite22]     |      ❌<sup class="fn-m">!</sup>      |         ❌<sup class="fn-m">!</sup>       |  ❌<sup class="fn-m">!</sup>   |
| CodeVirtualizer           |         ✅[^cite25]     |❌<sup class="fn-m">!</sup>|❌<sup class="fn-m">!</sup>      |        ❌<sup class="fn-m">!</sup>       |         ❌<sup class="fn-m">!</sup>       |         ❌<sup class="fn-m">!</sup>       |     ❌<sup class="fn-m">!</sup>   |    ❌<sup class="fn-m">!</sup>   |          ❌<sup class="fn-m">!</sup>       |   ✅[^cite25] |         ❌<sup class="fn-m">!</sup>       |  ❌<sup class="fn-m">!</sup>   |
| ReWolf's Virtualizer      |         ❌<sup class="fn-m">!</sup>             |            ❌<sup class="fn-m">!</sup>            |          ❌<sup class="fn-m">!</sup>      |        ❌<sup class="fn-m">!</sup>       |         ❌<sup class="fn-m">!</sup>       |         ❌<sup class="fn-m">!</sup>       |     ❌<sup class="fn-m">!</sup>   |    ❌<sup class="fn-m">!</sup>   |          ❌<sup class="fn-m">!</sup>       | ✅[^cite26]  |         ❌<sup class="fn-m">!</sup>       |  ❌<sup class="fn-m">!</sup>   |
| Enigma Protector          |         ✅[^cite27]              |            ❔              |          ❔       |         ❔        |         ❌<sup class="fn-m">!</sup>       |          ❔        |     ❔     |    ❔     |          ❔        |      ✅[^cite27]       |         ❌<sup class="fn-m">!</sup>       |  ❌<sup class="fn-m">!</sup>   |
| Obsidium                  |         ❌<sup class="fn-m">!</sup>             |            ❔              |          ❔       |         ❔        |         ❌<sup class="fn-m">!</sup>       |          ❔        |     ❔     |    ❔     |          ❔        |      ✅[^cite28]       |         ❌<sup class="fn-m">!</sup>       |  ❌<sup class="fn-m">!</sup>   |
| ASProtect                 |         ❌<sup class="fn-m">!</sup>             |            ❔              |          ❔       |         ✅[^cite29]       |         ❌<sup class="fn-m">!</sup>       |          ❔        |     ❌<sup class="fn-m">!</sup>    |   ✅[^cite29]     |          ❔        |      ❌<sup class="fn-m">!</sup>      |         ❌<sup class="fn-m">!</sup>       |  ❌<sup class="fn-m">!</sup>   |

</span>

### Malware Obfuscations

| Application               | Instruction Substitution | Control Flow Modification | Indirect Branches | Opaque Predicates | Function Inlining | Function Outlining | Dead Code | Junk Code | Constant Unfolding | Virtualization | Self Modification | Jitting |
| ------------------------- |:------------------------:|:-------------------------:|:-----------------:|:-----------------:|:-----------------:|:------------------:|:---------:|:---------:|:------------------:|:--------------:|:-----------------:|:-------:|
| ANEL                      | ❌<sup class="fn-m">!</sup> | ✅[^cite30] | ✅[^cite30] | ✅[^cite30] | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ✅[^cite30] | ✅[^cite30] | ✅[^cite30] | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> |
| Smoke Loader              | ❌<sup class="fn-m">!</sup> | ✅[^cite31] | ❔ | ✅[^cite31] | ❌<sup class="fn-m">!</sup> | ✅[^cite31] | ❌<sup class="fn-m">!</sup> | ❔ | ❔ | ❌<sup class="fn-m">!</sup> | ✅[^cite31] | ❌<sup class="fn-m">!</sup> |
| Nymaim                    | ❌<sup class="fn-m">!</sup> | ✅[^cite32] | ✅[^cite32] | ✅[^cite33] | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❔ | ❔ | ✅[^cite33] | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> |
| xTunnel                   | ❌<sup class="fn-m">!</sup> | ❔ | ❔ | ✅[^cite34] [^cite35] | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ✅[^cite36] | ✅[^cite34] | ❔ | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> |
| Uroburos/Turla            | ❌<sup class="fn-m">!</sup> | ❔ | ❔ | ✅[^cite37] | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❔ | ✅[^cite37] | ❔ | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> |
| FinSpy VM                 | ❌<sup class="fn-m">!</sup> | ❔ | ❌<sup class="fn-m">!</sup> | ✅[^cite39] | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ✅[^cite39] | ✅! | ✅[^cite39] | ✅[^cite38] [^cite39] | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> |
| ZeusVM                    | ❌<sup class="fn-m">!</sup> | ❔ | ❔ | ❔ | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❔ | ❔ | ❔ | ✅[^cite40] | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> |
| Swizzor                   | ❌<sup class="fn-m">!</sup> | ❔ | ❔ | ❔ | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ✅[^cite41] | ❔ | ❔ | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> |

### 'Legitimate' Obfuscations

| Application               | Instruction Substitution | Control Flow Modification | Indirect Branches | Opaque Predicates | Function Inlining | Function Outlining | Dead Code | Junk Code | Constant Unfolding | Virtualization | Self Modification | Jitting |
| ------------------------- |:------------------------:|:-------------------------:|:-----------------:|:-----------------:|:-----------------:|:------------------:|:---------:|:---------:|:------------------:|:--------------:|:-----------------:|:-------:|
| BattlEye                  | ✅<sup class="fn-m">!</sup> | ✅<sup class="fn-m">!</sup> | ✅<sup class="fn-m">!</sup> | ✅<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ✅<sup class="fn-m">!</sup> | ✅<sup class="fn-m">!</sup> | ✅<sup class="fn-m">!</sup> | ✅<sup class="fn-m">!</sup> | ✅<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> |
| EasyAntiCheat             | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ |
| Photoshop                 | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ |
| Spotify                   | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ | ❔ |
| PatchGuard                | ❌<sup class="fn-m">!</sup> | ✅[^cite42] [^cite45]| ✅[^cite43] | ❔ | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❔ | ❔ | ❔ | ❌<sup class="fn-m">!</sup> | ✅[^cite44] | ❌<sup class="fn-m">!</sup> |
| Skype                     | ❌<sup class="fn-m">!</sup> | ✅[^cite46] | ✅[^cite46] | ✅[^cite46] | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❔ | ❔ | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> | ❌<sup class="fn-m">!</sup> |

Note: A lot of the above applications utilize various forms of control flow modifications that are not control flow flattening. As such, I have generalized that category in the table above.

# A Conclusion, For Now

I have made some bold and ambitious statements in this post. Systems like [PatchGuard][2] are completely unknown to me. Malware like [Turla][3] and [Sednit][4] are state-sponsored threats with high-priority targets. Not to mention, the obfuscation employed by all of above is said to be challenging, and very difficult to work through. There is a lack of easy, straightforward solutions; and the research to engineer such tooling is generally contained within academic circles.

There's no doubt the road ahead is a long and challenging one. It will require me to learn a great deal about how these techniques are implemented, and how we can better go about circumventing them. Additionally, I am learning quickly that there are a myriad of legal obstacles in the way of my research. I want to make it clear: I don't wish to be implicated in any legal manner for the sharing of this information, nor would I like to aid in the breaking of DRM schemes, or development of malware-focussed binary protection systems. So in the future, I expect the methods I use to share this information with you will fall firmly within the bounds of what is fair, legal, and educational.

That being said, I make this promise to you now: my research **will** apply to modern binary obfuscation. I will not diverge from [the goals I stated initially](#goals).

# Additional Reading

Above I mentioned that the most notable research into binary deobfuscation is largely contained within academic circles. Usually these resources are dry, and difficult to read through. So, in the spirit of not bogging you down with boring and speculative research, below I have done my best to list resources that I found to be both practical and engaging.

- *Diablo - Deobfuscation: By Hand* (<https://diablo.elis.ugent.be/node/54>)
- *The Tigress C Diversifier/Obfuscator: Transformations* (<http://tigress.cs.arizona.edu/transformPage/index.html>)
- *Intermediate Representation - Wikipedia* (<https://en.wikipedia.org/wiki/Intermediate_representation>)
- *Usenix - Disassembling Obfuscated Binaries* (<https://www.usenix.org/legacy/publications/library/proceedings/sec04/tech/full_papers/kruegel/kruegel_html/node3.html>)
- *Optimizing Compiler (Specific Techniques) - Wikipedia* (<https://en.wikipedia.org/wiki/Optimizing_compiler#Specific_techniques>)
- *Compiler Optimizations for Reverse Engineers - Rolf Rolles, Mobius Strip Reverse Engineering* (<https://www.msreverseengineering.com/blog/2014/6/23/compiler-optimizations-for-reverse-engineers>)
- *Udupa, Sharath K., Saumya K. Debray, and Matias Madou. "Deobfuscation: Reverse engineering obfuscated code." 12th Working Conference on Reverse Engineering (WCRE'05). IEEE, 2005.* (<https://ieeexplore.ieee.org/abstract/document/1566145>)
- *Deobfuscation: recovering an OLLVM-protected program* (<https://blog.quarkslab.com/deobfuscation-recovering-an-ollvm-protected-program.html>)
- *Aspire - Publications* (<https://aspire-fp7.eu/papers>)
- *Creating Code Obfuscation Virtual Machines - RECon 2008* (<https://www.youtube.com/watch?v=d_OFrP-m2xU>)
{: #list-spaced}

# References
[^cite0]: *Control Flow Flattening (Wikipedia)* - <https://github.com/obfuscator-llvm/obfuscator/wiki/Control-Flow-Flattening>
[^cite1]: *\[PDF\] Masking wrong-successor Control Flow Errors employing data redundancy* - <https://ieeexplore.ieee.org/abstract/document/7365827>
[^cite2]: *Indirect Branch (Wikipedia)* - <https://en.wikipedia.org/wiki/Indirect_branch>
[^cite3]: *Opaque Predicate (Wikipedia)* - <https://en.wikipedia.org/wiki/Opaque_predicate>
[^cite4]: *Inline Expansion (Wikipedia)* - <https://en.wikipedia.org/wiki/Inline_expansion>
[^cite5]: *Eilam, Eldad. \*Reversing: Secrets of Reverse Engineering\*. Wiley, 2005. Print.* - <https://www.wiley.com/en-us/Reversing%3A+Secrets+of+Reverse+Engineering+-p-9780764574818>
[^cite6]: *The Tigress C Diversifier/Obfuscator: Function Splitting* - <http://tigress.cs.arizona.edu/transformPage/docs/split/index.html>
[^cite7]: *Constant Folding (Wikipedia)* - <https://en.wikipedia.org/wiki/Constant_folding>
[^cite8]: *Virtual Machine (Wikipedia)* - <https://en.wikipedia.org/wiki/Virtual_machine>
[^cite9]: *Self-Modifying Code (Wikipedia)* - <https://en.wikipedia.org/wiki/Self-modifying_code>
[^cite10]: *Just-in-time Compilation (Wikipedia)* - <https://en.wikipedia.org/wiki/Just-in-time_compilation>
[^cite11]: *The Tigress C Diversifier/Obfuscator: Dynamic Obfuscation* - <http://tigress.cs.arizona.edu/transformPage/docs/jitDynamic/index.html>
[^cite12]: *Obfuscator-LLVM: Instruction Substitution* - <https://github.com/obfuscator-llvm/obfuscator/wiki/Instructions-Substitution>
[^cite13]: *Dead Code (Wikipedia)* - <https://en.wikipedia.org/wiki/Dead_code>
[^cite14]: *Unreachable Code (Wikipedia)* - <https://en.wikipedia.org/wiki/Unreachable_code>
[^cite15]: *Thomas Ridma. "Seeing through obfuscation: interactive detection and removal of opaque predicates." In the Digital Security Group, Institute for Computing and Information Sciences. 2017.* - <https://th0mas.nl/downloads/thesis/thesis.pdf>
[^cite16]: *Palsberg, Jens, et al. "Experience with software watermarking." Proceedings 16th Annual Computer Security Applications Conference (ACSAC'00). IEEE, 2000.* - <https://ieeexplore.ieee.org/document/898885>
[^cite17]: *Hex-Rays: IDA and obfuscated code* - <https://www.hex-rays.com/products/ida/support/ppt/caro_obfuscation.ppt>
[^cite18]: *The Tigress C Diversifier/Obfuscator: Transformations* - <http://tigress.cs.arizona.edu/transformPage/index.html>
[^cite19]: *Oreans: Themida Overview* - <https://www.oreans.com/Themida.php>
[^cite20]: *VMProtect Software Protection: What is VMProtect?* - <http://vmpsoft.com/support/user-manual/introduction/what-is-vmprotect/>
[^cite21]: *PELock: Software protection system* - <https://www.pelock.com/products/pelock>
[^cite22]: *Obfuscator-LLVM: Instruction Substitution* - <https://github.com/obfuscator-llvm/obfuscator/wiki/Instructions-Substitution>
[^cite23]: *Obfuscator-LLVM: Control Flow Flattening* - <https://github.com/obfuscator-llvm/obfuscator/wiki/Control-Flow-Flattening>
[^cite24]: *Obfuscator-LLVM: Bogus Control Flow* - <https://github.com/obfuscator-llvm/obfuscator/wiki/Bogus-Control-Flow>
[^cite25]: *Oreans: CodeVirtualizer Overview* - <https://oreans.com/CodeVirtualizer.php>
[^cite26]: *\[PDF\] ReWolf's x86 Virtualizer: Documentation* - <http://rewolf.pl/stuff/x86.virt.pdf>
[^cite27]: *The Enigma Protector: About* - <https://enigmaprotector.com/en/about.html>
[^cite28]: *Obsidium: Product Information* - <https://www.obsidium.de/show/details/en>
[^cite29]: *David, Robin, Sébastien Bardin, and Jean-Yves Marion. "Targeting Infeasibility Questions on Obfuscated Codes." arXiv preprint arXiv:1612.05675 (2016).* - <https://arxiv.org/pdf/1612.05675>
[^cite30]: *Carbon Black: Defeating Compiler-Level Obfuscations Used in APT10 Malware* - <https://www.carbonblack.com/2019/02/25/defeating-compiler-level-obfuscations-used-in-apt10-malware/>
[^cite31]: *CERT.PL: Dissecting Smoke Loader* - <https://www.cert.pl/en/news/single/dissecting-smoke-loader/>
[^cite32]: *ESET: Dymaim Obfuscation Chronicles* - <https://www.welivesecurity.com/2013/08/26/nymaim-obfuscation-chronicles/>
[^cite33]: *CERT.PL: Nymaim Revisited* - <https://www.cert.pl/en/news/single/nymaim-revisited/>
[^cite34]: *\[PDF\] ESET: En Route with Sednit \(Part 2\)* - <https://www.welivesecurity.com/wp-content/uploads/2016/10/eset-sednit-part-2.pdf>
[^cite35]: *\[PDF\] Sebastien Bardin, Robin David, Jean-Yves Marion. "Deobfuscation: Semantic Analysis to the Rescue". Presentation at Virus Bulletin (2017).* - <https://www.virusbulletin.com/uploads/pdf/conference_slides/2017/Bardin-VB2017-Deobfuscation.pdf>
[^cite36]: *\[PDF\] Robin David, Sebastien Bardin. "Code Deobfuscation: Intertwining Dynamic, Static and Symbolic Approaches". Presentation at BlackHat Europe (2016).* - <https://www.robindavid.fr/publications/BHEU16_Robin_David.pdf>
[^cite37]: *\[PDF\] ESET: Diplomats in Easter Europe Bitten by a Turla Mosquito* - <https://www.eset.com/me/whitepapers/eset-turla-mosquito/>
[^cite38]: *Github: Rofl Rolles' Static Unpacker for FinSpy VM* - <https://github.com/RolfRolles/FinSpyVM>
[^cite39]: *Mobius Strip Reverse Engineering: A Walk-Through Tutorial, with Code, on Statically Unpacking the FinSpy VM (Part One, x86 Deobfuscation)* - <https://www.msreverseengineering.com/blog/2018/1/23/a-walk-through-tutorial-with-code-on-statically-unpacking-the-finspy-vm-part-one-x86-deobfuscation>
[^cite40]: *Miasm's Blog: ZeusVM Analysis* - <https://miasm.re/blog/2016/09/03/zeusvm_analysis.html>
[^cite41]: *Pierre-Marc Bureau, Joan Calvet. "Understanding Swizzor's Obfuscation Scheme". Presentation at RECon (2010).* - <https://archive.org/details/UnderstandingSwizzorsObfuscationScheme-Pierre-marcBureauAndJoanCalvet>
[^cite42]: *CSDN iiprogram's Blog: Pathguard Reloaded, A Brief Analysis of PatchGuard Version 3* - <https://blog.csdn.net/iiprogram/article/details/2456658>
[^cite43]: *Uninformed: Subverting Patchguard Version 2; Anti-Debug Code During Initialization* - <http://uninformed.org/index.cgi?v=6&a=1&p=5>
[^cite44]: *Uninformed: Subverting Patchguard Version 2; Overwriting PatchGuard Initialization Code Post Boot* - <http://uninformed.org/index.cgi?v=6&a=1&p=12>
[^cite45]: *Uninformed: Subverting Patchguard Version 2; Obfuscation of System Integrity Check Calls via Structured Exception Handling* - <http://uninformed.org/index.cgi?v=6&a=1&p=8>
[^cite46]: *\[PDF\] Philippe Biondi, Fabrice Desclaux. "Silver Needle in the Skype". Presentation at BlackHat Europe (2006).* - <https://www.blackhat.com/presentations/bh-europe-06/bh-eu-06-biondi/bh-eu-06-biondi-up.pdf>

[0]: https://github.com/obfuscator-llvm/obfuscator/wiki
[1]: https://llvm.org/
[2]: https://en.wikipedia.org/wiki/Kernel_Patch_Protection
[3]: https://en.wikipedia.org/wiki/Turla_(malware)
[4]: https://en.wikipedia.org/wiki/Fancy_Bear
[5]: https://en.wikipedia.org/wiki/Widevine
[6]: http://tigress.cs.arizona.edu/index.html
