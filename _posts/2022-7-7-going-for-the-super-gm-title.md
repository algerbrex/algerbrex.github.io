Going For The Super GM Title
------------------------

Per usual, progress has been slow on Blunder, but it's come along here and there. The current goal, as the title suggest,
is to get Blunder to be playing around 2700-2800, or the level of a what's informally known as a super grandmaster, 
such as Ding Liren, or Ian Nepomniachtchi. Currently Blunder 8.0.0 has not been tested throughly, but I estimate it's rating 
to fall close to 2700 on the CCRL 4/40 rating list, probably something like 2670, so not too much needed to break 2700 - hopefully.

The biggest improvment of Blunder 8.0.0, the latest release of Blunder at the time of this writing, has been improving the evaluation
by generating my own training dataset from several hundred thousand self-play games. This self-play games were the result of about 2
month of self-play testing, at time controls of 10+0.1s, 8+0.8s, and 5+0.5s. So not the highest qualtiy games, but nonetheless, the
results speak for themselves: Against a small gauntlant Blunder gained roughly 34 Elo at a time control of 5+0.5s.

The first step to generating the games was to actual retrieve them from the trash - or more precisely the recycling bin. I had
fiddled with self-play evaluation tuning in the past and failed miserable, so for now I was just deleting the games.pgn files,
as I used them as a form of redundancy in case a test finished during the day but my computer somehow crashed or restarted. I could run
the pgn file through [ordo](https://github.com/michiguel/Ordo) to retrieve the Elo gain. So I had towrite a small Python script to go through the recycling bin, restore a 
single games.pgn file, rename it to games_N.pgn, where N was a running counter used to prevent name conflicts, and then store it in a new folder.

Afterwards I went and cleaned up my PGN parser, which utilized Go's regex library, and refactored my FEN generartion code. The biggest
change I made was selecting, at most, 10 random FENs from each game, and discarding the rest. I believe this was the biggest contributor
towards making this self-play tuning much more succesful than past versions, as the positions has much better variety. I also made
sure to exclude positions where the side to move was in check, the position was likely from the opening book, the position was near the
very end of the game or likely part of a boring draw, or positions from games which were less than 20 full-moves. Thanks to the author of Velvet,
Martin Honert, for giving me the idea to try being more strict with my FEN selection, from skimming through his Python FEN generation script.
Otherwise, here's the relevant code:

```Go
pgns := parsePGNs(infile)
numPositions := 0
fens := []string{}

for i, pgn := range pgns {
	log.Printf("Extracting positions from game %d\n", i+1)

	search.Pos.LoadFEN(pgn.Fen)
	gamePly := len(pgn.Moves)
	possibleFens := []string{}

	if gamePly <= 40 {
		continue
	}

	for _, move := range pgn.Moves {
		search.Pos.DoMove(move)
		search.Pos.StatePly--

		if search.Pos.InCheck() {
			continue
		}

		if search.Pos.Ply > 200 {
			continue
		}

		if search.Pos.Ply < 10 {
			continue
		}

		if gamePly-int(search.Pos.Ply) <= 10 {
			continue
		}

		pvLine := engine.PVLine{}
		search.Qsearch(-engine.Inf, engine.Inf, 0, &pvLine, 0)

		fields := strings.Fields(applyPV(&search.Pos, pvLine))
		result := OutcomeToResult[pgn.Outcome]

		possibleFens = append(possibleFens, fmt.Sprintf("%s %s - - 0 1 [%s]\n", fields[0], fields[1], result))
	}

	samplingSize := engine.Min(samplingSizePerGame, len(possibleFens))
	for i := 0; i < samplingSize; i++ {
		fens = append(fens, possibleFens[rand.Intn(len(possibleFens))])
		numPositions++
	}
}

rand.Shuffle(len(fens), func(i, j int) { fens[i], fens[j] = fens[j], fens[i] })

file, err := os.OpenFile(outfile, os.O_APPEND|os.O_WRONLY, 0644)
if err != nil {
	panic(err)
}

seen := make(map[string]bool)
for _, fen := range fens {
	if seenBefore := seen[fen]; !seenBefore {
		seen[fen] = true
		_, err := file.WriteString(fen)
		if err != nil {
			panic(err)
		}
	} else {
		log.Print("skipping duplicate position")
		numPositions--
	}
}

log.Printf("%d positions succesfully extracted!\n", numPositions)
file.Close()
```

As can be seen, to actually extract the FENs, I applied the principle variation of a quiescene search, and got the tail-end position,
This made the most sense to me, since such positions are what the actual evaluation function is going to be encountering.

This whole process gave me ~1.3M FENs to tune with. I extracted 300K, mixed them with the well known Zurichess dataset, and got a new training set of
~1M positions, which I used to tune Blunder's evaluation parameters from scratch.

As I mentioned, the resulting evaluation seem to play much nicer, not Blundering nearly as much, and statiscally it showed Blunder had gotten stronger.
These results were somewhat unexpected for me, but nevertheless, I'll now defintely devote more time to exploring using self-play tuning in the future.

Besides the above changes, the biggest strength gainging change so far has been trying a slightly different late-move reduction scheme, 
where we first search with a null-window and reduced depth, and if this search fails low, instead of assuming the null-window caused the
fail-low, and doing another search with a full-window but reduced depth, I switched to the opposite. Now it's assumed the reduced-depth
caused the fail-low, so we search to depth-1, with a null-window.

So far this change looks promising, although the Elo gain has dropped from 18 Elo to 11 Elo after 2K games. But for a very simple change and no lines of
code added, I'd be pretty happy with 8-11 Elo. Which would hopefully put the dev version of Blunder at roughly 50 Elo stronger than Blunder 8.0.0, and
hopefully above 2700. If testing does show Blunder dev is significantly stronger than Blunder 8.0.0 at decently slow time controls (e.g. 15+0.2s or
5+0.5s), then I'll likely be releasing Blunder 8.1.0 within the next week or so.

Once I've confirmed I've broken 2700, I'll likely begin turing my focus towards getting a neural net in Blunder. Not right away, I want to really try to exhaust
the power of HCE and search tuning. In an ideal world I'd be able to break 2800 comfortable, and only then really start looking towards neural nets. But
with the rate that development has been going, I think I'm starting to exhaust my testing and programming capabilites using my current approaches. I'll make
the choice once Blunder reaches 2700, so I have a bit of time to consider my options. If I do end up adding a neural net, as part of the Blunder 9.0.0 release,
then I'd really like it to be simple and original. A simple architecture, likely something like 768 -> 16 -> 1, trained using my own data and trainer. Because
of this I likely won't see a massive 200-300 point Elo gain like I might if I used more established methods, but as I've written before, Elo is not my only
goal in working on Blunder. Orginality, understanding, learning, and having something I can call my own, and more importantly, understand inside and out, is
more what I'm after!
