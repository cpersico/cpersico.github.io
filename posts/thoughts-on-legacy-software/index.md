<!--
.. title: Thoughts On Legacy Software
.. slug: thoughts-on-legacy-software
.. date: 2019-07-19 22:27:38 UTC-03:00
.. tags: software design, legacy code, experience
.. category:
.. link: 
.. description: 
.. type: text
-->


Once upon a time, I joined a company where I was faced with what had become their Gordian Knot. 


## The Beast

During the development of the newest company product (an on-prem server-client architecture) the decision was made to adapt the previous one (a desktop application) as a component behind a CLI. The application was a massive monolith containing UI code, 
internal states, license management, a knowledge database, the UI main loop, etc.  
After the refactor the resulting component was a super complex monolith engine **over 100K LOC** in size.

In order to interact with this engine, a CLI was created but it kept the internals of the desktop app, which had to reconstruct states from previous sessions from files so the input was exceptionally convoluted, it 
was a single endpoint that required a combination of massive and complex XML files and, depending on which files 
it received, it would determine what task to perform.  Every functionality was interconnected through a huge core class that was in charge of everything, from initialization, execution, analysis and output generation.   
Besides, the CLI would keep behaving like a desktop app, recovering from errors and using
threaded code which was no longer necessary.

## The Problem

Our day-to-day was full of support tickets where we had to diagnose what happened in this huge codebase where almost anything was possible. 
Also, the tests required mocking of the core class and that made them complex and often unreliable. 
The suite took 40 minutes to run on our CI environment.

We scheduled a little bit of time from sprint to sprint for maintenance tasks, but its progress was very limited.
We need to simplify this, to discard all the desktop internal code and, most importantly, redefine the input to be just what we needed to perform each task (actually it was just a couple of strings) but were to start? we couldn't just stop working on the product.

For years it had shackled progress and innovation. Eventually, the opportunity for improvement appeared and the solution was very simple.  
After that, all pieces seemed to fall in place on their own and the gears started turning.

I learned a valuable lesson about software design from that.



## The Action Plan

The stack at the moment consisted of the platform of the new product, the actuator (in charge of invoking the engine) and the engine itself.
The ownership was divided into two teams, one in charge of the platform and the actuator and my team, in charge of the engine.

For illustration purposes here's a simplified diagram.

![diagram v1](/images/thoughts-on-legacy-software/vnc_1.png)

We needed to tear that monolith down into services, each one in charge of a single functionality. Some of the problems we were facing had to
do with coordinating our sprints with the Platform's team to make sure nothing broke, but the pace of the projects made it
impossible as there were many fronts to attack. On our side we couldn't do much about the input either, because the core class was traversal to
every functionality, there was no way that refactoring this would work. We needed to start from scratch and simplify each service all while honoring the
contract we had with the Platform.

We were stuck with this for quite a while until I had the chance to develop a new engine that would integrate with a completely different system.
Since we didn't want to limit ourselves from the start (and we had the urgency of coming up with a new product) after some convincing I was
able to join forces with the other teams to sync and implement a public interface for each service.
The signature of every endpoint would be clean and concise, requiring only the necessary to perform each task.
The new engine would be created from scratch, so this was ideal while the old one would receive whatever it was it needed through a special parameter `extra_params` in the endpoint.

![diagram v2](/images/thoughts-on-legacy-software/vnc_2.png)

Here's what we achieved:

* The other teams became consumers of a public interface instead of the CLI.
* There was no need (or at least much less) to sync with other teams to do maintenance work on our stack.
* We were able to break down the monolith **one functionality at the time** under the condition of committing to the new public API.
* The new services were coded from scratch, so they were simpler and easier to test, **because we were very familiar with the subject matter**.
* During the rollout process of new standalone functionality, we could do AB testing vs the old engine.
* The test suite takes < 5 min for all projects.
* The public interface required only the minimum necessary to do each task.


## The Aftermath

After a couple of months it looked like this:

![diagram v3](/images/thoughts-on-legacy-software/vnc_3.png)

We reduced the LOC count to **<10K** and greatly increased the testability. Eventually, the support tickets regarding our stack became less and less frequent. The monolith engine it's still there, but now it has only one responsibility. As for the public interface layer, it
also has the responsibility of transforming the output from the new services (which is our ideal output) to the old output generated by the 
desktop app (because the Platform expects that), but that is another battle...


## Lessons Learned

This is a story about imperfect software design, about working on something huge under constraints beyond our control, patience and choosing your battles.
What I learned is that the opportunity for change **_may_** come someday and you may not have control over it, all you can do is **be ready**. 
Understand the problem in deep, know your constraints, think outside the box and spend your time, not frustrating, but preparing to make the best 
out of the opportunity when **_(if)_** it comes.

And most importantly, have patience and a clear mind.