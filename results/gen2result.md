# Generation 2 results

## Summary

- **Champion: Opus 5.5 (`bots/gen2/bot_b`)**, crowned as `hall_of_fame/gen2_bot_b`.
- Qwen (`bot_c`) topped both league runs on placement points, but by small margins, and in head-to-head tests it was level with the unchanged gen1 champion. Its main change was tuned against the house bots.
- Opus came out ahead in every head-to-head test (vs Qwen and vs the old champion). None of these differences is statistically decisive, so this is the best-supported choice rather than a proven one.
- Gen2 improved on gen1 only slightly. Against varied opponents, the variants built on the champion were within noise of it.

## Setup

- **Candidates:** the unchanged gen1 champion (`bot_e`, Opus's gen1 bot) plus four variants.
  - Opus (`bot_b`) got the winner prompt: improve its own champion.
  - Qwen (`bot_c`), GLM (`bot_d`) and Astra (`bot_a`) got the loser prompt: choose a base (champion, own bot or a combination).
  - Astra additionally saw every gen1 bot.
- **Starting points, measured by line similarity to the gen1 champion:**

| Bot | Model | Starting point |
|---|---|---|
| bot_b | Opus 5.5 | 97% similar: refined the champion |
| bot_c | Qwen | 97% similar: refined the champion |
| bot_d | GLM | 93% similar: modified the champion |
| bot_a | Astra | 4% similar: continued its own gen1 bot |

## League results

Two runs of `tournament.py`, 20 rounds each (40 tables per bot), subprocess mode. All bots finished every game cleanly (`OK`).

| Bot | Model | Run 1 placement | Run 2 placement | Run 1 game pts | Run 2 game pts |
|---|---|---|---|---|---|
| bot_c | Qwen | **4.513** | **4.100** | 20.28 | 18.85 |
| bot_b | Opus 5.5 | 4.287 | 4.075 | 19.38 | **18.89** |
| bot_e | old champion | 4.037 | 3.975 | 18.80 | 18.48 |
| bot_a | Astra | 3.388 | 3.487 | 17.00 | 17.15 |
| bot_d | GLM | 2.175 | 2.050 | 11.88 | 11.55 |

- **Run 1** (`results/gen2_run1.csv`): reference opponents drawn from the gen1 champion and the house bots.
- **Run 2** (`results/gen2.csv`, `--seed baselines`): the four non-champion gen1 bots were added to the reference pool as fixed baselines, because three of the five candidates were near-copies of the champion and the field had become one family.
- Qwen's lead over Opus shrank from 0.23 to 0.025 placement points, and Opus finished ahead on game points.
- Qwen's lead over the old champion shrank from 0.48 to 0.13. Much of the run 1 "improvement" came from near-copies of the champion playing each other.

## Head-to-head results

`h2h.py`, 50 duplicate sets each, 5-seat tables with fillers `baselines/gen1_bot_a`, `baselines/gen1_bot_d` and `house:call`, same seed for every test so the tests compare directly.

| Matchup | Sets won | Game points per set | Standard error |
|---|---|---|---|
| Qwen vs old champion | 24–22 (4 tied) | +0.06 | 0.46 |
| Opus vs old champion | 25–22 (3 tied) | +0.28 | 0.52 |
| Opus vs Qwen | 29–19 (2 tied) | +0.64 | 0.50 |

- **Order in every test:** Opus first, then the old champion and Qwen level.
- No difference reaches two standard errors. Opus vs Qwen is the strongest at about 1.3.
- Raw output: `results/gen2_h2h_opus_vs_oldchamp.txt` and `results/gen2_h2h_opus_vs_qwen.txt`.

## What each model changed

- **Qwen:** one main change, a "VPIP floor": shove-sized raises from players who play most hands are read as near-random rather than premium. Its own docstring calls this the largest measured win "against the house gauntlet" and "neutral vs strong bots". It was tuned against `house:allin` and `house:random`, which is why its edge disappeared against varied opponents.
- **Opus:** broader changes to opponent reading:
  - preflop fold and raise tendencies pooled across spots, so they're read in a few hands
  - the same loose-player idea applied to all raises
  - a bet-size tell: a postflop bet bigger than that player's usual size is read as a stronger range

  These are general skills rather than exploits of one bot type.
- **GLM:** modified the champion and made it clearly worse. It lost chips in the league in both runs and was the slowest bot (23–26 ms/hand).
- **Astra:** continued its own line rather than adopting the champion. It placed 4th in both runs, with strong results against the house bots but weaker placement, the same pattern as gen1.

## Decision

The league was won by Qwen on placement, but by margins inside run-to-run noise. Following the gen1 precedent (when the league is close, head-to-head tests decide), Opus was crowned gen2 champion: it led every head-to-head comparison, and its improvements are general rather than tuned to the house bots. Qwen's gen2 bot remains in `bots/gen2/bot_c`.

Caveat: this decision was made after the league result, and none of the head-to-head differences is statistically decisive. Opus and Qwen share about 97% of their code, so in practice they are close to the same bot.

## Lessons for gen3

- **Small refinements of the champion are hitting diminishing returns.** Gen3 prompts should ask for larger, deliberate changes aimed at where the champion loses against the baseline styles.
- **Models that can test against the house bots will tune to them.** Prompts should say that the house bots are not the target field.
- **Keep the gen1 baselines in the reference pool.** Without them, the league flatters variants of the champion.
