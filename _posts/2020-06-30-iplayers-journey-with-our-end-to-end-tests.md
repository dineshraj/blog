---
layout: post
title:  iPlayer’s journey with our end-to-end tests
date:   2020-06-30 12:22:47 +0000
categories: technology it computers testing software development 
image: /assets/iplayer-e2e/social.jpg
---

![iPlayer’s journey with our end-to-end tests][intro]


A story of how iPlayer Web achieved a stable end to end testing platform

At iPlayer we take our testing seriously and always strive to provide an optimal user experience, free of bugs and without broken user journeys. This is why as part of our Continuous Integration/Continuous Deployment pipeline we have a suite of end-to-end tests (E2Es) which run every time any code is merged into our master branch.

{% include image.html url='/assets/iplayer-e2e/pipeline.jpg' caption='the build pipeline for the iPlayer Playback page' %}

The value of E2Es lies in the fact that they directly emulate user interaction and user journeys. Whereas unit and integration tests are heavily mocked, meaning that you start with certain assumptions that other parts of the system are working correctly, E2Es test the entire system from start to finish without anything being mocked and just as a user would experience it.

<!-- more -->

E2Es also directly click on elements rather than programatically triggering clicks, meaning it can also spot things like styling issues or if an element is obscured by another element on the page.

However, as you may have experienced, E2Es can suffer from a fair share of problems including — but not limited to — slow running times, unreliability and limitations in the amount of platforms they can run on.

This article will detail the journey we have taken over the past several months to get to the place we are now. Hopefully this will help you skip a couple of steps to get to your desired destination. It is worth pointing out that our changes in solutions did not occur straight away and we did spend a fair bit of time trying out each approach.

Our current solution is by no means perfect. Whilst we have arrived at a place with reasonable reliability and execution time, we are still on this journey and striving to improve.

We will focus on the three main stages of our E2E journey; our [local selenium instances](https://www.selenium.dev/documentation/en/introduction/the_selenium_project_and_tools/), [Zalenium](https://opensource.zalando.com/zalenium/) hosted on AWS EKS and finally using [BrowserStack](https://www.browserstack.com/).

## Local Selenium Instances

Initially our set up involved having local Selenium instances running in separate docker containers that used WebDriver 4. This ran via our Jenkins instance, which shared resources with other jobs.

However, this had a multitude of issues. One of which was the fact we wrote our own custom runner, and this only ran the test suites in series rather than parallel. We could have chosen to re-write this but there were other deeper problems with our set up, the main one being that Selenium instances seemed to often have environment issues and instability. This led to unreliable tests that were frequently failing.

Running our tests via Jenkins was also suboptimal as Jenkins was quickly getting overwhelmed when running lots of jobs. Increasing the size of the Jenkins instances did not seem to help this.

Finally, the way in which we set up our Selenium tests only allowed us to run the test suites in Chrome — ideally we would like our set up to be cross platform so we can test features and user journeys across different combinations of operating systems and browsers.

## Zalenium

Due to a need to upgrade our previous set up to a solution that was more performant, reliable and robust, we turned to Zalenium. Zalenium is an open source project which packages a [Selenium Grid](https://www.selenium.dev/documentation/en/grid/). It is designed to run on Docker or Kubernetes.

It has several built in features, including auto-scalability, recording videos of test runs, which is extremely useful for debugging, and having a nice user interface.

{% include image.html url='/assets/iplayer-e2e/zalenium.jpg' caption='a screenshot of Zalenium in action' %}

Zalenium requires a lot less maintenance from us to run tests reliably. As mentioned above, Selenium Grid had some stability issues but Zalenium takes care of this by destroying and rebuilding the test containers after each test run.

Initially this worked well, and we were able to run our tests consistently and reliably, but unfortunately as we moved more tests over they started intermittently failing.

After some investigating, we found this was due to the auto-scaling not working properly. The feeling was that we could have solved this issue but it would have taken up too much development time.

One other downside to this approach was that it ran on Kubernetes and in our case in AWS EKS (Elastic Kubernetes Service). This is a managed Kubernetes but there is still a lot of configuration to do.

It is worth noting here that as part of this upgrade we also upgraded from WebDriver 4 to WebDriver 5, which allowed us to remove our custom test runner mentioned above and utilise the build in parallelisation that WebDriver 5 offers.

However, despite this, due to limits with our Zalenium set up we had a maximum of 8 test suites running at once, and this meant simply running end to end tests for two apps could fill them up.

It is also worth noting that Zalenium was initially written on top of the Selenium Grid, before Docker and Kubernetes were around, so whilst it is now designed to run on these platforms, it was always going to have some limitations.

## BrowserStack

Due to the ongoing issues described above, we again decided to find a different solution. After further research we made the move to BrowserStack.

Our move from Zalenium promised to solve a lot of our issues. Like our previous solution, BrowserStack is based on a Selenium Grid, but it is fully managed by BrowserStack, meaning no extra configuration is required. So instead of managing our own grid, or setting up AWS EKS configuration we could simply point our tests at BrowserStack.

{% include image.html url='/assets/iplayer-e2e/browserstack.jpg' caption='BrowserStack provides a nice UI to view your test results' %}

Video storage also comes for free with the BrowserStack account, which meant there was no need to worry about setting up S3 buckets for this.

But the main positive is that our tests ran with a higher pass rate percentage (the last 10 runs of the Homepage E2Es have a 100% pass rate) and we were able to have a suite of E2Es that mostly fail for the right reasons.

Whilst we do have some occasional false positives, overall we have a much improved experience with our tests and this helps us spot bugs that may otherwise get through to the user.

Our current goals with our E2Es is to now improve our debugging capabilities for BrowserStack, so we can find out why something has failed quickly, and also to get the tests running on several different platforms.

I hope this article has been useful in showing you the journey we had in order to get to the solution we have now.

[intro]: /assets/iplayer-e2e/social.jpg "iPlayer’s journey with our end-to-end tests"
