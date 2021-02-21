---
layout: post
title: Toys 'R' Us-eful
subtitle: Insights from toy simulations
---

Back in the *Before Times™*, i.e. March, before the first lockdown, the first mask worn, the first uncomfortable interaction at the supermarket, the new phrase 'social distancing' was appearing more and more and there was lots of talk of flattening the curve.

I had been following developments around the coronavirus for some time, after all I can see the hotel where the first UK case was reported from my bedroom window, but now people were talking about it in terms of infection rates and R numbers - something I could understand more than biological mechanisms.

So one evening in early March I sat down and created a basic model for infections in a population. This was primarily just an interest project but it also gave me an opportunity to understand some of the mechanisms behind upcoming public health policy.

# Individual based modelling

I decided to base my model on one which I have used numerous times before, first as an undergraduate and later when creating a demo for my research group to show, the Ising model.

The Ising model is one of the simplest model of magnetic materials in which a material is represented by a number of spin sites where the spin can be in one of two states - up or down (+1 or -1). The model predates modern computers however it's still a useful tool for studying some simpler materials.

The key mechanism which underlies the Ising model is the flipping of spin states via a Monte Carlo method. The process is as follows:

* Choose a random spin site
* Calculate the energy change of the system if the spin is flipped
  * If the energy of the system is lowered then flip the spin
  * If the energy of the system is increased then test for spin flip against a probability distribution
    * P = exp(-(del(E)/T))
      * P: spin-flip probability
      * del(E): change in energy of the system if the spin is flipped
      * T: the temperature of the system
    
    * The test is done by generating a random number between 0 and 1 and seeing if it is lower than the calculated P
    * If so then the spin is flipped and the process continues







The experience, and the weeks that followed, taught me a few things about the usefulness, and pitfalls, of making toy models for an area outside your own expertise. A week or so after I spent the evening messing with intervention levels, the MRC Centre at Imperial College published their report into non-pharamceutical interventions for managing the pandemic.

I was surprised by how well my simple model captured the core mechanics used in the academic work, although of course it was not parameterised so didn't provide any quantifiable information like the MCR group. Perhaps this is more understandable when noting that the lead author Neil Ferguson himself has a PhD in computational physics!

However it also highlighted just how poorly models are understood by media and the wider public. I often saw *inputs* to the model, usually the base mortality rate for COVID-19, presented as outputs of the research. There also seemed to be more focus given to individual infection risks rather than the statistical study that it was.

