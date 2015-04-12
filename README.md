# Hacker News API

## Overview

In partnership with [Firebase](https://www.firebase.com), we're making the public Hacker News data available in near real time. Firebase enables easy access from [Android](https://www.firebase.com/docs/android/), [iOS](https://www.firebase.com/docs/ios/) and the [web](https://www.firebase.com/docs/web/). Servers aren't left out; there's also [REST](https://www.firebase.com/docs/rest/) support.

If you can use one of the many [Firebase client libraries](https://www.firebase.com/docs/) you really should. The libraries handle networking efficiently and can raise events when things change. Be sure to check them out.

Please email api@ycombinator.com if you find any bugs.

## URI and Versioning

The API is based at https://hacker-news.firebaseio.com.

We hope to improve it over time, and may later enable access to private per-user data using OAuth. The changes won't always be backward compatible, so we're going to version the API. This first iteration will live at https://hacker-news.firebaseio.com/v0/ and is structured as described below.

For versioning purposes, only removal of a non-optional field or alteration of an existing field will be considered incompatible changes. *Clients should gracefully handle additional fields they don't expect, and simply ignore them.*

## Design

The v0 API is essentially a dump of our in-memory data structures. We know, what works great locally in memory isn't so hot over the network. Many of the awkward things are just the way HN works internally. Want to know the total number of comments on an article? Traverse the tree and count. Want to know the children of an item? Load the item and get their IDs, then load them. The newest page? Starts at item maxid and walks backward, keeping only the top level stories. Same for Ask, Show, etc.

I'm not saying this to defend it - It's not the ideal public API, but it's the one we could release in the time we had. While awkward, it's possible to implement most of HN using it.

## Items

Stories, comments, jobs, Ask HNs and even polls are just items. They're identified by their ids, which are unique integers, and live under https://hacker-news.firebaseio.com/v0/item/<id>.

All items have some of the following properties:

Field | Description
------|------------
id | The item's unique id. Required.
deleted | `true` if the item is deleted.
type | The type of item. One of "job", "story", "comment", "poll", or "pollopt".
by | The username of the item's author.
time | Creation date of the item, in [Unix Time](http://en.wikipedia.org/wiki/Unix_time).
text | The comment, story or poll text. HTML.
dead | `true` if the item is dead.
parent | The item's parent. For comments, either another comment or the relevant story. For pollopts, the relevant poll.
kids | The ids of the item's comments, in ranked display order.
url | The URL of the story.
score | The story's score, or the votes for a pollopt.
title | The title of the story, poll or job.
parts | A list of related pollopts, in display order.
descendants | In the case of stories or polls, the total comment count.

For example, a story: https://hacker-news.firebaseio.com/v0/item/8863.json?print=pretty

```json
{
  "by" : "dhouston",
  "descendants" : 71,
  "id" : 8863,
  "kids" : [ 8952, 9224, 8917, 8884, 8887, 8943, 8869, 8958, 9005, 9671, 8940, 9067, 8908, 9055, 8865, 8881, 8872, 8873, 8955, 10403, 8903, 8928, 9125, 8998, 8901, 8902, 8907, 8894, 8878, 8870, 8980, 8934, 8876 ],
  "score" : 111,
  "time" : 1175714200,
  "title" : "My YC app: Dropbox - Throw away your USB drive",
  "type" : "story",
  "url" : "http://www.getdropbox.com/u/2/screencast.html"
}
```

comment: https://hacker-news.firebaseio.com/v0/item/2921983.json?print=pretty

```json
{
  "by" : "norvig",
  "id" : 2921983,
  "kids" : [ 2922097, 2922429, 2924562, 2922709, 2922573, 2922140, 2922141 ],
  "parent" : 2921506,
  "text" : "Aw shucks, guys ... you make me blush with your compliments.<p>Tell you what, Ill make a deal: I'll keep writing if you keep reading. K?",
  "time" : 1314211127,
  "type" : "comment"
}
```

job: https://hacker-news.firebaseio.com/v0/item/9235551.json?print=pretty
```json
{
  "by" : "nickporter",
  "id" : 9235551,
  "score" : 1,
  "text" : "We change the way brick-and-mortar retailers interact with their data. We help them make better decisions and grow their business.<p>The landscape is currently dominated by legacy software from SAP, Oracle, IBM, etc. We&#x27;re showing our customers what beautifully crafted software looks like.<p>We&#x27;re looking for driven individuals to help architect and implement the next iteration of our platform. Our customers include multinational retailers from across the world, so you&#x27;ll get a chance to learn the inside operations of some of the biggest consumer-facing companies. Oh and we&#x27;re processing over 1 billion dollars worth of transactional data.<p>The ideal candidate can easily work in 2 of these areas of the stack:<p><pre><code>    - UI&#x2F;UX Design\n    - Frontend       (Angular, D3, CoffeeScript, SASS, Jade, Grunt)\n    - Backend        (Node, Python, Go, SQL)\n    - Infrastructure (Spark, Docker, CoreOS, Kubernetes, AWS)\n</code></pre>\nWe are constantly improving and evolving our stack, so it&#x27;s not a big deal if you don&#x27;t know some of the tech that&#x27;s listed. What&#x27;s important is that you have a passion for your craft and that you&#x27;re able to learn quickly. Our team is growing, so this is an awesome opportunity to be part of the core product and engineering team and lay the foundation of the platform.<p>What we offer:<p><pre><code>    - A fully stocked fridge\n    - An office in SOMA, San Francisco, 10min away from Bart or Caltrain\n    - Paid lunches\n    - Flexible vacation time\n    - Full healthcare package\n    - Generous stock option plan\n    - Whatever gear you need\n    - Visa sponsorships\n</code></pre>\nSend an email to `nick@42technologies.com` if you&#x27;re interested!",
  "time" : 1426811086,
  "title" : "42 (YC W14) is hiring engineers to help fix retail",
  "type" : "job",
  "url" : ""
}
```

poll: https://hacker-news.firebaseio.com/v0/item/126809.json?print=pretty

```json
{
  "by" : "pg",
  "descendants" : 54,
  "id" : 126809,
  "kids" : [ 126822, 126823, 126993, 126824, 126934, 127411, 126888, 127681, 126818, 126816, 126854, 127095, 126861, 127313, 127299, 126859, 126852, 126882, 126832, 127072, 127217, 126889, 127535, 126917, 126875 ],
  "parts" : [ 126810, 126811, 126812 ],
  "score" : 46,
  "text" : "",
  "time" : 1204403652,
  "title" : "Poll: What would happen if News.YC had explicit support for polls?",
  "type" : "poll"
}
```

and one of its parts: https://hacker-news.firebaseio.com/v0/item/160705.json?print=pretty

```json
{
  "by" : "pg",
  "id" : 160705,
  "parent" : 160704,
  "score" : 335,
  "text" : "Yes, ban them; I'm tired of seeing Valleywag stories on News.YC.",
  "time" : 1207886576,
  "type" : "pollopt"
}
```

## Users

Users are identified by case-sensitive ids, and live under https://hacker-news.firebaseio.com/v0/user/.

Field | Description
------|------------
id | The user's unique username. Case-sensitive. Required.
delay | Delay in minutes between a comment's creation and its visibility to other users.
created | Creation date of the user, in [Unix Time](http://en.wikipedia.org/wiki/Unix_time).
karma | The user's karma.
about | The user's optional self-description. HTML.
submitted | List of the user's stories, polls and comments.

For example: https://hacker-news.firebaseio.com/v0/user/jl.json?print=pretty

```json
{
  "about" : "This is a test",
  "created" : 1173923446,
  "delay" : 0,
  "id" : "jl",
  "karma" : 2937,
  "submitted" : [ 8265435, 8168423, 8090946, 8090326, 7699907, 7637962, 7596179, 7596163, 7594569, 7562135, 7562111, 7494708, 7494171, 7488093, 7444860, 7327817, 7280290, 7278694, 7097557, 7097546, 7097254, 7052857, 7039484, 6987273, 6649999, 6649706, 6629560, 6609127, 6327951, 6225810, 6111999, 5580079, 5112008, 4907948, 4901821, 4700469, 4678919, 3779193, 3711380, 3701405, 3627981, 3473004, 3473000, 3457006, 3422158, 3136701, 2943046, 2794646, 2482737, 2425640, 2411925, 2408077, 2407992, 2407940, 2278689, 2220295, 2144918, 2144852, 1875323, 1875295, 1857397, 1839737, 1809010, 1788048, 1780681, 1721745, 1676227, 1654023, 1651449, 1641019, 1631985, 1618759, 1522978, 1499641, 1441290, 1440993, 1436440, 1430510, 1430208, 1385525, 1384917, 1370453, 1346118, 1309968, 1305415, 1305037, 1276771, 1270981, 1233287, 1211456, 1210688, 1210682, 1194189, 1193914, 1191653, 1190766, 1190319, 1189925, 1188455, 1188177, 1185884, 1165649, 1164314, 1160048, 1159156, 1158865, 1150900, 1115326, 933897, 924482, 923918, 922804, 922280, 922168, 920332, 919803, 917871, 912867, 910426, 902506, 891171, 807902, 806254, 796618, 786286, 764412, 764325, 642566, 642564, 587821, 575744, 547504, 532055, 521067, 492164, 491979, 383935, 383933, 383930, 383927, 375462, 263479, 258389, 250751, 245140, 243472, 237445, 229393, 226797, 225536, 225483, 225426, 221084, 213940, 213342, 211238, 210099, 210007, 209913, 209908, 209904, 209903, 170904, 165850, 161566, 158388, 158305, 158294, 156235, 151097, 148566, 146948, 136968, 134656, 133455, 129765, 126740, 122101, 122100, 120867, 120492, 115999, 114492, 114304, 111730, 110980, 110451, 108420, 107165, 105150, 104735, 103188, 103187, 99902, 99282, 99122, 98972, 98417, 98416, 98231, 96007, 96005, 95623, 95487, 95475, 95471, 95467, 95326, 95322, 94952, 94681, 94679, 94678, 94420, 94419, 94393, 94149, 94008, 93490, 93489, 92944, 92247, 91713, 90162, 90091, 89844, 89678, 89498, 86953, 86109, 85244, 85195, 85194, 85193, 85192, 84955, 84629, 83902, 82918, 76393, 68677, 61565, 60542, 47745, 47744, 41098, 39153, 38678, 37741, 33469, 12897, 6746, 5252, 4752, 4586, 4289 ]
}
```

## Live Data

The coolest part of Firebase is its support for change notifications. While you can subscribe to individual items and profiles, you'll need to use the following to observe front page ranking, new items, and new profiles.

### Max Item ID

The current largest item id is at https://hacker-news.firebaseio.com/v0/maxitem. You can walk backward from here to discover all items.

Example: https://hacker-news.firebaseio.com/v0/maxitem.json?print=pretty

```json
9130260
```

### New and Top Stories

Up to 500 top and new stories are at https://hacker-news.firebaseio.com/v0/topstories and https://hacker-news.firebaseio.com/v0/newstories. 

Example: https://hacker-news.firebaseio.com/v0/topstories.json?print=pretty

```json
[ 9129911, 9129199, 9127761, 9128141, 9128264, 9127792, 9129248, 9127092, 9128367, ..., 9038733 ]
```

### Ask, Show and Job Stories

Up to 200 of the latest Ask HN, Show HN, and Job stories are at https://hacker-news.firebaseio.com/v0/askstories, https://hacker-news.firebaseio.com/v0/showstories, and https://hacker-news.firebaseio.com/v0/jobstories.

Example: https://hacker-news.firebaseio.com/v0/askstories.json?print=pretty

```json
[ 9127232, 9128437, 9130049, 9130144, 9130064, 9130028, 9129409, 9127243, 9128571, ..., 9120990 ]
```

### Changed Items and Profiles

The item and profile changes are at https://hacker-news.firebaseio.com/v0/updates.

Example: https://hacker-news.firebaseio.com/v0/updates.json?print=pretty

```json
{
  "items" : [ 8423305, 8420805, 8423379, 8422504, 8423178, 8423336, 8422717, 8417484, 8423378, 8423238, 8423353, 8422395, 8423072, 8423044, 8423344, 8423374, 8423015, 8422428, 8423377, 8420444, 8423300, 8422633, 8422599, 8422408, 8422928, 8394339, 8421900, 8420902, 8422087 ],
  "profiles" : [ "thefox", "mdda", "plinkplonk", "GBond", "rqebmm", "neom", "arram", "mcmancini", "metachris", "DubiousPusher", "dochtman", "kstrauser", "biren34", "foobarqux", "mkehrt", "nathanm412", "wmblaettler", "JoeAnzalone", "rcconf", "johndbritton", "msie", "cktsai", "27182818284", "kevinskii", "wildwood", "mcherm", "naiyt", "matthewmcg", "joelhaus", "tshtf", "MrZongle2", "Bogdanp" ]
}
```
