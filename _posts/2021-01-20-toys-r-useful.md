---
layout: post
title: Toys 'R' Us-eful
subtitle: Insights from toy simulations
---

Back in the *Before Times™*, i.e. March, before the first lockdown, the first mask worn, the first uncomfortable interaction at the supermarket, the new phrase 'social distancing' was appearing more and more and there was lots of talk of flattening the curve.

I had been following developments around the coronavirus for some time, after all I can see the hotel where the first UK case was reported from my bedroom window, but now people were talking about it in terms of infection rates and R numbers - something I could understand more than biological mechanisms.

So one evening in early March I sat down and created a basic model for infections in a population. This was primarily just an interest project but it also gave me an opportunity to understand some of the mechanisms behind upcoming public health policy.

# The Ising model

I decided to base my model on one which I have used numerous times before, first as an undergraduate and later when creating a demo for my research group to show, the Ising model.

The Ising model is one of the simplest model of magnetic materials in which a material is represented by a number of spin sites where the spin can be in one of two states - up or down (+1 or -1). The model predates modern computers however it's still a useful tool for studying some simpler materials.

The key mechanism which underlies the Ising model is the flipping of spin states via a Monte Carlo method. The process is as follows:

* Choose a random spin site
* Calculate the energy change of the system if the spin is flipped
  * If the energy of the system is lowered then flip the spin
  * If the energy of the system is increased then test for spin flip against a probability distribution
    > P = exp(-(del(E)/T))
    > P: spin-flip probability
    > del(E): change in energy of the system if the spin is flipped
    > T: the temperature of the system
    
    * The test is done by generating a random number between 0 and 1 and seeing if it is lower than the calculated P
    * If so then the spin is flipped and the process continues

This highlights one of the main uses of Monte Carlo methods, testing for statistical outcomes arising from individual occurences - in this case the magnetisation of a material due to spin-flipping at elevated temperatures.

(I have previously coded a simple version of the Ising model with visual output to terminal using Curses that can be found [here](https://github.com/InsomniacVegan/ising-model) if you want to see the spin-flipping in action)

# Infection modelling

My curiosity was sparked when I realised that each spin site could be treated as an individual and the binary spin-state could simply be the infected status of the individual. I also added an additional binary state for whether the individual was immune.

In this case the model can borrow heavily from the Ising implementation but change the probability test to rely on relevant factors controlling infection in the population. Since I was only interested in trends for the data, rather than any quantified results, I chose two broad factors to determine infection probability; the level of interventions in place to bring infections down, I, and the 'base' infectiousness of the virus given as R_0.

Thus our probability is now given as:

> P = exp(-(I/R_0))

With the infection mechanism in place, the next step was how to allow the population to interact and how to incorporate time. Again I went for simplicity and chose to use a nested for loop structure of *total time* and *daily interactions*.

An interaction occurs as follows:

  * Choose two random members of the population
  * If neither are infected then proceed to the next interaction
  * If both are infected or immune then proceed to the next interaction
  * Otherwise attempt to infect the members based on the above probability

After the total number of daily interactions occur, time is advanced by one day with infection and immunity status determined by how long it has been since initial infection, e.g. after 10 days the infected state is returned to False.

# Coding implementation

With the general outline of the model sketched out it was time to throw some code together, in reality this is happening as I develop the model to test various ideas. Since the idea is to use the model primarily as a quick learning tool, I went with a Jupyter notebook where I could combine the code with more extensive commentary and equations. The notebook is available from my [Github](https://github.com/InsomniacVegan/CV19-model).

As the simulation is small scale I could make full use of an object oriented approach without worrying about the computational cost too much. The obvious choice is to package the individual as one class and then wrap the simulation backend into a separate class.

## Individual

An individual in this simulation can be thought of as a unit cell of infection and immunity, in fact some research in the area takes this concept further and literally treats the infections in a cell based manner.

As such it makes sense to combine the data, the binary infection and immunity states, with methods to infect the individual and progress any existing infection or immunity.

> class Person:
>    """Base individual for COVID-19 model"""
>    
>    def __init__(self):
>        # Infection
>        self.infected   = False
>        self.t_infected = 0
>        
>        #Immunity 
>        self.immune     = False
>        self.t_immune   = 0
>    
>    
>    def infect(self):
>        # Do  not reinfect
>        if self.infected or self.immune:
>            return
>        self.infected   = True
>        self.immune     = True
>    
>    def progress(self):
>        
>        # Progress infection if occured but reset after 10 days
>        if self.infected:
>            self.t_infected += 1
>            if self.t_infected >= 10:
>                self.t_infected = 0
>                self.infected = False
>                
>        # Progress immunity if occured but reset after 365 days           
>        if self.immune:
>            self.t_immune += 1
>            if self.t_immune >= 365:
>                self.t_immune = 0
>                self.immune   = False

It is worth noting the inclusion of a check in the infection mechanism to prevent reinfection while the individual is already infected or immune, although really only a check on immunity is needed here. This is included to make our implementation of infection in the main simulation *individual agnostic*. By placing this check inside of the infection method then we are free to call *infect* on any individual without worrying about damaging our statistics by causing prolonged infections.

## Simulation

The simulation wrapper handles the progressing of the model through time and the tracking of statistics regarding the population.

Upon initialising the simulation the population is generated and an initial seed infection occurs.

> class Simulation:
>     """Main simulation wrapper"""
>     
>     def __init__(self, pop_size = 100, seed_pop = 1):
>         
>          # Initialise numpy rng
>         np.random.seed()
>         
>         # Simulation variables
>         self.pop_size = pop_size
>         self.seed_pop = seed_pop
>         
>         # Create population
>         self.population = [Person() for i in range(self.pop_size)]
>         
>         # Infect seed population
>         infected_indices = np.random.randint(0,self.pop_size,self.seed_pop)
>         for i in infected_indices:
>             self.population[i].infect()
>         
>         # Track statistics
>         self.total_infections        = []
>         self.current_infection_level = []
>         self.immunity_level          = []
>     
>     
>     def run(self, time=100, daily_interactions=100, intervention_level=1, r_0=1):        
>         
>         # Simulation parameters
>         self.time = np.arange(0,time,1)
>         self.daily_interactions = daily_interactions
>         
>         # Precompute probability
>         infection_probability = np.exp(np.divide(np.multiply(-1.0, intervention_level),r_0))
>     
>         total_infection_counter = 0
>     
>         # Simulation loop
>         for day in self.time:
>                 
>             for interaction in range(daily_interactions):
> 
>                 # Select two random people
>                 person_1, person_2 = np.random.randint(0,self.pop_size,2)
> 
>                 # If both are immune then select a new interaction 
>                 # Infected individuals are automatically immune
>                 if(self.population[person_1].immune and self.population[person_2].immune):
>                     continue
> 
>                 # If neither are infected then also generate a new interaction
>                 if(not self.population[person_1].infected and not self.population[person_2].infected):
>                     continue
> 
>                 # If random number [0,1) falls under probability criteria then an infection occurs
>                 if (np.random.random() < infection_probability):
>                         # Infection will only take place if not already infected/immune
>                         self.population[person_1].infect()
>                         self.population[person_2].infect()
>                         total_infection_counter += 1
> 
>             # Update statistics
>             self.total_infections.append(total_infection_counter)
>             self.current_infection_level.append(len([person for person in self.population if person.infected]))
>             self.immunity_level.append(len([person for person in self.population if person.immune]))
>     
>             # Generate endemic infection
>             if (self.current_infection_level[-1]==0):
>                 if np.random.random() < 0.01:
>                     self.population[np.random.randint(0,self.pop_size)].infect()
>     
> 
>             # Progress each person's infection
>     
>             # N.B This could be made faster by only tracking infected
>             # or immune individuals but simplicity is the goal here
>             for person in self.population:
>                 person.progress()

The simulation also includes a mechanism for endemic infection, where if there are no infections in the population then there is a small chance of an infection appearing. This is to simulate potential viral dormency or external introduction of the virus, e.g. from travelling.





The experience, and the weeks that followed, taught me a few things about the usefulness, and pitfalls, of making toy models for an area outside your own expertise. A week or so after I spent the evening messing with intervention levels, the MRC Centre at Imperial College published their report into non-pharamceutical interventions for managing the pandemic.

I was surprised by how well my simple model captured the core mechanics used in the academic work, although of course it was not parameterised so didn't provide any quantifiable information like the MCR group. Perhaps this is more understandable when noting that the lead author Neil Ferguson himself has a PhD in computational physics!

However it also highlighted just how poorly models are understood by media and the wider public. I often saw *inputs* to the model, usually the base mortality rate for COVID-19, presented as outputs of the research. There also seemed to be more focus given to individual infection risks rather than the statistical study that it was.

