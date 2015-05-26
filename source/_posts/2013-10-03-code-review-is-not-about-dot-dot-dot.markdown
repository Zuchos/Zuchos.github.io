---
layout: post
title: "Code Review is not about..."
date: 2013-10-03 15:11:24 +0200
comments: true
categories: [code review, agile, codebrag]
---
In SML we do code reviews. We do them on daily basis. Actually the point that we are now is a result of long journey that we made. We try different strategies and tools until we went to the place that we are now (but it doesn't mean that we are going to end up here).<

During this journey we found many risks and traps that are waiting for a newcomer. That's what this post is all about, traps & misconceptions on code review.

{% img centered /images/posts/2013/puppet.jpg %}


**Code control:** many organizations uses CR for controlling codebase. Most of them are using pre-commit strategy. In many cases its because those projects are open-source with hundred of commiters. In real life this is quite rare scenario, so if you hired someone it means that you trust him enough to let him commit code to repository. I know that in some organizations there will be temptation to make procedure that will force developers to "review" and "approve" every commit, but it will not guarantee the quality. Moreover people will soon treat code review as "stupid" corporate procedure and will try to hack-it (such changing password every month e.g. people are using passwords like: mypass1, mypass2 etc.).  
<!--more-->

{% img centered /images/posts/2013/Codebrag.png %}
**Hall of blame:** Don't user CR for finding scape goats or guilty ones. Let's assume that there was a failure and you found a person who "reviewed" the "bad code" and blame him for not pointing it out. I will cause that development in your company will drastically slow down. People will be pointing out every semicolon not at the right place, because they will be afraid of being scape goat. Your team members will start feeling unconfident and the lack of trust.

{% img centered /images/posts/2013/duty.jpg %}
**"Code" of duty:** Don't push on your developers to much. If you force them to make review every day for an hour, soon they will hate it and treat as unfunny duty. Code review is about learning, praising others and giving feedback so it's very social activity. CR could be fun, don't spoil it.

{% img centered /images/posts/2013/wolfs.jpg %}
**I'm not my code:** If your code was reviewed by someone and he left some comment (sometimes even not too nice), don't get angry. He isn't saying that you are a poor developer. That wasn't his intention nor code review is about that. All he did, was criticizing some piece of code (not the author). Code review is all about code, not about you. Don't treat code review tool as forum with "trolls" and people fighting each other. When you will be writing comments try not to be rude or too strict. Try to imagine that your are on the other side reading it.

To sum it up, there is plenty of ways to make code review wrong. Those 4 are the one that I experienced or anticipated, so beware. I promise to write post about "what code review is about".  

If you wan't to try out code review tool (fruit of our experience & practices) visit [codebrag.com](http://codebrag.com/)

####Used photos:  
1. ["The Puppet Master"](http://www.flickr.com/photos/strenggeheim/2856486144/in/photolist-5mqeP7-5Kpkk8-5KHyC7-5QTtZk-5QTu62-5TSxo2-67sWs5-6m5Zj4-6MvHAZ-6MVMR1-72BPWY-7fs94o-7fCN6a-7gaSkT-7uyJ7v-7vfFKQ-aFogBm-dc9uia-7RAsHq-aaFTbm-awno9t-8kUyiE-8eh9AD-ebrXeJ-8vJGPw-cpb97h-9nD8HE-bSmHAz-7zr6Ed-9yxUuq-9yuTXZ-9yxUHG-9PMTjC-aabDNu-8ekogC-aou2w1-8ekmkN-8eh8TR-8eknaf-8ekmHQ-aSuD2H-aeWYt2-ar27s3-edGq5V-cior97-9NQK5R-b2sWzM-dky9bw-8932kC-83xoQR-9ULYTQ/) by Henk
2. ["ready for duty"](http://www.flickr.com/photos/mythoto/8294069500/in/photolist-dCVgUh-8AH8J8-9mHYrq-7GVnTL-ggR7Ev-8CdcUW-94aGjt-e9UbC3-81MVTr-81MVWe-81Dvod-aTPEvk-fv4y8m-8ctrmT-ac5J4M-8H6pEw-fhu8R4-9XSaoD-atHTtb-c4Y2sN-9oxmaz-9Y9pxG-cC2rGA-8ZPXUJ-ci9xwW-dfTxvG-7Y7WtL-aD4TuF-aMQuoR-b7FPRi-dHLsn5-8Gtcdm-9x7J69-bhkMsB-dNVmzY-bSvsrB-bDAJxG-bV1JNH-bFd7vE-cA5ufs-9n5BYB-9n5ren-aaeUht-7YTgVs-bFpGEi-bFpGDz-882iJX-9oGVj7-bYJSoQ/) by Leonard John Matthews  
3. ["Polar wolf's argument"](http://www.flickr.com/photos/8070463@N03/10039483454/in/photolist-gi9YEj-gdGxwt-gcXRyX-g8WzrJ-g6BRPW-g6AXgu-g5ybsw-g5n24y-g26JkD-fYfAbU-fUX3fc-fU1C29-fQFeNm-fQsoKa-fPrCxP-fLzErr-fLQaBU-fKu8PR-fKibjz-fKib46-fKzKSQ-fKi8L6-fJV2fN-fJRA2o-fJz3Vn-fJqF9Y-fJqH1m-fJqFp7-fJ985F-fJqG5q-fJqH85-fJqFzo-fJ98LB-fJ992T-fJ97Kg-fJqF5G-fJ999X-fJqFWq-fHq3H1-fCgwUH-fCk9Vm-fAcLWL-g6Lh1d-fXESAq-fUsx7D-fSMPTC-fSMNBQ-fSMMZQ-fSMNKh-fSMP7E-fSLEjg) by Tambako the Jaguar