* Tell me about a time when you faced a challenging problem at work. How did you solve it?

Situation: While building a distributed query system for Voltus simulations, we had to manage over 10B+ elements spread across partitions.

Task: I needed to design a scalable and intuitive way to access partitioned data, compatible with our Python interface.

Action: I introduced a relative-path syntax for cross-partition access and built a lazy-loading data layer to minimize memory usage. I also optimized communication protocols between master and workers.

Result: We significantly improved usability and performance, and the system scaled to real-world design sizes without bottlenecks.


* Tell me about a time when you worked on a team project. What was your role, and how did you contribute?

Situation: On the Lattice Boltzmann Solver project, we had to design a distributed engine for numerical computations.

Task: I led the engine partitioning and inter-task communication design.

Action: I defined task scheduling rules, implemented a message-passing system for boundary data, and helped align everyone on a shared abstraction for dependencies.

Result: The project achieved scalable performance, and my design became the foundation for follow-up simulation engines.

* Tell me about a time when you had to meet a tight deadline. How did you manage it?
   (work under pressure)

Situation: Existed GUI lacks query system, Before the demo for the new query system to customer, we need a working python interface to our new HPC data format.

Task: I was responsible for delivering a working prototype in just under two weeks.

Action: I quickly scoped down features to focus on core functionality, wrote Python bindings using Pybind11, and created clean documentation to support user testing.

Result: We successfully demonstrated the prototype, received positive feedback, and the customer's saying was "that exactly what we want!"

* Tell me about a time when you disagreed with a colleague or manager. How did you handle it?
  Tell me about a time when you improved a process or system.(existed interface is a messy and redundant)
  Tell me about a time when you successfully persuaded someone to see things your way.()

Situation: When our team tried to deliver the python interface for our HPC data format, a colleagure proposed to wrap all possible C++ data type into python new data types which is most straightforward, while I want to hide the value type of the data object and construct new data type through argument rather than 
naming it explicitly.

Task: I need to convince the colleague, also the team to adopt this complex implementation, while leaving clean interface for python user.

Action: I create intermediate class with shared pointer to hide value type through virtual function call, also I include another shared pointer to operation to register the value type for future casting from python data type to internal C++ data type.

Result: Eventually, this python binding interface is well structured without sacrificing too much efficiency, also provide a really clean interface for the python users. Get good feedback from both of the customer and internal group.

* Tell me about a time when you failed at something. What did you learn from the experience?
  (work across team communication)

Situation: I once tried to optimize a memory pool too early without understanding upstream access patterns. To avoid the memory fragment, I allow the small memory block request to occupy the large block. While this let following large memory block has no big enough block, then need to allocate one more large block from OS, and this results in more memory usage.

Task: Improve memory usage — but I ended up introducing more usage instead.

Action: After rolling back, I communicated with the upstream team to understand their memory request patterns, then redesigned the memory pool allocation policy. Several memory block queues are stored within the memory pool based on the memory block size. The large queue is specified for the large memory block request, while allowing temporary small memory block to occupy only when this small memory is released back to memory pool before the next time of the large memory block request.

Result: The revised solution improved memory usage, which is critical for the GPU run. It taught me the importance of not optimizing in isolation.

* Tell me about a time when you took the lead on a project. What was the outcome?

Situation: At Cadence, I initiated the AuData project — an HPC data storage system with schema support.

Task: Build a performant and extensible system that worked well with both Python and C++.

Action: I designed the schema abstraction, implemented serialization, created the Python bindings, and coordinated across teams.

Result: The system was adopted by multiple groups internally and received praise for its usability and performance.

* Tell me about a time when you received constructive criticism(bad feedback). How did you respond?

Situation: A senior engineer once commented that my documentation lacked user perspective.

Task: Improve clarity for others using my module.

Action: I asked follow-up questions to better understand their concerns, then rewrote the documentation with real use-case examples and better inline comments.

Result: The revised documentation got much more positive feedback, and I’ve since made user-focused documentation a habit.



* Tell me about a time when you had to manage multiple priorities. How did you stay organized?

Situation: I was juggling bug fixes for AuData reported by other group, documentation for the usage of AuData and preparing for a demo to customer.

Action: I prioritized tasks by impact and deadlines, blocked focused hours for deep work, and communicated clearly with users about trade-offs. 

Result: All deliverables were completed on time, and the demo helped secure further investment in the Voltus query work.

* Tell me about a time when you had to deal with a difficult team member.
Situation: A collaborator was not responding to code review requests and was blocking progress.

Action: I reached out privately, listened to their concerns, and learned they were overwhelmed. We rebalanced the task load and set clearer deadlines.

Result: They became much more engaged, and we completed the project smoothly.

* Tell me about a time when you had to adapt to a significant change at work.
  Tell me about a time when you had to learn a new skill quickly. How did you go about it?(learn CUDA and SIMD instruction)
  
Situation: Our team decided to add GPU support for AuData module originally written for CPUs.

Task: Port the module for GPU without sacrificing performance or flexibility.

Action: I abstract platform differences(ABI design), restructured memory handling logic(Underlying memory allocation, storage class) and abstract operation(CPU loop to SIMD kernel and GPU kernel) I validated performance across multiple backends.

Result: The module ran efficiently on both GPU and CPU(including SIMD)

* Tell me about a time when you went above and beyond for a customer or client.
  
Situation: A partner team needed to evaluate our HPC AuData library under a tight deadline.

Task: Deliver a minimal, tailored Python interface for their testing use-case.

Action: I refactored core logic into a clean module, wrote a user-friendly python demo, and made myself available after hours to answer their questions.

Result: They integrated our system successfully and hit their deadline. Later, they became long-term collaborators on the project.


