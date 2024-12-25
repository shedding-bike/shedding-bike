+++
title = "Why is building a to-do list app so darn hard?"
date = "2023-03-15"
authors = ["jack"]
draft = false
+++

Why are Todo Lists (a.k.a. personal productivity systems) so hard to build well?

I'm genuinely curious. I was listening to the last episode of [Cortex](https://www.relay.fm/cortex/), and one of the hosts (CGP Grey) brought up a similar point regarding personal productivity platforms. OmniFocus, the reigning champion of the industry for professionals looking for a deeply customized system, has been staggering in their ability to ship the next version of their application. Much of the market consists of various different packagings of the same offering. Grey's thesis of these platforms essentially boils down to this:

1.  Todo lists are very personal systems that requires deep customizations
2.  Yet, they need to stay very out of the way ("fit into the workflow") of their user

Plus, a point that I have noticed which was brought up briefly:

1.  its very easy to build a crappy one, and very hard to build a good one

I particularly like Grey's phrasing of point 3.: "there's high desire market saturation, but no practical market saturation in the offering." That is, "everyone has a good idea of what a good to-do list software is, and no one has been able to write one that generally works very well for "most" people.


## Learnings and Next Steps {#learnings-and-next-steps}

I tried once. [Condution](https://www.condution.com/) was an early experiment between me and a few of my friends to try to solve some of the issues we saw in the to-do list market at the time. There was some response within the community: within about 6 months, we got about 1000+ MAU, 10,000+ registered users; but I think there's an undeniable sense in me that, while Condution solved the specific problem we saw in the market, it still doesn't solve the fundamental issue that to-do list platforms have:

Condution, like all other platforms, isn't for everyone. And that is a problem. Peter Thiel gave this [famous talk](https://www.youtube.com/watch?v=3Fx5Q8xGU8k) in 183B which outlines the fact that much of the illusion of "no competition" comes from the small companies trying desperatly to dominate a fiercely competitive market; to win, one truly has to dominate a market. The person productivity space is one which the battle for hearts and minds have created an exciting explosion of choices for the consumer, but no one system has yet emerged to engulf it all.

The feedback that I tracked from building Condution boils down to a few asks:

1.  "can we have this repeating task/date/filter behavior?" (yes, but it has to be a ticket for each one of these and god knows when it can happen)
2.  "can you be a calendar app with scheduling?" (aaaaaaaaaa so much API work; plus, scheduling is hard)
3.  "why is [this native feature] not supported?" (because we run a PWA wrapped in a WebView on mobile)
4.  "self hosting when?" / "API when?" (oh, we tried. alas)

I think this boils down to a few things:

1.  users want myriad behavior that we don't possibly have time to implement ourselves
2.  users want their ability to process their own data (i.e. API, calendar, etc.), at their own pacing, without going through our server

And, to be honest, I don't think anyone on the market (barring lots of elisp code + Org mode, which really fails at "users want an experience that's actually not limited to Desktop Emacs nuts like me") achieves these two objectives.


## We've Seen This Before {#we-ve-seen-this-before}

Here's the thing. This is not the world's first rodeo to this problem. Visual Studio Code, the text editing sensation which today holds something like 75% market share, was released in 2015. Let's examine two random blogs I found in 2013 writing about text editors:

-   [this one](https://web.archive.org/web/20230509053757/https://www.theregister.com/2013/03/11/verity_stob_text_editor/)
-   [and this one](https://web.archive.org/web/20230608151954/https://www.bloggersentral.com/2013/09/awesome-text-editors-for-web-developers.html)

I'm of the humble opinion that Text Editors circa 2013 has the same exact set of problems as to-do lists now. Technology was mature enough for everyone to build one; the space is crowded enough that its was worth writing a listicle literally titled ("10 most fascinating text editors"); and each of us developers had a persnickety opinion about what our Text Editor/IDE should do: from editing text, to running code, to making coffee.

And then, a disruptor.

1.  that has easy to implement plugins and a web-based core which allows any and all users to customize their experience down to the tat
2.  and which makes the core experience of editing text and setting up autocomplete (bare bones basics) so easy that its inconceivable you would use anything else

Our friend Visual Studio Code.


## And so, a proposal {#and-so-a-proposal}

I don't know what such a disruptor of the field would look like, but I'd like to try again. I would like to take a crack at building a to-do list app (YET AGAIN!) that solely focuses on these two tenants:

1.  ultimate and intuitive extensibility by the user
2.  absolutly smooth down to a tat introductory ("basic") experience, which is minimally opinionated but can accommodate as many modalities as possible

I hope to do this while not compromising the platforms the product is available on, and the nativeness of this experience.

It's a lot to ask, and its yet unclear what a solution may look like. But it never hurts to take a crack. If you want to help out with this probably open-source effort, reach out.

---

and also, defer/start dates. We should have those.
