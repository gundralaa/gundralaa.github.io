---
layout: post
title: "Markov Beats"
---
An intresting idea I've recently been exposed to is the Markov Chain. The model is simple yet powerful. By linking together a set of states with a set of transition probabilities, you can create intresting state models that express probablistic natures.

With this idea, I was inspired to apply the model to a create a drum loop that introduces elements of suprise based on the modification of thee transition proabilities within the chain.

### Conceptual Overview
A simple markov chain is defined by a set of *S states* and *n timesteps* such that the state at time n is some *X_n in S*. With this setup a large number of configurations can be built such that the probability can be define as being in some state *S_1* at *time n* given the base states starting at 0.

```
P[X_n = S_1 | X_n-1 = S_2, X_n-2 = S_2, ....] = p
```
Like the statement above, many other combinations could be defined on the finite or countable set S at some step n. However, the way that Markov Chains are defined allows the exploitation of the memoryless property. The chain is define as a transition from one state to the next **regardless** of the states before. Therefore, we can define the probability of *X_n = S_1* where *X_n-1 = S_2* as some p such that:
```
P[X_n = S_1 | X_n-1 = S_2 ] = p
```
From this follows the idea that the probability transition out of any state must sum to 1.

![Graph Structure](https://ds055uzetaobb.cloudfront.net/brioche/uploads/uRaFsaTqz3-group-4.png?width=400)

The graph above represents a two state markov chain with transition probabilities affixed to directed edges between the states notifying the direction of transition.

### Periodicity
A markov chain with certain properties can have an invariant distribution that defines a vector *v* such that the transition matrix *A*
```
v = vP
```
It can be proven for aperiodic chains this invariant distribution defines the fraction of time in certain states as the number of timesteps *n* approaches *infinity*. The aperiodicity of a chain allows for the system to theoretically be in any state *S* at any time *n*. For the drum loop this idea can be both powerful and hindering as a consistent rythm might not be achieved without enforcing a period upon the chain. Due to intrest in manipulating the invariant distribution of the system, I will be implementing an aperiodic structure to the markov chain.

### System
Our simple drum loop will consist of a sequence of 16 beats that are filled using the output of a markov chain simulation. The starting state will be chosen at random. The beat has a snare kick and an empty state (no sound). Heres the semi psuedocode for a proposed implementation.

```javascript
var beats = 16;
var song = [];
var states = ["kick", "snare", "blank"];
var transitions = {
    "kick" : [0.4, 0.2, 0.4],
    "snare" : [0.4, 0.2, 0.4],
    "blank" : [0.3, 0.3, 0.4],
};
var cur = states[getRandIntUniform(0, 2)];

for (int i=0; i < beats; i++) {
    song[i] = cur;
    cur = states[
        getRandIntDist(transitions[cur], 0, 2)
    ]
}
```
The code assumes two helper functions needed to sample from a uniform distribution and a defined distribution. Overall, this javascript module could be then ported to a more robust feature that will be more fully developed.


