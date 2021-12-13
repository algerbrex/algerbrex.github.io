## Diminishing Returns

One of the interesting things you run into when you start programming a chess engine is the idea of diminishing returns.

At the start of writing a new chess engine, there are many tried-and-true ideas you can implement to gain several hundred points worth of Elo, such as transposition tables, null-move pruning, simple move ordering, piece-square tables, and repetition-detection, to name a few. I implemented all of these ideas in Blunder, as well as some others, and eventually was able to cross the [2000 Elo mark](http://ccrl.chessdom.com/ccrl/404/cgi/engine_details.cgi?print=Details&each_game=1&eng=Blunder%205.0.0%2064-bit#Blunder_5_0_0_64-bit) with version 5.0.0.

However, as an engine becomes stronger, it often can become much harder to make solid, consistent progress. There aren't as many obvious ideas to implement like at the start, and it starts to become clear why chess programming is such a challenging (yet rewarding) field. I've started experiencing this myself with Blunder in recent weeks. The current release version of Blunder should be somewhere around 2450 Elo, and the current development version should be at 2500. But many ideas I've tried in the past week have been pretty unsuccessful at bringing improvements, many of them being adding knowledge to the static evaluation.

The three main ideas that have stuck were Knight outposts, dynamic time control logic, and re-tuning the evaluation. All were able to make Blunder a bit of a better player, particularly the dynamic time control. Even in the latest version Blunder's time control is still strictly linear, and Blunder 7.4.0 will be the first version to have some sort of dynamic time control, albeit very basic. And the knight outposts, though not as impactful, still certainly seem to help.

As for re-tuning the evaluation parameters, although it technically did improve Blunder (gaining 20+ Elo), I'm still not sure what to make of the changes. For the re-tuning, I opted to try diminishing the value of a pawn, and ramping up the piece value. I also decied to allow the king safety to be more granular. 

On the one hand, the chess Blunder plays now seems to be much better, and more agressive, in many cases. I've seen it play multiple games verus other engines with little to no king safety, where it will ruthelessly hunt the king down and either trade into a winning endgame, or deliver a flashy checkmate. [Here](https://www.chess.com/analysis/game/pgn/3w7YxzQRxS).

However, in some others cases I observed, the current evaluation would cause the engine to blunder quite badly, and in some openings, it would completly break down. Not to mention the evaluation is quite a bit more jumpy in its ranges than before, which I've always taken to be a not-so-great sign.

So at the moment I'm a bit torn. I'm quite fond of the way re-tuning the evaluation has changed Blunder's personality - I've always wanted a strong tactical, agressive engine that would be more than happy to take the intiative to its opponet - and from testing the new evaluation does seem to be stronger, but asthetically, the chess it plays can look pretty bad sometimes, especially in sharp, imbalenced endgames (e.g. Queen and pawns vs pawns, knight, and rook). 

Right now I'm leaning towards re-verting the changes, as I belive there are other valid considerations when creating a chess engine besides the maximum amount of Elo possible, and not to mention the change wasn't a huge jump in play strength, so I wouldn't be giving up much. 

However, the current evaluation terms have inspired me to try to replicate Blunder's current agressive play-style, with a more sound overall evaluation. Currently, I'm running a texel tuning session with some new king safety ideas to see what kind of improvements are possible. Hopefully pretty soon in the future I'll be posting an update on how this tuning session goes.
