---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
title: "Logically Constrained Reinforcement Learning Tutorial"
---

- [Overview](#overview)
- [Reinforcement Learning](#reinforcement-learning)
- [Temporal Logics and Automata](#temporal-logics-and-automata)
- [Logically Constrained Reinforcement Learning](#logically-constrained-reinforcemen-learning)
- [Advanced exercises](#advanced-exercises)

## Overview

This tutorial walks through the pieces needed to understand logically constrained reinforcement learning, connecting standard RL intuition with temporal logic specifications and the automata constructions that make those specifications actionable.

## Reinforcement Learning 
 We begin with a refresher on the reinforcement-learning loop: agents observe a state, choose an action, receive a reward, and transition to a new state.

1. Open Andrej Karpathy’s Gridworld demo ([reinforcejs/gridworld_td](https://cs.stanford.edu/people/karpathy/reinforcejs/gridworld_td.html)).
   > This is a toy environment called Gridworld that is often used as a toy model in the Reinforcement Learning literature. In this particular case:
   >
   > **State space:** GridWorld has 10x10 = 100 distinct states. The start state is the top left cell. The gray cells are walls and cannot be moved to.
   >
   > **Actions:** The agent can choose from up to 4 actions to move around.
   >
   > **Environment Dynamics:** GridWorld is deterministic, leading to the same new state given each state and action.
   >
   > **Rewards:** The agent receives +1 reward when it is in the center square (the one that shows `R 1.0`), and -1 reward in a few states (`R -1.0` is shown for these). The state with +1.0 reward is the goal state and resets the agent back to start.
   >
   > In other words, this is a deterministic, finite Markov Decision Process (MDP) and as always the goal is to find an agent policy (shown here by arrows) that maximizes the future discounted reward. My favorite part is letting Value iteration converge, then change the cell rewards and watch the policy adjust.
   >
   > **Interface:** The color of the cells (initially all white) shows the current estimate of the value (discounted reward) of that state, with the current policy. Note that you can select any cell and change its reward with the Cell reward slider.
2. Before running anything, inspect the grid and reward layout, sketch what you think the optimal policy is, and note which states look risky versus attractive.
3. Reset the environment, click “go slow,” then click “toggle td learning” to start training; watch which zones the agent explores and how each state’s estimated value updates over time until the policy stabilizes—does it match your initial hypothesis?
4. Reset the agent and repeat the experiment with a very small exploration rate and then a large one; compare how quickly the policy improves and whether it gets stuck in suboptimal loops.
5. Feel free to play around modifying the reward of any cells and rerunning training to observe how the optimal policy adapts.

## Temporal Logics and Automata
We motivate temporal logics as a language for expressing behavioral constraints over entire trajectories—goals that involve ordering, persistence, or repetition rather than single-step rewards. In this setting, every Linear Temporal Logic (LTL) specification can be translated into a finite automaton whose accepted words are exactly the trajectories that satisfy the specification.

> Recall the key temporal operators: `F p` (eventually p becomes true), `G p` (globally, p holds at every step), and `X p` (in the next step, p holds).


Using the Spot LTL visualizer—an interactive tool maintained by the Spot research team—we experiment with these translations to build intuition for the acceptance conditions and the structure of the resulting automata.

1. Open the Spot web interface ([spot.lre.epita.fr/app/](https://spot.lre.epita.fr/app/)), set “Acceptance” to “Generalized Büchi,” and enable the “small” and “complete” options.
2. Enter the formula `F goal` to generate an automaton for “eventually reach the goal”; note which states are accepting.
3. Replace it with the safety formula `G !unsafe`; observe how the automaton enforces avoidance by making every unsafe transition go to a rejecting sink.
4. Compose both requirements as `F goal & G !unsafe`; compare it to the previous automata—why does this construction enforce both constraints?
5. Consider the formula `F (goal1 & X (F goal2)) & G !unsafe`; provide one sequence of labels that satisfies it and one that violates it.
6. Generate the automaton for that formula in Spot—how does the construction enforce visiting `goal1` before `goal2` while still avoiding `unsafe`?

## Logically Constrained Reinforcement Learning

Logically-Constrained Reinforcement Learning (LCRL) is a model-free framework that couples an agent with an automaton for a given Linear Temporal Logic (LTL) property, shapes rewards on the fly, and synthesizes policies that maximize the probability of satisfying the specification in discrete and continuous-state-action MDPs.
In this context, the only addition to a standard RL environment is a labeling function that maps each state to the atomic propositions referenced by the LTL property. As in standard RL, the underlying MDP can stay unknown; the agent only needs each observation to include its label so it can synchronize with the automaton on the fly.

![LCRL Architecture](https://raw.githubusercontent.com/grockious/lcrl/master/assets/lcrl_overview.png)

1. Clone the repository and install it in editable mode so you can tweak the examples locally:
   ```bash
   git clone https://github.com/grockious/lcrl.git
   cd lcrl
   pip3 install -e .
   ```
2. Inspect `src/lcrl/automata/goal1_then_goal2.py`; the `step` function there mirrors the Spot automaton you generated earlier.
   <div align="center">
   <img src="https://raw.githubusercontent.com/grockious/lcrl/master/assets/F(goal1%20%26%20XF(goal2))%20%26%20G!unsafe%20-%20by%20SPOT.png" alt="Spot automaton for F(goal1 & X(F goal2)) & G !unsafe" />
   </div>
3. Inspect `src/lcrl/environments/SlipperyGrid.py` and `src/lcrl/environments/gridworld_1.py`; the first implements the slippery grid dynamics, while the second layers the labeling function on top.
   <div align="center">
   <img src="https://raw.githubusercontent.com/grockious/lcrl/master/assets/layout_1.png" alt="Slippery grid layout annotated with propositions" />
   </div>
4. Create a directory `aims/` for your experiments and add `aims/train.py` with the starter script below. It trains an agent for the `goal1_then_goal2` specification in `gridworld_1` using Q-learning:
   ```python
   from lcrl.train import train
   from lcrl.environments.gridworld_1 import gridworld_1
   from lcrl.automata.goal1_then_goal2 import goal1_then_goal2

   if __name__ == "__main__":
       MDP = gridworld_1
       LDBA = goal1_then_goal2
       task = train(
           MDP,
           LDBA,
           algorithm="ql",
           episode_num=3_000,
           iteration_num_max=4_000,
       )
   ```
5. Run `python aims/train.py` and discuss the output.

## Advanced Exercises

1. Create and run a new training script that mirrors the structure below:

   ```python
   from lcrl.train import train
   from lcrl.environments.frozen_lake_4 import FrozenLake
   from lcrl.automata.frozen_lake_4_5_6 import frozen_lake_4_5_6

   if __name__ == "__main__":
       MDP = FrozenLake
       LDBA = frozen_lake_4_5_6
       task = train(
           MDP,
           LDBA,
           algorithm="ql",
           episode_num=3_000,
           iteration_num_max=4_000,
       )
   ```
2. Inspect `src/lcrl/automata/frozen_lake_4_5_6.py` to understand the reference automaton.
3. Implement your own automaton class for the specification “goal1 then goal2 then goal4 then goal3 while avoiding unsafe” (note the reordered goals).
4. Train an agent in `frozen_lake_4` using the automaton you just created and analyze the learning outcome.