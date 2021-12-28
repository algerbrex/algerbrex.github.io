## Evaluation Progress

In the previous blog post I mentioned how I had been tinkering with the evaluation to try to keep Blunder agressive while also making the evaluation more sound. 
One idea I decided to try to implement was a table to initalize the safety counter for each king. If that doesn't make much sense, let me explain a little. 

The way Blunder does king safety currently is an approach used in many, many engines. But before I explain the approach, it's important to think about how king safety 
ought to be scored in the first place.

Unlike other evaluations features, king safety is a bit weird and fickle. For example, if all you have is a lone queen hovering around your opponets king, you really 
don't have much of an attack, and may in fact be opening yourself up to an attack. However, if you slowly build up pressure around your opponets king by bringing your 
queen and supporting it with your other pieces, things become much more dangerous for your opponet, and he better make sure he's able to adequately defend his king.

The way we capture this idea of gradually building up an attack is by using non-linear scores for king safety. We can use a counter and track different small 
details of each king's safety - does the enemy have a queen? Are there several pieces ganging up around the king? Is the king castled? Are there open files near the king? How
many squares does each attacking piece cover? Each of these features by themselves might be worth 2 points, or 5 points, or 3 points.

This counter is then used as an index into a pre-computed table of non-linear king safety scores. This way, we can capture the idea of gradually building up an attack. For
example, while a queen attacking the area around a king might only be 5 points and only get a score of, say, 10 centipawn, if there are two open files near the king, and a 
knight and rook are also putting pressure around the king, we might have 35 points and the score might jump to 100 centi-pawn!

To make things a little more conrete, here's a code sample of how Blunder implements king safety. Below is the non-linear attack table which will later be
indexed by the attacker counter for each side:


```Go
var KingAttackTable [100]int16 = [100]int16{
	0, 0, 1, 2, 4, 6, 9, 12, 16, 20, 25, 30, 36,
	42, 49, 56, 64, 72, 81, 90, 100, 110, 121, 132,
	144, 156, 169, 182, 196, 210, 225, 240, 256, 272,
	289, 306, 324, 342, 361, 380, 400, 420, 441, 462,
	484, 506, 529, 552, 576, 600, 625, 650, 676, 702,
	729, 756, 784, 812, 841, 870, 900, 930, 961, 992,
	1024, 1056, 1089, 1122, 1156, 1190, 1225, 1260, 1260,
	1260, 1260, 1260, 1260, 1260, 1260, 1260, 1260, 1260,
	1260, 1260, 1260, 1260, 1260, 1260, 1260, 1260, 1260,
	1260, 1260, 1260, 1260, 1260, 1260, 1260, 1260, 1260,
}
```

Next, each piece on the board is evaluated, excluding the king, since the king safety calculations depend on evaluating all other pieces first:

```Go
phase := TotalPhase
allBB := pos.SideBB[pos.SideToMove] | pos.SideBB[pos.SideToMove^1]

for allBB != 0 {
    sq := allBB.PopBit()
    piece := pos.Squares[sq]

    switch piece.Type {
        case Pawn:
	    evalPawn(pos, piece.Color, sq, &eval)
	case Knight:
		evalKnight(pos, piece.Color, sq, &eval)
	case Bishop:
		evalBishop(pos, piece.Color, sq, &eval)
	case Rook:
		evalRook(pos, piece.Color, sq, &eval)
	case Queen:
		evalQueen(pos, piece.Color, sq, &eval)
    }
    
    phase -= PhaseValues[piece.Type]
}
```

Each piece's mobility is evaluated, and conviently, the information used to evaluate mobility is also useful in evaluating king attacks. For each piece, the number of squares 
they attack in the enemy king's "zone" is counted, and a different point bonus is given for each square, depending on the piece type. The king zone I use are the 16 outer
squares around the king, and the 8 inner squares. Here's an example of how knight attacks are evaluated:

```Go
usBB := pos.SideBB[color]
moves := KnightMoves[sq] & ^usBB
...
outerRingAttacks := moves & eval.KingZones[color^1].OuterRing
innerRingAttacks := moves & eval.KingZones[color^1].InnerRing

if outerRingAttacks != 0 || innerRingAttacks != 0 {
	eval.KingAttackers[color]++
	eval.KingAttackPoints[color] += uint16(outerRingAttacks.CountBits()) * uint16(MinorAttackOuterRing)
	eval.KingAttackPoints[color] += uint16(innerRingAttacks.CountBits()) * uint16(MinorAttackInnerRing)
}
```

The constants `MinorAttackOuterRing` and `MinorAttackInnerRing` are the point values added to the king attack counter. These values are tuned using 
[Texel Tuning](https://www.chessprogramming.org/Texel%27s_Tuning_Method) and are currently `1` and `3` respectively.

Once all of the attack information is collected, it's combined with information about open files near the king, as well as what I mentioned above concering the king's position.
And finally the points are used to index the table I showed, and this value is the penalty given to the sides king being evaluated:

```Go
// Evaluate the score of a king.
func evalKing(pos *Position, color, sq uint8, eval *Eval) {
	eval.MGScores[color] += PSQT_MG[King][FlipSq[color][sq]]
	eval.EGScores[color] += PSQT_EG[King][FlipSq[color][sq]]

	enemyPoints := InitKingSafety[FlipSq[color][sq]] + eval.KingAttackPoints[color^1]
	kingFile := MaskFile[FileOf(sq)]
	usPawns := pos.PieceBB[color][Pawn]

	// Evaluate semi-open files adjacent to the enemy king
	leftFile := ((kingFile & ClearFile[FileA]) << 1)
	rightFile := ((kingFile & ClearFile[FileH]) >> 1)

	if kingFile&usPawns == 0 {
		enemyPoints += uint16(SemiOpenFileNextToKingPenalty)
	}

	if leftFile != 0 && leftFile&usPawns == 0 {
		enemyPoints += uint16(SemiOpenFileNextToKingPenalty)
	}

	if rightFile != 0 && rightFile&usPawns == 0 {
		enemyPoints += uint16(SemiOpenFileNextToKingPenalty)
	}

	// Take all the king saftey points collected for the enemy,
	// and see what kind of penatly we should get by indexing the
	// non-linear king-saftey table.
	enemyPoints = min_u16(enemyPoints, uint16(len(KingAttackTable)-1))
	if eval.KingAttackers[color^1] >= 2 && pos.PieceBB[color^1][Queen] != 0 {
		eval.MGScores[color] -= KingAttackTable[enemyPoints]
	}
}
```

One last important detail to mention is that if you look closely at the above code, you'll see the king safety is only considered for the middlegame, and only when the
attacking side has at least two attackers and a queen. This helps Blunder better understand that a king isn't necessarily unsafe in the endgame if its exposed, and that there's
usually not much to an attack without a queen or a good amount of attackers.

Overall, Blunder's king safety is getting better, and it looks like its starting to do a decent job at recognizing when it has an opportunity to exploit its opponets 
king safety, or lack thereof. To see the full evaluation code, visit [here](https://github.com/algerbrex/blunder/blob/develop/engine/evaluation.go).
