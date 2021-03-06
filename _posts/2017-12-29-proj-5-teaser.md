---
layout: post
title: Metis Project 5 — Sax | Not Sax (Teaser)
---

As a professional musician in my former life, the saxophone paid my rent. Thus, I thought it fitting to pay it tribute for this project, and see if I could train a convolutional neural network to detect its presence in audio clips. I created a volume of 24,000 audio samples from several hundred songs in my library, logging a record of each one in a MongoDB database at the time of creation. To label these samples, I wrote a web app using Flask and JavaScript, which I then hosted at [audiosamplelabeler.com](http://audiosamplelabeler.com) so anyone anywhere could pitch in and assist in the labeleing effort. As of 12/8/17, 2,000 out of 24,000 samples have been labeled, and the effort will continue indefinitely.

Thankfully, 2,000 labeled samples turned out to be enough to train and test a PoC neural net classifier model. To build the model, I turned to [PyTorch](http://pytorch.org), which makes up for a steeper learning curve and relatively sparse documentation with a high degree of customizability and sheer power. I trained and tested on 128p by 108p spectrograms, which allowed for relatively quick training, and enough information for the model to start picking out patterns.

PyTorch has neither a train-test-split nor cross-validation utilities, so I had to write my own for each. Next on the list are utilities for grid search and random search.

When the cross validation function was up and running, I kept running into a problem where the model would get stuck in what I started to call "The 0.693 Desert," and then "Death Valley." After the cross-entropy loss hit this value, there was very little chance of recovery, and it would stay here for each remaining epoch. While there is no way to say exactly what my hundreds-of-dimensions loss function actually looks like, I imagine it is something of a saddle point, which, on one side, presents a giant basin with a local minimum of 0.693, while on the other, an abyss in which one can easily minimize loss to the point of significant overfitting. 

I was able to address this problem through a combination of methods. First, I changed the initialization settings so that weights were initialized on a normal distribution instead of uniform (mean of 0, standard deviation dynamic according to layer inputs and kernel size). This tightened up the weights and kept them closer to zero, which lowered the chance that a larger initial value would result in a large and irrecoverable step into Death Valley. Second, I increased the batch size to 16, which ensured that the steps taken would be more even due to a larger variety of samples. Finally, I used the Adam optimization method, the dynamic learning rate and momentum of which allowed for smaller, more cautious steps at the beginning, and then significant acceleration once on the right track.  

Through these efforts, I was able to cut down the frequency of Death Valley runs to less than 1%, whereas these constituted the vast majority on my first attempts at model training.

After training on the full train set and testing on train and test, I was able to attain accuracy scores of around 84% for the train set and 77% for the test set, quite satisfactory for a for a PoC model. I believe that the model could perform even better, and as such I would like to take the following steps to improve both model and process:

1. More labeled data (at least 5,000 samples, if not 20,000+).
2. Spend more time researching and tuning the CNN architecture, not only for increased accuracy, but to mute the tendency for overfitting.
3. Use higher-resolution spectrograms, which may or may not be helpful due to more information.
4. Experiment with the transfer learning options offered by PyTorch, since my data is limited, even with 24,000 samples.
5. Move to GPUs to greatly accelerate the process of model selection and training.

A more in-depth series of posts on this project will be coming soon.
