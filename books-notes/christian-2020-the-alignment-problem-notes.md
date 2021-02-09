## The Alignment Problem -- notes

I like this book a lot.
It covers a lot of ground from the early days of AI to today.
It is a book that can motivate someone working in AI.
What other AI books are there that are good motivational books, and talk about
the big picture?

The alignment problem: how do we design AI systems that have some independence
(and may even be smarter than us) while ensuring that their actions align with
our intentions and our values?

Pitts was a self-taught logician.
He was homeless in Chicago, and McCulloch, who was a professor, invited him to
stay at his house.
Together they developed the artificial neuron.

Later, Rosenblatt invented the perceptron, which is a 1-layer neural net, and
predicted that if the weights of a perceptron can be set to a value that does
well on a task, then there should be an automated training procedure that
discovers that configuration.

Minsky and Papert published a book and proved that there exist very simple
functions such as xor that cannot be learned by perceptrons.
Nobody knows how to train multi-layer perceptrons.
The field stagnates.

Later, Hinton (re)discovers back-propagation, and later his postdoc LeCun trains
convnets to recognize digits, the first successful commercial application of
neural nets.
But progress stagnates again because the training sets are too small and the
computers too slow by orders of magnitude.

In the 2000s, Amazon creates Mechanical Turk, and Fei Fei Li uses it to create a
giant set of labeled image data called ImageNet.
A competition is organized to train models to do well on ImageNet.
Alex Krizhevsky figures out how to use GPUs for neural net training and achieves
state of the art performance on cifar-10.
Sutskever encourages him to try ImageNet.
The two of them with Hinton win the 2012 competition by a very large margin.
AlexNet was the event that started the current large wave of neural net work.
Techniques used in AlexNet that are commonplace today: relus as activation
functions, data augmentation, dropout.

Compass is some software that, based on a person's history, predicts how likely
this person is to reoffend.
Propublica published a famous article showing that, under certain metrics,
Compass discriminates against people of color.
Later, Kleinberg and others showed that it is mathematically impossible to
satisfy both Compass's measure of fairness (called calibration) and Propublica's
measure; there is no version of Compass about which the article wouldn't have
been written.
So how do we decide if Compass is fair or not?
There is no solution to the metrics issue; it is an open problem to decide what
metrics tools should use.

The problem with using a model to predict crime is that the model trains on past
data, and as a result it says that what happens in the past will keep happening.
But our motivation is reform, to improve policing, not leave it as is.

Paul Meehl and Robyn Dawes:  
The idea that simple linear models outperform human judgment in many problems,
e.g., sex minus fights predicts marriage happiness.
However, humans are great at finding *which* variables to consider in the simple
model.
That's extremely difficult for a machine to figure out.

Cynthia Rudin: computer scientist who tries to discover the simplest possible
linear models (features that matter) from data.
Rudin managed to find simple models that are highly accurate, and can be run on
pen and paper by doctors.

Saliency methods: methods that create a heat map on an image showing which
pixels were most important when the network made a decision.
Uncovers surprising things, e.g., the net may have ignored the parts you thought
were important.

Matt Zeiler invented deconvolution in order to understand what each neuron of a
net learns.
This allowed him to find inefficiencies in AlexNet and win the ImageNet
competition.

Chris Olah and others found which features matter most to a net when it
recognizes, say, a dog.
Then, they start with random noise to create an image recognized as a dog, and
the resulting image may have lots of eyes, and other crap like that.
The takeaway is that the net doesn't understand that a dog must have two eyes.
It would be good to be able to specify properties like this.
Another example is that the net expects arms in photos of dumbbells.
When the net sees an image, it doesn't know which part of the image the label
refers to.

I vaguely remember a webpage where you could draw something with your mouse,
and a neural net predicts what the object is.
That net may have more understanding about properties of objects than the ones
trained on real images from the web.

Temporal difference learning:
Learning a guess from an earlier guess.
Refining your guesses as the world changes.
Dopamine spike when the outlook suddenly looks better.
Dopamine drop when the outlook suddenly looks worse.

Say you are trying to teach an agent how to accomplish a complex task.
When the agent's initial behavior is random, the agent may be very far from
accidentally doing actions that complete the task.
So, it is important for the agent, when it is in an intermediate state, to have
a sense of how close/far it is from the goal.
That's where TD learning helps.
TD learning was discovered in a purely theoretical context and later found to
explain dopamine behavior in monkey brains.

BF Skinner was working on a secret project during WW2 to train pigeons to become
suicide bombers.
At some point, they built a miniature bowling alley and were trying to teach a
pigeon to play bowling.
They wanted to give the first food reward when the pigeon rolled the ball.
But the pigeon did nothing.
Then they decided to reward when the pigeon just looked at the ball, and from
that point reward for actions that are increasingly closer to the objective.
This technique is called shaping and it worked marvelously.
His collaborators left science to start an animal training company based on this
principle, and it became the biggest company of its kind in the world (training
animals for movies, Seaworld, etc).

Reinforcement Learning (RL) in a nutshell.
The agent starts doing random actions.
You reward the ones that bring the agent closer to the goal, and it eventually
learns to prefer those.
Epsilon-greedy RL means that most of the time (say 99% of the time) the agent
performs actions that maximize the reward, and 1% of the time the agent performs
random actions (presumably to get out of local maxima).
Epsilon-greedy works very well in situations where the rewards are easy to
discover by random actions (called dense rewards).
But in many real-world scenarios, the environment is very complex and it is hard
to find rewards randomly.
This issue is called sparse rewards, or just sparsity.

Shaping (teaching an easier problem first and gradually harder problems) has
been found to work when training computers as well.
Some NLP researchers were trying to teach a computer to predict the next word
in an English sentence without success.
Then, they trained only on simple child-level sentences and the machine learned
to predict there, and was then able to be trained and predict on sentences of
arbitrary difficulty.

In RL, designing reward functions is maybe the most challenging part.
You want to be rewarding for intermediate accomplishments, because in any
realistic scenario, the end-goal rewards are sparse, but you have to be careful
otherwise the rewards create perverse incentives that don't lead towards the
goal.  
Examples:  
*  Soccer-playing robots that were rewarded for ball possession hogged the ball
   and never tried to score.
*  Bicycle robot rewarded for moving towards the target but not penalized for 
   moving away from it ends up biking in circles near the origin point.
*  Child rewarded for cleaning up dumps the dirt again to clean it again.
*  Child rewarded when taking younger sibling to the bathroom fills him up with
   water so he can go more often.

Andrew Ng and Stuart Russell came up with the idea of rewarding states of the
world instead of actions of the agent.
Doing this improves reward functions in some cases, e.g.,
*  Reward bike robot based on how far it is from the origin, not how it got
   there.
*  Reward child for the clean floor instead of the act of cleaning.

As part of his PhD, Bellemare‎ built an environment called ALE where people
could try various RL algorithms on Atari games.
The RL agent only knows the pixels on the screen and from that it needs to learn
how to play the entire game.
After his PhD, he went to Deepmind where he and others applied deep learning to
RL, and created a system that beat the state of the art in many Atari games,
often surpassing human performance.
They published a paper in Nature and this was the start of Deep RL.

Daniel Berlyne was a psychologist who studied intrinsic rewards in
animals.
Curiosity / desire to explore new stuff.
(He himself had many interests and rarely worked nights / weekends because he
was exploring his other interests.
Despite that he had a very successful career.)
Curiosity is useful in sparse games.
When you aren't achieving much, don't do random actions; favor actions you
haven't done before.

Points are extrinsic rewards.
Novelty and surprise are intrinsic rewards.
Game agents and people can be motivated by intrinsic rewards.
Agents that are programmed to be curious did well in "Montezuma's revenge",
where the point rewards are sparse.
They managed to explore all the game's screens and reach the end.

There is a trick that fools RL agents trying to search novel/surprising areas of
their environment.
If there is some static noise, or a TV screen playing stuff, etc, the agent
keeps finding the source of randomness surprising and gets stuck.
It can't classify all the random behaviors as essentially the same and move on.
The author hypothesizes that this also may be a cause for humans' addiction to
gambling and TV (seems unlikely to me, at least not without additional causes).

Scientists have found that monkeys and chimpanzees imitate what they see less
than humans do.

A newborn, within the first hour of its life, sticks its tongue out when it sees
an adult do it, even though it has never seen itself in the mirror.
It knows somehow that it is similar to the adult and imitates.
Psychologists believe that humans' ability to imitate is fundamental to their
development, not a result of their development.
This has been a shift in psychology research.

Scientists have performed experiments where young children over-imitate an
adult, e.g., the adult opens a clear drawer without food and then another one
with food, and the child does the same instead of going for the food directly.
Otoh, a monkey goes for the food directly.
This was eventually explained as a sophisticated behavior of children: they see
the adult as trying to teach them something, and they trust that the adult knows
something they don't.

Inspired by imitation in humans, ML researchers created an agent learning to
play Montezuma by watching YouTube videos of people playing the game first.
The agent did much better than the RL agents based on exploration and learning
from scratch.

A team of two climbers who first climbed Dawn Wall in Yosemite took several
years to plan the route and do it.
The next climber who replicated their route needed only a few weeks.

General lesson: when we know that something is possible, it is much easier for
us to do it, especially when we first see someone else do it.

Imitation was also very helpful in the first successful self-driving cars in the
early 1990s.
(The human controlled acceleration and braking and the car was steering, and was
able to drive on the highway.)

A big difficulty with imitation learners is that they only ever see the human
playing expertly.
Since they never see precarious situations, they don't know how to recover from
them.
So when as novices they inevitably make a mistake, they go into uncharted
territory and catastrophic failure follows.
This problem is called cascading failures.  
What people have done to address this issue:
* The early car researchers used data augmentation to generate images where the
  car is not centered on the road, and tagged them with the correct steering
  command for recovery.
* A later car project (that trained in a video game environment IIRC) used a
  cool trick called interaction.
  The human is driving the virtual car, and at random moments the computer
  takes control, then randomly again passes it back to the human.
  This way, the human shows how to recover from mistakes.
* Researchers wanted to train a drone to follow a trail in the forest.
  They had a hiker walk the trail with three cameras, one facing forward, one a
  bit left and one a bit right, and told the hiker to always look straight.
  Then, they tagged the left images with "go right" and the right with
  "go left", so the drone had lots of data on how to recenter on the trail.

IRL (Inverse Reinforcement Learning): a technique where the agent doesn't
know the reward function to be maximized, but must infer it from a
demonstration.
IRL has been used in robotics to achieve things not achieved before, e.g.,
complicated helicopter tricks, and robot-leg back-flip.

Inferring the human's objective is also important for problems where it is hard
to describe it, but we know it when we see it, and also for AI safety issues.

The book doesn't describe how IRL works so I don't know the algorithm even at a
high level.

The open category problem: when you train an image classifier on k classes,
whatever image you then show to it, it will assign it to a class.
Instead you want it to say "I don't know".
Very important.

Bayesian neural nets: weights aren't numbers, but probability distributions.
You can sample from the net.
You show it an image several times.
If it predicts different labels, this shows some uncertainty in its knowledge.
Bayesian neural nets handle uncertainty but are not practical to train.
Then, people discovered ensembles.
You train many nets and ask them the same question, and average their
predictions.
Ensembles approximate BNNs but are still expensive to train.
Then, people found that you can use dropout during inference.
You ask the model to infer the same input many times, each with different
dropout, and average the results.
Turns out that is also a good approximation of BNNs and it is feasible in
practice!
