# Scoring flow: recording one ball

## 1. Who's batting and bowling

The server works out who is on strike from the previous balls. The scorer only
chooses people at these moments, each as its own action:

- **Start innings**: picks the two opening batters and the opening bowler
- **Start over**: picks the bowler for the new over
- **New batter**: picks the incoming batter after a wicket

### Start over checks

| Check                                | Why                                         | Error           |
| ------------------------------------ | ------------------------------------------- | --------------- |
| Bowler is in the bowling team's XI   | Only playing XI members can bowl            | 400 Bad Request |
| Bowler didn't bowl the previous over | Cricket rule: no consecutive overs          | 409 Conflict    |
| Bowler hasn't used up their quota    | Each bowler can bowl a limited no. of overs | 409 Conflict    |

### New batter checks

| Check                              | Why                             | Error           |
| ---------------------------------- | ------------------------------- | --------------- |
| Batter is in the batting team's XI | Only playing XI members can bat | 400 Bad Request |
| Batter hasn't already batted out   | A batter can't bat twice        | 409 Conflict    |

## 2. Inputs for recording a ball (what the scorer sends)

- `runsOffBat`: number of runs the batter scored (0–6)
- `extraType`: none | wide | no_ball | bye | leg_bye
- `extraRuns`: number
- `wicket` (optional): { type, dismissedPlayerId, fielderId? }
  - type: bowled | caught | lbw | run_out | stumped | hit_wicket

## 3. Record ball checks (reject the ball if any fail)

| Check                  | Why                                                      | Error        |
| ---------------------- | -------------------------------------------------------- | ------------ |
| Match is live          | Can't score a match that hasn't started or is finished   | 409 Conflict |
| An over is in progress | A bowler must be chosen before the first ball of an over | 409 Conflict |
| No batter is missing   | After a wicket, the new batter must be chosen first      | 409 Conflict |

## 4. What gets calculated after the ball

- **Legal ball?** Yes unless extraType is wide or no_ball
- **Strike rotates?** If total runs ran is odd
- **Over complete?** balls get calculated for the over
- **Innings over?** number of balls
- **Bowler finished quota?** total overs bowled by the bowler

## 5. Undo

- Deletes only the last delivery (highest sequence)
- If no deliveries exist → error
- Everything else recalculates from the remaining deliveries

## 6. Open questions

- What if the scorer picks the wrong bowler for a whole over? We can change the
  whole over, but not other events.
