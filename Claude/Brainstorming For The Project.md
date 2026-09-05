---
fileClass: Project
Category: Claude
Status: Active
Author: Levi Kuhn
Last Updated: 8/30/2026
Version: 0.2
tags:
---
l# Overview
Go into plan mode and use this document for your planning. Don't ask permission to modify it or work in .claude/plans. This is your plan file. Please leave this Overview alone and build the plan in the following sections. 

There is a website https://barricade.gg where you can play a multiplayer online version of a game similarly known as "Quoridor", with slight rule set variation. It does not have a public API. 

I'd like to create an AI model trained through reinforcement learning via self-play. I know that OpenSpiel has an existing AI model for Quoridor with a slightly different rule set. In a different prompting session I learned that I can alter the existing game rules of that model before training. This approach seems time efficient and more conducive to actually completing the project. 

Can you help me organize my thoughts here and give suggestions for things you can think of and help organize this set of ideas into more of a cohesive plan? I'm not ready to move to implementation or design yet. I'd like to keep brainstorming with your help so please think very deeply.
# Restating the Goal (so we're aligned before answering anything below)

Train a Quoridor-family agent by AlphaZero-style self-play, using OpenSpiel's existing `quoridor` game as the starting point, modified so its rules match the barricade.gg variant. The near-term deliverable is a strong self-play agent evaluated offline; interacting with barricade.gg itself (no public API) is a separate, optional, and legally-loaded question.

Two things dominate everything else: (1) the exact rule delta between barricade.gg and OpenSpiel's Quoridor, and (2) whether the goal is "an agent that plays well" or "an agent that plays well *on barricade.gg*." Almost every question below hangs off those two.

# Open Questions

## 1. Rules and Game Definition

1.1. What exactly are the barricade.gg rule differences? This is the single highest-value unknown and the one only you can answer. Needed at minimum:
- Board size (9x9 assumed?) and starting positions.
- Walls per player. OpenSpiel defaults to `board_size^2 / 8` = 10 on a 9x9.
- Wall legality: is the "you may not fully block a player's path to their goal" rule enforced? OpenSpiel enforces it via a path search on every wall placement.
- Pawn jump rules when pawns are adjacent: straight jump allowed? Diagonal jump allowed only when blocked by wall/edge, or always? This is the most common variant divergence and it changes the action space.
- Draw / repetition / move-limit rules. OpenSpiel has *no* repetition rule and just caps games at `4 * board_size^2` moves. Does barricade.gg have a draw condition, a move limit, or only clock loss?
- Any barricade.gg-specific mechanics not in classic Quoridor (special wall types, more than 2 players, scoring beyond win/loss, tie-break by walls remaining or distance).
Levi: The board is 9x9. Both players start in the middle of the bottom and top row. Each player gets 10 walls. You may not fully block a player's path to victory. When the two players are adjacent, special movement applies: you can jump over your opponent to the square directly behind them. If there's a barricade (or the board edge) directly behind your opponent, you can't jump straight — instead you move diagonally to either square beside them. Barricade.gg I believe only operates on clock loss. However, let's make the draw condition the same as chess, three repeated positions means draw. There are no tie breaks for a draw. 


1.2. How will you *verify* the rule delta rather than assume it? Suggestion: hand-record 20–30 real games (or positions you construct deliberately) from barricade.gg, then write a conformance test that replays them against your modified OpenSpiel game and asserts identical legal-move sets and outcomes. Do you have a way to capture game records from the site, or is this manual transcription?
Levi: Online games on Barricade.gg are recorded in this format: 
"1.e2e8
2.e3e7
3.e4e6
4.he5hd4
5.f4f6
6.g4hf4
7.h4vh6
8.vh8e6
..."
The number is the turn. The first 2-3 characters are the first players move, the last 2-3 characters are the second players move. Turn 4 has 2 walls placed, turn 6 has 1 wall placed, turn 7 has 1 wall placed, and turn 8 has 1 wall placed. Are you able to decipher and integrate this notation into the project?

1.3. If the rules turn out to be *identical* to OpenSpiel's Quoridor, does the project still hold together, or does the variation matter to the assignment/goal?
Levi: If they are exactly the same that is fine. I am interested in the learning component of reinforcement learning. 

1.4. Is this 2-player only, or do you care about the 3/4-player modes OpenSpiel also supports?
Levi: This is a 2-player only game. 
## 2. Scope and Definition of Done

2.1. What is the actual success criterion? Candidates, pick one primary:
- Beats OpenSpiel's plain MCTS bot at N simulations, X% of the time.
- Beats a strong shortest-path heuristic baseline.
- Beats *you* consistently.
- Reaches some rating on barricade.gg (implies bot play — see §6).
Levi: The first three are good success criterion. I would say the primary goal should be the beating OpenSpiel's plain MCTS bot, while my secondary goal is for it to beat me. 

2.2. Is there a hard deadline (class project? see `Class/Project.md` conventions in this vault) and what's the time budget in wall-clock weeks and in hours/week?
Levi: There is no hard deadline. I'm unsure of how long this project will take, how long do you estimate? I am willing to put at least 2 hours per week for this project. 

2.3. Is the deliverable the *agent*, the *training pipeline*, or a *write-up/analysis*? That changes how much effort goes into reproducibility, logging, and ablations versus raw strength.
Levi: I want to make this project public and put it on my resume if possible. This means I would want good documentation. I think the strength is a nice metric to have but I dont want it to come at the cost of documentation. Which would you recommend?

2.4. Is there a stopping rule — what does "good enough, ship it" look like so training doesn't expand to fill all available time?
Levi: Good enough to beat me 9/10 times would be a good metric. I'm not sure about the plausibility of any percentage win rates against OpenSpiel's plain MCT bots, perhaps you can make some estimates.
## 3. Tooling and Implementation Path

3.1. Which AlphaZero implementation? OpenSpiel ships several and they are not equal:
- **C++ `alpha_zero_torch`** (LibTorch): fastest, but a community contribution, explicitly "not regularly tested nor maintained by the core team," with known pybind11 interference issues.
- **Python AlphaZero** (TF-based): easier to modify and debug, much slower actor throughput.
- **Roll your own loop** on OpenSpiel's Python/C++ bindings using its MCTS as the search, with PyTorch for the net: most control, most work.
The vault template mentions "I will build the C++ server and run tests myself" — is a C++ build the assumed path already?
Levi: I have now removed "I will build the C++ server and run tests myself". This was a mistake from porting the obsidian template from a previous project. Please ignore. Tell me more about alpha_zero_torch/AlphaZero, where would the difficulties lie?

3.2. What OS/toolchain are you building on? OpenSpiel + LibTorch on Windows is meaningfully harder than on Linux/WSL. Is WSL2 or a Linux box acceptable?
Levi: My computer is Windows, is there an easier alternative? What would you recommend?

3.3. Where do the rule changes live — a patched fork of `quoridor.cc`, or a new game registered as e.g. `barricade` alongside it? A separate game name keeps the upstream game intact for A/B comparison and makes rebasing on upstream far less painful. Any objection to the fork-as-new-game approach?
Levi: Give me more context. Why would you not just update the existing rule set? How hard would it be to implement if you had a separate game name?

3.4. How do you want to track upstream OpenSpiel? Pin a commit and never move, or rebase periodically?
Levi: Give me more context, I dont understand what you are asking me.

3.5. Licensing: OpenSpiel is Apache 2.0. Is this project going to be public (GitHub), and does the class or anyone else have a claim on it?
Levi: This project is going to be public. There is no "class", this is a personal project.  
## 4. Learning Setup

4.1. What hardware is available for training? A single consumer GPU, CPU-only, cloud credits, university cluster? This is the biggest single constraint on what's realistic and should be answered before any design work.
Levi: I have a gaming pc with an average CPU/GPU I plan to use. 


4.2. Do you want to start on a smaller board (5x5 or 7x7) to get the whole pipeline working end-to-end in hours instead of days, then scale to 9x9? I'd strongly recommend it, but it costs you a curriculum/transfer decision: retrain from scratch at 9x9, or transfer weights?
Levi: I'll go with your recommendation. But give me a little more reasoning for why this is the better choice.

4.3. State representation: what goes into the input planes? Beyond pawn positions and wall occupancy, do you want engineered features (each player's BFS shortest-path length, walls remaining)? Pure-AlphaZero purism says let the net learn it; pragmatism says shortest-path distance is expensive for a conv net to compute and cheap for you to hand it. Which side do you want to land on?
Levi: Am I sacrificing anything if I let the net learn it? Elaborate more on the pros and cons of the decision.

4.4. Action space: OpenSpiel encodes moves and walls on the `2*board_size-1` "diameter" grid. Keep that encoding (simplest, reuses their code) or design your own? If jump rules differ, does the encoding still cover every legal move?
Levi: Based on the very slight difference I mentioned in 1.1, I believe that we will be able to reuse the existing OpenSpiel action space. Do you agree based on my description?

4.5. Symmetry / data augmentation: Quoridor has a left-right mirror symmetry, and a color-swap symmetry (flip the board and swap players). Do you want to exploit both to multiply training data? Worth deciding early because it constrains the representation.
Levi: It makes sense to because it could multiply the training data by 4. How does it constrain the representation?

4.6. What are the knobs you'll actually tune, and which do you fix by fiat? (MCTS simulations per move, c_puct, Dirichlet noise, temperature schedule, replay buffer size, net width/depth, checkpoint cadence.) Fixing most of them up front is how this project finishes.
Levi: I am not familiar with these "knobs". Can you explain each of them?
## 5. Evaluation

5.1. What's the baseline ladder? Suggested rungs: random → shortest-path greedy heuristic → OpenSpiel MCTS at 100 sims → MCTS at 1000 sims → previous checkpoints of yourself.
Levi: Those sound like good benchmarks. Based on hardware limitations, I'm not sure we will beat the MCTS on 1000 sims but we can aim for it. 

5.2. How do you measure progress — Elo computed from a round-robin against frozen past checkpoints? Fixed benchmark position suite? Both?
Levi: What is "fixed benchmark position suite"?

5.3. First-player advantage in Quoridor is real. Are all evaluations played as paired games with colors swapped?
Levi: Sure. 

5.4. Do you want an interpretability/analysis component (e.g. does the learned policy discover known human wall openings), or is strength the only metric?
Levi: What would an interpretability component look like?
## 6. barricade.gg Interaction (decide *whether*, before deciding *how*)

6.1. Do you actually need to play on barricade.gg, or is it just the rules reference and inspiration? If it's only a rules reference, this whole section collapses and the project gets much simpler and safer.
Levi: It's just rules reference. I can test it online by hand if necessary. One of my biggest aims is seeing if it can beat me at the game, if I want to expand that goal we can cross that bridge when we get there. 

6.2. If you do want to play there: what do barricade.gg's Terms of Service say about automated play, bots, and scraping? This needs to be read before anything is built, not after. Automated play against unwitting human opponents is the kind of thing that gets accounts banned and is a genuine ethics question for a class project.
Levi: This is not for a class project, however the ethics are important. The ToS states: "You agree not to: Cheat or use any unauthorized third-party software, bots, engines, or artificial assistance during gameplay". While we are adopting the rule set, we will not use it on the site. 

6.3. If ToS permits it (or permits it against yourself/a friend), what's the interface — browser automation against the web client, or an inspection of the site's network traffic? Both are brittle; browser automation is the more defensible.
Levi: ToS does not permit engines. 

6.4. Is there a "human evaluation" path that avoids the issue entirely — you play the bot yourself, offline, and record results?
Levi: I will probably play the bot offline and record results.
## 7. Project Logistics

7.1. This note's frontmatter has no `Author`, `Last Updated`, `Version`, or `tags`, though `Templates/Claude.md` includes them. Want them added here for Dataview consistency?
Levi: I have updated the frontmatter. If its still not consistent let me know. 

7.2. `Vault Setup.md` refers to the vault as `Barricade.gg-AI` under `C:\Users\LJK\...`, but the actual folder is `BarricadeAI` under `C:\Users\LeviJKuhn\...`, and the template lists the author as Mason Bendixen. Worth correcting so the setup note stays reproducible?
Levi: I have updated these mistakes. They were remnants of a previous project template. If they are still inconsistent let me know. 

7.3. How do you want this project's notes split across the vault — Research / Planning / Implementation / Documentation notes in `Projects/`, with this file staying the Claude-facing plan? Where should the rule-delta spec live?
Levi: 

# Ideas and Suggestions

**Write a rules-delta spec before writing any code.** One note, a table with three columns: rule, OpenSpiel behavior, barricade.gg behavior. Everything downstream — the game fork, the conformance tests, the action encoding — is generated from that table. It is also the single artifact most likely to be wrong, so it deserves to exist on its own where it can be reviewed and corrected.

**Register the variant as its own game, don't patch Quoridor in place.** `barricade` as a sibling of `quoridor` means you can pit the two rule sets against each other, keeps upstream tests green, and makes any future OpenSpiel update a merge you can survive.
> **Revised in Round 2:** superseded. The rule delta turned out to be a single rule (threefold repetition), so neither a fork nor a patch is warranted — implement it as a Python wrapper around the stock game and keep the pip-installed OpenSpiel. See §3.3 in Round 2.

**Get the pipeline green on a 5x5 board first.** A 5x5 Quoridor with 3 walls each trains to something clearly non-random very fast. The point isn't the agent, it's proving that self-play → replay buffer → training → checkpoint → evaluation loop actually closes. Every bug you'd hit at 9x9 you'll hit at 5x5 for a fraction of the compute.

**Build a heuristic baseline early and keep it forever.** A bot that always walks its own BFS shortest path, and places a wall only when the opponent's path is shorter than its own, is ~50 lines and is a surprisingly hard opponent for an undertrained net. It's your smoke test for "is training doing anything at all."

**Consider whether you need AlphaZero at all for the first milestone.** OpenSpiel's MCTS with a *perfect-information rollout* evaluator plays decent Quoridor with zero training. If your goal is "strong bot," a tuned MCTS is a legitimate strong baseline and a fast path to a working artifact; the neural net is what beats it, and framing the project that way gives you a deliverable at every stage instead of only at the end.

**Feature the shortest path.** Quoridor's value function is dominated by "difference in shortest-path lengths." Giving the network that as an input plane (or two scalars) is likely worth more than a deeper net. If you want the purist version too, that's a clean ablation for the write-up.

**Exploit the mirror symmetry.** Free 2x on data with a few lines in the training loop. The color-swap symmetry is a further 2x, but only if the state encoding is canonicalized to "current player is always moving up."

**Plan for checkpoints and resumption from day one.** The C++ AlphaZero writes a `config.json` and resumes from the latest checkpoint; whatever path you choose, assume training will be interrupted, because it will be. Related: decide now where checkpoints live and whether they go in git (they should not — the vault is already a git repo, and a `.gitignore` that keeps model weights out is worth writing before the first run, not after).

**Log obsessively, plot weekly.** Value loss, policy loss, game length, wall-usage rate, and win rate versus each baseline rung. Game length and wall-usage are the two that tell you the agent is learning *Quoridor* rather than learning to shuffle.

**Beware the no-repetition gap.** OpenSpiel's Quoridor has no repetition rule, so a self-play agent can learn to shuffle back and forth forever in drawn-ish positions and burn your compute on 300-move games. If barricade.gg has a repetition or move-limit rule, implement it in the fork early; if it doesn't, consider a training-only move cap anyway.
> **Revised in Round 2:** confirmed and now scoped. This is *the* rule delta. Levi's answer to §1.1 sets the rule as chess-style threefold repetition; OpenSpiel has none and only draws at its 324-move cap. Implement in a wrapper, not a fork.

> **Revised in Round 2:** moot for now — §6.1 puts barricade.gg play out of scope, so there is no deployment clock to tune against. Keep it in mind only if that changes.

**Time control is a rule too.** If barricade.gg games are clocked and you ever intend to play there, the agent's simulation budget per move is a hard constraint, and an agent tuned at 1600 sims/move that gets 200 in practice is a different, weaker agent. Decide the deployment budget before tuning.

# Suggested Order to Answer These

1. §1.1 rule delta and §6.1 whether barricade.gg play is in scope — these two gate everything.
2. §2 scope, deadline, and success criterion.
3. §4.1 hardware, then §3.1 which AlphaZero implementation (hardware largely picks it for you).
4. §5 evaluation ladder — define it *before* training, so the numbers mean something.
5. Everything else is a design-phase decision, not a brainstorming one.

# Round 2 — Answers, Corrections, and the Notation Spec

## Headline: your rule set is, with one exception, already OpenSpiel's Quoridor

I read OpenSpiel's `quoridor.cc` against your §1.1 description. Every rule you listed is already implemented:

| Rule (your description) | OpenSpiel `quoridor` | Delta |
|---|---|---|
| 9x9 board | `board_size` default 9 | none |
| Both pawns start middle of their back row | same | none |
| 10 walls each | default `board_size^2 / 8` = 81/8 = 10 | none |
| May not fully block a path to the goal | enforced in `IsValidWall()` via a path search on every placement | none |
| Straight jump over an adjacent opponent | implemented — comment reads *"In two-players: A normal jump is allowed. We know that spot is empty."* | none |
| Diagonal side-step only when the straight jump is blocked by a wall or edge | implemented — *"We are jumping over the other player against a wall, which side jumps are valid?"* then checks both diagonals | none |
| Threefold repetition = draw | **not implemented.** OpenSpiel has no repetition rule; it declares a draw only at the move cap (`4 * board_size^2` = 324 moves) | **the only delta** |

This is the most important finding in this round and it changes the shape of the project: **there is no meaningful rule fork.** You are not modifying a game, you are adding one rule to it. Sections 3.3 and 3.4 below are re-answered in that light, and two bullets in the Ideas section above are marked as revised.

It also means §1.3 resolves cleanly in your favor — you said you're in this for the reinforcement-learning component, and now essentially all of your effort can go there instead of into rules engineering.

## 1.2 — The barricade.gg notation, deciphered

**Yes, I can read it, and it's unambiguous.** Here is the grammar:

```
game    := turn+
turn    := <int> "." move move?          # trailing move may be absent on the final turn
move    := pawn | wall
pawn    := <file><rank>                  # file a-i, rank 1-9 — the destination square
wall    := ("h"|"v") <file><rank>        # file a-h, rank 1-8 — the wall's anchor square
```

The 2-vs-3 character ambiguity is only apparent: parse left to right, and if the next character is `h` or `v` you are looking at a 3-character wall token, otherwise a 2-character pawn token. `6.g4hf4` splits cleanly into `g4` + `hf4`; `8.vh8e6` into `vh8` + `e6`. No lookahead needed.

**Wall semantics** (inferred — see the evidence below):
- `h<file><rank>` — horizontal wall sitting on the boundary between rank *r* and rank *r+1*, spanning files *f* and *f+1*. Blocks vertical movement in both of those files.
- `v<file><rank>` — vertical wall on the boundary between file *f* and file *f+1*, spanning ranks *r* and *r+1*. Blocks horizontal movement in both of those ranks.

That is the standard "name the wall by the square to its south-west" convention, and it means wall anchors only ever use files a–h and ranks 1–8, which matches your sample (no wall token in it exceeds `h` or `8`).

**Your sample game decoded**, with P1 starting on e1 and P2 on e9:

| Turn | P1 | P2 |
|---|---|---|
| 1 | e1→e2 | e9→e8 |
| 2 | e2→e3 | e8→e7 |
| 3 | e3→e4 | e7→e6 |
| 4 | wall h/e5 (blocks rows 5-6 across files e,f) | wall h/d4 (blocks rows 4-5 across files d,e) |
| 5 | e4→f4 | e6→f6 |
| 6 | f4→g4 | wall h/f4 (blocks rows 4-5 across files f,g) |
| 7 | g4→h4 | wall v/h6 (blocks files h-i across ranks 6,7) |
| 8 | wall v/h8 | f6→e6 |

**Why I'm confident in the anchor convention.** Three placements in your sample only make sense under this reading:
- Turn 4, P1's `he5`: P2's pawn is on e6 walking *down*; the wall blocks e6→e5. Purposeful.
- Turn 4, P2's `hd4`: P1's pawn is on e4 walking *up*; the wall blocks e4→e5. Purposeful. This one also rules out a leftward span — files c,d would block nothing.
- Turn 6, P2's `hf4`: P1's pawn is on f4; the wall blocks f4→f5. Purposeful.

The alternative readings (span leftward from the anchor, or sit on the boundary *below* the anchor rank) each make at least one of those three walls a wasted move, and turn 5's mutual sidestep to the f-file is exactly what you'd expect from two players walking around walls placed as described.

**One five-minute confirmation before trusting this.** Play a throwaway game on the site, place four walls at known positions — one horizontal near a1, one horizontal near h8, one vertical near a1, one vertical near h8 — and read the notation back. The corners are where an off-by-one in the anchor convention becomes obvious. Paste the result here and I'll lock the spec.

**How this gets integrated.** A ~60-line parser converting a record to a list of OpenSpiel actions, plus the inverse for writing records out. The two tests that matter:
1. **Round-trip**: parse a record to actions, serialize the actions back to text, assert byte-identical output.
2. **Conformance**: replay every move through the OpenSpiel state and assert each one is in `legal_actions()`. If any real game contains a move the engine calls illegal, you've found a genuine rule delta — this is the cheapest possible rules-verification harness, and it's why the notation matters beyond convenience.

Twenty to thirty transcribed games is plenty. A side benefit: those games are also a small supervised warm-start dataset if you ever want to pretrain the policy head before self-play.

**Known gaps in the format**: it records no result, no resignation, no clock. You'll want a wrapper file (JSON or YAML frontmatter) carrying `winner`, `termination`, and a source URL per game.

## Answers to your follow-ups

### §2.2 — How long will this take?

At 2 hours/week, hands-on, with the phase plan below:

| Phase | Hands-on hours | At 2 hr/wk |
|---|---|---|
| 0. Environment + "a random bot plays a legal game" | 3–6 | 2–3 weeks |
| 1. Baselines + evaluation harness (this is your foundation) | 6–10 | 3–5 weeks |
| 2. Notation parser + conformance tests + repetition rule | 4–8 | 2–4 weeks |
| 3. AlphaZero loop working end-to-end on 5x5 | 8–14 | 4–7 weeks |
| 4. 9x9 training, tuning, babysitting | 10–20 (plus weeks of *unattended* GPU time) | 5–10 weeks |
| 5. Documentation, README, write-up, "play it yourself" script | 6–10 | 3–5 weeks |

Roughly **40–70 hours of your time, spread over 5–8 months.** The good news is that Phase 4's cost is mostly wall-clock your GPU spends while you're asleep, not hours at the keyboard — which suits a 2 hr/week budget unusually well.

One piece of advice about the budget shape: 2-hour sessions are a bad fit for Phase 0 specifically. Environment setup is the one task where an interrupted session costs you the whole session. If you can find one 4-hour block anywhere, spend it there.

### §2.3 — Documentation vs. strength: my recommendation

**Documentation, clearly — and it isn't close.** Here's the reasoning rather than just the verdict.

Strength is unverifiable to a reader. "Beats MCTS at 1000 simulations 87% of the time" is a number only you can reproduce, on a game and a baseline nobody else has calibrated. A reviewer skimming your GitHub for ninety seconds cannot tell a strong agent from a weak one.

What they *can* evaluate in ninety seconds: a README that explains the problem, a training curve that goes up, an ablation table showing you tested a hypothesis, and a `play.py` that lets them lose to your bot in their browser or terminal. Those are the artifacts that make a project legible.

The deeper point: those two goals are not actually in tension the way you're framing them. Every piece of infrastructure that makes a project well-documented — the evaluation harness, the logging, the benchmark suite, the ablations — is *also* what makes the agent strong, because it's how you find out that training isn't working. Undocumented RL projects fail silently. The 5x5 phase and the baseline ladder aren't documentation overhead; they're the debugging apparatus.

So: **target strength = "beats me 9/10 and beats the shortest-path heuristic ≥95%," and every hour beyond that goes into the write-up.** Write docs as you go, in the vault, and curate them into the public README at the end.

### §2.4 — Realistic win rates against the MCTS baselines

An important caveat first, because it affects your primary success criterion: **plain MCTS is a much weaker baseline in Quoridor than it is in most games.** OpenSpiel's default MCTS evaluator uses random rollouts, and random play in Quoridor is close to useless as a signal — two random walkers on a 9x9 board scattering walls take hundreds of moves to finish, and who wins is nearly independent of the position at the root. In games like Go or Hex, random rollouts carry real information; in a race game they mostly carry noise.

Practical consequences:
- MCTS @ 100 sims: expect a trained agent to beat it **>95%**, possibly quite early in training. Treat clearing this rung as a smoke test, not a milestone.
- MCTS @ 1000 sims: **90%+ is realistic**, and it's not the compute wall you're worried about — the wall is the evaluator's quality, not its budget.
- The **shortest-path heuristic** is likely to be your genuinely hard rung, and I'd expect an undertrained net to lose to it badly. Make this your headline baseline.
- Stronger honest baseline, if you want one: give OpenSpiel's MCTS a *shortest-path-differential evaluator* instead of random rollouts, and let it run at 1000 sims. That is a real opponent and a much more meaningful number in a README.

So I'd gently amend §2.1: keep MCTS as the primary *stated* criterion if you like, but understand that heuristic-MCTS and your 9/10 personal target are the ones carrying real information.

### §3.1 — What alpha_zero_torch actually is, and where the pain lives

AlphaZero is the algorithm: self-play games are played using MCTS guided by a neural network that outputs a policy (move priors) and a value (win probability); the games are stored; the network is trained to predict the search's move distribution and the game's actual outcome; the improved network makes the next batch of self-play stronger. That loop is the whole idea, and it's about 400 lines of Python. The implementation is not the hard part — the plumbing and the compute are.

OpenSpiel ships three versions:
- **Python (TensorFlow)** — readable and modifiable, but per the OpenSpiel docs it "uses one process per actor/evaluator, doesn't support batching for inference and does all inference and training on the cpu." That last clause is disqualifying: it will not touch your GPU.
- **C++ TensorFlow** — fast, threaded, batched, GPU-capable, but requires building OpenSpiel from source against TensorFlow's C++ API, which is genuinely unpleasant.
- **C++ LibTorch (`alpha_zero_torch`)** — same performance story, and the README says outright that it's a community contribution "not regularly tested nor maintained by the core team," may "fail to build or function properly without warning," and has "known problems with the C++ PyTorch: interferences with pybind11 versions."

Where the difficulties concentrate, in order:
1. **The build.** You must set `OPEN_SPIEL_BUILD_WITH_LIBTORCH=ON` and `OPEN_SPIEL_BUILD_WITH_LIBNOP=ON`, download a LibTorch matching your exact CUDA version, and get its ABI to agree with the pybind11 in your Python environment. On Windows this is worse than on Linux. Realistically 3–10 hours of your budget, with a real chance of ending in a wall.
2. **Debuggability.** When training silently fails to improve — and it will, at least once — you want to print tensors, dump a replay buffer, and plot a value head. In C++ that's a rebuild each time.
3. **Maintenance risk.** An unmaintained component that breaks on an upstream change, in a project you touch two hours a week, can cost you a month of calendar time to a problem that isn't yours.

**My recommendation: don't use any of them.** Use OpenSpiel for what it is unambiguously excellent at — a correct, fast, tested game implementation plus a solid MCTS — and write the AlphaZero loop yourself in PyTorch. Reasons that apply specifically to you:
- You said the learning component is what you're actually interested in. The AZ loop *is* that component. Delegating it to a C++ binary and keeping the build headaches is the worst trade available.
- It runs on your GPU with zero build effort, because you're just using stock PyTorch.
- It is the part of the repo a reader will look at. "I implemented AlphaZero" reads very differently from "I ran someone's AlphaZero."
- It's roughly 400–600 lines and the pieces are individually simple.

The cost you're accepting: Python self-play is slower than C++ self-play. The mitigation is batched MCTS (run many self-play games concurrently and evaluate the network on batched leaf positions rather than one at a time), which recovers most of the gap and is a good exercise in itself.

### §3.2 — Windows

Good news that changes the calculus: **OpenSpiel now publishes Windows wheels on PyPI** (2.0.2, August 2026, Python 3.11–3.14). So the first thing to try is simply:

```
pip install open_spiel
python -c "import pyspiel; print(pyspiel.load_game('quoridor').new_initial_state())"
```

If that works — and combined with the roll-your-own-PyTorch recommendation above, it means **no C++ build at all**, on your existing Windows install, with native CUDA PyTorch. That is a 20-minute Phase 0 instead of a multi-week one.

If the wheel misbehaves, fall back to **WSL2 + Ubuntu**, which gives you the better-tested Linux wheel, full CUDA passthrough to your GPU, and a normal Unix toolchain, at the cost of an afternoon. I'd keep WSL2 as plan B rather than plan A now.

Only if you later decide you need C++-speed self-play does the build question come back — and by then you'll have a working pipeline to compare against, which is the right time to take on a risky dependency.

### §3.3 — Fork vs. patch vs. neither

I asked this when I assumed a real rule delta. Given that the only delta is threefold repetition, here's the fuller picture.

Why you wouldn't just edit `quoridor.cc` in place:
- It requires building OpenSpiel from source forever. Under the pip recommendation above, you'd be giving up the wheel to change one rule.
- Upstream's own `quoridor_test` now tests your modified game, so their tests may fail and yours are entangled with theirs.
- You lose the ability to run stock-vs-modified comparisons.
- Every upstream update becomes a merge conflict in code you didn't write.

Cost of a separate registered C++ game (`barricade` beside `quoridor`): copy ~1,100 lines, rename the registration struct, add a test file, and commit to building from source. Call it a day of work plus the permanent build tax. It's the right answer when the rule delta is large. Yours isn't.

**So: neither.** Implement threefold repetition as a thin Python wrapper around the stock game — hold a `Counter` of position hashes, and have your wrapper report a draw when any count hits 3. Fifty lines, no build, no fork, and it lives in *your* repo where it's yours to test. If the delta ever grows past what a wrapper can express, escalate to the separate-game option then.

One design note: repetition makes the game formally non-Markov in the board alone — the same board can be a draw or not depending on history. Chess has exactly this property and AlphaZero handles it fine by ignoring it in the network input and letting the search see terminal draws. Do the same: don't feed repetition counts to the network, just let the wrapper end the game.

### §3.4 — What I was asking about upstream tracking

Plainly: OpenSpiel is actively developed and changes over time. If your project builds on version X and you later pull version Y, behavior can shift underneath results you've already recorded. The question was whether you want to (a) **pin** — choose one version, write it down, never move during the project — or (b) **rebase** — periodically update and re-verify.

**Recommendation: pin.** Put `open_spiel==2.0.2` in a `requirements.txt`, note the version in your README, and don't touch it. You get nothing from upstream during this project and you'd be risking silently invalidated training runs. Revisit only if you hit a bug that upstream has fixed.

### §4.2 — Why start on a small board

Three reasons, in descending importance:

1. **Bug classes are board-size independent, but their cost isn't.** The bugs that kill AlphaZero projects are sign errors in the value target, an action-index mismatch between MCTS and the network, a replay buffer that never evicts, an evaluation harness that forgets to swap colors. Every one of them shows up identically at 5x5 — and at 5x5 you find out in twenty minutes instead of after two days of GPU time and an ambiguous result you can't interpret.
2. **You get a real "it learns" signal fast.** 5x5 games are short and the branching factor is small (16 wall placements versus 128). A run that produces a clearly-better-than-random agent in under an hour tells you the loop closes. Until you've seen that once, you don't know whether a flat 9x9 curve means "undertrained" or "broken" — and that ambiguity is where hobby RL projects die.
3. **It fits your schedule.** A 5x5 experiment fits inside a 2-hour session. A 9x9 experiment does not.

On the transfer question: **retrain from scratch at 9x9.** The 5x5 phase's deliverable is the *pipeline*, not the weights, and transfer would add a confound to your first real training curve. (If you build the network fully convolutionally with a global-pooling head, transfer becomes possible later as a nice experiment — worth designing for, not worth depending on.)

Caveat to expect: 5x5 with 3 walls each may turn out to be a near-trivial first-player win. That's fine — you're testing the machinery, not studying the game.

### §4.3 — Learned vs. engineered features: what you'd actually be sacrificing

**What you sacrifice by making the net learn it: mostly time, but possibly the ceiling.**

The specific problem is that Quoridor's value function is dominated by one quantity — the difference between the two players' shortest-path lengths to their goals — and shortest path is a *global, iterative* computation. A convolutional layer propagates information one cell per layer. To compute a path that snakes around walls across a 9x9 board, the network needs depth roughly proportional to the path length, and it has to learn to implement something like a BFS in its weights. That is a genuinely hard thing to learn, it eats most of your network capacity, and it's the single biggest reason a small net on a consumer GPU underperforms on this game.

Meanwhile OpenSpiel already computes shortest paths internally — it has to, to check wall legality — so handing it to the network costs you essentially nothing.

**Pros of engineered features** (a plane or scalar per player for shortest-path length, plus walls remaining): dramatically faster time-to-strength; a much smaller network suffices; your limited compute goes into learning *strategy* rather than re-deriving arithmetic.

**Cons**: it's a hand-crafted prior, so the "learned from scratch" story is weaker; if the feature has a bug, the network trusts it and you silently cap your ceiling; and there's a real possibility the network would have learned something *better* than shortest-path (e.g. path robustness under future walls) that the feature now discourages.

**Recommendation: build both, and make it your headline ablation.** Same network, same hyperparameters, one with the features and one without, trained on 5x5 where a full run is cheap. Then your README contains a plot answering "does hand-engineering the dominant heuristic still matter in the AlphaZero era?" — which is a far more interesting artifact than either agent alone, and it directly serves your §2.3 goal. For the 9x9 run that you actually want to be strong, use the features.

### §4.4 — Yes, the existing action space works

**I agree, and now with more confidence than when you wrote it.** OpenSpiel encodes every action as a point on the `2*board_size-1` = 17x17 "diameter" grid, where even/even coordinates are squares and the odd coordinates are wall slots — 289 action indices total, covering all 128 wall placements and every pawn destination. Since your jump rules are *exactly* the ones OpenSpiel already implements, every legal move you described already has an index. Nothing to change.

The one thing to keep straight is that the action space is *fixed and larger than the legal set* at any moment (correct AlphaZero practice): the policy head outputs 289 logits, and you mask to `legal_actions()` before softmax. Forgetting the mask is a classic silent bug — worth a test.

### §4.5 — How symmetry constrains the representation

The 4x figure is optimistic (see the caveat at the end), but the constraints are real and they're the reason to decide now:

**Mirror (left-right) symmetry** requires an action-index mirror map: for every action, the index of its reflection. Pawn moves are easy. Walls are where the bugs are — a horizontal wall spans *two* files, so mirroring column *f* lands on column `board_size - 2 - f`, not `board_size - 1 - f`. That off-by-one produces walls one square off from where they belong, training silently degrades, and it's nearly invisible unless you test for it. Write a test that mirrors a position twice and asserts you get the original back.

**Color-swap symmetry** is the one that genuinely constrains you: it only works if the state is encoded **from the perspective of the player to move** — rotate the board 180°, swap the pawn planes, swap the wall-count planes, so that "the player to move is always advancing upward." If instead you encode absolute planes ("player 0 here, player 1 there") then a color-swapped sample is a *different* input meaning a different thing, and the augmentation is simply invalid.

That's the constraint: **commit to a canonical, side-to-move-relative encoding on day one.** Retrofitting it later means rewriting the encoder, the MCTS value signs, the replay buffer, and every checkpoint you've trained. It's also good practice independent of augmentation, because it halves what the network must learn.

Two more details: augmentation must be applied to the **policy target** as well as the input (permute the target vector through the same index map), and the true speedup is well under 4x because mirrored samples are highly correlated — you're regularizing more than you're adding information. Still worth it; the implementation is about twenty lines once the index maps exist.

### §4.6 — The knobs, explained

| Knob | What it controls | Suggested |
|---|---|---|
| **MCTS simulations per move** | How many search iterations before choosing a move in self-play. More = better training targets, linearly slower self-play. The single biggest cost/quality lever. | 100 at 5x5, 200–400 at 9x9 |
| **c_puct** | Exploration constant in the search's selection formula: how much the search trusts the network's move priors versus the values it has measured. Too low, the search never questions the net; too high, it wanders. | fix at 1.5–2.0 |
| **Dirichlet noise (α, ε)** | Random noise mixed into the priors *at the root only* during self-play, so the agent tries moves its current policy dislikes. Without it, self-play collapses onto one line and never discovers anything. α scales inversely with branching factor (Go 0.03, chess 0.3). | α ≈ 0.15, ε = 0.25 |
| **Temperature schedule** | For the first N moves, pick moves in proportion to visit counts (diverse openings); afterwards pick the most-visited move (strong play). Without it, every self-play game is nearly identical. | temp 1.0 for 20 moves, then 0 |
| **Replay buffer size** | How many recent positions training samples from. Too small overfits to the current generation; too large trains on stale play the agent has outgrown. | ~20x the positions produced per iteration; 100k–500k at 9x9 |
| **Network width/depth** | Residual blocks x filters. You are compute-bound, not capacity-bound — a bigger net makes every self-play move slower, which is where your time actually goes. | start 4 blocks x 64 filters |
| **Train/self-play ratio** | Gradient steps per self-play game. The most common cause of instability: train too hard on too little fresh data and the net overfits, self-play degrades, and the whole loop spirals. | start conservative, ~1 epoch over fresh data per iteration |
| **Checkpoint cadence + gating** | How often you snapshot, and whether a new net must *beat* the current one (say 55% over 100 games) before it becomes the self-play generator. AlphaGo Zero gated; AlphaZero dropped it. | **gate** — it costs eval games but prevents silent collapse, which matters when you check in weekly rather than daily |
| **Learning rate / batch size / weight decay** | Standard supervised-training knobs. | Adam @ 1e-3, batch 256, wd 1e-4 |

**Fix everything in that table by fiat except three**: simulations per move, replay buffer size, and learning rate. Those are the ones worth an experiment. Tuning the rest is how a two-hours-a-week project turns into a two-year project.

### §5.2 — What a fixed benchmark position suite is

A curated file of positions, each with a known-correct answer, that you run the *network alone* against — no games played. You measure policy accuracy ("did it put mass on the right move?") and value accuracy ("did it predict the right winner?").

Why it's worth having alongside match results: match results are **noisy and slow** (you need hundreds of games to distinguish a 52% agent from a 48% one) and they tell you *that* the agent is worse, never *why*. A position suite is instant, deterministic, and diagnostic.

For Quoridor there's an unusually good source of free ground truth: **positions where both players have zero walls left are pure races, and you can compute the exact winner with two BFS calls.** So:
- **~200 auto-generated wall-less race positions** with exact ground-truth values. Your value head should approach 100% here. If it doesn't, it hasn't learned the most basic fact about the game, and no amount of further training will fix a representation problem.
- **~30 hand-picked tactical positions**: one move from reaching the goal; positions where exactly one wall placement wins; positions where a jump is forced.
- Optionally, **positions from your transcribed barricade.gg games** where you know what a strong human played.

Run it at every checkpoint, plot the accuracy alongside your loss curves. It's about two hours of work and it will save you far more than that.

### §5.4 — What an interpretability component would look like

Concrete options, cheapest and most interesting first:

1. **Value head vs. shortest-path heuristic — where do they disagree?** Scatter the network's predicted win probability against the shortest-path differential across thousands of positions. Most points will lie on a curve. The interesting ones are the outliers: positions where the net is confident *against* the path count. Pull a dozen of those up and look at them — that's where your agent has learned something the heuristic doesn't know, and it's the most publishable thing in this whole project. This is my recommendation for the centerpiece: cheap, visual, and it directly answers "did the network learn anything, or is it just doing arithmetic?"
2. **Opening book.** What does the policy head play from the start position, and how does that change across training generations? Quoridor has documented human opening theory — showing the agent rediscovering (or rejecting) it is an immediately legible result.
3. **Wall-economy curve.** Plot walls remaining against move number, by generation. Human wisdom says walls are more valuable late; watching an agent learn patience is a nice narrative.
4. **Occlusion sensitivity.** Delete one wall from an input position and re-evaluate. Rank walls by how much they move the value. It produces a heat-map of "which walls matter here," which is a good figure.

All four are analysis scripts over checkpoints you'll already have. None requires extra training. Pick one or two — I'd take #1 and #2.

### §7.1 / §7.2 — Consistency check

The vault fixes landed. Three small things remain:

1. **`Last Updated:` is empty** in this note's frontmatter (and in `Templates/Claude.md`). `Vault Setup.md` specifies `{{date:M/D/YYYY}}` — QuickAdd fills that at creation time, but this note predates the fix. I've set it to 8/30/2026 and bumped `Version` to 0.2; revert if you'd rather track versions differently.
2. **`Authors:` vs `Author:`** — `Vault Setup.md` says to add `Authors: Levi Kuhn` (plural), while `Templates/Claude.md` and this note use `Author:` (singular). Dataview treats those as different fields, so pick one. Singular is what's actually in use.
3. **`tags:` is empty** everywhere. Fine if you're not using them, but the Dashboard would support tag-based views if you ever want them. Suggested for this note: `rl`, `alphazero`, `quoridor`.

Also, a typo carried over into `Templates/Claude.md`: *"add tests for anything you chance"* should be *"change."*

### §7.3 — Where the notes should live (you left this blank; here's my proposal)

**Keep the code out of this vault, in its own public repo.** The vault is your thinking space — messy, personal, full of half-formed questions like this document. A public portfolio repo needs a curated README and a clean history. Trying to be both makes both worse. So:

- **`BarricadeAI` vault** (private or public, your call) — planning, research, decisions, this file.
- **`barricade-zero` repo** (public) — code, tests, the curated README, training curves, the play script.

Inside the vault, using your existing categories:

| Note | Category | Purpose |
|---|---|---|
| `Claude/Brainstorming For The Project.md` | Claude | this file — the running conversation and plan |
| `Projects/Rules Spec.md` | Documentation | the rules table above plus the notation grammar. **The rule-delta spec belongs here** — it's the reference every test is written against |
| `Projects/OpenSpiel Research.md` | Research | findings about OpenSpiel's API, the AZ implementations, version pinning |
| `Projects/AlphaZero Design.md` | Planning | state encoding, action space, network architecture, symmetry maps |
| `Projects/Training Log.md` | Implementation | dated entries per run: config, curve, what you changed, what happened |
| `Projects/Evaluation.md` | Documentation | the baseline ladder, the benchmark suite, results tables |

The one I'd argue hardest for is **`Training Log.md`**. At two hours a week you will not remember what you changed three weeks ago, and "why did generation 12 get worse" is unanswerable without it. Dated append-only entries, five minutes at the end of each session.

## Revised project shape

Given that the rule delta collapsed to one rule and the build burden collapsed to a pip install:

- **Phase 0 — Foundations.** `pip install open_spiel` on Windows; play a random game; write the coordinate mapping and its round-trip test.
- **Phase 1 — Baselines and evaluation.** Random bot, shortest-path heuristic bot, OpenSpiel MCTS at 100/1000 sims. Paired-color match harness, Elo, results table. *Build this before any training.* Everything afterward is measured with it.
- **Phase 2 — Rules fidelity.** Notation parser, 20–30 transcribed games, conformance test, threefold-repetition wrapper.
- **Phase 3 — AlphaZero on 5x5.** Encoder, network, batched MCTS, self-play loop, replay buffer, training, gating. Ship when it beats the heuristic bot on 5x5.
- **Phase 4 — 9x9.** Scale up, add shortest-path features, run the ablation, tune the three knobs that matter, iterate on GPU time while you sleep.
- **Phase 5 — Publish.** README, curves, ablation table, the value-vs-heuristic analysis, and a script that lets a stranger play your bot.

Each phase ends with something demonstrable, which matters when the calendar is long.

## New open questions (Round 2)

R2.1. **Do you want to write the AlphaZero loop yourself** (my recommendation, §3.1) or use an existing implementation? This is the biggest remaining fork in the road and everything downstream depends on it.

R2.2. **Can you run the pip install check?** `pip install open_spiel` then `python -c "import pyspiel; print(pyspiel.load_game('quoridor').new_initial_state())"`. If that prints a board, Phase 0 is nearly done and we skip WSL2 entirely. If it errors, paste the error.

R2.3. **The four-wall notation confirmation** described in §1.2 — five minutes on the site, and it locks the parser spec.

R2.4. **What's your own playing strength?** "Beats me 9/10" is your stopping rule, so it's a load-bearing number. Are you a casual player or have you studied openings? It changes the target by a lot.

R2.5. **Separate public code repo, or code inside this vault's repo?** I've argued for separate in §7.3 — confirm before Phase 0 so the git history starts in the right place.

R2.6. **GPU specifics** — which card, and how much VRAM? It sets the network size and batch size, and whether 9x9 self-play is an overnight job or a week-long one.
