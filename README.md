# Pokerbot-Hackathon-MAC-2026

## Introduction
A no-limit hold'em bot for the MAC poker bot hackathon, evolved over several generations of AI-written variants.

## Methodology
This bot was developed using an evolutionary training loop with AI-generated variants.

**Loop**
1. Generation 1: Five bots are written independently by different AI models from the same prompt (`prompts/gen1promptv4.md`).
2. Evaluate: All bots play each other using the `macpoker` engine with duplicate deals (every bot plays every hand from every seat) to cancel out card luck.
3. Select: The best bot is crowned champion. It goes into the next generation unchanged, alongside four new variants: the champion's model gets `prompts/winnerprompt.md`, the others get `prompts/loserprompt`.
4. Repeat. The plan was 5-8 generations; I ran out of time and stopped after gen 2 (see Findings).

**Considerations**
Selecting only against the current generation can cause counter-strategies to cycle (like rock paper scissors). To prevent this, every candidate is also evaluated against:
- House bots: the built-in `call`, `checkfold`, `allin` and `random` bots, a constant yardstick of progress.
- Previous champions (`hall_of_fame/`): a new bot must hold up against past strategies.
- Fixed baselines (`baselines/`): the non-champion gen1 bots, added in gen2 because the field was turning into near-copies of the champion.

When the top two are within noise, `h2h.py` plays them head to head over many duplicate sets to decide.

## Running
```bash
python tournament.py gen2 --subprocess --crown
python h2h.py bots/gen2/bot_b/main.py bots/gen2/bot_c/main.py baselines/gen1_bot_a/main.py house:call --sets 50 --subprocess
```
`tournament.py` ranks a generation by average placement points (scored like the real tournament) and, with `--crown`, copies the winner into `hall_of_fame/`.

## Results
| Generation | Champion | Notes |
|---|---|---|
| 1 | Opus 5.5 (`hall_of_fame/gen1_bot_b`) | Near-tie with GPT6 Astra; [results/gen1result.md](results/gen1result.md) |
| 2 | Opus 5.5 (`hall_of_fame/gen2_bot_b`) | Qwen led the league narrowly but lost head to head; [results/gen2result.md](results/gen2result.md) |

Per-generation scores are in `results/gen*.csv`.

**Final bot: `bots/gen2/bot_b` (Opus 5.5, also `hall_of_fame/gen2_bot_b`).**

## Findings
I ran out of time and stopped the loop at generation 2. Two things made stopping there the right call:
- **Overfitting had started.** Three of the five gen2 bots were near-copies of the champion, so the league began measuring "best version of this bot" rather than "good against different styles". Once the gen1 baselines were added as opponents, the gen2 improvements over the old champion shrank to within noise. Qwen's league-winning change turned out to be tuned to the house bots and was neutral against stronger ones.
- **Diminishing returns versus testing time.** Each further generation needed full league runs plus head-to-head checks to separate near-identical bots, and the hackathon clock ran out before another generation could be tested properly. Submitting a well-tested gen2 bot was a better bet than an untested gen3.

Other observations:
- In both generations, the bot that won the most chips was not the one with the best placement. Only finishing order counts in this format, so consistency beat big wins.
- Small, general improvements (Opus's opponent reads) held up against varied opponents; changes tuned to specific opponent types (Qwen's) did not.
- With five models and two generations the sample is small, so the model comparison is an observation, not a ranking. Gen1 was the controlled comparison (same prompt, no shared code); from gen2 on, bots built on each other's code.

## Future work
After the hackathon I may keep running the loop for fun and personal testing, to see whether more generations, with the gen1 baselines in place from the start, keep improving the bot or level off.
