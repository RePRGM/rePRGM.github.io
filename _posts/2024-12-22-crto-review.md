---
layout: post
title: CRTO Review
---

![Lets Make Malware](/assets/rto.png)

# CRTO Review
It is 12/22/2024 and I have just received notification that I successfully passed the exam for the Red Team Ops (RTO) course offered by Zero Point Security. The last time I did an exam and course review was in 2020 reviewing and comparing the OSCP and the up-and-coming at the time eCPPT. So, it's long past due for a new review.

While I will be mainly focusing on RTO here, I'd also like to do a bit of a comparison to the Red Team Apprentice Course (RTAC) offered by KFiveFour that I took earlier this year prior to RTO as well. So, let's jump in.

## RTO Course
The course itself is very comprehensive in that it covers a lot of different topics in the offensive security space. There are 29 modules in total covering everything from external reconnaissance to data exfiltration. 

Each module contains anywhere from three (excluding the module on exam preparation) to 14 different lessons as well. The vast majority of lessons are text-based with the occassional video included as well. This can be a pro or con depending on your viewpoint. 

How long it takes you to get through every lesson will entirely depend on you and your background. As someone who has been in the space for a while now, I did not take my time going through the lessons as much of it is review for me. 

The biggest differentiator, in my book, is that while the topics discussed are similar to what you'll find in many other courses, here you will be learning how to do things from a Command-and-Control (C2) Server in a, primarily, Windows environment. 

The C2, in this case, is Cobalt Strike. This is another huge draw to this course. Additionally, as this is a Red Team focused course, you will learn about Operational Security (OpSec). 

This includes topics such as bypassing or evading Anti-virus solutions (in this case Windows Defender) to a small degree as well as understanding some of the common detection points of various techniques and tools.

Of note is that the course is also a bit of a walkthrough for the labs. I like this approach. Too often in this space do we throw people to the wolves with sayings like "*try harder*". Being able to figure things out for yourself is key to success but it's nice to be able to just follow along at the start. 

Another thing of note is that, while the course is comprehensive in the amount of different things it covers, it does not dive very deep into any of those topics. Again, this could be a pro or con depending on your viewpoint. It encourages you to do research of your own instead of relying too heavily on it.

Lastly, while many courses approach things from a (Kali) Linux standpoint, RTO takes a different approach. As mentioned above, it teaches you to operate primarily from a Windows environment alongside WSL. The Team Server binary is hosted on Linux, however, this is not Kali. It is a pretty standard Ubunutu distro. The tools you'll be using are in their own folder on the Windows box.

## Labs
The bread and butter of any course in this space. The labs are hosted by SnapLabs which means they are accessed through the browser. 

You can pause and reset them at any time, although pausing them means you will need to restart the team server the next time. 

Inside the lab, you have access to a variety of different machines (File and Web Servers, Exchange Servers, DNS server, SQL server, DC, workstations, etc) to play around with and get familiar with different vulnerabilities, tools, and techniques. 

Additionally, you have access to an Elasticsearch, Logstash, and Kibana (ELK) stack instance. This allows you to see what defenders see every time you launch a different attack. Understanding what logs and other artifacts various Tactics, Techniques and Procedures (TTP) generate is paramount, so this inclusion is huge in my book.

The biggest downside to the labs is that they are not internet-connected. This means you cannot easily bring in other tools you may want to use. As I understand it, this is due to the fact that Cobalt Strike is present and Forta (understandably) wants to protect their product. Visual Studio is present however and copy-paste works so there *is* a way to get other tools and scripts into the environment if you really want to do so. 

## Exam
I cannot say much here to maintain the integrity of the exam for myself and others. Biggest thing of note is that the exam is hosted on SnapLabs and you can pause and resume the environment as needed. You have 48 hours total to complete it over four days, but honestly, you really shouldn't need all of it. There also is no report required, which was a massive plus for me.

Everything you need to pass is in the course for the most part. I would recommend researching Cobalt Strike Malleable C2 Profiles more in depth as you will need a good one to bypass Defender in both the labs (after you've enabled it) and the exam. The course covers how to do this, of course, but it never hurts to know more. Researching the Arsenal Kit (something else also covered in the course) is also a good idea.

Beyond that, I will say the exam was very straight-forward. There are other exams famous for throwing extra things in, simply to waste time, with the excuse of "*making things more realistic*" and "*testing your time management skills*", but this is not one of them. Enumerate well and you know exactly what your next step is. 

## Overview and RTAC Comparison
Overall, this was mainly a refresher for me but a welcome one. If I had to describe this course, I would do so by calling it the "*Cobalt Strike primer course*". 

If you have done other offensive security certifications or courses before, or CTFs or labs involving Active Directory or if you have some professional experience in this space already, you likely are already familiar with much of what this course teaches. The biggest thing here is learning how to do those same things through Cobalt Strike. 

And this is where I have a bit of an issue with this course being required for, or listed on job descriptions for, Red Team roles. 

My issue is not with Zero Point Security, to be clear, but this is not a "*how to red team: beginner's edition*, in my book, as seemingly many organizations are making it out to be. 

Someone who has gone through this course, based on that alone, is not ready for a Red Team role. They could confidently say they were familiar or comfortable with Cobalt Strike. They could say they understand a bit about evading or bypassing Defender. But that's really about it. 

And this is where RTAC comes in. If you wanted the closest thing to a "*how to red team: beginner's edition*", it would be RTAC. This isn't a completely fair comparison (apples to oranges, really), of course, as the course formats, goals, and other things (pricepoint is a big one) are completely different. But RTAC covered things a bit differently. 

One of the more important things, in my opinion, was the use of Beacon Object Files (BOF). Cobalt Strike, by default, uses a Fork & Run routine. To put things extra simply, it means sacrificial processes are created when you run certain things and the results returned before killing the process. As with most things Cobalt Strike, this behavior is highly signatured by endpoint protection. 

BOFs (essentially extra, custom functionality for your Beacons) instead run in the same process as your beacon. This *does* run the risk of killing your beacon if the BOF does anything crazy but it is much more opsec-safe than fork & run. Guess what you'll be making (almost exclusively) use of in RTAC?

Other things RTAC covered (and had students actually *do*) were the importance of logging, which comes in handy for deconfliction, persistance techniques (*sidenote: RTO does cover this as well but there was an interesting method covered in RTAC that isn't in RTO*), and some other ideas for staying under the radar. Ever heard of timestomping? 

While you want to stay off disk as much as possible, sometimes dropping a file on disk is unavoidable. When that is the case, making that file stand out less is key. Timestomping lets you blend in by tricking Windows into thinking it was uploaded at a different date and time than it was. What about where to upload that file to blend in better? That's covered too as well as what to name it.

Like I said before, it's really an apple to oranges comparison and in reality, they both complement each other but those are my thoughts on the matter. I'd probably recommend RTO and RTAC (in that order and in addition to some prior experience) to new Red Teamers.
