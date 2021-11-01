The following notes are a result of the lightning talks and community discussion that took place at the ModRef 2021 workshop on Oct'25 2021. 

The ModRef session was collaborative exercise to identify challenges for the next five years of modelling, with the goal of producing a jointly-written white/research paper to guide future research. The main topic is on the integration of modelling as a component in larger systems. This session consisted of a series of lightning talks pointing out pain-points or desired features for modelling, then a series of focussed discussions to identify common features, possible solutions and research directions.

# Problems Identified

Our sophisticated CP algorithms and tools may not be easy to integrate within larger systems in some cases. This gives rise to several problems, for example:

- No support for object-oriented models.
- No common interface for CP solvers, e.g., it is not easy to swap solvers within code, which is not the case in the SAT community which has an established MiniSAT interface.
- Not easy to share preprocessors for CP solvers, e.g., each solver has to do its own presolving.
- There is no easy way to make use of solvers in an incremental setting, e.g., most solvers are one-shot solvers, but adding and removing constraints/variables and resolving needs to be done from scratch.

# Current Situation

We have great modelling languages and systems that translates a high-level problem specification into a lower-level format, e.g., MiniZinc, Essence. However, the lower-level format is typically text-based, e.g., FlatZinc. This gives rise to two problems:

- Writing text parsers may be difficult and inconvenient.
- The problem of incremental CP still remains.

# Possible Solution

The first step in addressing the above problems would be to come up with a standardised API for constraint programming solvers. With such an API, integrating CP technology in a larger system would mainly come down to communicating through the API, making it easy to develop high level algorithms, swap in and out different solvers, and other kinds of tools.

It is important to note that here the goal is _not to replace_ MiniZinc or similar tools, but perhaps move away from a text-based representation of Flatzinc towards an API-style interaction.

# Open Questions

We need to:

- Find specific use cases which could give better insight into what this API should look like.
- Agree on the functionality and purpose of the API. 	

## Specific use cases

- Scenario 1: Online scheduling where we would like to add and remove tasks (variables and constraints) on the go without needing to restart from scratch.
- Scenario 2: Model checking or abstraction refinement problems where the algorithms solves an approximation, and then modifies it depending on the model (TODO expand with more details).
- Scenario 3: Industry users may like to have an easier time writing their own algorithms on top of solvers.

## API Functionality

- There were many concerns that this may be difficult to standarise across solvers. For example, even MiniZinc does not support all the functionality of solvers, so possibly the API might need to be too large.
- We could start 'small', try to set up some 'basic' functionality (also needs to be agreed upon), and work from there.
	1. Starting small has the advantage that more solver developers would be willing to support the new API.
- Need to identify what can be shared among solvers, e.g., (creating bool/int variables, posting linear, non-equal, alldiff, element constraitns, solving). 
	1. But also allow individual solvers to have freedom in having their own API extensions.

# How to move forward?

These could be topics for our next meeting.

- What would we like to do on a high-level? Perhaps this is the most critical question we need to address first before moving further.
	1. What is the goal of the API? Perhaps link to use cases.
		* Is it to be a modelling-like library, or is to provide a common interface to different solvers?
		* Function-based versus registry-based
	2. Is what we are looking for really an 'API' or should it be called differently? Maybe we are looking for something in between a modelling language and low-level solver interfacing, e.g., this way we can include some type-safe bindings.
	3. Can we decide on these points _without_ deciding on which programming language to use?

- Can we agree on a base API?
	0. What is the smallest API that makes sense as a starting point?
	1. What is a _variable_ in this API? How to create a variable? Ideally we need to not make any assumptions about how the variables are represented internally in solvers.
	2. What is a constraint? Are there 'base' constraints? MiniZinc assumes there are no base constraints, and asks the solvers to give info on how to rewrite. 
	2. What does _remove_ a constraint mean?
	3. Do we want to have _assumptions_? Here by assumption is meant a set of atomic constraints (e.g., X >= 3), which is given to the solver, and we expect the solver to report a solution that satisfies the given assumptions. There is a connection between adding and removing constraints and assumptions.
	4. Do we want to require solvers to produce a _cores_, i.e., a subset of the assumptions which in combination with the underlying model lead to infeasibility? Solvers that do not support core extraction could simply return all the assumptions in case of unsatisfiability.
	5. Choice points may be interesting, i.e., after the user defines a 'choice point', the user can revert the solver state back to a given choice point.
	6. How is this API different from MiniZinc Python? Is it complementary? Do we want to do more or less than MiniZinc, or perhaps different things?
	7. Maybe can start out as an API that wraps around a text-based format?
	8. Does the API also take care of the state of the solver? This should be the case if we would like to achieve incrementality.
	9. How to address intermediate solutions?

# Random

These are some comments that did not make it into the above text, but are nevertheless relevant.

Conjure has json support. May be useful to look into this and see how it relates here.

XCSP has an interesting format, we could learn a few things by looking into their abstractions.

SAT community
- PySAT
- IPASIR interface (incremental interface)
- https://github.com/biotomas/ipasir

SMT Community
-SMTLib format
- http://smtlib.cs.uiowa.edu/theories-Core.shtml


Technical problems
- Parallelism may be problematic, e.g., if c++ does hidden parallel things behind the scenes, may be a problem, may be easy for the user to write bad code.
- Python to c unsafe if you spawn threads
- Do we assume the program resides in the same thread, or do we assume processes are talking to each other?