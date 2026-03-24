**Research Proposal: Morphological-Neural Co-development Mechanism Based on Spatial Dynamic Graphs and Its Embodied Verification**

# 1. Abstract

Currently, mainstream embodied AI systems mostly adopt a paradigm of "pre-defined physical structure + learned control strategy." The morphology and control networks of robots are often designed in isolation. Although the field of Artificial Life has long proposed the concept that morphology and control should co-evolve, existing developmental models often struggle to describe real structures in physical space. Furthermore, embodied verification bridging virtual simulation and physical entities remains difficult to achieve.

This research aims to explore the performance of evolved artificial life in terms of polymorphism and behavioral patterns after introducing environmental inputs and physical constraints. To this end, a unified representation framework based on **spatial dynamic graphs** is proposed. This framework uses Compositional Pattern Producing Networks (CPPNs) as the genotype, breaking the limitations of static generation in traditional neural networks, and allowing artificial life to dynamically adjust its topology based on environmental information during development. 

Additionally, this research proposes to build a **modular physical system** based on distributed MCUs and bus communication, mapping the topologies evolved in simulation onto distributed hardware composed of actuators, sensors, microcontrollers, and physical buses. During the embodiment phase, it further explores the potential of utilizing compliant sensory mediums on the entity's surface as an **implicit physical reservoir** to synergize with the distributed network. Through the physical embodiment of artificial life, this research aims to verify the effectiveness of **morphological computation** in real multimodal environments.

# 2. Research Background and Problems to be Solved

The co-evolution of morphology and neural networks has always been a core topic in the field of Artificial Life. As early as 1994, Karl Sims' research on virtual creatures$^\textsf{[1]}$ demonstrated the decisive role of morphological diversity in the emergence of behavior. However, with the development of deep learning, research focus gradually shifted towards pure network weight optimization, while the exploration of morphogenetic mechanisms has lagged. Several of the most representative existing approaches for morphogenesis and evolution still have significant limitations:

1. **Fixed Phenotype Generation Patterns:** Indirect encoding schemes based on NEAT and CPPN$^\textsf{[2]}$ (such as HyperNEAT) can generate complex spatial connections, but their phenotype generation is usually one-off and static. Once decoding is complete, the network structure is fixed, lacking a "developmental" process of dynamic growth through environmental interaction.
2. **Limited Spatial Expression:** The highly regarded Neural Cellular Automata (NCA)$^\textsf{[3]}$ demonstrate amazing capabilities in morphological regeneration and local updating, but their underlying data structure relies heavily on fixed voxel grids. Such rigid grids are difficult to use for simulating the 3D extension and mutual squeezing during biological tissue division. Moreover, the substrate limitation deprives the model of pathways to interact with the external environment.
3. **Lack of Physical Constraints:** Pure virtual evolution often ignores the physical laws limiting information transfer in the real world, and the evolved morphologies often lack reasonable governing rules. This makes migrating topologies evolved in simulation to physical entities extremely difficult. For example, Xenobots$^\textsf{[4]}$ designed biological structures capable of various locomotion modes in a virtual environment, but the actual assembly of cells is difficult to manipulate, allowing the final living entity to only roughly exhibit the preset functions.
4. **Engineering Bottlenecks of Pure Physical Computation:** The recently emerging Physical Reservoir Computing (PRC)$^\textsf{[5]}$ has demonstrated the elegant potential of utilizing flexible bodies directly for morphological computation. However, relying solely on compliant mediums encounters various engineering bottlenecks in real-world applications: the inherent damping of physical materials leads to rapid state dissipation, making it difficult to maintain long-term memory; environmental drift, such as variations in temperature and humidity, easily disrupts the original non-linear mapping; furthermore, limited by the input bandwidth of actuators and the signal-to-noise ratio of sensors at the readout layer, weak high-order features are often submerged by environmental noise, rendering the system highly fragile.

In summary, designing a developmental model that can break through rigid grid limitations, support continuous physical space expansion, and integrate real physical constraints is the core problem this research seeks to address.

# 3. Research Objectives and Core Hypotheses

This research is dedicated to establishing a novel framework for the **co-development of morphology and control** at the intersection of Evolutionary Robotics and Artificial Life. While achieving absolute "open-ended evolution" remains the ultimate vision of the field, this proposal prioritizes engineering convergence. By introducing rational local constraints and mechanisms, this study aims to observe the morphological polymorphism and behavioral patterns of artificial life, fundamentally verifying the following hypothesis:

**Hypothesis:** Compared to traditional deterministic developmental mappings, the introduction of environment-dependent developmental mechanisms combined with spatial graph topologies containing physical attributes can overcome phenotypic rigidity. This "co-evolution" of morphology and neural networks not only facilitates the emergence of functional robustness in simulation but can also be successfully deployed onto distributed physical hardware, demonstrating spontaneous embodied adaptability in unstructured, real-world environments.

# 4. Research Methods and Implementation Plan

One of the core of this plan is to design a system architecture capable of crossing the boundary between virtual and reality, allowing evolved organisms to be validated in simulation engines and replicated in reality using existing electromechanical engineering technologies.

## 4.1 Topological Representation of Life: Dynamic Spatial Graphs with Physical Dimensions

To overcome the defects where artificial neural networks lack spatial features (making it difficult to integrate with physical structures) and voxel models lack topological expansion flexibility, this proposal designs a **specialized dynamic spatial directed graph**:

Various types of cells are treated as nodes in the graph, and physical connections and information pathways between nodes are represented by directed edges. The edges contain specific parameter lists in the form of multidimensional vectors, used to store or facilitate the emergence of key attributes such as connection distance, excitation/inhibition, activation state, and delay.

In terms of engineering implementation, a decoupled architecture utilizing a **Node Dictionary** and a **Global Sparse Adjacency List** is adopted:

- **Node Dictionary:** $\boldsymbol{\mathcal{V}} = \{\boldsymbol{v}_1, \boldsymbol{v}_2, ..., \boldsymbol{v}_N\}$, stores the physical states and attributes of the cells themselves.
- **Sparse Adjacency List:** $\boldsymbol{\mathcal{E}} = \{\boldsymbol{e}_{ij}\}$, records directed connections with distance and topological logic, along with the parameter vectors of the connections.

This data structure not only provides meaningful physical boundaries for multimodal sensor inputs but also supports the dynamic insertion of new nodes during operation (simulating growth and division), providing support for possible mechanical feedback such as physical stretching and squeezing.

<figure style="text-align: center;">
  <img src="graph_neuron.webp" width="70%" alt="Example of life's structural topology">
  <figcaption style="font-size: 0.9em; color: grey;">Figure 1: Example of a spatial dynamic graph topology with connection attributes</figcaption>
</figure>

This data structure allows for the dynamic insertion of new nodes during the organism's development to simulate cell division. The stretching and squeezing during biological tissue growth can also be simulated and replicated through computational updates on the directed edges, laying the foundation for subsequent mapping to robotic joint modules with real dimensions.

Because loops are allowed in the graph, and the connection establishment process is gradually co-generated by the genotype and environmental vectors, this simulation mechanism is expected to produce preliminary, observable spontaneity and adaptability.

## 4.2 Genotype Design and Phenotypic Developmental Rules

**Genotype:**

The genotype is a CPPN with cross-scale expression capabilities, serving simultaneously as a morphogenesis generator and a network builder.  
Because the available evolutionary directions encompass both the morphological structure and the internal neural network structure, designing isolated construction genes would lead to the credit assignment problem. Therefore, a construction mechanism with cross-scale expression capability is directly introduced as the genotype, ensuring that the growth of the body and the neural network occur synchronously.

- **Inputs:** Node's own state vector, local environmental information vector, adjacency list connection parameter vector.
- **Outputs:** Increment of the node's own state, increment of local environmental information, generation state of new connections/new nodes.

**Dynamic Development of the Phenotype:**

Development begins from the initial node. Under the influence of different environmental conditions, the genotype will express itself in different ways. For example, when the input tensor meets a certain condition, the genotype's function as a morphogenesis generator takes effect, instantiating a morphological structure. Correspondingly, input (sensor) and output (actuator) nodes are generated, providing conditions for further network establishment.

The construction of the phenotype and the transmission of neural signals proceed based on time steps. Regarding structural construction, within a time step, cell nodes will decide on the formation of new structures based on input information.

Regarding neural signal transmission, by introducing a time accumulation effect and physical transmission constraints, nerve impulses proceed sequentially and continuously, preventing uncontrolled positive feedback generated within loops.  
For instance, a hidden state variable, the accumulated membrane potential $U_i^{(t)}$, is defined. At time $t$, the node state update follows this rule:

$$U_i^{(t)} = U_i^{(t-1)}\cdot(1-\lambda) + \sum_{j\in \mathcal{N}(i)}S_j^{t-\Delta t_{ji}} \cdot w_{ji}$$

Here, $\lambda$ is the natural decay rate of the membrane potential, $\Delta t_{ji}$ is the physical transmission delay based on topological distance, $S_j^{t-\Delta t_{ji}}$ is the nerve impulse transmitted from upstream, and $w_{ji}$ represents connection attributes, such as excitatory and inhibitory connections. This forces the network to find optimal layouts to meet real-time signal requirements, continuously optimizing its topology.

## 4.3 Training and Evolutionary Mechanisms

Because the dynamic generation of graph structures and variable morphology involve non-differentiable discrete operations, traditional backpropagation algorithms will become ineffective. This research will use Genetic Algorithms (NEAT + GA) to drive evolution.

- **Fitness Evaluation:** During training, the reward model or fitness evaluation system is one of the most critical modules. Although the ultimate mode of open-ended evolution cannot be directly achieved immediately due to the difficulty of environmental system design and result convergence, evaluation systems with specific mechanisms can still be designed to enable winning individuals to develop richer morphological and behavioral traits. For example, random simple scenarios can be generated to input multimodal information into test individuals, and comprehensive macro-survival performance can be evaluated through competition and other mechanisms. During scenario design, learning experiments can be incorporated to test whether the individual tends to generate new behavioral patterns after several rounds of pattern repetition.
- **Speciation:** Regarding the speciation rules in the NEAT algorithm, since this project introduces morphological components, speciation can be further refined based on steady-state morphological features, in addition to directly calculating speciation from the genotypes.
- **Morphological Stability:** To ensure the final evolved morphology is manufacturable, the number and complexity of instantiated somatic modules will be evaluated. Inferior individuals that proliferate infinitely or fail to maintain a stable morphology will be eliminated.

## 4.4 Physical Embodiment and Sim2Real

Embodiment is the core component for verifying the morphological computation mechanism in this proposal. This research will directly map abstract topological graphs and virtual morphologies onto real electromechanical systems. 

Before evolution begins, a series of standardized bodily modules providing parameterized interfaces needs to be designed.

After evolution begins, the somatic instances created by the phenotype will map to the corresponding bodily modules, using the modules' parameter interfaces to perform fine-tuning (e.g., length parameters). By instantiating manufacturable modules, artificial life first evolves in a simulated environment and can then be replicated in a real-world environment.

- **Bodily Modules:** For example, joint modules with rotational degrees of freedom (corresponding to upper arms, forearms, etc.), containing output interfaces for actuators like motors, as well as input interfaces for stress/tactile sensors and temperature/humidity sensors. To adapt to complex physical contacts in the real world, the exterior of some modules can be wrapped with a layer of compliant sensory medium (e.g., a silicone skin). Additionally, central nervous modules with high processing performance and frame modules connecting morphological features can be designed. An MCU is embedded inside each module, equipping it with a local neural network to handle local reflexes and sensor data aggregation.
- **Perception Modules:** For example, vision modules and hearing modules. These can be located anywhere on the body, but their morphological generation patterns will be subjected to adaptive selection, ultimately evolving into a rational layout.
- **System Bus:** All modules are topologically connected via a physical bus, simulating biological nerve pathways, and providing routing for inter-module communication, central perception, and control.

After simulated evolution concludes, individuals with high fitness are selected, and their morphological architectures are assembled into physical entities using real modules. Within the actual structure, the **gene expression starting from the initial node restarts in the real-world environment**.

- **Physical Awakening:** Initially, although the assembled body has physical connections, it is completely paralyzed logically. As the CPPN expresses within the central modules, the neural network begins to redevelop based on the environment, the topology continuously expands, and nerves gradually enter each module along the physical bus, establishing connections with the internal MCUs. The modules are awakened and integrated into the overall network. In the real-world environment, the phenotype can interact with the actual environment, generating corresponding short-term memories and stress-response habits.
- **Realization of Virtual Individuals:** The phenotypic parameters from the virtual environment can be directly flashed into the MCUs of each module, allowing the physical individual to rapidly acquire the identical capabilities it had in the virtual environment, serving as a foundation for further adaptation and training.
- **Implicit Physical Reservoir Synergy:** This is a crucial extension generated during the embodiment phase. The flexible silicone medium wrapped around the bodily modules, with its hysteretic deformation and damping attenuation during environmental interaction, naturally forms a high-dimensional, non-linear "physical reservoir"$^\textsf{[5]}$. Although this complex soft dynamics is not computationally modeled during the simulation phase, because the physical distributed MCU network possesses the characteristic of "secondary growth" in response to environmental feedback, the neural network spontaneously treats this physical skin as an excellent morphological pre-processor during its real-world development$^\textsf{[6]}$. The network implicitly learns to extract the features of the compliant medium, thereby achieving a synergy between transient physical computation and upper-level long-term memory.

# 5. Verification Objectives and Evaluation Metrics

To verify the effectiveness and embodied capabilities of the evolutionary system, the following evaluation metrics are established:

1. **Phenotypic Diversity:** Testing the degree of variance in the final spatial graph structures developed from the same genotype when inputted with different environmental information.
2. **Functional Robustness:** Testing the recovery rate of the system's core survival functions (such as locomotion capability) after random deletion of network nodes or alteration of physical parameters in morphological modules.
3. **Sim-to-Real Behavioral Consistency:** Comparing the behavioral patterns of the physical robot with its simulated counterpart, testing reaction mechanisms by applying the same types of external stimuli.
4. **Embodied Environmental Adaptability:** Evaluating whether the physical robot, once deployed in the real world, can rely on dynamic phenotypic expression to evolve new reflexes or adaptive behaviors when facing previously unencountered stimuli (such as varying ground friction or human interference).
5. **Spontaneity:** Observing whether the individual can still generate certain behaviors when external inputs are artificially cut off.

# 6. Research Strategy

The ultimate goal of this research is to establish a life architecture that possesses both morphological computational capability and neural network expressiveness. By mapping physical constraints from the real world into the organism's construction pathways, the morphological features and neural behaviors of artificial life become better suited to real environments, providing a viable solution for its embodiment in the real world. 

The anticipated **core deliverables** of this research are defined as:

- **Morphological-neural polymorphic co-development mechanism based on spatial dynamic graphs and behavioral verification.**
- **Morphological computation and artificial life embodiment verification based on distributed electromechanical networks.**

The preliminary expected research timeline is as follows:

- **Months 1-6 (Algorithm and Data Structure Development):** Complete the underlying data structure construction for the spatial dynamic graphs. Program the functional implementations of the genotype and phenotype.
- **Months 7-12 (Virtual Evolution and Feature Observation):** Design the evolutionary environment, conduct simulated evolution, and test the system's capability to generate artificial life topologies with stable structures and basic functions. Preliminarily evaluate the feature emergence and capabilities of artificial life in the virtual environment.
- **Months 13-18 (Distributed Electromechanical System R&D):** Design standardized bodily modules and implement the capability to establish cross-module networks via buses. Realize the expression process of the dynamic graphs in actual hardware modules.
- **Months 19-24 (Embodied Evaluation and Thesis Writing):** Re-run simulated evolution using actual module data, and manufacture the final physical artificial life based on the results. Conduct behavioral tests on the physical entity, compile Sim2Real comparison data, and complete the writing and defense of the graduation thesis.

# 7. References

[1] K. Sims, "Evolving 3D morphology and behavior by competition," *Artificial Life*, vol. 1, no. 4, pp. 353-372, 1994.

[2] K. O. Stanley, "Compositional pattern producing networks: A novel abstraction of development," *Genetic Programming and Evolvable Machines*, vol. 8, no. 2, pp. 131-162, 2007.

[3] A. Mordvintsev, E. Randazzo, E. Niklasson, and M. Levin, "Growing neural cellular automata," *Distill*, vol. 5, no. 2, p. e23, 2020. [Online]. Available: https://doi.org/10.23915/distill.00023

[4] S. Kriegman, D. Blackiston, M. Levin, and J. Bongard, "A scalable pipeline for designing reconfigurable organisms," *Proceedings of the National Academy of Sciences (PNAS)*, vol. 117, no. 4, pp. 1853-1859, 2020.

[5] K. Nakajima, "Physical reservoir computing—an introductory perspective," *Japanese Journal of Applied Physics*, vol. 59, no. 6, p. 060501, 2020.

[6] H. Hauser, A. J. Ijspeert, R. M. Füchslin, R. Pfeifer, and W. Maass, "Towards a theoretical foundation for morphological computation with compliant bodies," *Biological Cybernetics*, vol. 105, no. 5-6, pp. 355-370, 2011.
