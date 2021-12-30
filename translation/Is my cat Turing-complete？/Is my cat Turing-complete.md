Is my cat Turing-complete?
==========================

[November 24, 2021December 13, 2021](https://belaycpp.com/2021/11/24/is-my-cat-turing-complete/)[Chloé \`Senua\` Lourseyre](https://belaycpp.com/author/senuachloe/)

**Author:** Chloé Lourseyre  
**Editor:** Peter Fordham

_This article is an adaptation of a Lightning Talk I gave at CppCon2021. A link to the video will be given here as soon as it’s available._

I’ll touch on a lighter subject this week, nonetheless quite important: is my cat Turing-complete?

Meet Peluche
------------

Peluche (meaning “plush” in French) is a smooth cat that somehow lives in my house.

![](https://belaycpp.files.wordpress.com/2021/11/112314403092001-1.png?w=1024)

She will be our test subject today.

Is Peluche Turing-complete?
---------------------------

### What is Turing-completeness

Turing-completeness is the notion that if a device can emulate a Turing machine, then it can perform any kind of computation[1](#notes).

It means that any machine that implements the eight following instructions is a computer (and can thus execute any kind of computation):

*   `.` and `,`: Inputting and outputting a value
*   `+` and `-`: Increase and decrease the value contained in a memory cell[2](#notes).
*   `>` and `<`: Shift the current memory tape left or right.
*   `[` and `]`: Performing loops.

So, if Peluche can perform these eight instructions, we can consider her Turing-complete.

### Proof of the Turing-completeness

#### Input and output

First, I tried to poke Peluche if I could get a reaction.

![](https://belaycpp.files.wordpress.com/2021/11/112314403092003-copie-1.png?w=1024)

![](https://belaycpp.files.wordpress.com/2021/11/112314403092004-copie-1.png?w=1024)

She looked at me, then just turned around.

![](https://belaycpp.files.wordpress.com/2021/11/112314403092005.png?w=1024)

![](https://belaycpp.files.wordpress.com/2021/11/112314403092006.png?w=1024)

So here it is: I poked her, and I got a reaction. So she can process inputs and give outputs.

Input/output confirmed!

#### Increase and decrease memory value

The other day, I came back from work to this:

![](https://belaycpp.files.wordpress.com/2021/11/112314403092008.png?w=1024)

Kibbles everywhere…

But then I took a closer look and realized that the slabs could be numbered, like this:

![](https://belaycpp.files.wordpress.com/2021/11/112314403092010.png?w=1024)

This looks pretty much like a memory tape to me! Since she can spill kibbles on the tiles and then eat them directly from the floor, she can increase and decrease the values contained in a given memory cell.

Increase/decrease confirmed!

#### Shift the current memory cell left or right

Another time, I was doing the dishes and _inadvertently_ spilled some water on Peluche. She began to run everywhere around the kitchen, making a huge mess.

![](https://belaycpp.files.wordpress.com/2021/11/112314403092012-copie.png?w=1024)

If you look close (at the tip of the red arrow), you may notice that while making this mess, she displaced her bowl.

Displacing her bowl means she will spill her kibbles in another tile. This counts as shifting the memory head to edit another memory cell.

Shift of the memory tape confirmed!

#### Perform loops

So, after this mess, I (obviously) had to clean up.

No more than five minutes later, I went back to the kitchen to this:

![](https://belaycpp.files.wordpress.com/2021/11/112314403092015.png?w=1024)

Yeah… she can _DEFINITELY_ perform loops…

Loops confirmed!

We have just proven that Peluche is, indeed, Turing-complete. So now, how can we use her to perform high-performance computations?

What to do with her?
--------------------

Now that I’ve proven that Peluche is Turing-complete, I can literally do anything with her!

Thus, I tried to give her simple code to execute[3](#notes):

```
😾😾😾😾😾😾😾😾
😿
🐈😾
🐈😾😾
🐈😾😾😾
🐈😾😾😾😾
🐈😾😾😾😾😾
🐈😾😾😾😾😾😾
🐈😾😾😾😾😾😾😾
🐈😾😾😾😾😾😾😾😾
🐈😾😾😾😾😾😾😾😾😾
🐈😾😾😾😾😾😾😾😾😾😾
🐈😾😾😾😾😾😾😾😾😾😾😾
🐈😾😾😾😾😾😾😾😾😾😾😾😾
🐈😾😾😾😾😾😾😾😾😾😾😾😾😾
🐈😾😾😾😾😾😾😾😾😾😾😾😾😾😾
🐈😾😾😾😾😾😾😾😾😾😾😾😾😾😾😾
🐈😾😾😾😾😾😾😾😾😾😾😾😾😾😾😾😾
😻😻😻😻😻😻😻😻😻😻😻😻😻😻😻😻🐾
😸
🐈🐈🐈🐈🐈🐈🐈🐈🐈🙀😻😻😻😻😻😻😻😻😻
🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐾🐾🐾🙀😾😾😾😻😻😻😻😻😻😻😻😻😻😻😻😻
🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐾🐾🐾🐾🙀😾😾😾😾😻😻😻😻😻😻😻😻😻😻😻😻😻😻
🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐾🐾🐾🐾🙀😾😾😾😾😻😻😻😻😻😻😻😻😻😻😻😻😻😻
🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐾🙀😾😻😻😻😻😻😻😻😻😻😻😻😻😻😻
🐈🐈🐈🐈🙀😻😻😻😻
🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐾🙀😾😻😻😻😻😻😻😻😻😻😻😻😻😻😻😻
🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐾🙀😾😻😻😻😻😻😻😻😻😻😻😻😻😻😻
🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈😾😾🙀🐾🐾😻😻😻😻😻😻😻😻😻😻😻😻😻😻
🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐾🐾🐾🐾🙀😾😾😾😾😻😻😻😻😻😻😻😻😻😻😻😻😻😻
🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐈🐾🐾🐾🐾🙀😾😾😾😾😻😻😻😻😻😻😻😻😻😻😻😻😻
🐈🐈🐈🐈🐈🐈🐾🐾🙀😾😾
😻😻😻😻😻😻🙀
```

The result was final: she wouldn’t do a thing.

Though they can, maybe cats are not designed to execute code after all?

About “cat-computing”
---------------------

Jokes aside, _cat-computing_ is the name I give to this generalized practice. In my experience, it happens quite often that when someone discovers a new feature of a language, they begin to use it everywhere, just because they can and they want to.

However, just like you can execute code using a cat[4](#note) but shouldn’t, it’s not because you _can_ use a feature that you _should_.

> They were too busy wondering if they could to think about whether they should.
> 
> Dr Ian Malcolm, Jurassic Park

Wrapping up
-----------

Cat-computing seems to be a rookie mistake (and it is), but even the most experienced developers sometimes make rookie mistakes (and there’s no shame in that).

Every three years, a new version of C++ is published. Every time, it makes me want to use the new features in every possible situation. Though this is a good opportunity to build some experience around that (one of the best ways to avoid misuses of a feature is to perform these misuses once, in my opinion), this is also favorable ground for acquiring bad practices.

Always ask yourself if a feature is necessary[5](#notes) before using it, or else you may do cat-computing.

Also, cat-computing is animal abuse, so don’t do it😠.

Thanks for reading and see you next week!

(_No cats were harmed during the writing of this article, but one was gently poked._)

**Author:** Chloé Lourseyre  
**Editor:** Peter Fordham

![](https://belaycpp.files.wordpress.com/2021/11/112314403092020.png?w=1024)

Addendum
--------

### Notes

1.  This is a simplified definition, very inaccurate but accurate enough for this example. If you want the real definition, go there: [Turing completeness – Wikipedia](https://en.wikipedia.org/wiki/Turing_completeness)
2.  I did not state it explicitly, but a Turing machine has a “memory tape” with “memory cells” on it. The machine is always pointing to a memory cell, which is the mentioned “current” memory cell.
3.  You may not be able to read this sample of code — this is a fancy new language I designed called “braincat”.
4.  Actually, you can’t execute code using a cat, I know, but it’s for the sake of the metaphor that I assume you can.
5.  Of course, necessity occurs when there is a known benefit to the feature. I’m not talking about “absolute necessity” but about “practical necessity”.
