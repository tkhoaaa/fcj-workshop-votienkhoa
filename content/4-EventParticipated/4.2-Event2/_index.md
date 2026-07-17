---
title: "Event 2: Cloud Architect - Game Show"
date: 2026-06-20
weight: 3
chapter: false
pre: " <b> 4.2. </b> "
---

# Event Reflection: "Event 7: Cloud Architect - Game Show"

### Purpose of the Event

- Reinforce foundational knowledge of cloud services (AWS) and system architecture design patterns.
- Sharpen situational analysis, breaking down business requirements to choose the optimal technical solution under time pressure.
- Strengthen teamwork, strategic communication, and risk management in a competitive, head-to-head environment.

### Organizers & Participants

- **Organizer** - AWS Study Group
- **Players** - 8 competing teams (each team had 5 members, randomly selected after the registration round)
- **Role** - Competing contestant

### Highlights

#### Head-to-Head competition format
- The 8 teams were paired for direct elimination matches. Teams took turns answering sets of case-study questions with increasing difficulty.
- The team that accumulated more points advanced to the next round.
- **Tie-breaker rule**: If two teams finished the question set tied on points, "Question 11" (Sudden Death) was released. Whichever team buzzed in and gave the correct answer first won immediately.

#### Technical focus: AWS Service Selection
- The questions weren't about dry theory; they put players in real-world scenarios. For example: "A system is hitting a bottleneck processing orders on Black Friday and needs to decouple its microservices without losing data. Should you use AWS SQS or Amazon Kinesis?"
- Contestants had to know the limits, characteristics, and pricing model of each AWS service to make the most accurate architectural decision.

#### Risk-management tactics through the Skill set
- **Minimum Risk (1 use per match)**: A defensive skill. Used for questions where the team was still uncertain or working from shaky assumptions. A wrong answer preserved the score (no deduction). A correct answer earned half the points for that question.
- **Star of Hope (1 use per match)**: An offensive skill (high risk, high reward). Activated when the team had fully analyzed the problem and was 100% confident in the answer. A correct answer earned double points (x2); a wrong answer deducted the corresponding points (x2).

### What I Learned

#### System Design Thinking
- **Don't guess, don't hide uncertainty**: The game-show setting showed that rushing to pick an AWS service just because it "sounds familiar" usually costs points. Architectural thinking requires surfacing every trade-off and staying anchored to the original requirements.
- **Minimal and on-target**: Designing architecture is like writing code: the best solution is the simplest one that meets the actual need. Don't "invent" extra complex services when the problem only calls for static storage (S3) rather than an expensive file system (EFS).

#### Technical Architecture
- Strongly reinforced my ability to distinguish the use cases of similar AWS services (for example: when to use DynamoDB vs RDS, ALB vs NLB, CloudFront vs Global Accelerator).
- Gained a clear awareness of cloud design "anti-patterns," especially design mistakes that blow the budget or create a "single point of failure."

#### Strategy & Teamwork
- **Surgical communication**: Short discussion windows forced members to communicate directly, skip vague opinions, and go straight to verifying the feasibility of each AWS service.
- **Risk management**: Learned to assess the reliability of a line of reasoning before deciding to "go all in" with the *Star of Hope* card or to play it safe with the *Minimum Risk* card.

### Applying It to My Work
- **Building test cases for architecture**: Applying the competition mindset to real work: before deciding to integrate any AWS service into a project, I need to spell out the verification criteria (load requirements, cost, latency) just like solving the case studies in the game.
- **Planning problem-solving**: When facing complex tasks or system bugs, always follow the principle of decomposing the situation. Break the problem into clear verification steps instead of offering a solution based on gut feeling.

### My Experience at the Event

Taking part in the **Cloud Architect Game Show** offered a completely different lens compared to one-way seminars. This was where architectural design thinking got thrown into a real, time-pressured challenge.

#### Parallels with my programming philosophy
The most interesting thing about the event was how precisely it reflected the working philosophy I always follow: "Caution matters more than Speed." Facing a tricky architecture question, the urge to buzz in and answer fast is strong. But our team agreed on a principle: never give an answer based on vague guessing. Every choice (for example, picking Lambda over EC2) had to rest on clear assumptions extracted from the wording "traps" in the question itself.

#### Team coordination and decision-making
Working with the other 4 members of my team was a great communication experience. Debating to eliminate wrong answers was tense but deeply logical. The most memorable moment was when the team decided to activate *Minimum Risk* on a complex network load-distribution question, and used *Star of Hope* at just the right time on a familiar decoupling design scenario, turning the game around.

#### Lessons learned
- Truly understanding one service inside out (its nature, strengths, and weaknesses) matters more than knowing the names of dozens of different services.
- In system architecture, as in programming, every decision to change or integrate a technology must be clearly verifiable. Don't apply complex architecture patterns on a whim if a minimal solution can fully solve the problem.

#### A few photos from the event
![Photo taken at the event](/images/4-EventParticipated/event2.jpg)

> Overall, the event wasn't just an entertaining playground but a sharp, practical test. It reinforced my belief in goal-oriented working principles, putting accuracy and careful analytical thinking first when facing any cloud system.
