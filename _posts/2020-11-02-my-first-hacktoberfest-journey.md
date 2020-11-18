---
title: My Hacktoberfest journey
date: '2020-11-02 13:13:56'
tags:
- open-source
- hacktoberfest
image: /images/hacktoberfest/hacktoberfest.jpeg
---
### About Hacktoberfest
The Hacktoberfest is a month long event organised on October every year. It is for everyone to contribute and get started with open source. To participate in hacktoberfest, anyone has to make 4 valid pull requests to the repositories that are participating in the event. Those who completes the challenge with 4 PRs, can choose a limited edition Hacktoberfest 2020 tee or plant a tree, as sponsored by Digital ocean and its partners.

Though I wanted to participate in the event last time, I couldn't. This time, however I managed to plan and find the time to participate. Let me explain my journey in participating and complete the hacktoberfest challenge.

### Planning
In the last week of Sepetember, I got a mail to register for Hacktoberfest 2020. I registered to Hacktoberfest over the weekend. Then I started searching for repositories in Github asking for help. 

- I came across a repository called [`Ciphey`](https://github.com/Ciphey/Ciphey/). It is a cool tool, that decodes any encoded text. The [issue] (https://github.com/Ciphey/Ciphey/issues/429) asked to create a simple Github action, that runs the binary from a terminal and run some tests.  There was also another [issue](https://github.com/Ciphey/Ciphey/issues/433) in the repo, that asked to add the UUencode decoder.  The Ciphey team had a discord server, and I joined there to get instant help. With a little bit of search in their excellent documentaion and some questions asked, I was able to figure out what needs to be done.

- Then there was another repo [morning.thechels.uk](https://github.com/MatBenfield/morning.thechels.uk). This repo is a personal website that has the today's weather info, share prices, news and articles from the configured websites. Let me explain first on how the website works. The website ran on Jekyll and as many of us know, Jekyll is a static site generator. The author had a github action that ran some python script everyday morning 5.00 A.M GMT. The author needed help in adding details on Covid data. The [issue](https://github.com/MatBenfield/morning.thechels.uk/issues/15) was there with all the necessary information to add covid 19 data to the website. 

- [https://hacktoberfestswaglist.com/](https://hacktoberfestswaglist.com/) is another open source project, that maintains the list of swags offered by participating repositories. That website helped me find one of the companies, [`LoginRadius`](https://www.loginradius.com/) welcoming guest blog posts in their website. 

### Real Work
Been able to spend time over the weekends, I managed to get 4 PRs ready by the end of 2 weeks of October. These are my 4 PRs,

- [https://github.com/Ciphey/Ciphey/pull/446](Terminal test for Ciphey #446)
- [https://github.com/Ciphey/Ciphey/pull/467](Add uuencode)
- [https://github.com/MatBenfield/morning.thechels.uk/pull/18](Adding Corona data)
- [https://github.com/LoginRadius/engineering-portal/pull/401](Blog post on creating a new blog with jekyll & github pages).  My blog published [here](https://www.loginradius.com/engineering/blog/setup-blog-in-minutes-with-jekyll/), if you are interested.

![Hacktoberfest PR completion](/images/hacktoberfest/hacktoberfest-pr-snapshot.png)

The end result is atmost satisfaction, that I am able to help me out a few maintainers. Aha, it feels so great when your PR is accepted.