---
title: "A New Vision for ArduPilot.org: Insights from Our Latest Analytics"
date:
  created: 2023-10-13
  updated: 2023-10-13
draft: false
categories: 
    - ArduPilot
tags:

authors:
  - khancyr
---
![khancyr-analytics-blog-title-image|500x500](https://discuss.ardupilot.org/uploads/default/optimized/3X/e/e/ee5f17826e08cdddc60acdc6b20b1d3c49ff7e4f_2_500x500.jpeg)
We've recently integrated analytics into our website, [ardupilot.org](https://ardupilot.org/), and the associated wiki. With these analytics, we aim to better understand our visitors and refine their experience.

<!-- more -->
# Understanding Analytics

Analytics provides insights into user behavior, such as:

* Origins of our traffic
* Most-clicked links
* Duration of visits

All that information is interesting to improve the website experience. ArduPilot didn't have such tools available, so we were blind on what to improve. Unfortunately, analytics tools always come with a cost. Therefore, a funding proposal was done (https://discuss.ardupilot.org/t/approved-add-some-analytics-to-ardupilot-org-to-help-scope-some-refactoring/98559/9) to allow some funds for it. In case, you are interested to improve the project, the funding proposal are open to anybody that want to contribute to ArduPilot, you can see there for the process : https://ardupilot.org/dev/docs/how-the-team-works-development-fund.html#submitting-proposals.

Numerous analytics tools exists but we had some strict requirements :

* GDPR compliance
* easy maintenance
* not intrusive for website visitors

![|303x80](https://lh4.googleusercontent.com/f7XHDu-0eB4Sr1MdSKsHpB3bqVdiWY8-8bG53N5iO3r_ArecGALtmf03_4yjaMpEPumqB96giOAX3R880I9h2gwUwVpV9s1S6o1oYMTCjhxdAZ5PKO79m9OEwjwXdJUkBH92wknnBAPjgGQ3j8nfK80)

This discards the most used analytics that Google Analytics, and we chose to try https://plausible.io/ that some of us had experience with. Unlike many other analytics platforms, Plausible respects user privacy by not using cookies, not collecting personal data, and not tracking users across sites. It provides insights into website traffic and user behavior without compromising the user's privacy.

So yes, we are tracking you on ardupilot.org and the wiki. But we don’t store any personal information nor sell anything.

# How did we deploy this ?

This was a two step work. First, by following [Plausible documentation](https://plausible.io/docs/self-hosting), we deploy an instance on a cloud instance we own. Then we add the analytics JavaScript plugin into the websites we wanted to monitor.

We could have chosen to use a Plausible hosted instance, but we were unsure of how much traffic we would have, so we chose to do the deployment ourselves. Data shows that we were right as the cost would have been higher.

# What Insights Did We Gather?

All the data is freely available here : https://plausible.ardupilot.org/ardupilot.org/

On the latest 3 months (1 July to 1 October) :

![|602x599](https://lh3.googleusercontent.com/CQ7oNo7pziPPBHpIgeY_xcE-4IHCTfoc6a5vRmpEDW56Oxd8PB02zyAc9LmXrJO-slQiJbNPSVaFUmbOWcH1GeNMigSgi_KoMjyoITO_yPU5l3LnSDUIRVqljr3k2_QqPQkpyWaA2vNI6xrPkCRPVms)

* **806k** unique visitors, with **3.8M page views**
* An average of **8.6k daily visitors**
* Majority of our visitors arrive via Google
* 75% access the site from computers

That is quite a number of visitors. Proof that ArduPilot is still popular, and that the main page is important.

***Top 3 pages visited :***

1. [ardupilot.org](https://ardupilot.org) : 59.3k

2. [ardupilot.org/planner/docs/mission-planner-installation.html](https://ardupilot.org/planner/docs/mission-planner-installation.html) 37.1k

3. [ardupilot.org/copter/docs/parameters.html](https://ardupilot.org/copter/docs/parameters.html) 23.3k

***Top 5 countries :***

1. 🇺🇸 United States: 133k

2. 🇮🇳 India: 90.2k

3. 🇺🇦 Ukraine: 49.6k

4. 🇷🇺 Russian Federation: 43k

5. 🇨🇳 China: 42.3k

![|602x243](https://lh6.googleusercontent.com/Jwkws6ZFi4cNvArv6HQ5DRZJx56BrRbV1T1WzflrozpQsijBNFA9Adw2qw83Vgw1a7kmZytMZLMnB2_Lw3dc2h2inwZnlf-Us4L5t8bOtfeweF82yMP09R0jRDt8RK_sZl-nOGlDzIJIzfbRhnIQ31k)

***Top 3 exit link :***

1. [https://firmware.ardupilot.org/Tools/MissionPlanner/](https://firmware.ardupilot.org/Tools/MissionPlanner/)

2. [https://firmware.ardupilot.org/](https://firmware.ardupilot.org/)

3. [https://discuss.ardupilot.org/](https://discuss.ardupilot.org/)

![|570x461](https://lh3.googleusercontent.com/NBldGrQg1tQ0XVXZiw0OyU1U88NHA8Ai-V892Hy6qzIZDma_z2NjiH61hL1dkKkOxlTFOxAhuWhlIwQ5poyqUukpbMTN7Uni_D03OPzB0BUMd9IiK6QVI2VKx-FIg-9RU6LbVBE6_ymp61usnfywR6A)

# What is interesting ?

The first idea about ArduPilot analytics was to analyze traffic on ardupilot.org main page to refactor it. Indeed, the page got an old design for years, it can be slow to load and some don't like the general design.

Our current frontpage is held on the same repository as the wiki : [here](https://github.com/ArduPilot/ardupilot_wiki/tree/master/frontend) and you can see that the history is pretty much correcting some typos. We are using a static webpage using Bootstrap as framework and loading a lot of images. This led us to have low performances on loading, especially on mobile.

Mobile is only 25% of our traffic but we could totally put a little work on it to improve performance. Since the start of the analytics gathering, a bunch of performance PRs were already done but we still have room for improvements !

https://pagespeed.web.dev/ reports taken from France on ardupilot.org :

***Mobile version:***

![|602x520](https://lh6.googleusercontent.com/zlOfR10Si2fdokYkonAJOjzEfxhKqAjztrsIlZoSoGtOtY99YvEhoGwkzHKwZoNge1KmZAOO7sQEcJO9Aj2AqKYt2PuVRx0kUnxPKgtkf1d3ePgxffyP8Lu08_0gbuqp5-er5DiVcoRZgdDYfT1TnLA)

***Desktop version:***

![|602x556](https://lh5.googleusercontent.com/kVt4FRxzcSlNNr8AmOHGUZ_PlsN90EL1o5q0uQINNlASJCUU3CUId3FE1vTbjvyMIIcy3hzMH8-BeoWEuciX-cF90iD4eLLffLR1yo3CUCL0eK6XnzyfydCYmb5L6UUlSIXrWT__TvkeVg5GnrdcQSU)

From the performance analysis, we are shown that our main page layout isn’t great nor is the use of Bootstrap, even if it helps to have responsive design, it slows the image loading …

But what is really interesting is that the blog section on the main page displays good interest from visitors. That is truly great ! But it doesn't solve my personnel issue of having this on the front page as this hides what is ArduPilot for new people. The question would be to know if people are looking at the main page as a blog post aggregator or are just visiting the website for information ? In case you want an RSS feed, you look are [our previous post](https://discuss.ardupilot.org/t/creating-an-rss-feed-from-ardupilot-discuss-forum-topics/104128)

The ArduPilot Monthly Update post also shows great interest, that is a good signal that our community wants to follow the overall direction the development team takes.

The fact that most people come from Google links suggests that SEO optimization is crucial for the ArduPilot website and we are doing quite well on this point.

The visit duration fluctuates between 302 to 358 seconds (or 5 to 6 minutes). This suggests that visitors spend a decent amount of time on the site, possibly reading or interacting with content.

# About the Wiki :

We can see that the most demanding entry is Mission Planner installation by far ! That isn’t surprising as that is the main GCS that people use with ArduPilot (no stat on this unfortunately). That highlights the need for updated and easily accessible content regarding it.

[https://ardupilot.org/dev/docs/raspberry-pi-via-mavlink.html](https://ardupilot.org/dev/docs/raspberry-pi-via-mavlink.html ) also shows great interest. It would be a sign that we should spend a bit more time on this wiki section to update how to use a Linux computer to connect with an ArduPilot flight controller.

Another thing that is already in work in the dev team is how to build a firmware. As the dev team use ArduPilot for years and are generally seasoned developers, some commands are really easy for us and we forget that it can be another world for newcomers or people that really need to build ArduPilot firmwares. We forward the build instruction from the wiki to the [BUILD.md](https://github.com/ArduPilot/ardupilot/blob/master/BUILD.md) file we got on ArduPilot source code. This isn’t probably the best way to teach people about building, but we are looking at it.

# Some interesting finding :

One link that is true high in the list is https://arkelectron.com/product/arkv6x/. This is a great board, but it truly outpaces other boards. I questioned myself about why there is so much interest in this one specifically ? Is that because it is an USA made board and we got a lot of traffic from the USA ? From the analytics data it shows that is coming from this page : [https://ardupilot.org/copter/docs/common-autopilots.html](https://ardupilot.org/copter/docs/common-autopilots.html) and the arkv6x is the first board in the list ! This is confirmed by the fact that the Pixhawk 3 Pro, which doesn’t seem to be in sales anymore, is the second one. Such as search engine first links are the most used, the placement in the autopilot list we show is important … even if it is in alphabetical order here. We could do an auto shuffling on the list to see if the statistics change ! But it doesn’t explain why CUAV boards aren’t popular. Mysteries !

Most of our users are looking at the website from Windows OS and with Chrome browser. I didn’t find a fridge or a toaster browsing but we got some hits from FreeBSD and Symbian^3 which are unexpected !

Some are still using [Yahoo!](https://search.yahoo.com/), which interestingly gives correct results for ArduPilot stuff.

We also show a good user base that tries to focus on online privacy with Linux + Firefox + DuckDuckGo combo !

# Conclusions

The integration of analytics into ardupilot.org has equipped us with invaluable insights into our visitors' behavior and preferences. We've learned that ArduPilot continues to be a significant presence in the drone community, attracting a diverse range of users from across the globe. The popularity of certain content, such as the Mission Planner installation and the Raspberry Pi connection guide, emphasizes areas where we should invest more time and resources. Performance and design issues, particularly for mobile users, have been identified and are on our radar for improvement.

Perhaps most fascinating are the subtle behaviors and patterns that analytics have unveiled. The unexpected interest in some specific boards, likely influenced by its placement on our list, reminds us of the nuanced impacts of content presentation. Similarly, the prominent use of certain operating systems and browsers, as well as the noticeable nod towards online privacy by a segment of users, provides direction for future optimizations.

In essence, these analytics don't just inform us—they inspire action. They emphasize the importance of continuous refinement, ensuring that ArduPilot remains user-centric and aligned with the evolving needs of our community. We're excited for the enhancements ahead and deeply appreciate the community's ongoing engagement and feedback.

If you aren’t a drone developer but a web developer, you can help us to make our websites much better !