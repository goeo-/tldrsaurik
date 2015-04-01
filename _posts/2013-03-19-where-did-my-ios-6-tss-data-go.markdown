---
layout: post
title: "Where did my iOS 6 TSS data go?"
date: 2015-04-01T07:10:00-00:00
---

<div class="original">
Normally, I get to write articles (or give talks, or leave many-page long comments on various websites and forums) on interesting aspects of technology, confusing aspects of business, protocols and the people who standardized them, or new tools that I've been working on; writing these articles can be grueling at times, but at some level the task is not just rewarding, but fun.

Sadly, that is not why I have been working on this article: instead, I am here to be the bearer of bad news that will likely cause me to get a ton of hatemail. :( Specifically, I am writing this to tell everyone that the TSS data Cydia saved for iOS 6.0-6.1.2 is unusable. I'm also going to attempt to explain some background on the process, what the mistake was, and what users can now do instead.

In the process, I am going to explain a few parts of the iOS software security mechanism that I have not seen described often outside of technical presentations at security conferences. Additionally, I will summarize, from the perspective of a user, what is currently possible with cached APTicket information (something I actually did not know much about before writing this).

My goal is that by the end of this article readers will understand enough of the process to appreciate the mistake, the history of the issue, and why it was never caught. Additionally, it will hopefully be slightly more clear when cached APTicket information is usable (as it turns out that cached APTickets have a rather narrow range of uses) and how affected users might still be able to get this data.

(For those who don't care about the long explanation below: the only devices where saved iOS 6 APTickets are actually ever useful are the iPhone 3G[S], iPhone 4, and the 4th generation iPod touch; on these devices, users can, and probably "should", save the APTicket that is currently allowing their device to boot. This can be done with a tool that can dump this information using the limera1n bootrom exploit, such as redsn0w or iFaith, and upload the replacement to Cydia.)
</div>

<div class="tldr">
Apparently no one noticed but the blobs from iOS 6.0-6.1.2 saved by Cydia are broken. In the following few sentences, I’ll explain what blobs and shit like that are and what it can possibly mean for us all.
</div>

## What is TSS?
<div class="original">
One aspect of maintaining a "secure" system is to verify that the software that is installed on it is the software that you were expecting. This is done by using cryptographic "signatures" that can demonstrate the authenticity of a block of data, such as the operating system running on a device such as an iPhone. Apple "signs" all of the software that they put on an iPhone.

As the device boots up, each step verifies the signature of the next step before moving onwards. Assuming that each step works correctly, you can then feel safe that the entire system is running the software that it was intended to run, with no modifications that might do things you don't want (whether it be actual malware, or adding cool features that you didn't authorize).

Of course, that assumption is unrealistic for a system as complex as the iPhone: there are many bugs, large and small, that allow attackers to bypass various checks. Particulary tricky attacks can allow for things like evasi0n: an "untethered jailbreak", where the software has been permanently modified in a way that will persist across reboots, re-attacking exploits each boot.

Therefore, one has to plan how bug fixes will be rolled out and required: it isn't sufficient to just sign software, as once a bug is found, users could always just downgrade to that known buggy version, take control, and then potentially work their way back up; certainly, though, even if they are stuck at the old version, that may already be a sufficiently costly loss.

The way vendors typically solve this is by having restrictions past just "valid signature" when new software is installed: the most common being "the software being installed must not be older than the software it is replacing, either by checking a version number or an encoded date (which is, really, just another way of encoding a version number)". Most systems use this scheme.

Apple, however, decided that this wasn't sufficient, as this allows one to actively maintain a device effectively at an old version for a long time. Additionally, it means that if you have a device you haven't used in a very long time, and many software revisions have been released between then and now, you could choose to upgrade to any of those versions, not just the latest.

Apple's solution to this, which they deployed with iOS 3.x (iOS 2.x was signed, but had no downgrade protections at all), was to have every single installation of the operating system verify, at that moment over the Internet, that the version of the software being installed is "the one we intended to allow you to install". The verifier is the Tatsu Signing Server, or "TSS".
</div>

<div class="tldr">
Software usually checks itself to see if it’s not hacked. Here is some really irrelevant things about what apple did different. Finally, I explain that TSS is what signs restores and updates, sorta. I actually don’t explain it all too well.
</div>

## What is SHSH?
<div class="original">
The important detail of the signing process is then to know exactly what is signed. Over the years, this has changed. The system that I had described a few years ago in a previous article, "Caching Apple's Signature Server", involves "personalizing" the files that are used as part of the iOS operating system software installation process (which is known as "restoring").

This personalization is done by adding data to each file that is specific to the device on which the software will be installed, and then having the resulting "personalized" file be signed again by Apple. (I say signed "again", as the original files distributed by Apple are already signed, but they do not contain this device-specific information and are entirely generic.)

The device-specific information that was used for this process is called, depending on where you find it, the "unique chip ID" or the "ECID" (an acronym that no one is sure of the meaning behind, but probably means something like "exclusive chip ID"). This ECID is a small block of data represented as a number that is unique to every iOS device that Apple has shipped.

This ECID is then sent to Apple, along with the list of files that are being signed. Apple then returns a "blob" for each file, which is a block of data that replaces the existing signature on the file with both the personalization information (which is the ECID), sometimes some random data seemingly just "for good measure", as well as a new signature that signs the entire file.

The actual signature inside of this blob is the SHSH, or signature hash (maybe "signed hash"; again, no one is really that certain, but we largely agree on "signature"). As the ECID of a device never changes, if you can then save the SHSH of a personalized file, you can always use it later to install that file, even if Apple is no longer willing to sign it: we thereby like saving these.
</div>

<div class="tldr">
Apple's digital signature protocol for iOS restores and updates. SHSH is downloaded from Apple, unique to every device and firmware. Only the latest firmware is signed.
</div>

## How is each SHSH computed?
<div class="original">
I now must make a slight detour to describe signatures: the way these work involves something called a "hash"; when you "sign" a file, you first take a "hash" of the file and then "encrypt" that hash using an encryption key that only you have (a "private key") but that can be decrypted by a second "public" key that you are able to widely distribute ahead of time to anyone who cares.

The result of "hashing" is sometimes called a "digest" because what a hash effectively does is to make a smaller version of the file. This smaller file is typically a fixed length that is unrelated to the size of the original file: it is instead determined by the choice of hashing algorithm. The algorithm that is used for verifying iOS is SHA-1, which generates a 160-bit digest.

When you are constructing a digest using SHA-1, you can actually do it "piecemeal": you can get part of the data and stop, and then later come back when you have the second part. At any moment, the only data you need is 160-bits of data and the length you have so far hashed. This means that there is an efficient way to "continue hashing a file from where you left off" when using SHA-1.

Apple uses this trick to generate its personalized signatures: instead of sending the entire file into the server, or even storing the entire file on the server, requiring it to still run the hashing algorithm on potentially hundreds of megabytes of data, Apple sends only a "partial digest" to the server as part of a TSS request, which is then completed by the server and signed with their key.

Finally, inside of an iPhone software (IPSW) update file (which is really just a ZIP file) is a "build manifest" that contains a list of filenames and, for each one, its digest (which is the hash of its data, including the signature it has attached to it from Apple) as well as one of these partial digests for the file without the signature block on the end (so it can be completed by TSS).

The personalization process thereby involves taking the build manifest, building a TSS request, sending it to Apple, getting the result, and then modifying each of the files that were listed by replacing the signature section with the blobs returned by TSS. The resulting modified files are sent to the device, which verifies the ECID inside of them, and also validates the signature hash.
</div>

<div class="tldr">
Irrelevant magic.
</div>

## Caching Apple's Signature Server
<div class="original">
When Apple started doing this, we figured out how it worked, and the course of action was clear: to setup a man-in-the-middle attack on this server that would simply store every single SHSH that was returned for every file of every firmware version for every device owned by all of the people who cared about being able to downgrade (both jailbreakers, and App Store developers who need to test their apps on earlier firmware versions).

I built this system as a service and wrote an article about how the process worked and how it could be used. Initially, the system acted only "in the middle", but it was immediately enhanced to save all of the ECIDs of all of the users in a massive database, so it could go on its own every time Apple released a new firmware version in order to request everyone's SHSH information "en masse".

Eventually, though, you end up with so many ECIDs on file that you are trying to request from a number of servers hundreds of thousands of times fewer, that it becomes very obvious how to shut down such an operation. For a couple years now, I have thereby not been able to run the "man in the middle" proxy server, nor am I able to provide the service of automatically saving peoples' SHSH for them.

However, I have still been able to help users automatically get their data stored by having the Cydia application do it "in the field": Cydia requests SHSH blobs from Apple and uploads them to my servers whenever it notices you don't have the information stored on my servers. Additionally, I provide an API that allows third-party tools that retrieve and manage the same information to upload it to me for safe keeping.
</div>

<div class="tldr">
You man in the middle attack a request and save the file. Also irrelevant, I bring up my server.
</div>

## How did APTicket change things?
<div class="original">
When iOS 5 came out, Apple changed the game slightly: SHSH, the system that I had known a lot about and had spent an inordinate amount of my life building (over and over again, due to changing limitations and scale requirements) tools to retrieve and store data for, was on its way out; a new verification scheme called APTicket was clearly setup to replace it entirely in some upcoming device.

The person who then spent a lot of time looking into APTicket was not me: it was actually MuscleNerd, the developer of redsn0w. Additionally, as recently as late September of 2012, at JailbreakCon, I can be heard asking questions from the audience to iH8sn0w about APTickets after his presentation, as all of my knowledge of how they worked was secondhand ;P.

Since then (a few months ago), I spent some time reverse engineering the APTicket verification system, and thereby know some more about it; however, I was mostly concentrating on finding bugs in the certificate parser, and only was looking at a single stage of the bootup process, so I continue to not know everywhere an APTicket is used, or certainly how various devices are affected by their presence.

However, at a high level, what the APTicket is is a single block of data that is signed as a whole and that contains the digest of all of the files that will be used as the device boots. In various ways this is much more efficient than having to personalize each and every single part: you don't even need to rewrite the files, as the APTicket can just store the pristine original digest of the files from the IPSW.

Additionally, it drastically reduces the number of signatures that Apple's servers need to perform: every time they release a new version of iOS, their requirement that the software must be signed for every specific device requires an immense number of crytographic signatures; the APTicket mechanism reduces the number of signatures they have to calculate by a factor of between 10 and 20 times.

Further, the APTicket has a special quirk: a "nonce" (a word with a fairly interesting history) is stored inside of it (meaning that it, too, is signed during the restore process by TSS) that is verified in addition to the ECID. This "nonce" is a number that is generated at random during the restore process, and which thereby makes the APTicket specific to every single restore and uncacheable.
</div>

<div class="tldr">
This new thing that apple made to make my life miserable. 
</div>

## Caching the Uncacheable
<div class="original">
The question then is, if APTickets solve the problems that SHSH had: why do people still save them? One would expect at this point that when Apple introduced APTickets, that all of the work attempting to store and process this data for later use would have stopped. Obviously, this is not the case. There are two things going on here: backwards compatibility and a few implementation mistakes.

First, Apple can't just reflash the bootloader on the existing devices that are in the field, and those verify the first step using SHSH. This means that if you have SHSH data stored for that first stage, it doesn't matter whether you have an APTicket for the remaining stages or not: you can trick the first stage. If nothing else, this lets you downgrade to iOS versions that did not use APTicket.

Second, this is a complex verification system, involving a lot of little steps; as an example, at some point you have to reset the nonce: if the nonce never changes, then you can save the APTicket data for whatever fixed nonce you have, and it all looks very much like it did back when we only had the ECID to worry about. It is my understanding that this is how MuscleNerd's "re-restore" trick works.

Taking advantage of these mistakes has allowed for a few interesting tricks, such as allowing people to switch from one version of iOS 5 to another, or allowing the iPad 2 to be downgraded to iOS 5 from iOS 6 (if you have TSS information saved for iOS 4, to use as an intermediary). For more information on how you can use these techniques, users may wish to read the JailbreakQA FAQ entry on SHSH and APTicket.
</div>

<div class="tldr">
I don’t know what saurik is trying to convey here.
</div>

## So, what actually is possible?
<div class="original">
While I have mentioned a number of interesting tricks, they are mainly related to either old versions of iOS or old devices. With newer devices, APTickets are more deeply ingrained in the bootup process; and with newer versions of iOS, most of the mistakes that had been used have now been fixed. As newer devices never had access to install older versions of iOS, these limitations start overlapping.

In particular, many older devices are subject to an exploit called limera1n, which attacks the first-level bootloader of the device: this is something that Apple can't fix, and allows us to bypass or alter everything that comes after it in the bootup sequence. Using this exploit, it is possible to downgrade these exploitable devices to any version for which you have your TSS information saved.

Another opportunity allows you to sidestep the APTicket process by way of iOS 4, which predates the introduction of that feature. This allows devices that have TSS information for both iOS 4 and iOS 5 to downgrade to iOS 4 (even from iOS 6) momentarily and then upgrade to iOS 5. There is only one device, however, where this matters: the iPad 2. Older devices have limera1n, and newer devices did not have iOS 4.

Finally, for devices running iOS 5, we have the ability to restore any other version of iOS 5. This is only relevant on the iPad 2, iPad 3, and the iPhone 4S (as earlier devices are exploitable with limera1n, and later devices came out after iOS 6 and thereby cannot use iOS 5). While the iPad 2 is also the device from the previous category, this can work even if you don't have iOS 4 TSS information saved.
</div>

<div class="tldr">
Nothing as all exploits are patched and no new ones can exist because memes.
</div>

## What does Cydia's TSS service do?
<div class="original">
Moving back for a moment to the service I have provided for storing TSS information, it is important to know that it was designed for SHSH. The reason I built it was to have a concrete way to use saved SHSH blobs: all previous descriptions (including geohot's original purplera1n.com, which allowed users to save one critical SHSH blob "for a purplera1ny day"), relied on a tool like redsn0w to one day exist.

At the time, however, we did not have such a tool, and thereby needed something "simpler". My idea was to build an implementation of the TSS server, one that could even transparently provide information from Apple by passing requests through, that would be able to save information on its way through and then later replay it when you could no longer make the requests for this information from Apple.

This meant that you could use iTunes itself to perform restores of any version you had ever previously installed by tricking it into connecting to my server instead of Apple's server (which we did using "etc/hosts" entries, which is easy to configure on both Windows and Mac OS X). I also provided an open-source implementation of TSS that others used to learn from and build local tools and servers.

As you used the server, I also saved your ECID, and would then attempt to request any newer version of iOS on your behalf. Over time, I ended up having tens of millions of ECIDs on file, and had built out infrastructure allowing me to rapidly dump, in parallel, large numbers of SHSH blobs from Apple's servers as fast as I could (which in a way was not fast enough, and in a way was too fast for Apple ;P).

Over time, of course, it became impossible to have all of this service centralized, so I began saving the information using Cydia itself, based on an API I setup called "TSS@home" (which made more sense in its original incarnation, where devices would download work units and save TSS on behalf of others, but ended up being deployed to Cydia as a feature that saved SHSH only for the running device).

Offering such a service, where data is uploaded by random devices, has a serious problem: you have to deal with people who might be purposely uploading invalid or corrupt blobs. I thereby reached out to MuscleNerd while building TSS@home, who provided to me a sketch of an algorithm that could be used to validate blobs as they were uploaded in the same way that an iPhone would verify them during boot.

At this point, while I had only been trying to distribute the load (both for me and for Apple, as they seemed to have multiple, geographically dispersed servers handling this particular system) of requesting large numbers of SHSH blobs to computers around the world, what I ended up with was a service that allowed users to upload SHSH data, verify that data, and then retrieve it later. The developers of other tools that worked with SHSH thereby reached out to me asking how they could upload blobs.
</div>

<div class="tldr">
It caches blobs for you.
</div>

## What about APTicket storage?
<div class="original">
When Apple introduced APTicket, the data my server was storing was no longer relevant. However, as part of the process of downloading SHSH information, Apple sends an APTicket as well. I thereby modified my service to also store the APTicket that was downloaded as part of the SHSH collection process (with some further help from MuscleNerd to learn how to validate and refer to APTicket instances ;P).

An additional modification was made to keep Cydia (or any other client using my TSS@home "check" API) to not claim that the SHSH information was stored for a particular ECID running a particular version of iOS unless an APTicket was also stored. No real changes were made to the system to make these APTickets happen: they were simply saved as a side effect of the requests that were already being made.

Apparently, this is what people needed, and for something like a year users were very happy with the APTickets that I was saving. As I described earlier, however: I really don't know much about APTicket, even now, and thereby have been reliant on the people building the tools that use APTicket to tell me what changes I needed to make, either to the verification process or the requests themselves.
</div>

<div class="tldr">
It stores them.
</div>

## So, what is the problem here?
<div class="original">
At this point, I think I have described everything I need in order to explain the current situation: all of the APTickets Cydia itself requested from Apple for iOS 6 are useless. The word "useless" is important, as it is not accurate to use the word "corrupt": the data that was uploaded was not lost or damaged, and in fact all of the tickets that were stored verified per the algorithm from MuscleNerd.

Instead, the requests being made via Cydia to collect SHSH information for iOS 6 did not result in useful tickets. This is because, in order to better emulate the requests Apple had been making when I first started the service, I filter the manifests I send to Apple to only include information about files that had the partial digests I discussed earlier, as only files that have partial digests are relevant for SHSH.

However, the APTicket signs complete digests, not partial digests, and so even descriptions of files that do not have partial digests need to be sent to TSS to get a complete ticket. What really should therefore be used as a filter is "files with digest information at all", not just those that have partial digests (there is never a partial digest without a full digest), effectively finding all "real" files.

The result is that the APTickets that were downloaded and saved by Cydia itself are not sufficient to boot a device. However, tickets that were downloaded or otherwise obtained by tools such as redsn0w, iFaith, or TinyUmbrella, will work fine. If those tickets are uploaded to Cydia and then downloaded back, they also will continue to work: it is only tickets downloaded by Cydia clients themselves that were affected.
</div>

<div class="tldr">
We did an oopsie and the apticket data was not complete.
</div>

## Why did no one notice this?
<div class="original">
As stated before, my knowledge of APTickets has been entirely relegated to what I am told by the people who build tools that use APTickets: when APTickets were introduced, I assumed my service would simply become obsolete, but then extended the service at request of other developers as it seemed like it could still provide some benefit to the community (although increasingly less and less, it seems).

Meanwhile, all of the people who build tools that use APTickets also build tools that download APTickets. It therefore happened that developers such as MuscleNerd and iH8sn0w do not personally test their tools against data saved for users by Cydia's TSS client. As for myself, I've never actually used an APTicket as a user: until writing this article, I did not even know in what situations they were usable.

Still, iOS 6 has been out for months, and one would have expected that some users would have noticed the problem. However, given the complex limitations that we have on our usage of APTickets, many users (even in larger discussions on sites like JailbreakQA) believed themselves to be misusing the tools, or simply thought the tools needed to be updated, and so seemingly never reported the issues.

Finally, there simply hasn't been much reason for users to attempt to use any of the TSS information for iOS 6 during these months: for the entire release cycle of iOS 6.0, there was no untethered jailbreak available, so people mostly wished to use TSS to downgrade to iOS 5. Then, until a few weeks ago, iOS 6 had an untethered jailbreak available, so people did not need to restore or downgrade: they would just restore.

Thankfully, I have been told by MuscleNerd that changes have been made to redsn0w to make any similar situation we may potentially run into in the future (where the manifest information being used by our tools differs, causing Cydia to save unusable information) something that will be noticed while the signing window is still open (in fact, he believes the very first day), allowing things to be corrected.
</div>

<div class="tldr">
No one really needed a downgrade nor any exploit was public yet.
</div>

## What does this mean for me now?
<div class="original">
Honestly, due to the various limitations on the exploits we have for the reuse of APTickets, not many users are affected by this issue. First, if you have a recent device, iOS 6 APTickets are entirely useless: you cannot use them to downgrade, you cannot use them to upgrade, you cannot even use them to just restore the version of iOS you are currently running; cached responses for iOS 5 are of general interest, not iOS 6.

The set of devices that are able to run iOS 6 and that are also old enough to be subject to this exploit is actually fairly small: only the iPhone 3G[S], iPhone 4, and the 4th generation iPod touch meet these requirements. In particular, no iPad, nor any recent iPhone (not even the iPhone 4S) has any known way to use cached iOS 6 TSS information. This means that 74.2% of Cydia users are not affected at all.

Secondly, for the remaining 25.8% of Cydia users for which cached iOS 6 APTickets could be useful, by far the primary purpose is to restore or otherwise recover the version of iOS you are currently running untethered. As an example, a user is currently running 6.1.2 and accidentally upgrades to 6.1.3. Alternatively, they accidentally break their iOS installation so badly that they need to restore.

In either of these two use cases, the alternative to an APTicket exploit is to upgrade to iOS 6.1.3; as this scenario is only applicable to people running older devices (those subject to limera1n), iOS 6.1.3 is still jailbreakable (as should all future versions of iOS on these devices), but the result will not be an untethered jailbreak: many users (including myself) really hate using tethered jailbreaks.

Thankfully, this situation is actually fairly easily solved: redsn0w has the ability to dump the full TSS information from a device (also using that same limera1n exploit). I thereby encourage users of devices capable of being exploited by limera1n (the iPhone 3G[S], iPhone 4, or 4th generation iPod touch) to download this tool right now and use it to upload complete TSS information.
</div>

<div class="tldr">
Blobs are useless for almost everyone because there has not been a new downgrade exploit in about year.
</div>

<div class="original">
Note: I have been told by MuscleNerd that there is a minor issue in the current version of redsn0w that will cause blobs retrieved from the device to not be uploaded to Cydia's servers. He had intended to get a new version out by the time he had to leave for HITBSecConf2013 (an international security conference at which evad3rs is giving a presentation about evasi0n), but schedules did not permit this. I had then hoped that this new version of redsn0w could be released before this article, but due to the longer delay I have decided that this information needed to be released sooner.
Using the currently released version of redsn0w will still (as far as I understand) copy the active TSS data from your device and store them locally on your computer. It is then my understanding that redsn0w will be able to upload this information at a later time from your computer. Alternatively, there is a program called iFaith, developed by iH8sn0w, that can be used to immediately upload your TSS information; however, this program is only available for Windows (so users of OS X will definitely have to wait until the new version of redsn0w is available).
</div>
