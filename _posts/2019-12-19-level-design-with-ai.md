---
layout: post
title: "Level Design with AI + Human Preference!"
author: "Mohammad Reza Taesiri"
categories: games
tags: [Game, AI, RL, NEAT, CPPN]
image: ai-level-design/heading.jpg
video: true
---

One of the most exciting challenges in the field of Video Game and Game AI is generating (good) games and levels with no to minimal human intervention. There are few attempts, like  Procedural Content Generation, down this line but I believe we still have a long way to go. One of the interesting papers that I have come across is "**Co-Creative Level Design via Machine Learning**". This paper introduces the concept of a collaboration of humans and AI to create a level, that's it, merging humans creativity and machine learning algorithms to assist designers to have a better level designing experience. This paper is exploring existing methods and show their shortcomings for level generation and opens a new door to game AI researchers.  The second paper of the same group, "**Integrating Automated Play in Level Co-Creation**" builds on top of the previous idea and adds a technique to further help and improve the quality of level design experience by adding some feedback from Automated Plays.

## High-level overview

"Co-Creative Level Design via Machine Learning" or CCLDML for short, introduces a new turn-based level editor tool to help a level designer to make "new levels". A level designer can interact with Level Editor and whenever he/she wants, by pressing a button, an AI agent will "Collaborate" and modifies what the designer had created. You can see a demo of this tool below:

<p style="text-align: center;" id="demo1">
<iframe width="560" height="315" src="https://www.youtube.com/embed/UkMeM5Ty1lA" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p>

This paper introduces a Computational Creativity tool, It does not invent a new algorithm for doing a particular task. It just put what is already out there to create something new, more powerful and useful. At its core, It has 3 methods of creating levels; Markov Chains, Bayes Nets, and LSTMs. Each one of these techniques previously used to generate or create new content, based on the distribution of what it saw before. For example, when someone uses LSTMs to predict the next character in a text sequence, after some training, LSTM will learn some underlying probabilistic distribution of characters. It can understand and memorize that, in the last N steps, the character ``x`` ninety percent of the time followed by a character ``y`` and the other ten percent, followed by either ``z``, ``w``. After the training phase, you can sample from LSTM to generate new text.

For video game generation using LSTMs, one can convert a level into a sequence and feed it to a LSTM. In the Super Mario Maker example, each individual pixel can have a certain type, (i.e Block, Pipe, Enemy,  ...). You can flatten a 2D level and treat level data as a sequence, just like any other sequence data and use LSTM over that.

{% include images.html url="../assets/img/ai-level-design/LSTM-Level-Generation.png" description="One way of feeding a Level data into LSTM" %}

The idea behind Markov Chains is also similar. This time we create a weighted graph from what we saw so far and after that, we do sampling by doing a *random walk* on the graph we constructed. Here is a very simple example of this:

{% include images.html url="../assets/img/ai-level-design/MC-Level-Generation.png" description="Generating Tiles with Markov Chain" %}

### Automatic Play as Feedback

In the second paper "Integrating Automated Play in Level Co-Creation", or IAPLCC for short, the authors suggested using two techniques to further assist the level designer. These two techniques are "Reachability Check" and "Survival Analysis".  "Reachability Check" just uses the A* algorithm to determine if there is a path from a starting point to the endpoint at the generated level. In "Survival Analysis", multiple agents will be thrown into the level and we keep statistics of success/fail and report it as a metric for survival rate. This metric can give the designer a clue about how hard the level will be. (This metric is not equivalent to human performance, but It can be used to compare two levels).

----

## My Thoughts

Here are some of my thoughts related to these papers.

## **As a Creativity tools**

The concept of **Collaboration** very powerful in the context of creativity and has explored before. An example of that is the "**Drawing Apprentice: Co-Creative Drawing Partner**" and other works related to it. In either of these papers, the Authors created an AI-based tool that collaborates with the player/user. The problem that I have with these tools is the way they are operating at the moment.

### Blind Sharing

As a game designer, I think I am the God of my world; I don't share my canvas with anybody else, Yet alone let him/her/it to draw on my canvas. In the [demo of **CCLDML**](#demo1) you can see that most of the time, the demonstrator is trying to delete what the AI is created. This is a source of frustration and even in the User study inside the paper, users report their level of frustration. I strongly believe there must be a way to control and preview what the AI is gonna do and just allow the desirable changes.

### Artistic Control

Another problem is that the designer has no artistic control over the AI. Of course, the AI uses what designer made in the past as an input and will act on it, but I think it is still important to have control over what AI will do. For example, the number of tiles its gonna modify, or a way for specifying the total number of tiles with type ``X`` that the AI will generate.

### Loss of Generality

I think in the second paper, IAPLCC, the authors are making a strong assumption about the type of game the designer would like to make. One can make a platformer game inside Super Mario Maker that doesn't require going form a location ``A`` to ``B``. For example, if I want to make a Survival/Wave-based game that the player must stay alive as long as he/she can; knowing a certain path is reachable inside the game is not  very helpful!

## **As an Algorithm**

As an algorithm, I think the used techniques are fairly limited. In Machine Learning literature, we have far more sophisticated models and tools that can be utilized here. In the next section, I will suggest methods, that I think will improve on the currently existing solutions.

----

## Possible Alternatives

Here I will suggest things that I think are helpful in the context of game level generation and also can be used to bridge ideas between humans and machines.

### Compositional Pattern Producing Networks

Compositional Pattern Producing Networks or CPPNs was invented by Kenneth O. Stanley back in 2007. It has been used before in many different contexts, from generating 2D patterns,3D shapes to Robotics. At its core, CPPNs are just neural networks, but unlike traditional neural networks, they pretty much can have any types of functions as nodes. (The choice of the name CPPN rather than regular Artificial Neural Network is described in Ken's paper).

{% include images.html url="../assets/img/ai-level-design/picbreeder.png" description="Generating 2D patterns with CPPNs - Picbreeder" %}

{% include images.html url="../assets/img/ai-level-design/endlessforms.png" description="Generating 3D Shapes with CPPNs - EndlessForms" %}

I don’t think generating level with vanilla CPPN be a good idea, (I am not sure, maybe it is or it is not!!), but we certainly can use them somehow in the process. [For example, someone tried to generate trains using CPPNs, but to me, the net results are not very interesting at this stage. [See here for the results](https://silky.github.io/posts/2018-04-15-cppns-for-procedural-landscape-generation.html#bonus-content-17-apr-2018)]

### Deep Reinforcement Learning from Human Preferences

In traditional (Deep) Reinforcement Learning, you need a very important piece called “Reward Function”. This function is used to assess the performance of your RL agents and gives it feedback and will be used as a training signal to our RL algorithm. Coming up with a good reward function is a really hard task and requires lots of effort. Designing a Naïve reward function can cause our agent to exploit a given environment in ways we don't imagine. You can read more about this [here](https://openai.com/blog/faulty-reward-functions/) and [here](https://openai.com/blog/emergent-tool-use/#surprisingbehaviors). You can see the result of choosing a bad reward function here:

<p style="text-align: center;" id="demo3">
<iframe width="560" height="315" src="https://www.youtube.com/embed/tlOIHko8ySg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</p>

Of course, we can, in theory, replace a Reward function with a human input, but in that case, our RL algorithm requires thousands and thousands of human evaluation. "Deep reinforcement learning from human preferences" is a collaboration between DeepMind and OpenAI to tackle this problem. Instead of using a "Reward Function", We use a "Reward Model". This model will be trained to match human preference. In this case, we can train our RL model faster (we don't need thousands and thousands of human inputs anymore) and we also can have human feedback in the loop, Yay! best of both worlds!

<p style="text-align: center;" id="demo2">
<iframe width="560" height="315" src="https://www.youtube.com/embed/oC7Cw3fu3gU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> </p>

One might think using Reinforcement Learning requires to design a reward function for a good level, and designing this reward function is equally challenging as generating a good level. I believe with the help of this method we can get away from explicitly designing a reward function. (There are also other techniques like Inverse-RL which I don't want to talk about them here.

----

## References

1. [Integrating Automated Play in Level Co-Creation](https://arxiv.org/pdf/1911.09219)
1. [Co-Creative Level Design via Machine Learning](https://arxiv.org/pdf/1809.09420)
1. [Learning from Human Preferences](https://openai.com/blog/deep-reinforcement-learning-from-human-preferences/)
1. [Drawing Apprentice: Co-Creative Drawing Partner](https://gvu.gatech.edu/research/projects/drawing-apprentice-co-creative-drawing-partner)
1. [Compositional Pattern Producing Networks: A Novel Abstraction of Development](http://eplex.cs.ucf.edu/papers/stanley_gpem07.pdf)
1. [Evolving Neural Networks through Augmenting Topologies](http://nn.cs.utexas.edu/downloads/papers/stanley.ec02.pdf)
1. [Picbreeder](http://picbreeder.org/)
1. [EndlessForms](http://endlessforms.com/)
1. [Faulty Reward Functions in the Wild](https://openai.com/blog/faulty-reward-functions/)
1. [Emergent Tool Use from Multi-Agent Interaction](https://openai.com/blog/emergent-tool-use/)
1. [Learning from Human Preferences](https://openai.com/blog/deep-reinforcement-learning-from-human-preferences/)

## Image Credits

1. Diragrams made with [Draw.io](https://www.draw.io/)
1. Mario's Sprites took from [here](https://github.com/mguzdial3/Morai-Maker-Engine)