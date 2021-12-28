## Progress Report

Since the release of Blunder 7.4.0, I've been making slow but decent progress towards 2600.

Most of the changes so far have come from tweaking and refining features already in Blunder, and the current development version should be around 30 Elo stronger than 7.4.0, 
which hopefully puts it close to 2600 on the CCRL, and puts it at 2570 from my own gauntlet testing at bullet time controls. So I'll just wait and see where it falls on the 
CCRL whenever testers get an opportunity to slip it in.

One of the biggest changes so far has been re-working the criteria for Blunder using certain contempt factors during the middle and endgame. In the early days of Blunder, 
I remember it wasn't enough to avoid repetition detection by just returning a value of zero when my engine started repeating moves. So for the middle game, I returned a 
-25 cp penalty for drawing, and only returned a draw value of 0 in the endgame.

The gain here was in better tweaking when Blunder decided it crossed over from the middle to the endgame, as I remember watching several games Blunder played which were drawn 
endgames, and I would've been perfectly happy to see Blunder hold the draw, but the engine was still penalizing itself for drawing. Seeing this I imagined with my previous 
configuration, I was punishing Blunder too harshly and causing it to try to strive for a win too much is drawn endgames and consequently losing. And I seemed to be correct, 
and making a small tweak to this code gained Blunder a respectable 10 Elo.

As for the next ideas I'm going to be trying I'm honestly not too sure, and I'm really starting to miss that day where Elo gains came in 50, 100, or even 200 point chunks 
:-) But on a positive note, I am taking this as a sign that Blunder's starting to move into a more mature development stage. I'll probably be adding a pawn hash in at some 
point, if not now then in the next version, and I'll probably try some little tweaks in the search here and there to see if any of them are worth 10-15 Elo. I also plan on 
experimenting with a more aggressive LMR formula soon, and I'll probably try re-tuning the evaluation with one of Etheral's datasets, after adding in some more positional 
knowledge like rooks on semi-open files and the bishop pair.

When looking at long-term goals, I've already come a lot farther than I ever thought I could when I started 6-7 months ago. Honestly speaking, while I'm sure it's very fun 
to have a 3000+ Elo program, I'd be quite satisfied if I can get Blunder to break 2800, playing around the level of a super-GM. I certainly won't stop the development of 
Blunder when hit this milestone, but I'll probably also start focusing more on making Blunder more user-friendly. I'm not entirely sure why, as I doubt many players would 
care to use yet another chess engine when they have plenty of other good options available. But for the few that I hope will use Blunder to analyze their games or play against, 
I'd like it to be an enjoyable piece of software to work with.

I'm also not sure when I'll finally stop doing Blunder 7.x.x, and make another major release of Blunder. Ideally, this would probably be whenever 
I start touching neural network stuff. I likely won't start with NNUE, although I will be studying it, and I'd like to try working with a simple neural network first.
