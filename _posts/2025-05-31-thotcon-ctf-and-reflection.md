---
layout: post
title:  "THOTCON in review"
date:   2025-06-04 00:00:00 -0500
categories: conference ctf
---
After being laid off by a previous employer, I decided as a way to relax, I'd try out another hacker conference. I went to DEFCON a few years ago in Vegas, and discovered that actually, I don't like Vegas very much. I did enjoy all the activities and things to see at the conference, but Vegas in August is incredibly hot, it's expensive year round, it's crazy crowded (and not in a fun way), and the decades of smokers in casinos have left their permanent mark in a way that won't ever go away, everywhere smells like garbage. I figured a smaller conference would be a lot of fun, but if it was too small, it would stop being fun again. So, I figured, the biggest city in the midwest would be a good choice!

In January, I bought my tickets to THOTCON, booked a hotel in Northalsted near the lake with credit card points, and got some Amtrak tickets for 50 bucks each way. I figured I'd make a week of it, see the sights in Chicago before the conference, and I had a great time doing so! Chicago has some amazing restaurants, some really nice walkable neighborhoods, a large public transit system, and all around I had an amazing time. 

On this trip, I brought the same Chromebook I brought with me to DEFCON. It's a servicible machine with a long battery life, it's close enough to Linux that I can function, but I don't have to do as much configuration to feel safe enough to hop on a public wireless network. Also it has all my stickers on it, and I don't want to switch. I'm sure once Google kills their Chromebook division I'll be forced to stop using it. I wiped and rebuilt my Linux container (Chromebooks give you access to a Linux shell via an LXC container), which caused me some problems. I had some trouble at DEFCON setting this machine up to run podman, and I had forgotten exactly what problems I had, so I kind of stumbled around, trying to get things to work that actually weren't necessary for proper functionality. I remember needing to set up something to do containers within my LXC container, but I didn't remember what, so I assumed that I needed to get `podman machine` working. That's a pretty heavyweight way of running containers by running a VM, then running the containers inside that VM. That would mean that I'd be running containers inside of a VM inside of a container inside chromeos. What a mess. Turns out I didn't need that. I had already done all the crostini host stuff I needed to do to get containers in containers working last time, and I could just run a container directly without needing the VM. I spent a few hours on my train ride trying to maintain enough of a cell signal to download and run some containers, just to verify everything was working. Both tethering and the onboard wifi dropped off in a few places between stations, and I kind of assume that the onboard wifi on Amtrak is more or less just a cellular modem, so that "tracks".

There were some really excellent talks at THOTCON this year. The keynote on the first day was Cliff Stoll, author of The Cuckoo's Egg. More or less, it was [this talk](https://youtu.be/1h7rLHNXio8), but a lot more fun. Cliff has so much energy when he's talking, and it's completely contagious. He used an overhead transparency projector for his talk that was basically illegible. The VC folks had a slide deck that contained the exact same information, and they'd play the game of VC turning up the brightness of the power point, projected onto the same screen with the overhead projector, until Cliff noticed and started shouting to "Get rid of that crap!" until his back was turned, at which point they'd turn it back up so the crowd could read the slide, and so on through the whole talk. He was literally climbing the walls in that auditorium, at one point.

Other talks of interest which I attended included a talk about various side channel attacks (interesting, but I wish it had gone more in depth about the practical implementation details), a talk about Terraform dependency trust, and how that software supply chain is just as ripe for attack as any other software supply chain. There was a talk about email exploits and phishing trends, one about using log analysis to find defunct domains still referenced in your logs (the subject domain in this talk was a now-defunct registrar), yet another WebDAV exploit (WebDAV has been a favorite target for years because of how many holes are in it, yet clients are still available built in to many hosts shipped with otherwise fully patched operating systems today), 

The oddest thing I did was participate in an egg challenge. An OPer gave me an egg on Friday, and I kept it safe all day and brought it back to my hotel, and on Saturday starting at noon, there were various egg related challenges. I won't go into too much detail because I think this'll be an event again in the future, but there were various egg survival challenges which I avoided, because I thought this was purely an egg survival challenge (which it is, but there aren't enough points at the end of the competition to win without participating in earlier rounds). The egg challenge was host to some high drama! There was an Eggsassin, who knocked several competitors out of the competition by cracking their eggs, until eventually she herself was knocked out by a counter-eggsassin, ending her reign of terror. This was strictly within the rules, however nobody else seemed to exercise this type of strategy. I have several ideas about how to do better at this event next THOTCON (0xE for Egg) (which sounds like it's going to a schedule of every other year), including new eggtelligence gathering, eggsassination, and propegganda techniques.

I also tried my hand at the CTF. I didn't do particularly well, but I did have fun. One of the challenges that I had a lot of fun with had to do with [RFC-2324](https://datatracker.ietf.org/doc/html/rfc2324), Hyper Text Coffee Pot Control Protocol. Someone set up a functioning, RFC-compliant (including RFC extensions), but virtual server that responded with valid HTCPCP. It was a little tricky forming these requests, however once they were figured out, it wasn't so bad. Some google-fu and we could make requests like so:

To get the current status of the coffee pot:

```
curl 172.20.2.2:8001 --http0.9 --request 'GET'
```

Some server metadata was available like so:

```
curl 172.20.2.2:8000 --http0.9 --request 'PROPFIND'
```

Requests to brew coffee could be made like this:

```
curl 172.20.2.2:8000 --http0.9
    --request 'BREW'
    --header  'Scheme: coffee'
    --header  'Content-Type: message/coffeepot'
    --header  'Accept-Additions: milk-type/Cream, syrup-type/Vanilla'
    --data    'coffee-message-body=start'
```

And reqeusts to stop pouring coffee additives could be made like so:

```
curl 172.20.2.2:8000 --http0.9 --request 'WHEN' --header  'Addition-type: milk-type/Cream'
```

Unfortunately I didn't save output, so I can't show you responses. One CTF challenge included interacting with this server in Esparanto and adding cream to a cup.

There were also some DNS challenges that let me use some of the tooling I saw in the registrar takeover talk I saw. I don't think those tools actually solved any of the challenges, but it was fun to play around with them. This particular toolchain was bbot from Black Lantern Security. I hadn't come across it before, but it seems like more of like a tool orchestration tool, something that will follow stuff in common fields on your behalf. Before we get into that, I kind of expected one of the answers about how to enumerate DNS would be to AXFR the contents of some zone, like so:

```
podman run --mount type=bind,source=/home/asdf/output,target=/root/.bbot/scans/ -it blacklanternsecurity/bbot -t bsides312.org -om json -p subdomain-enum
```

But no dice with the axfr.

One of the gripes I have about this tool is that the actual scan output is always sent to a file rather than to stdout. The docs say that the json output module (specified here with `--om json`) is supposed to output to stdout by default. It doesn't. So mapping a directory in is pretty much the only way to get the contents of the scan, and that's not included in their docker instructions. Oh well. The contents of a scan contain documents that look like this:

```
{
  "type": "DNS_NAME",
  "id": "DNS_NAME:5839af9a266a960b2b719832c9b8480a68cad40a",
  "uuid": "DNS_NAME:f2d938db-f137-4c7a-8ace-74ba7643e3dc",
  "scope_description": "in-scope",
  "netloc": "mud.bsides312.org",
  "data": "mud.bsides312.org",
  "host": "mud.bsides312.org",
  "resolved_hosts": [
    "54.203.105.0"
  ],
  "dns_children": {
    "A": [
      "54.203.105.0"
    ]
  },
  "web_spider_distance": 0,
  "scope_distance": 0,
  "scan": "SCAN:9c78fb8eb1c234a161ae595a7b7c492559221341",
  "timestamp": "2025-06-01T00:48:02.765673",
  "parent": "URL:f79da84f4233fc6358da5611bbb76ddf8c780698",
  "parent_uuid": "URL:34e17d0d-5582-484f-8472-289d406c538b",
  "tags": [
    "soa-error",
    "ns-error",
    "mx-error",
    "a-wildcard-possible",
    "srv-error",
    "a-record",
    "cloud-ip",
    "txt-error",
    "cloud-amazon",
    "in-scope",
    "subdomain",
    "cname-error",
    "wildcard-possible",
    "aaaa-error"
  ],
  "module": "host",
  "module_sequence": "httpx->excavate->host",
  "discovery_context": "URL_UNVERIFIED has host DNS_NAME: mud.bsides312.org",
  "discovery_path": [
    "Scan fleecy_joe seeded with DNS_NAME: bsides312.org",
    "speculated OPEN_TCP_PORT: bsides312.org:443",
    "httpx visited bsides312.org:443 and got status code 200 at https://bsides312.org/",
    "HTTP_RESPONSE was 63.76KB with text/html content type",
    "Excavate's URLExtractor emitted URL_UNVERIFIED https://mud.bsides312.org/, because HTTP response (body) contains full URL",
    "URL_UNVERIFIED has host DNS_NAME: mud.bsides312.org"
  ],
  "parent_chain": [
    "DNS_NAME:b8585c2f-3296-45b9-80ab-24f8e615796b",
    "OPEN_TCP_PORT:f4483d60-b478-4e04-bb9d-034e09a1d771",
    "URL:34e17d0d-5582-484f-8472-289d406c538b",
    "HTTP_RESPONSE:34185f94-5839-4978-b014-644b450570d9",
    "URL_UNVERIFIED:42d65bea-3730-46c4-aee0-85910d78037c",
    "DNS_NAME:f2d938db-f137-4c7a-8ace-74ba7643e3dc"
  ]
}
```

Like I mentioned before, this tool seems to do a few things I would normally do manually, automatically. This document shows a check for open ports after a subdomain has expired, and has identified that https is open and replying with a 200 on this server. Cool, but I don't think this tool discovered anything that I didn't already know about from nmapping hosts here and there and trying things myself before running it, but it seems like it's supposed to be sort of an all-in-one type thing, and I should use it as a starting point rather than the last thing I try.

In the future, I think that it'd be an appropriate CTF strategy to just look at each of the challenges before attempting any. One of the challenges was to meet one of the OPers outside at noon to eat surstromming, worth 200 points. Most other challenges were worth 10-30. Knowing about these things in advance would be better than finding out about them at 3 PM after they're done.

THOTCON programming ran on Friday and Saturday, and on Sunday, BSides312 took place at the same venue. I got a free ticket to BSides312 through a really simple challenge, there was a ticket code that was pretty easily findable in git history, removed from a comment a few weeks ago. I'm not linking it here, because they'll probably do it again in the future too. BSides put up a fun MUD, based on DerbyMUD which was also used a couple years ago at DEFCON safe mode. I don't really know anything about MUDs, but I gave it a try anyway. I moved around kind of aimlessly, trying to get used to how things work, and ultimately moved on to other things. There was also the BattleTech MUX that will be at DEFCON this year, which again, I joined, I tried a few commands out, and ended up going to more talks. I might try to play around with that some more in the next several months, since this will be a contest at DEFCON with prizes, but I doublt I'll get very far, since this isn't really my typical bailiwick.

The first talk of the convention at BSides was about community, and having each other's backs. It kind of buttered the audience up, said some very kind things about hackers as a group, and had us do some psychological tricks to make us feel more together and close. While I recognized these types of tricks, I still fell for them, and came out of this talk feeling good about the folks around me. Everyone's looking for acceptance and community and connection, and wants to be understood on their own terms, and it's difficult to find that in the world. Then I went to the next talk room, and listened to some guy talking about how badly he wants the death penalty for petty crimes while I waited for the talk to start. Kind of ruined the vibe.

There were some really interesting talks about stuff I was familiar with, and even more talks about things I wasn't. The only Cloud talk introduced me to a few tools. [AWSPX](https://github.com/ReversecLabs/awspx) is a visual tool for exploring various aspects of your cloud environment, and [cartography](https://github.com/cartography-cncf/cartography) is the Neo4j graph cloud data ingestion tool that sounded cool every time I thought about personal projects to work on. I look forward to trying them both out in the future.

Through all this, I met some really great people who are incredibly smart, curious, and fun to be around. The OPers worked so hard to put on a great event (both were a ton of fun), the speakers put together talks that were incredible, the villages covered a lot of the classic bases, but also included some really unique stuff. Special shout out to the Eggsassin, you're good company and I hope to see you again!
