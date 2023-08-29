---
layout: post
title:  "DefCon Wrap up and thoughts"
date:   2023-08-29 12:41:18 -0500
categories: jekyll update
---

DefCon is the largest hacker meetup in the world. It happens every year in Las Vegas in the first couple weeks of August. I was lucky enough to have WalMart send me for work this year, and

Before the trip, I wanted to be prepared and have all the equipment I'd need while I was there. I didn't want any trouble with accidentally leaking any credentials, so I created a new email address and got a tracfone phone and phone number just for the conference. I still brought my regular phone in case of emergency, I just left it turned off in my hotel most of the time. I also picked up a used chromebook off Craigslist to do all the CTFs and labs I wanted to do, which worked out pretty well, for the most part. It was an i5 with 8 gigs of memory, and it was capable of running a Linux container and giving me a shell to do whatever I wanted in that container. It was a bit tricky to set up the container-in-container configuration so I could run podman within the ChromeOS-provided LXC container, but eventually I got that working during the conference. Prior to the conference, I made sure all my devices were up to date, that they weren't creating any unencrypted http traffic, and that wifi and bluetooth were turned off, just as safety precautions. I also brought a Flipper, hoping that I could learn a few things, but the only thing I ended up doing with it was cloning my room key.

The conference takes place across 3 hotels (The Flamingo, Harrah's and the Linq) and Caesar's Forum, which is a huge convention center. Even with this much space, it seemed crowded, and like there wasn't enough room for everything that was going on. I stayed across the road at Caesar's palace, because I didn't fully understand that the Forum and Palace weren't the same thing. I'd recommend future attendees stay at the Linq, which seems like the most centrally located hotel for the conference. Even if you stay at Ceasar's Palace, it's not too far of a walk, not too big of a deal, but it is a minor inconvenience stumbling home at the end of the night

The first thing to do is get your badge, or go to "LineCon" as some call it. Basically folks try to make standing in line interesting and as much of a party as possible with beach balls, music, and just a lot of crazy behavior. The line does actually go pretty quickly once it's moving on Thursday night. Some people get to the line as early as the previous day, which is nuts to me. I showed up about 2 hours before they started handing and the line was already pretty long. They started handing out badges at 5 PM on Thursday, and that was really the only event on Thursday. Badges are required for all parts of the conference except for the talks that happen before the badges are given out. I saw a couple of those talks, waited in line for my badge, and was back at my hotel by 8 PM.

Friday, Saturday, and Sunday are all a blur for me since there was so much going on. There was so much to see and not nearly enough time to see it, and yet I managed to see so many interesting talks and so many cool demos at the various villages. Here are some of the things that stuck in my mind:

- Talk about hacking 15 year old Lexmark printer firmware at Pwn2Own, and making $20k in the process
- Talk about hacking Boston metro transit cards to add an unlimited amount of money
- Talk about how, when you do identify a vulnerability or security issue in something, even if you disclose it in good faith, you may be arrested, the police may arrive at your house with a warrant to take all your electronics. There is no solution to this as of yet other than keeping your mouth shut
- Demo of audio and visual deep fakes
- CAN Bus hacking demo
- Fuzzing cloud provider APIs for lower environment endpoints to potentially avoid CloudTrail ([available here](https://youtu.be/61C_lEQ5qNM) the same talk at a different conference)
- How P25 trunked radio works and how to monitor modern P25 systems with cheap SDR dongles
- The Recon Village's cold calling event, where people volunteer to try to get 3 pieces of information (like SSID, store number, and address) from randomly dialed numbers. Very entertaining to watch
- Removing tape seals without leaving evidence using a syringe full of rubbing alcohol

There was lots more, but those were the things that really stuck in my mind.

There are a ton of CTFs with many different focuses. I tried my hand at the Recon CTF (which involved scraping social media for information that might lead to a flag), the Cloud CTF (which was really difficult, and I'm proud to have finished in the top quarter), and I started a random CTF unaffiliated with any of the villages as well. If you want to do these, find a quiet place and tether to your phone over USB. They have a wifi network for the whole conference, but you need a certificate to access the secure network, and you've got to wait in line to get one.

Us cloudsec folks didn't really cross paths except for lunch on Friday and Saturday. If you want to see people from work, coordiation is hard. I'd recommend either planning lunch or dinner all together for a couple days, or just expecting not to see each other.

Lastly, now that DefCon is over, there's now 2LineCon, as in the two lines that show for a positive COVID test, for all the people who caught COVID at the convention. There aren't too many mitigations for disease in place, most people aren't masking, and the little C02 monitor I carried around (C02 being a useful proxy for air recirculation) fluctuated from 1500 PPM to 2500 PPM depending on where I was, which means air circulation isn't great. There are 30k people coming from all over the world to this convention. Either expect to be sick after the convention, or wear a mask.

So, here are my takeaways:

- Download the HackerTracker app early and plan out what you want to see well before the conference starts. You won't have time to plan once you're there.
- info.defcon.org is your friend
- Stay in a con hotel
- Be ready to walk a ton
- Mask, or plan to be sick
- Plan team meetups beforehand
