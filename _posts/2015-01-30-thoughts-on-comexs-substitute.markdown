---
layout: post
title: "Thoughts on Comex's Substitute"
date: 2015-04-01T06:30:00-00:00
---

<div class="original">
(For some context, as a lot of people end up posting my long responses to places where I didn't originally post them: today, comex announced and released on GitHub a pre-alpha implementation of Substitute, a project that is designed to replace my Substrate. He is doing this because iMods, a project to build a more-closed ecosystem, is aiming to replace Cydia, and for this to happen they need their own implementation of Substrate, the only key project I develop which is closed source.)
</div>

## Open Hardware, not Software
<div class="original">
So, an interesting aspect of my overall "mission" is that I consider "open source" to be an increasingly short-term game: it is an issue that only exists right now because binaries are relatively inscrutable and unmodifiable. Substrate itself is designed to undermine these restrictions: it allows developers to make the kinds of changes that "classically" require source code. In general, I also believe the algorithms for reverse engineering have started getting better faster than the technology behind binary obfuscation.

(As a demonstration of this: do not for a second believe that comex is not able to just read through Substrate as if it were open source. I know that I would only have mild difficulty doing this, and comex is much better at this kind of work than I am. In Substitute's unrestrict.c, he claims Substrate might have a bug: I don't think he's right, but the issue comes down to the analysis of multi-process interaction, a sufficiently subtle point that makes it very clear that comex "read the code" for MSUnrestrictProcess.

Other parts of the code for Substitute also have comments about duplicating functionality for things that Substrate is doing, things that are not open source. At some point in the not-too-distant future, I believe that we will find the distinction between closed source and open source crumbling—not just for people like comex and myself, but for the average developer—a point I made in one of the best talks I've ever given, a five minute "Ignite" talk at Foo Camp 2010. If anyone is bored, it is only five minutes ;P.)
</div>

<div class="tldr">
So here I fail to properly describe Open Source in three paragraphs.
</div>

<div class="original">
My mission has thereby concentrated on fighting for open hardware and, in general, open platforms. I use software licensing as a weapon in that war. (BTW, playing this kind of "long game" is also something Stallman does: he advocated for Ogg Vorbis to use a BSD-styled license rather than GPL because he thought it more important to undermine MP3—for which patent restrictions make it difficult to even use in open source software—than to guarantee that the good implementations of Ogg will always be open.)
</div>

<div class="tldr">
My mission is to make open hardware because software isn't enough. BTW, Stallman is a saint.
</div>

## One Case Study: ldid
<div class="original">
My first open source project that was core to this community is actually an interesting case study in this war: a couple core members of the original group of jailbreakers were tied to the interests of RiPDev's Installer (who had employed them), and their business model was to sell apps (note: not extensions, apps, which was the focus at the time). Their strategy was to have the jailbreak locked down in a way that would require apps go through them, potentially using Apple's own codesign signature mechanism.

At the time, the understanding behind codesign was a guarded secret, and the one tool for "pseudo-signing" new executables was to be closed source. I made it my job to, without learning anything from them, write a tool that parsed and updated those segments and to release it as open source. There was seriously an argument I had with some people where they were pissed at me for having released that knowledge to the community, because they had intended it to be restricted to them.

To be clear: I wasn't the only person on the team arguing, and this wasn't the only part of that battle. Their timeline for controlling codesign was really tight, and they were cutting corners, which meant that knowledge of pseudo-sign played a large part, but it is also important to point out that others on the core jailbreak team were also "on my side", which is why the timeline was so tight and why so many corners had to be cut. ldid being open source was simply "the nail in the coffin" for this particular effort.
</div>

<div class="tldr">
Fuck old competitors. Here I complain about something old because anecdotes are cool
</div>

<div class="original">
When I built Cydia, I embraced the community of third-party sources, putting them front-and-center to the experience, something that I have continually pushed further (such as with the new Sources interface released last year, which pulls that feature out of the depths of the UI and forces users to experience the concept). This is how I think about platforms: when I have had control over something, I use that control to build an egalitarian base for other people to build their own kingdoms.
</div>

<div class="tldr">
I made Cydia because third party sources are needed
</div>

## Open Source is not Enough
<div class="original">
Which then brings us to why Substrate has been closed source, and why that is also to me a weapon in the same battle. This starts from a simple premise: that open source software merely existing does not lead to open platforms, as the priority of the vast majority of users align with short-term interests (for various reasons, some good and some bad) and so they will happily buy into and empower closed platforms in order to get a relatively small amount of functionality, comfort, or glitter (see iOS itself ;P).

When I started working on Substrate, I still had what I now consider to be a fundamentally flawed model for the goal of open source: all of my software was released under BSD licenses, and I regularly had arguments with Brian Fox (a friend of mine, and first employee of the Free Software Foundation) about how I considered GPL to be an awkward... even a "corrupting"... force, and would use examples such as gcc being crippled, seemingly on purpose, with respect to being able to export syntax trees.

The overall mistake I tended to make was to not look at the "long term" consequences of these licensing decisions: I was instead looking at it from a perspective of "making my knowledge as available as possible to everyone is always a good thing". The obvious next place for this thought process is of course GPL: to trade some short-term freedoms (for developers to build it into closed-source software) for the long-term guarantee that the source code will always be open and that it can always be modified.
</div>

<div class="tldr">
Open Source is not Enough because third party sources
</div>

<div class="original">
So, now we look at Substrate: why do I keep it closed source? It clearly isn't that I like closed source, as almost all of my other key projects (Cydia, WinterBoard, Cydget, Veency, ldid) are open. It isn't possible to paint me as someone who is against things being open (and comex thankfully did not try this): I even have a good "batting average" on having software that not only was open but continues to be open as compared to other key developers in the ecosystem (DHowett, rpetrich, BigBoss, limneos, ...).
</div>

<div class="tldr">
I made Cydia Substrate closed Source because third party sources
</div>

## Rock Your Phone 1.0
<div class="original">
The central reason is that we have already seen others come in and use an embrace-and-extend strategy on Substrate in order to build a totally-closed ecosystem: that was Rock Your Phone, with their Rock Extensions. People always think back to Rock 2.0, which was a reasonable "Cydia competitor" that supported APT repositories; but the mission of Rock 1.0 was to build a siloed, commercial-first (if not even commercial-only: I was told by them I should sell Cycorder, which had been free) store.

The reason why Rock got forced to support APT repositories (which became Rock 2.0) was that iOS 3.1 required some major reimplementation work on Substrate, and I made the project closed source. This meant that for Rock's software to operate, they needed access to a legitimate copy of Substrate, and their strategy for doing that was to support APT repositories and then just download it in the same manner as Cydia. A world where Rock 1.0 became the core ecosystem would be a world very unlike the one we have.
</div>

<div class="tldr">
Rock was evil
</div>

## Importance of Mobile
<div class="original">
I can totally appreciate the comments about non-mobile platforms (and am intending to do more there), but frankly: setting the strategy on non-mobile platforms is missing the lesson from Wayne Gretzky, who would always say "I skate to where the puck is going to be, not where it has been.". We are just about at the point (which is conservative on my part: we may already be there) where tablet sales have surpassed laptops. It is really sad to look at short-term benefits when we have serious long-term issues to deal with.
</div>

<div class="tldr">
Substrate on Mac would be useless because tablet sales surpassed laptops
</div>

<div class="original">
In a way even more important than their sheer sales numbers, however, is that the "war" over whether computing—hardware, not software—will be open or closed is being waged on these mobile devices, not on laptops and PCs. When we lose the war for open tablets, to the extent to which laptops still exist we will see a major—if not even sudden—shift on those platforms, but for now the only reason we are seeing laptops becoming more closed over time is due to the way this is playing out on tablets and phones.
</div>

<div class="tldr">
It's more important to fight against corporations than to let people make tweaks
</div>

<div class="original">
(Which brings me to a side point: the paragraph from comex about why he refused to cooperate with me on Substrate in the kernel makes me really angry. It is essentially him saying that he doesn't believe that cooperation is valuable because of what I find to be a very selfish analysis of who would benefit from the existence of that work. I, for example, wanted Substrate in the kernel for me, I know others who have as well, and on Android I was even doing some similar work recently: work I haven't yet done a write up for.)
</div>

<div class="tldr">
I'm really angry
</div>

## Closed Platforms are Bad
<div class="original">
A funny thing about all of this is that, in fact, surprisingly few people are somehow (I say "somehow", as I find it surprising sometimes) in a position to build something like Substrate. Whenever iMods has claimed they were going to find someone to work on a replacement for Substrate, I've put together a little list in my head of the people who could do it and why they would likely not work on it; what I had failed to realize for comex, though, is that I generally think of him as being uninterested in all this stuff.

The awkward-to-me result is that we now see comex—someone who talks a lot about open systems—building tools for people who are specifically against the idea of open platforms: who talk a lot about commercialization, strong DRM to defeat piracy, and how allowing third-party repositories outside the centralized, curated experience would fundamentally undermine what they consider required to build a clean experience. comex is now working for the very ideas I thought we wanted to defeat.

This makes me really disappointed, and this isn't something I can fix. Essentially, this undermines the strategy that I've had in my head for what this community was doing, and why it mattered in the greater picture. I don't do this because I want people to have funny features on their phones: I do this because we are fighting a battle against companies—not just companies like Apple, but companies like Rock Your Phone and iMods—who want to see "open" be traded away for "ease of use" or "profitability".
</div>

<div class="tldr">
I didn't expect comex to do this, also he's evil because iMods is evil
</div>

## What This Means Now
<div class="original">
So, I don't really know what to do. I really can't get into a massive long-term argument with comex: I almost left the community before (years ago) because of comex (during the time when we were all having arguments over the deployment of unionfs), and I just don't have it in me anymore to "fight" (not that there's anything that could really be done) against people whom I thought were "on my side". (I thereby do not address this to comex, and don't entirely care what his response to what I'm writing is.)
</div>

<div class="tldr">
I am a kid on YouTube who wins an argument by not replying.
</div>

<div class="original">
Right now, we are in the middle of an DMCA exemption comment period. In the next day or two, Britta and I are going to be launching some documentation and a small campaign related to this. I promise to everyone that I will continue to be part of things through the end of this DMCA exemption cycle. This push is going to come with some style updates to Cydia's home page, so you will see those changes, and there are a couple payment related things that are "almost done" which will happen in the near future.
</div>

<div class="tldr">
Irrelevant thing about the DMCA thing because pathos
</div>

<div class="original">
I also have some changes I was working on to Substrate (a different way of loading and filtering that is finally ready, that will come with an easier-to-use SDK) that will probably be released. We will see if it makes sense for me to continue bothering. I am, in all honestly, really tired of fighting and arguing with people who don't see the end game. I spend way too much of my time burning myself out on reddit and IRC trying to get people to "understand", and it has driven me at times to the brink of insanity.
</div>

<div class="tldr">
I am not going to do much now that iMods and Substitute exist because other people playing in the sandbox with me is disgusting and must be stopped.
</div>

<div class="original">
The key thing I think I'm going to do is "retract" somewhat: while you are probably all quite used to me being active on places like /r/jailbreak, having to deal with the escalating comments from a few posters there about iMods (especially some of the recent stuff, largely due to coolstar's Anemone, a monolithic replacement for the WinterBoard ecosystem, also due to iMods) really took a toll on me, and the return of comex is much more than I can mentally handle.
</div>

<div class="tldr">
I will not use reddit because all I do is complain about how this cannot be added into Cydia because Cydia is not a store even though it is but hey FUCK COOLSTAR!!!!
</div>

<div class="original">
I will, of course, continue to be available "behind the scenes" to the key developers and repositories, but I just don't feel like I can continue to directly participate in the public conversations much anymore without it killing me: I've already pulled back some this last month, and it has been really inspiring; but what I really need to do is just stop entirely and let other people handle all of this front-line stuff. (FWIW, I'm sorry I can't do even more: it is painful to realize one's limits.)
</div>

<div class="tldr">
iMods causes me depression but even though that happens, I WILL TRY MY BEST TO NOT BE A LAZY BUM
</div>
