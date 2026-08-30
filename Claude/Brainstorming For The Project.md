---
fileClass: Project
Category: Claude
Status: Active
Author: Levi Kuhn
Last Updated:
Version: 0.1
tags:
---
# Overview
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

**Get the pipeline green on a 5x5 board first.** A 5x5 Quoridor with 3 walls each trains to something clearly non-random very fast. The point isn't the agent, it's proving that self-play → replay buffer → training → checkpoint → evaluation loop actually closes. Every bug you'd hit at 9x9 you'll hit at 5x5 for a fraction of the compute.

**Build a heuristic baseline early and keep it forever.** A bot that always walks its own BFS shortest path, and places a wall only when the opponent's path is shorter than its own, is ~50 lines and is a surprisingly hard opponent for an undertrained net. It's your smoke test for "is training doing anything at all."

**Consider whether you need AlphaZero at all for the first milestone.** OpenSpiel's MCTS with a *perfect-information rollout* evaluator plays decent Quoridor with zero training. If your goal is "strong bot," a tuned MCTS is a legitimate strong baseline and a fast path to a working artifact; the neural net is what beats it, and framing the project that way gives you a deliverable at every stage instead of only at the end.

**Feature the shortest path.** Quoridor's value function is dominated by "difference in shortest-path lengths." Giving the network that as an input plane (or two scalars) is likely worth more than a deeper net. If you want the purist version too, that's a clean ablation for the write-up.

**Exploit the mirror symmetry.** Free 2x on data with a few lines in the training loop. The color-swap symmetry is a further 2x, but only if the state encoding is canonicalized to "current player is always moving up."

**Plan for checkpoints and resumption from day one.** The C++ AlphaZero writes a `config.json` and resumes from the latest checkpoint; whatever path you choose, assume training will be interrupted, because it will be. Related: decide now where checkpoints live and whether they go in git (they should not — the vault is already a git repo, and a `.gitignore` that keeps model weights out is worth writing before the first run, not after).

**Log obsessively, plot weekly.** Value loss, policy loss, game length, wall-usage rate, and win rate versus each baseline rung. Game length and wall-usage are the two that tell you the agent is learning *Quoridor* rather than learning to shuffle.

**Beware the no-repetition gap.** OpenSpiel's Quoridor has no repetition rule, so a self-play agent can learn to shuffle back and forth forever in drawn-ish positions and burn your compute on 300-move games. If barricade.gg has a repetition or move-limit rule, implement it in the fork early; if it doesn't, consider a training-only move cap anyway.

**Time control is a rule too.** If barricade.gg games are clocked and you ever intend to play there, the agent's simulation budget per move is a hard constraint, and an agent tuned at 1600 sims/move that gets 200 in practice is a different, weaker agent. Decide the deployment budget before tuning.

# Suggested Order to Answer These

1. §1.1 rule delta and §6.1 whether barricade.gg play is in scope — these two gate everything.
2. §2 scope, deadline, and success criterion.
3. §4.1 hardware, then §3.1 which AlphaZero implementation (hardware largely picks it for you).
4. §5 evaluation ladder — define it *before* training, so the numbers mean something.
5. Everything else is a design-phase decision, not a brainstorming one.
