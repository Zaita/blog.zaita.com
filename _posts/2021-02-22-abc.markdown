---
layout: post
title: 'Digital automation of security assurance work-flows'
date: "2021-02-22T00:00:00.000Z"
description: "Reducing noise for security professionals by automating the security assurance life-cycle"
categories: [certification, accreditation, risk, security assurance, automation, work-flow]
---

Note: This post is not about selling a product. The security assurance work flow product I am describing is open source and free for everyone. We even provide a default configuration that is based on our own internal security assurance life-cycle for free. This is not a sales pitch. If you'd rather just jump straight to the [SDLT Wiki](https://github.com/NZTA/SDLT/wiki), please feel free.

## Copying and evolving

At the end of 2018 I attended [Kiwicon](https://2018.kiwicon.org) and enjoyed a talk from Slack about their [Security Development Tool](https://github.com/slackhq/goSDL). This was an interesting talk as I am incredibly lazy... and enjoy automating tasks that do not provide any value add from my time. Coincidentally, the following Monday I was approached by one of our engineering managers to automate security assurance practices into their devops processes better. They were doing their own thing as the organisation had failed to provide them an adequate security life-cycle. Aha! I thought, the Slack SDL fits this perfectly.

Fast forward 24 hours and we had a working copy of the goSDL running internally, and we were most disappointed. The goSDL provided us some check-lists, but they were really aimed at assessing a single product through a largely static process. We have >1,600 systems. It was very rudimentary and required quite a lot of knowledge by the submitter to complete. While it did provide a risk rating, it didn't provide any context or next steps. We determined that we'd struggle to gain value from it, but we really liked the concept.

I spoke to the Head of Security and asked for a few weeks to expand on the concept with my own proof of concept. Two weeks later I had a demo of what I had nicknamed the SDLT or Security Development Life-cycle Tool. The user could pick from different deliverable types and then get assigned tasks relevant to their delivery. Everything was handled in this simple web-app. The feedback was immediately positive and the organisation offered to fund a version 1.

*Note: Funny enough, a member of the devops team thought the proof of concept was a real product and made an actual submission for a change he was working on, so that was definitely something that helped me get funding.*

## Building and selling the concept

In my mind, I wanted to have a customisable work-flow system for our security assurance life-cycle. I researched and reached out to some vendors to see what was in the market, but the same four problems always came up.
1. The licencing costs would be like $500k/pa 
2. The product didn't come with any default work-flows, we'd have to customise our own
3. They had broken risk assessment concepts 
4. They required programming or scripting

This didn't sit well with me. Why would I pay $500k/pa only to have to build all the logic I wanted. Why can't I have something that is effectively drag-n-drop with simple customisations. I really wanted something that allowed me to build simple representations of flow-charts. Some of them talked about their alignment to the New Zealand Information Security Manual, but when we inquired more they were just dumb control lists that weren't part of a wider process. Nobody provided an out of the box, easily customisable end-to-end assurance life-cycle.

I had some further discussions with the Head of Security and decided to bite the bullet and take the funding to build our own version 1. Surely if we built something useful for us, it'd be useful for other organisations? So why not open source it? We're a public sector organisation, so there is no benefit in keeping it closed source. After some.. tough discussions with our procurement team, I was able to get a contract sorted that ensured the product would remain open source under the BSD licence. The organisation would pay a retainer to a local development company to manage the project for us... win win fucking win!

I reached out to contacts I had at Catalyst NZ Ltd who an open source development specialist organisation to see if they'd be interested in coming on board as our development partner; they jumped at the chance. We would be the first NZ Government Agency to fund an open source project. I wanted to take it a step forward and got approval to open source our own internal flows and configurations. This means that anyone who picked up the SDLT would be able to start with our working flows as a base and improve from there. A full security assurance tool out of the box for no cost.. fuck yes.

## What the hell am I building?

Well. I want something that is like a flow chart. The user would see a set of questions on the front end, and we'd have some basic logic on the back end to only do relevant things. Right, time for some key principles.
1. Shortest Path First - The user only sees or answers questions that are relevant.
2. Only what is needed - Only give the user tasks that are necessary
3. Reduce the noise - Try to do everything without a Security staff member as much as possible
4. 80% Hands off - 80% of all submissions shouldn't require any special effort
5. Conquer the world - Start with security, but rope in other teams and stakeholders with tasks

These seem like a good place to start.  My concept was a security assurance work-flow tool that would handle >80% of all deliveries we did without requiring a dedicated security resource. We'd handle our obligations under the Privacy Act, Public Records Act and Official Information Act through the tool. We'd support different deliverable types including proof of concept, software trial, software as a service adoption, project release, technical change and bug fixes. We'd deliver this as a Software as a Service web application. 

The product would be open source, free for everyone and we'd provide our own internal configuration (flows/tasks etc) for nothing.

## What is the Security Development Life-cycle Tool?

The SDLT is a web application that walks a user through the security assurance life-cycle. They're taken step-by-step through all of the tasks they need to complete and submit to key stakeholders through a single platform. Where their deliverable has Privacy concerns, they'll be taken through a Privacy Threshold Assessment task that is automatically emailed to the Privacy Officers for approval. Approvals on all tasks and the submission are handled within the tool; no more printing/signing and scanning documents.

The SDLT provides a single front door to Security (and more... see below) for the organisation. The process is objective, repeatable, auditable and optimised. 

*Note: I say front door to security, but the reality is that we've also incorporated PCI-DSS, Privacy and Information Management as key stakeholders with their own tasks. There is work happening to also include procurement, legal and vendor services as well effectively making it a single front door for "go-live"*

## The high level user flow

The user would land on the front page and select the type of deliverable that they had. We'd bunch them into 4 categories that we call pillars.
1. Proof of Concept or Software Trial
2. Software as a Service
3. Solution/Project Delivery or Initial Software Release
4. Technical Change, Feature Release or Bug-Fix

After selecting the deliverable type, they'd go into the parent questionnaire assigned to that pillar. The pillar questionnaire would ask key information about the deliverable. Who is the business owner? Will you deal with any personal information? Will you be exposing any new public services? When is your go live? 
Depending on how the user answers, the submission would have tasks assigned for the user to complete prior to being able to submit for approval. If the user answers yes to personal information, then they have to do a Privacy Threshold Assessment. If they answer yes to exposing new public services, then they have to do a penetration test. Out of the box, we have approximately 30 tasks.

Tasks are broken down into 2 major types.
1. Where the task is completed externally. We provide a link to the user from the SDLT task to the document they must complete. Then we ask for a link to where they have stored the completed document in our document management system.
2. The task is completed entirely within the SDLT. We ask for information from the user and the tool determines the next steps.

Once a user has completed all assigned tasks on their submission, they can send the entire submission through for approval. The submission would be sent to the Security Architect team to review and approve. After a Security Architect has approved, the submission will be sent to the Chief Information Security Officer for a recommendation approval and to the Business Owner to formally approve and accept the associated risks.

## Doing it without programming

The SDLT is designed to be edited by people without a development background. We use SilverStripe as a CMS and everything is built on top of that. When adding a button to a question you can create an action to assign. The button can give the user a message, skip ahead to another question or allocate a task to be created on the submission.

For more information about the different types of tasks we provide out of the box please check out the [wiki](https://github.com/NZTA/SDLT/wiki/Basic-Concepts:-Tasks)

## Success or failure?

The SDLT was built on the crazy idea of building an open source work-flow tool for automating security assurance. Nothing in the market did what we wanted out of the box and those that provided frameworks wanted to be paid in kilograms of gold.

Two years on, we've had more than 400 submissions through the SDLT. Over 75% of those would never have come to security due to lack of resourcing and process. Our primary focus has been increasing adoption so that we can increase visibility of what is happening. We believe this has been incredible successful.

For those who noticed, above I noted that off the shelf products had terrible risk assessment concepts. We went on to build SDLT version 2 that includes a digital security risk assessment methodology. As this is a topic in it's own right, I'll write about this next. For those who are too keen to wait to hear more about this please check out the [GitHub Wiki](https://github.com/NZTA/SDLT/wiki/Basic-Concepts:-DSRA). The TLDR is that we can automate the risk assessment process from start to finish and have it meaningful and measurable over time.

## How do I get it?

As mentioned, the SDLT is open source. You can grab a copy from the [GitHub Repository](https://github.com/NZTA/SDLT) and follow the [Wiki Instructions](https://github.com/NZTA/SDLT/wiki) to deploy it.

Alternatively, Catalyst have stood up a [SDLT SaaS product](https://www.catalyst.net.nz/products/security-development-lifecycle-tool) that can be subscribed too. This is ideal for larger organisations who want to ensure they get updates and bug-fixes as we develop them.

## Free screenshots
If you just want to see what it looks like, here are some handy screenshots.

The SDLT Landing Page:
![The landing page](/images/sdlt_landing_page.png)
Answering questions within a pillar:
![Answering questions within a pillar:](/images/sdlt_pillar.png)
The summary screen listing tasks to be completed:
![The summary screen listing tasks to be completed](/images/sdlt_summary.png)
An example task questionnaire with a result:
![An example task questionnaire with a result](/images/sdlt_task_result.png)