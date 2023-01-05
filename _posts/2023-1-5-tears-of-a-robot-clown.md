---
layout: post
title:  Tears of a Robot Clown
categories: [AI]
excerpt: "Does generative AI have a sense of humor? If not, can it still tell jokes? What can this tell us about how these systems work? The second in a series of posts exploring the ramifications of generative AI."
---

(This is the second in a series of posts exploring the ramifications of generative AI.)

Not only are machine learning models a genie that can't be put back in the bottle (for better or for worse), but with the release of ChatGPT it's clear that the technology is improving faster than most people were anticipating. 

[No, we don't have "self-aware AI" yet, far from it](https://www.techdirt.com/2022/06/14/google-ai-fracas-shows-how-the-modern-ad-based-press-tends-to-devalue-the-truth/). But what we do have is truly impressive. Which makes it all the most important that we understand exactly what it is that we *do* have, what it can do, and what it can't.

I'm not going to touch on the issue of "is AI generated art merely plagiarizing elements of the training set," primarily because there's nothing I can meaningfully add to the debate. We know that these models [i]can[/i] reproduce high fidelity copies of popular low-entropy works (just ask ChatGPT to recite The Raven). At the end of the day, people who feel anxious about the future of their careers or feel violated by the use of their art in training data will insist that AI models [i]must[/i] plagiarize, while people who understand how the technology works insist that AI models don’t need to plagiarize to produce the outputs we observe.

Neither of these groups are going to change each other's minds, and this is all that I’m going to say about them.

What I will say is that in my own experience playing around with ChatGPT, I’ve been blown away by its ability to generate text content that is, as far as I can tell, entirely original. And not just original, but [i]funny[/i].

Specifically, I asked ChatGPT to write jokes. I gave it joke setups that I invented on the spot, and asked it to create the punchline, and explain the joke to me. Most of the “jokes” were, to put it bluntly, not funny. Some barely even qualified as wordplay. But there were a couple of gems that got a genuine laugh out of me. My favorites were:

Q: What is Frodo Baggins’ favorite sport?
A: Ring Toss.

Q: How many birds does it take to change a lightbulb?
A: None, they just sit in the dark and tweet about it.

Q: What is Oedipus’s favorite mixed drink?
A: A mom-osa.

Again, to the best of my knowledge, these are entirely original jokes. I searched but was unable to find any evidence of these jokes being told online. (If anyone reading this has heard any of these jokes before, or better yet, knows where I can find a written record of any of them, [please let me know.](mailto://the.solipsis.project@gmail.com))

So what gives? Can computers be funny now? If the humor wasn’t stolen from other writers, where did it come from? Presenting these jokes on their own suggests that the ChatGPT is funnier than I could ever dream of being.

But presenting these jokes on their own removes very important context that perhaps might elucidate what’s actually going on.

For starters, I edited the jokes to make them land better. The original Ring Toss joke asked what Frodo’s favorite *Olympic* sport was. I removed the world Olympic because Ring Toss is not an olympic sport. (I wonder if ChatGPT latched onto a connection between “Olympic” and “Ring” and was aiming for some double-wordplay that didn’t land.)

I also didn’t show you any of the jokes that *weren’t* funny. And there were a lot of them. More than there were funny ones. I didn’t save them, and I barely remember most of them. Here’s one I do remember, because of how profoundly *unfunny* it was.

Q: Why did Captain Picard go to space?
A: To boldly go where no one had gone before.

I mean… I get it. I get the reference. But that’s all it is. A reference. ChatGPT explained the joke as “humorously imagining Picard taking his catch-phrase literally.” But… that’s what he does every episode. There’s no joke here.

The others followed a similar pattern: the punchline was always thematically cohesive with the setup. Usually they even had some form of double meaning or wordplay, which it then explained. The problem is that wordplay is not necessarily funny. Even in the jokes that *were* funny, the generated explanation merely explained the wordplay, not why the joke was funny.

So if most of the responses weren’t funny, but they did all have some kind of wordplay, then it turns out what I’d created wasn’t a joke engine, but a word-association engine. And while humor is subjective and creative, word-association is a tractable problem. In fact, it’s a tractable problem that language models are quite good at, for understandable reasons.

When I asked ChatGPT to come up with Oedipus’s favorite mixed drink, it needed to generate a response that was associated with both Oedipus, and mixed drinks. And it turns out that the phrase “mom-osa” is all over the Internet, meaning a mimosa or mimosa-variant made by or for moms. ChatGPT didn’t invent the term… but it did connect it to Oedipus. It wasn’t trying to be funny, it just identified a word that could be associated with both parts of the prompt.

What ChatGPT did was make a million monkeys bang on keyboards, narrowed the search down by eliminating the results that had no word association, and then **tasked me with sifting through the noise to find out which ones made me laugh.**

That last part is the most important piece. {% include pullquote.html quote="These weren’t just spat out by a machine; they were the result of an interactive process in which I was an active participant. Without a human actor evaluating and editing the output, the quality degrades dramatically." %} This is more true the more sophisticated the project, but we see signs of it even here, with a relatively simple prompt.

It gave me a dozen jokes, and I chose three of them. ChatGPT couldn’t have done that choosing for me, because it had no idea which jokes were actually funny. I looked at a list of generated word associations (a very computable and not-funny task), and I saw the humor that occurred by chance in a quarter of them (a very funny but not very computable task).

So where did the humor in these jokes come from?

It didn’t come from the training data, because these jokes aren’t in the training data. And I’d argue it didn’t come from ChatGPT either, because while generative AI is constantly amazing us with the new things it can do, humor is such a subjective and context-sensitive pursuit that I expect it to be one of the last things that these models will conquer.

But there’s one other part of the system: me. In separating the chuckles from the duds, the humor I was observing wasn’t inherent in the responses, but rather something I was projecting onto them. By reading, selecting, and editing the responses, I used my own labor and my own creativity to arrive at something that I thought was funny.

Now in the case of these jokes, it wasn’t very much labor, but these aren’t very great jokes either. They’re just simple wordplay with a miniscule amount of human creativity sprinkled on top.

{% include pullquote.html quote="ChatGPT’s output was a Rorshach test: ChatGPT didn’t write jokes any more than Hermann Rorschach drew pictures of my parents fighting." %}

It’s said that you can find every possible string of text somewhere inside pi. (This isn’t actually proven to be true, but that’s beside the point.) Does that mean that pi is the [i]source[/i] of every possible string of text? Of course not, because to find them you need to already have an idea of what it looks like. The creativity is in the effort of extracting the string. Tools like ChatGPT reduce that effort, but you’ll still get out of it only what you put in.
