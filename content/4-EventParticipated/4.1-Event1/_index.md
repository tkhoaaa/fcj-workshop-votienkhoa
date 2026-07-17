---
title: "Event 1: AWS First Cloud AI Journey — Community Day"
date: 2026-07-23
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# AWS First Cloud AI Journey — Community Day

I spent a whole Saturday morning up on the 26th floor of Bitexco for the AWS First Cloud AI Journey Community Day, and honestly it was one of the more useful mornings I've had outside of my everyday internship tasks. It was organized by the AWS Study Group, and the room was a mix of cloud architects, GenAI engineers, DevOps people, and students like me who were mostly there to soak things up and pick up whatever we could. I went as a participant, not expecting much beyond the usual "here's a cool AWS service" pitches. But that's not how it went, which is why I bothered to sit down and write this.


## The sessions that actually stuck with me

There were six talks across the morning, from 8:30 to noon. Some landed harder than others, so I'll be honest about which ones actually got to me.

**"Context Is Everything" by Tinh Truong (GoTymeX).** This is the one I keep thinking about. His argument was that when AI hands you garbage, it's usually not the model's fault, it's that you didn't give it enough to understand the real problem. He broke context down into your goal, your situation, your constraints, and the relevant evidence, and the line that stuck with me was "context turns a vague request into a solvable problem." What I liked was that he also warned against the opposite mistake, what he called the "internet puller" problem, where people paste an entire PDF and fifty screenshots into one chat box and then wonder why the answers keep getting worse. More context doesn't mean better context. As someone guilty of exactly this habit, I felt a little called out, but I needed it.

**"GenAI-Powered Auto Audit for AWS Workload" by Pham Nguyen Hai Anh (G-AsiaPacific).** This one leaned more enterprise than I usually care about, but the way she described business-user pain points was sharp. She walked through Amazon Q Business, the natural-language querying, the 40-plus data connectors, the guardrails, and a demo of a PM assistant that drafts meeting minutes and schedules the next meeting on its own. I'll admit the deeper auto-audit details went a bit over my head, but the general idea of pointing an LLM at your own infrastructure to sniff out compliance gaps felt very practical.

**"From Edge to Origin: CloudFront as Your Foundation" by Nguyen Tuan Thinh.** I walked in thinking CloudFront was just "that CDN thing," and walked out a little scared, in a good way. The point about the pay-as-you-go paradox stuck with me: the 1TB free tier is generous, but a viral spike or a DDoS attack can turn into a five-figure bill overnight. The horror-story numbers he threw out (over $100K in extreme cases) are the kind of thing that makes you go set up billing alerts right away. Which I actually did, more on that below.

**"36 hrs with LotusHacks: Building UTMorpho" by Team VIB.** This was the fun one, and probably the one the whole room enjoyed most. The team retold their 36-hour hackathon run from "hour zero with a blank mind" to a working demo, and they were refreshingly honest about the bugs and the panic in the middle. The idea of focusing on solving a single problem instead of piling on features is something I need to hear over and over, because I always want to add this and that.

**"Non-Determinism of 'Deterministic' LLM Settings" by Duc Dao (Cloud Kinetics).** This one quietly untangled a knot for me. He explained why setting temperature to 0 still doesn't give you identical outputs, basically because floating-point addition running in parallel across GPU cores isn't associative, so shifting the order of the sum a little is enough to skew the probabilities and change the chosen token. Add server load balancing and hardware drift at the API layer on top of that. I sat there realizing this is exactly why my LingoRise exam generator occasionally spits out slightly malformed JSON even when I thought I'd locked everything down.

**"Enterprise-Grade Multi-Agent System" by Vy Lam (VPBank).** This was the most tightly structured talk, about credit scoring for startups. The core problem is that banks demand three-plus years of financial statements and startups just don't have them, so they get rejected even when they're very promising. Her solution was a "virtual credit committee" of specialized agents, a financial analyst, an IP evaluator, a risk auditor, working together under a central orchestration layer, instead of one single agent carrying everything. She also laid out a five-layer security model, perimeter, network, identity, application, and data, which was a lot to absorb but a good checklist.

## What I actually took away

The thread connecting almost every talk was this: the model is rarely the hard part. Context, memory, reliability, and guardrails are where the real work lives. That made me look at my own project differently.

A few things mapped straight onto LingoRise:

- The multi-agent talk validated a choice I'd accidentally half-gotten right. I split LingoRise's AI logic into a separate exam-generation flow and a separate Writing-assessment flow, and it turns out that's exactly the "split into specialists" instinct Vy Lam was arguing for, just at a smaller scale.
- Duc's non-determinism talk finally explained why I built that `extractJsonObject()` regex fallback parser back in Week 5. At the time it felt like a patchy hack. Now I understand it's a legitimate defense against LLM output drift, and I feel a lot more at ease keeping it around.
- The security session lined up neatly with the hardening I did in Week 10, rate limiting, input validation, prompt-injection defense. Hearing enterprise folks describe the exact layers I'd been fumbling toward on my own was pretty reassuring.
- And after the CloudFront talk I went home, set up billing alerts properly, and looked into Origin Access Control for my S3 resources. That's a very dry, very useful outcome, and exactly the kind of thing I wouldn't have prioritized on my own if I hadn't gone.

#### A few photos from the event
![Photo taken at the event](/images/4-EventParticipated/event1.jpg)