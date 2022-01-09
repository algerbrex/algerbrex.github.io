# New Year, New Tournaments, New Features

Happy new year everyone! I hope the holiday season has been restful for all, and 2022 presents many new opportunites for growth.

In the last couple of weeks, [Blunder 7.6.0 was released](https://github.com/algerbrex/blunder/releases/tag/v7.6.0), which is still currently being tested, 
but I estimate to be around 2650 at longer time controls, and a bit higher at blitz. Which means I've pretty well hit my goal of 2600 Elo, and now it's on 
to 2700!

While I've still been developing Blunder, trying little tweaks here and there, and the dev version is hopefully a bit stronger, I spent most of my time the
past week or so, letting Blunder be tested by the community.

The latest version has participated and is participating in several tournaments recently:

* It particpated in [Graham Banks 90th Amateur Series Division 7](http://talkchess.com/forum3/viewtopic.php?f=6&t=78968#p917197), scoring 26.0 points and finishing in second.
* It's currenlty particpating in Graham Banks [Blunder-Buss tournament](http://talkchess.com/forum3/viewtopic.php?f=6&t=79059#p917808), where after 12 rounds, it currently
  leads the pack with 8.5 points.
* Blunder is currently participating in the [27th edition of Carlos's blitz tournament](http://talkchess.com/forum3/viewtopic.php?f=6&t=79037&start=10#p917718), where 
  it finished third in the 3rd promotion leauge, and is still running in the 2nd promotion leauge.
  
Additonally, Blunder's currently being tested by the CCRL at the 40 moves in 15 minutes time control, which is it's first time being offically tested at a longer time control.
I'm curious to see where it ends up falling, and many thanks to Graham Banks for running these tests. And beyond the CCRL, several other testing groups such as 
[CEGT](http://www.cegt.net/) and [Bruce's Bullet Rating List](https://e4e6.com/) have also been kind enough to test Blunder.

I'm not entirely sure when the next version of Blunder will be released, mostly because I'm starting to realize how much of a toll running testing matches is taking on my
poor laptop, and I'll likely need to invest in a Desktop to continue testing, which may mean a couple of weeks before I can begin testing again. In the meantime, I'll likely 
work on more asthetic features for Blunder, features that might be more helpful for chess players than engine programmers. For example, Blunder 7.7.0 will have a command line 
option to display a deatiled break-down of how it statically evaluates a position. Here's an example of what it looks like for the 
[famous kiwipete position](http://www.talkchess.com/forum3/viewtopic.php?t=55787):

```
+------------------+--------------+--------------+------------+
|  Evaluation Term |  White Score |  Black Score | Difference |
+------------------+--------------+--------------+------------+
|     Material     |     39.64    |     39.64    |    0.00    |
+------------------+--------------+--------------+------------+
|    Positional    |     0.51     |     0.46     |    0.05    |
+------------------+--------------+--------------+------------+
|  Pawn Structure  |     0.00     |     0.00     |    0.00    |
+------------------+--------------+--------------+------------+
|     Mobility     |     -0.44    |     -0.44    |    0.00    |
+------------------+--------------+--------------+------------+
|    King Safety   |     0.00     |     -0.42    |    0.42    |
+------------------+--------------+--------------+------------+
|         -        |       -      |       -      |    0.47    |
+------------------+--------------+--------------+------------+
```

If you'd like to try this feature out yourself, it's been comitted to the development branch of Blunder. More rows will be added of course as more distinct evaluation 
categories are added to Blunder, as needed.

Overall though, I'm very exicited to work on Blunder over the coming months, and hopefully it won't be long before it hits the 2700 milestone.
