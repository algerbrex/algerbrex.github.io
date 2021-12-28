## Bugs and More Bugs

One of my goals when I started writing Blunder was to make sure my code was as bug free as possible. 

Of course most people probably don't expect a newer engine to consistenly run without crashing ever, but regardless, I wanted to produce a solid piece of software, and in many areas I put a focus on simplicitly over peformance.

However, while in the process of trying to establish a rating for Blunder in his Gambit Rating List, Ed Schroder - a longer timer in the chess programming community - has informed me of Blunder crashing several times during his testing.

This news isn't particularly suprising since even though I do my best to prevent most bugs from making their way into new releases, I'm only human and far from a perfect programmer, and a chess engine is still quite a complicated piece of software with many innerworkings.

Since Blunder currently has no logging capabilities, I'll likely need to run the same kind of testing Ed is running to determine where the bug is at. My guess right now is that
the history heuristics in Blunder are still overflowing, despite believing that I fixed this issue back in version 6.1.0. The reveleant search code is this

```Go
// Update the history heuristics table if the move that caused a beta-cutoff is quiet.
func (search *Search) updateHistoryTable(move Move, depth int8) {
	if search.Pos.Squares[move.ToSq()].Type == NoType {
		search.history[search.Pos.SideToMove][move.FromSq()][move.ToSq()] += int32(depth) * int32(depth)
	}

	if search.history[search.Pos.SideToMove][move.FromSq()][move.ToSq()] >= MaxHistoryScore {
		search.ageHistoryTable()
	}
}
```

which should update the history heurstic scores for a move if it raised alpha or caused a beta cutoff. The issue before was that while I checked to make sure no entires 
in the table were above `MaxHistoryScore`, I neglected to realize that if an entry in the table, which was of an unsigned 16-bit integer type, exceeded the maximum size of a 16-bit integer, it would overflow before I was able to check if it exceeded `MaxHistoryScore`, since `MaxHistoryScore`'s value was the maximum value of an unsigned 16-bit integer.

I fixed this issue by making the table entires 32-bit integers so I would be able to limit the size of an entry without dealing with potential overflow. With this fix,
I'm not sure what the issue still might be. But I'll likely start here. 

On the bright side, this has now given me the motivation to add logging to Blunder, so issues like this in the future can hopefully be handled much more smoothly.
