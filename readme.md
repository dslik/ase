# 1. ASE Overview

ASE (Asynchronous Systems Environment) is a runtime environment for creating distributed systems. It specifies a minimal approach for state management and code execution. There is nothing unique or innovative about ASE itself, rather, it is a curated collection of key ideas from the field of computer science that, when combined, reduces system complexity.

This work is based on earlier research conducted at the University of Victoria and published in the MacHack ‘98 Conference Proceedings as "The ADE Environment".

# 2. Foundation - The actor model of computation

## 2.1 State transitions on events

Hewitt et al teaches the generalization of computing into sequences of states, and an "actor" model where local states transition from state S<sub>n</sub> to state S<sub>n+1</sub> as a result of receiving event E<sub>n</sub>:

> "In a serial model of computation, computations are sequences of states, and each state in the sequence determines a next state for the computation by considering the program text in a Von Neumann computer or the specifications for the finite state control of an automaton in computability theory. In the actor model, we generalize the notion of a computation from that of a sequence of global states of a system to that of a partial order of events in the system, where each event is the transition from one local state to another." [1]

This can be denoted as E<sub>n</sub> -> S<sub>n</sub> [ -> S<sub>n+1</sub> ].

![image](https://github.com/dslik/ase/assets/5757591/338e2a1a-849f-460d-8e9a-49e5b918683c)

Figure 1 illustrates an existing state S<sub>n</sub> receiving an event E<sub>n</sub> that does not result in any changes to state S<sub>n</sub>.

Figure 2 illustrates an existing state S<sub>n</sub> receiving an event E<sub>n</sub> that results in changes from state S<sub>n</sub> to S<sub>n+1</sub>.

## 2.2 Events generating additional events

One potential result of an actor at state S<sub>n</sub> receiving an event E<sub>n</sub> is the sending of additional messages (events) E<sub>n+1</sub> through E<sub>n+m</sub>:

> "We present axioms for actor systems which restrict and define the causal and incidental relations between events in a computation, where each event consists of the receipt of a message by an actor, which results in the sending of other messages to other actors." [1]

This can be denoted as E<sub>n</sub> -> S<sub>n</sub> -> [ E<sub>n+1</sub> ... E<sub>n+m</sub> ]

![image](https://github.com/dslik/ase/assets/5757591/62d918f3-c22f-4533-b5ec-15380144f216)

Figure 3 illustrates an existing state S<sub>n</sub> receiving an event E<sub>n</sub> that results in the creation of two additional events, E<sub>n+1</sub> and E<sub>n+2</sub>.

## 2.3 Events creating new actors

A new actor S<sub>m</sub> can also be created at runtime, by sending a specific type of "creation" event:

> "Every actor is either one of a finite number of initial actors, or is created during the course of a computation. Every created actor is created in a unique event called that actor's creation event." [1]

This can be denoted as E<sub>n</sub> -> S<sub>n</sub> -> [ EC<sub>n</sub> -> S<sub>m</sub> ]

![image](https://github.com/dslik/ase/assets/5757591/1c75507e-21cb-4ba1-b447-eec5d83ab81a)

Figure 4 illustrates an existing state S<sub>n</sub> receiving an event E<sub>n</sub> that results in a creation event E<sub>f</sub>, which creates a new state S<sub>m</sub>.

## 2.4 Named actors

Actors can have names, and that actors' names can be communicated as part of events, to allow further events to be sent to that actor:

> "Hence, actor messages must be able to convey the names of other actors as well as data so that new actors may be introduced to old." [1]

## 2.5 Communication by names

Finally, communication can only occur when a sender knows the name of the receiver:

> "The most fundamental form of knowledge which is conveyed by a message in an actor computation is knowledge about the existence of another actor. This is because an actor A may send a message to another actor B only if it "knows about" B, i.e. knows B's name. However, an actor cannot know an actor's name unless it was either created with that knowledge or acquired it as a result of receiving a message. In addition, an actor cannot send a message to another actor conveying names he does not know." [1]

This use of names is now known as a capabilities-based system:

> "Conceptually, a capability is a token, ticket, or key that gives the possessor permission to access an entity or object in a computer system." [2]

# 3. Foundation - The fork-join model of parallel computation

## 3.1 Fork-join as subset of actor model

Hewitt et al later teaches that the fork-join model is a subset of the actor model of computation :

> "We shall say that E<sub>1</sub> is a fork event and that E<sub>6</sub> is a join event. In the above computational it will necessarily be the case that E<sub>1</sub> -act-> E<sub>6</sub> since this is the only way that E<sub>6</sub> can be activated. Therefore it will be the case that either E<sub>4</sub> -act-> E<sub>6</sub> or E<sub>5</sub> -act-> E<sub>6</sub>. The continuation ordering -cont-> enables us to present the history of the computational without having to be concerned as to which of the above possibilities actually occurred. Using the continuation ordering the symmetry of the above fork-join computation is demonstrated by the fact that the continuation ordering is the same for both of the above histories." [3]

## 3.2 Definition of fork and join

Conway teaches the concept of a "fork", where program execution branches and both branches are executed concurrently:

> "FORK is simply an instruction with two successors. It is written and acts like a branch instruction. However, if location 100 contains a FORK 200 instruction, then instructions at 200 and at 101 will be subsequently executed." [4]

Figure 4 above can be viewed as a fork, where instructions in state S<sub>m</sub> are executed concurrently with instructions in state S<sub>n</sub>.

Likewise, "join" takes two execution branches and merges them into one:

> "The JOIN, which is, in effect, the reverse of the FORK, has a vital additional job: it waits. In Figure 1, box C must not be begun until boxes A and B are completed." [4]

![image](https://github.com/dslik/ase/assets/5757591/aadf8e75-e5d4-4e5b-b54a-f8edeeb43ac0)

Figure 5 illustrates two existing states S<sub>n</sub>, and S<sub>m</sub>, where a join event E<sub>j</sub> results in the merging of S<sub>m</sub> into S<sub>n</sub>, creating a new state S<sub>n+1</sub>.

## 3.3 Private state for forks

Conway also teaches that private state should be allocated to the lifespan of a fork:

> "Indeed, if private storage is required, it belongs not to each processor but to each parallel flowchart path. The distinction between processors and paths is a crucial one; confusion over this matter seems to be muddying up much contemporary thinking about parallel processing." [4]

## 3.4 Fork-join as a superset of interrupts

Finally, Conway teaches that interrupts can be replaced with forks and joins:

> "In fact, external interrupts are unnecessary. Consider that an I/O instruction is simply a very lengthly one which may be executed in parallel with other code, such as computation and editing. Then simply precede it with a FORK and follow it with a JOIN, and all the functions of external interrupts are accounted for in a much more elegant manner." [4]

# 4. References

- [1] "Laws for Communicating Parallel Processes", Hewitt and Baker, 1977
- [2] "Capability-Based Computer Systems", Henrey M. Levy, 1984
- [3] "Actors and Continuous Functionals", Hewitt and Baker, 1977
- [4] "A Multiprocessor System Design,", Melvin E. Conway, 1963
