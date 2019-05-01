+++
draft = true
title = "The CD mindset"
subtitle = "The human factor in continuous delivery"
date = "2019-06-07"
tags = ["devops"]
image = "/drafts/XXXX-XX-XX-cd-mindset/assembly-line.jpg"
image_info = "Image by Land Rover MENA [CC BY 2.0](http://creativecommons.org/licenses/by/2.0) with slight colour modifications"
id = "asdf5"
url = "asdf5/continuous-delivery-mindset"
aliases = ["asdf5"]
+++

“Continuity” is a central keyword in the software industry, especially in modern web development. Continuous integration, continuous deployment, continuous releases: while these are slightly different categories, they all aim at the same goals – to shorten the feedback loop in feature development, to be able to respond to changing requirements faster and to minimise the exposure time of bugs. Continuous delivery is not just a technical niceness though, it has become a crucial business factor that ultimately enables agile product development.

The concept of building and shipping right away sounds simple and intriguing, but getting there actually deserves a lot of effort to be put into tooling and processes. Ideally, every code contribution will automatically be deployed and released to production. But even if releases are driven manually, for them to be happening frequently a rock-solid test suite is as needful as thorough real-time monitoring. Reliability is what makes the difference here.

While books[^1] can be filled about technical setups and concepts, this blog post touches a less obvious aspect that, however, is not less important: the human factor. Practicing successful continuous delivery is not just a matter of having the right tools in place, it’s a mindset question in great part. (This especially should not be underestimated if the endeavour is to introduce continuous delivery to a team that is used to scheduled, dedicated releases.)

## Work incrementally

Continuously integrating and delivering small chunks of work has several benefits as opposed to big releases. Most of all, the risks of changes are reduced by making them more manageable. Changes are split into smaller chunks, which can thus be gradually discussed, reviewed, tested, integrated, verified and re-evaluated as they are merged back into the main branch.

Generally, this aspect is the most crucial one and could easily make one blog post alone. Working both incrementally and iteratively is not just beneficial when it comes to continuous delivery, it also brings immense value to the product development process. However, in order to be effective this virtue must be exercised on an organisational level and not just in the development department.

- Baby steps; small increments that are continuously released
- Feature/dev flags
- Requires certain level of seniority

## Take responsibility



- Verify that your changes are successful
- Keep an eye on logs and monitoring

## Communicate proactively

- Keep everyone in the loop
- Inform about risks proactively

## Know the tools

- Always think forward, but mind the retreat
- Two-pronged data migrations

## Invest into the foundations

- Constantly invest in tests, etc.
- Continuously dedicate time for refactorings


[^1]: For technical aspects, “[Continuous Delivery](https://www.amazon.com/dp/0321601912)” by Humble/Farley provides a comprehensive overview.