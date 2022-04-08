# ECE499 Report : Reinforcement Learning

### Supervised by Professor. Mark Crowley

##### By Vincent Lin

##### Computer Engineering

##### 20674085



[toc]

## Introduction

### Background

The purpose of this report is to examine various Reinforcement Learning (RL) algorithms on different simulated environments provided by OpenAI gym (OpenAI). The goal is to study the performance of the algorithms on various domains.

### Reinforcement Learning (RL)

RL is a field from Machine Learning (ML) where it emphasizes on developing its own agents to make a serious of decisions within an environment. RL aims at building the experiences of an agent by trial and error, alongside with a reward and penalty system to determine whether an experience is good or bad. The goal is to transform an agent which takes random actions at first, to making sophisticated decisions after many (typically) trials.

### OpenAI gym

The RL algorithms will be trained on OpenAI. This technology is “a toolkit for developing and comparing reinforcement learning algorithms” (OpenAI, 2022). OpenAI provides a wide range of environments, from the entry level of classic control, toy text to more sophisticated 2D and 3D robots. OpenAI provides a well-documented wiki page from their Github with each environment’s description, observation, action space, episode termination and rewards. The details to these terminologies are:

- Observation: an environment-specific object illustrating the observation of the current environment state, these could include the current coordinate of the car from environment MountainCar-v0, and the velocity of the car. 
- Action space: the possible actions that the agent could take, it could be a discrete action space where agent could only pick a fixed range of non-negative numbers as their actions, or a continuous action space where agent could pick any numbers within a range provided. 
- Episode termination: specifies parameters on what terminates the environment.
- Rewards: the current reward policy built-in by OpenAI, however, users could develop their own rewards policy.

To sum up, OpenAI is a sophisticated toolkit for RL and environment-specific parameters will be discussed in later sections. This report will test various algorithms on the environments FrozenLake-v0 (non-slippery version), and CartPole-v0.

### RL Algorithms

As RL was first introduced in 1965 in an engineering literature, it has been studied and refined due to its importance and usefulness in the past few decades. There are many great algorithms which performs differently based on the characteristics of the environment. This report will focus on the following algorithms: Q-learning (QL), State-action-reward-state-action (SARSA) and double Q . 

There are a few concepts which apply to general RL algorithms, they are:

- Temporal Difference (TD): "TD is an approach to learning how to predict a quantity that depends on future values of a given signal" (Barto, 2007). 
- Q table: generally is a table where row and column represents the state and action respectively, each Q value represents the expected long term reward by taking that action from that state (Generally updated via TD).
- On-policy: meaning the algorithm uses the same policy for both updating the Q values and choosing the next action.
- Off-policy: meaning the algorithm uses different policies for updating the Q values and choosing the next action.
- π-greedy policy: a policy which features both exploitations and explorations, exploitation uses prior knowledge of the agent (Q table) and exploration uses π to determine whether a random action should be taken instead of exploiting.
- μ greedy policy: a policy which always chooses the best Q value.
- γ: discount rate of the future rewards, in other words, how important does the algorithm values the future rewards.
- α: learning rate when updating Q values, the larger it is, the more the sum of immediate reward and discounted estimate of optimal future value is roportionate d in the overall Q value (Venkatachalam, 2021). 

**actually leave it for the algorithms big section**

### Setup

The results of this report are produced on a MacBook Pro (2016), with a processor of 2.9GHz Dual-Core Intel Core i5, 8GB memory, written in python, and uses various the following libraries:

- gym
- random
- numpy
- time
- os

## Algorithms

### State-action-reward-state-action (SARSA)

SARSA is an on-policy TD control method. The algorithm uses π-greedy policy for both choosing an action and updating the Q value, and therefore it is an on-policy algorithm. The pseudo code is:

 ```pseudocode
 Initialize Q(s, a), for all s ∈ S, a ∈ A(s), arbitrarily, and Q(terminal-state, ·) = 0 Repeat (for each episode):
 	Initialize S
 	Choose A from S using policy derived from Q (e.g., ε-greedy) 
 	Repeat (for each step of episode):
 		Take action A, observe R, S′
 		Choose A′ from S′ using policy derived from Q (e.g., ε-greedy) 
 		Q(S, A) ← Q(S, A) + α[R + γQ(S′, A′) − Q(S, A)]
 		S ← S′; A ← A′;
 	until S is terminal
 ```

(Sutton & Barto, 2020, p. 106)

The code initializes Q tables based on the environment-specific spaces (states and actions). Then for each episodes, SARSA resets the game by resetting the states, then immediately choose an action using the π-greedy policy, it could be a random action or the best action derived from the Q table depending on the π value.

After taking an action, the code updates the Q value of the previous state and action by this equation

```pseudocode
Q(S, A) ← Q(S, A) + α[R + γQ(S′, A′) − Q(S, A)]
```

The equation is essentially updating the old Q value with a learning rate and discount rate applied immediate rewards and future rewards. Notice that the future Q value used here is the action taken and the state it arrived, which is the same result that the π-greedy policy produced. 

### Q-learning (QL)

QL is an off-policy TD control method. This algorithm is similar to SARSA, they both choose the next action using the π-greedy policy, however, it updates the Q value differently. Here is the pseudo code: 

```pseudocode
Initialize Q(s, a), for all s ∈ S, a ∈ A(s), arbitrarily, and Q(terminal-state, ·) = 0 Repeat (for each episode):
	Initialize S
	Repeat (for each step of episode):
		Choose A from S using policy derived from Q (e.g., ε-greedy) 
		Take action A, observe R, S′
		Q(S, A) ← Q(S, A) + α[R + γ*(maximum value of)Q(S′, a) − Q(S, A)]
		S ← S′
until S is terminal
 
```

(Sutton & Barto, 2020, p. 106)

The code's initialization is identical to SARSA and therefore will not iterate. One difference which differentiates QL from SARSA is the update function: 

``````pseudocode
Q(S, A) ← Q(S, A) + α[R + γ*(maximum value of)Q(S′, a) − Q(S, A)]
``````

The difference is the future reward used in the equation. Personally I believe the notation from Sutton & Barto is not clear enough, I would like to treat it as:

```pseudocode
Q(S, A) ← Q(S, A) + α[R + γ*(maximum value of)Q(S′) − Q(S, A)]
```

This is because the notation `Q(S', a)` represents the Q value of taking action `a` at state `S'`, however, the algorithm is actually trying to find the action `a` at state `S'` which gives the maximum value of Q. Therefore, I believe the second equation is better to understand. Notice that this is not the same policy as the π-greedy policy, which was used to select the next action. The Q value is updated using the action from the state  `S'` which produces the highest Q value, it may not be the action we actually choose in the state `S'` since there is a possibility that the algorithm chooses to explore other actions rather than expolit on the best action.

**sneak double QL in here as well just briefly explain difference**







