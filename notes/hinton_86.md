# Hinton - 1986 - Learning Distributed Representations of Concepts

A concept can be represented as activity patterns in a neural network.

There are two "obvious" ways to do this: encode features as neural network units (= nodes) and then represent concepts by strengths of connections between units, and to make each concept a unit and represent relationships with connections.

Another way to do this (this paper) is to assign groups of units to each concept. Individual units may represent sub-concepts (e.g. a unit active iff a concept is male); the author calls these "role-fillers". Propositions are stable combinations of role-fillers. This is interesting because the same concept can have different representations if it is fulfilling different roles. Instead of storing representations of concepts, we store representations of conjunctions of propositions and concepts.

Extra units that activate role-fillers to constrain their relationships are said to encode a micro-inference. Micro-inferences encode underlying domain regularities and hence store propositions, which causes sensible generalisation.

Good representations use units that allow useful micro-inferences to be expressed in terms of the units. Each role-filling unit partitions the set of concepts into those for which a unit is on and those for which a unit is off. Searching for good representations is searching the space of partitions. (Note that the units don't have to be binary, but it's assumed for this paper as an idealisation.)

