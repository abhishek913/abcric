# Scoring flow: recording one ball

## 1. Inputs (what the scorer sends)

- `runsOffBat`: number of runs the batter scored (0–6)
- `extraType`: none | wide | no_ball | bye | leg_bye
- `extraRuns`: number
- `wicket`: Bowled | catch | run out | fielder_id | bowler_id | batsman_id |

## 2. Checks (reject the ball if any fail)

| Check                                | Why                                                    | Error           |
| ------------------------------------ | ------------------------------------------------------ | --------------- |
| Match is live                        | Can't score a match that hasn't started or is finished | 409 Conflict    |
| Bowler didn't bowl the previous over | Cricket rule: no consecutive overs                     | 400 Bad Request |
| if batter is correct                 | If last ball one run or out battter should change      | 409 Conflict    |
| if bowler still can bowl             | one bowler can only bowl certain amount of overs       | 400 Bad request |
| if the batter is already out         | batsman can't bet twice                                | 400 Bad request |
| Bowler is from teams playing XI      | Player can't be outside of playing XI                  | 400 Bad Request |
| if batsman is already been out       | Batsman shouldn't bat twice                            | 409 Conflict    |

## 3. What gets calculated after the ball

- **Legal ball?** Yes unless extraType is wide or no_ball
- **Strike rotates?** If total runs ran is odd
- **Over complete?** balls get calculated for the over
- **Innings over?** number of balls
- If bowler has finished quota? total overs bowled by the bowler

## 4. Undo

- Deletes only the last delivery (highest sequence)
- If no deliveries exist → error
- Everything else recalculates from the remaining deliveries

## 5. Open questions

- What if the scorer picks the wrong bowler for a whole over? we can change the
- whole over but not other events
