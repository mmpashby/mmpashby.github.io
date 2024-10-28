---
layout: post
title: "Influencing Change With The Paved Road"
date: 2024-09-10 12:00:00 +0100
categories: [tech, devops, architecture]
tags: [tech, devops, architecture]
---



## Introduction

Let me say this upfront—if you're new to the tech world, you'll soon discover that one of the toughest challenges you'll face is influencing change. That's right, you can't simply rely on the classic "trust me, folks" approach. Building trust, helping teams adapt, and being in the trenches alongside them are essential. Throughout my career, I've influenced change in many tech teams, and every time, there's always something new to learn.

In this post, I'll explore what change really means, how we can influence it, and why it's such a critical skill in tech. I'll also introduce the concept of "The Paved Road," a method I’ve found to be highly effective in guiding change within technical engineering teams, where strong opinions and scrutiny are both common and justified.

## What is "Change"?

The dictionary definition of "change" is simply to "make (someone or something) different; alter or modify." Let me give you this insight for free: changing something without a solid value argument or a clear problem to solve will quickly become a huge cost sink. Much like personal change, we know that nothing happens overnight. You need to start by defining what you want to change and what value that change will bring.

For example, imagine Zita, who owns a coffee shop. Regular customers have been complaining that the takeaway coffee gets lukewarm very quickly after they leave. This is a genuine business risk because if enough regulars are unhappy, they may stop buying coffee altogether. This situation demands immediate investigation, and if the complaint is justified, a problem statement must be crafted. Let's say the investigation reveals that the takeaway cups aren't keeping the coffee warm enough. Now that Zita has a clear problem statement, they can focus on finding a solution to make their customers happy.

In tech, problem-solving is at the core of what we do, and often those problems require solutions that demand input and discussion from the engineering team. Some of the most crucial problems are those that can be addressed by defining standards. Standards call for patterns, and patterns are repeatable in software engineering. However, agreeing on these patterns and then building, measuring, and adopting them is no small feat. This is where The Paved Road comes in… read on if you dare!

## Influencing Change With The Paved Road

Let me introduce you to a rough-and-ready diagram I’ve refined over the past couple of years and then let's go through the numbered points on the flow to understand each component. Disclaimer: Any of the tech choices displayed in the diagram are not solid requirements and they are just there coincidently as best fits at the time I was refining this Paved Road implementation.

![The Paved Road](/assets/img/media/paved-road-uj.png)

If you're wondering, the Build, Measure and Adopt sections are intentional. I beleive you should always build first, and then measure immediately to see what the state of play is. You then build a Developer Experience (DX) to encourage adoption to get those usage metrics pumping. I will do a much more detailed post on Developer Experience (DX) at a later date, because this is also a topic I have experience in and I am very passionate about.

1. **Create an RFC** - This is the most important step in the whole flow. What is an RFC if you have never authored or encountered one before? An RFC is a document that describes a requirement/problem statement, and the proposed solution to solve/fix that requirement or problem. In my experience the RFC process should be kept very lean to encourage participation and collaboration. The following headers are all that are needed:

- **Abstract** - This can be a quick TLDR on the purpose of the RFC, so that stakeholders and collaborators can get a fast piece of feedback on what they're looking at and if they feel they can provide some valuable feedback.
- **Problem** - This section should be used to describe the problem or requirement that you want to address. You should make sure to provide as much information as possible to describe the problem, so that fellow contributors/commentators have all the information to hand.
- **Solution** - This section is describe the solution you are presenting to the audience