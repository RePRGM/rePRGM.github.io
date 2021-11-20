---
layout: post
title: eCPPT vs OSCP (Reviews and Comparison)
---

This is a re-post of a reddit post I made a while ago. Original link [here](https://www.reddit.com/r/oscp/comments/ho0j5z/oscp_vs_ecppt_my_experience_with_both/)

-----

As seems to be standard after passing, this will be my review of OSCP and since recently there seems to be growing interest in eLearnSecurity's PTP course and eCPPT certification, I'll also do a comparison of the two.

## Background

I decided to get into offensive security last August by going after CompTIA's Pentest+ exam with the goal of eventually doing OSCP, but knew I'd need to ease my way there. For context, I earned the CompTIA trifecta in High School, and then went to on to do Network Administration a couple years after, earning my CCENT and CCNA. I considered CEH as well of course, but decided against it for a few reasons.

* I don't have verifiable experience in Cyber Security so I would have had to pay for their training as well

* The more research I did, the more it seemed only HR likes it

* Pentest+ is regarded as being harder, so passing it meant I had a good chance of passing CEH later on if I decided I wanted or needed to get it

So, I studied. I found Ippsec purely by chance, and watched some of his videos here and there even though it was all entirely over my head. I did courses and practice tests on Udemy, tried to do boxes on Vulnhub, and just learned as much as I could. I passed Pentest+ that November.

From there I moved on to eLearnSecurity's PTS course and eJPT certification. More Ippsec. I think by this point I had found Hackersploit and maybe The Cyber Mentor on Youtube as well. More Vulnhub. I passed eJPT in December. Barely. I had a score of 16 or 17 out of the 20 possible. Passing was a 15. Since there were multiple-choice question alongside the pentest (if you can really even call it that), I guessed on a few questions I either flat-out didn't know, or was unsure about.

I had immediately went on to do the PTP course and eCPPT certification. At this point, I was watching Ippsec religiously, as well as TCM and hackersploit. I was looking into topics I came across in the PTP course that I didn't understand well, or just didn't get (DNS). I had signed up for HTB too. I don't think there was a single box I rooted completely on my own. I used the hints on the forums. There were times I went for hints out of pure laziness, evidenced by the fact I realized it was something I could have found had I tried... more. Other times, after looking at the hints and eventually figuring it out, I realized it wasn't something I would have figured out on my own. Sometimes, you just need the exposure to the concept or technology. I made mental notes on everything I learned.

I took the exam this past March and...failed. Pivoting is hard. Buffer Overflows are hard. That's all I can really say. I took the next week or two to go over Buffer Overflows more. I used TCM's Youtube course and it helped. This was before he made his ethical hacking course on Udemy, so that wasn't an option at the time. I took the exam again, and got the same environment to my surprise. I quickly got to the part where I was stuck the first time and eventually got through it. This time? I passed. That was in April.

From there I took a few weeks off. I had been studying constantly, going cert to cert, since August. I was starting to burn out during PTP.

## OSCP

But during my time off, I was still thinking about pentesting. I get bored when I'm not learning something new. After about a week or two, I was starting to watch Ippsec again. I started looking at OSCP reviews to get a sense of the course and exam. Recommendations, what I needed to do to prepare, any information I could get. I saw TJ Nulls OSCP-like machine list and went through a few. I'm pretty sure I used walkthroughs on either all or most after getting stuck. Again, sometimes you just need the exposure.

Eventually, I got tired of preparing and just decided to go for it. I fully expected to fail the first time or two given what I had read about the exam. But, you shouldn't be afraid to fail. In the end, no one cares how many times you failed something. People only care about the end-result. Did you pass? Did you accomplish what you set out to accomplish?

I bought 90 days of lab time in May, given that was what everyone seemed to recommend. I read through most of the PDF, skipping the scripting sections and maybe one or two other ones. I watched the majority of the videos as well. This took me about a month to get through, reading around 30 pages a day and sometimes when I had time, around 50. I'd also spend an hour or so on the accompanying videos given that they were so short.

I had planned to do work on the labs as well but realized it was eating up nearly all of my free time, so I decided to do labs on weekends and only if I really felt like it, did I do any reading or watch the videos. I ended up rooting about 15 boxes in the public network. I did not pivot to any other network, even though I had the keys to at least one of them. Why? I had already practiced pivoting doing PTP, and it wasn't going to be on the exam anyway. Once again, I used the hints on the forums extensively to make it through most, if not all, of the boxes. The exposure is good. Yes, I am a bit lazy and too impatient with enumerating. I made more mental notes of things I would have found with more enumeration and things I knew already but did not take into consideration. I also did not do any of the lab machines that were centered around client-side attacks or active directory. Why? Again, I had already worked on client-side attacks during PTP and neither that or AD would be on the exam.

I got bored (and a bit burned out) with the labs pretty soon and decided to just go for the exam. So, I scheduled to take it over the July 4th weekend.

I can summarize the exam pretty easily. I've seen it said that the entire thing was made to be doable in 12 hours and I completely agree. This is a beginner level certification. I used to hate seeing and hearing that. "The OSCP is NOT a beginner level cert", I thought. Well...it is. Though there is some context missing from that statement. The OSCP is a beginner level Cyber Security (or Offensive Security) cert. Cyber Security in general is more of an "intermediate" level thing.

Now, I have to caveat that with the fact that it actually took me around 16 hours to root 4 out of the 5 boxes. That last one I just couldn't get a foothold in, but I was overcomplicating the privilege escalation on one of the boxes, which is what made me take so long. I had a foothold on that box around 9 or 10 hours in. Follow Ippsec's advice here. Always try the simplest thing you can think of first when exploiting and doing privilege escalation.

Here is where my biggest advice comes. Triple check the flags you submit to the Control Panel before ending the VPN access. Check that the flag itself is correct, and also check that you submitted it to the correct IP. Guess what I didn't do that could have cost me my pass? I noticed after ending that I submitted one of the root flags to the wrong IP. So, for the past few days I was sure I had failed.

Also, take the time to do the lab report. Don't be lazy like me and decide it's too much work for too little reward. Those 5 extra points would have probably saved me had my mistake cost me the pass.

## Comparison

Course Material:

I enjoyed PTP a ton more than PWK. It holds your hand, yes. And that is a good thing. You're here to learn. There's also complete walkthroughs for every lab should you need it. PTP also covers a lot more of the boring aspects of pentesting like reports, legal issues, documentation, etc than PWK does. It does teach you a lot of Metasploit, and other automated tools as well. But, again this is a good thing. You're here to learn. And you do learn with automated tools. Of course, you can't completely rely on automated tools, but they are very helpful and useful since they will save you a ton of time. It is a pet peeve of mine to hear some of you moan about automated tools. In one of his videos, Ippsec even addresses it. "There is no difference between running Metasploit and running an exploit you found on Searchsploit". All of it is code. All of it should, ideally, be reviewed and understood before you run it.

The most memorable sections from each course is the Buffer Overflows. PTP does a great job at teaching you about history, naming conventions, etc. Meta-data. It has an entire chart with the names of registers on everything from 8-bit x86 to 64-bit x86_64 and the why's behind them. Do you need to know all of that? Not at all, but it's interesting and good to know. That said, PTP focuses on 32-bit Windows BOF. I loved how PWK (2020) goes over both Windows and Linux 32-bit BOF.

PTP also has entire sections for Powershell, Ruby, and Wifi and I felt like it covered Client-side attacks more than PWK did, but that is entirely subjective. The Powershell and Ruby sections are only available in the highest tier version, however.

Worth a mention: PTP has powerpoint slides (and videos) to go over. You can also get access to a PDF in the... highest tier version. It's good there's at least options. PWK only has a PDF (and videos). One gripe I had with PWK videos was that they are more or less just audio versions of the PDF. For some topics, seeing it done was a huge help for me, but I preferred PTP where the videos are there to complement the slides.

Labs:

Labs are completely different between the two courses and both have their ups and downs. PTP has dedicated labs that focus on each of the various topics. At times, I wanted a HTB-type environment where I had to figure out everything on my own instead of knowing what kind of attack I'd need to leverage based on what section the lab was attached to. Now, I should mention PTP actually does have a few labs like this, but they are only available in the highest tier version (Elite edition).

PWK has an open, shared environment that is similar to HTB. At times, I wanted to specifically practice something I had gone over in the PDF and videos, but there's no way to know ahead of time which box covers what attack or technique.

Exams:

Completely different, and really should not be compared. OSCP is harder. That said, its "unnaturally" or "artificially" difficult. What I mean by this is that its only difficult because of the tool restrictions and time limit. eCPPT is technically (that is, from a technical standpoint) harder. By that I mean pivoting opens up a whole new series of issues and considerations. You're also not just looking for flags. It is much more realistic. Treat it like a real pentest is probably about as much as I can say without delving into exam specifics.

*"Which should I do first?" "Should I do PTP to prepare for OSCP?"*

Everything depends on your specific circumstances. Both courses have a ton of overlap. I learned a lot from having to do things manually in PWK. Things that I didn't learn during PTP because you will use Metasploit a lot. And because, well, I didn't realize you can use Burp Suite to figure out what Metasploit is actually doing. Thanks Ippsec. I also learned a lot doing PTP. Things I wouldn't have learned in PWK. Both have value, but if I had to say, I'd say "yes". PTP will prepare you for the OSCP. And I agree with John Hammond (Another great Youtube resource) in that I think someone who passed the eCPPT could pass OSCP without studying for it. But not the other way around.

So, I'll end with this. OSCP is your money-maker. It has infinitely more value from its recognition alone. The course covers a lot of great things. I just think the exam falls a bit short. I'd actually love to see the next version of the exam be a (small) AD environment with every (or at least most) computers joined to the domain. eCPPT is something you do to learn. These two respective courses actually complement each other pretty well.
