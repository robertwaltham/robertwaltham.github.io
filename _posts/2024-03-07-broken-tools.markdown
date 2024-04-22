---
layout: post
title:  "Engineering Teams With Broken Tools"
date:   2024-03-07 08:59:06 -0700
categories: engineering
---
![Header](/assets/images/header.jpg)
### Can people use this tool without you?

I've seen this pattern on a number of different software engineering teams. A need is identified for some sort of tooling which is to be built to solve some sort of internal problem. This can include some sort of configuration tool, or tool that provides access to internal data, or a tool that passes data between systems, et cetera. An engineer has a bright idea of how to solve this and runs off to build it. Initially this is a success where - with the help of the engineer who built it - the tool solves the problem neatly.

The problem is so often that this engineer who built and maintains the tool is a single point of failure. If that person is unavailable (vacation, busy with other work, etc) no one understands how the tool works. People or teams quickly become blocked on work that relies on the tool. When that person leaves (promotion, leaving the company, etc) it may become a crisis, where on short notice the tool must be stripped out and replaced with something new. 

### I have been that guy

 I worked at a startup a few years ago where we built a customizable forms and accounting app for the oil and gas industry. The iOS client was a bit rough and ready but functional, and there were a handful of customers using it on a daily basis. Clients would contact account managers about problems in the field and the account managers would pass it off to the tech team to debug. Most of the client debugging involved using an S3 client to download zip files of logs from the individual sessions for each of the client's devices, and manually reading logs to find problems. This obviously took a lot of time and made it difficult to effectively support our clients.

In an effort to reduce the friction in support work, I built a little tool in Node that would ingest the zip files from S3 into an Elastic Search instance, providing an quick and dirty interface for searching logs. This worked really well and cut down the amount of time it took me to debug issues immediately. A few months after I left the company I had lunch with some former colleagues and I asked them about the tool. They said it stopped working and no one could figure it out, so they went back to manually downloading files from s3. 

### So how do you prevent this from happening?

- Let proof of concepts be proofs of the concept 
- Treat tools like fully fledged software projects
- Don't overlook the cost of maintenance 

### Broken gets fixed, Shoddy is forever

A number of projects I have worked on have fallen into what I call the proof of concept trap. This where the PoC of is not a proof of concept, and is actually more like a fully fleshed out MVP. This gets shipped as-is and, the result is something that works and fills the need; however, has a lot of sharp corners and bugs that only the builder understands. The trap is that from the outside, the tool looks like it is fully functional and "done" and thus priority will likely be given to client facing work instead.

If a proof of concept proposal looks like the end goal is shipping a fully end-to-end working solution consider cutting scope. Cut until the only output is the bare minimum to measurably show the value of the tool. The goal is to show the value and to advocate for the time and resources to build out the full solution, including time for bug fixes and maintenance.

One of the best PoCs I've built was a quick and dirty python script to dump JSON from an API, which I imported into excel and spent an hour massaging into a graph. This allowed me to present the data to the intended audience, and get buy in on building and maintaining a proper solution to generate that kind of report. 

### I'm a real software project now!

Modern software practice includes building out tests, documentation, process, deployment pipelines, and more. This enables teams to build on and maintain the software, to make changes or address production issues without fear of breaking the software. Knowledge about the inner workings of the software can be shared between individuals and teams, and silos within engineering orgs can be broken down. 

Internal tools are software projects. They have all the same complexities and risks as client facing software, and should be treated in the same way. Tests and documentation for tools should be built for the same reason as for other projects, they enable other engineers to learn and contribute to the project. If an individual is working on a tool, including requirements for documentation and tests in their roadmap is important to facilitate handoff. 

Feedback loops are also an important part of a software project, setting up communication between users and engineers. Building appropriate feedback loops for internal tools can be extremely useful for highlighting single points of failure, and mitigating proactively. 

As tools are often less impactful to the business than client facing software, it's important to be pragmatic about how much time to invest on such things and only build what is necessary. [^1]

### Sticking together a team with glue

The maintenance for tools is often not planned for, forgotten or de-prioritized and the single point of failure may result from this. An engineer takes it upon themselves to fix and upkeep the tool outside of the planning or estimation or roadmaps. This work gets bundled in with all the other "glue work" which keeps teams and orgs running smoothly. 

It's important to identify this work and highlight it as part of the lifecycle of the tool. This is an opportunity to spread around the glue work, and to share knowledge. 

### Don't let your ego cloud your judgement

Engineers love building tools as it's a chance to break out of architectural constraints, outdated tool choices and general cruft of long term projects so I can understand why we are so eager to run off and build them. It's far too tempting to shortcut straight from ideation to delivery for a tool project, throwing proper process and pragmatism to the wind. 

I would suggest a pragmatic approach would be to write a proposal that includes at least:

- Diagnosis of the problem to be solved by building a tool and the value to be gained by it
- Identification of the users
- Estimation of the costs to build, deploy and maintain 
- Exploration of alternatives (build vs buy, saying no, building as part of existing, etc)
- Summary of the value proposition for the tool weighing costs vs output

[^1]: I'd argue this applies to all software, not just internal tools