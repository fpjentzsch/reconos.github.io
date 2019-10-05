---
title: Projects & Publications
layout: page
---
todo: add/remove some projects, update publications
# Projects

## Epics

<img src="{{ site.url }}/assets/images/logo_epics_color.png" alt="" align="right" width="260" style="padding: 5px 5px 5px 20px">
Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.

## CRC 901 - On-The-Fly Computing

<img src="{{ site.url }}/assets/images/logo_sfb901.png" alt="" align="right" width="260" style="padding: 5px 5px 5px 20px">
Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.

## Dynamic partial reconfiguration

ReconOS defines a standardized interface for hardware threads, which simplifies exchanging them, not only at design time but also during runtime using dynamic partial reconfiguration (DPR). DPR allows for exploiting FPGA resources in unconventional ways, for example, by loading hardware threads on demand, moving functionality between software and hardware, or even multi-tasking hardware slots by time-multiplexing. ReconOS supports DPR by dividing the architecture in a static and a dynamic part. The static part contains the processor, the memory subsystem, OSIFs, MEMIFs, and peripherals. The dynamic part is reserved for hardware threads, which can be reconfigured into the hardware slots. Our DPR tool flow builds on Xilinx PlanAhead and creates the static subsystem and the partial bitstreams for each desired hardware thread/slot combination. Time-multiplexing of hardware slots is supported through cooperative multi-tasking.


## Adaptive network architectures

Researchers at ETH Zurich use ReconOS to implement adaptive network architectures that continuously optimize the network protocol stack on a per-application basis to cope with varying transmission characteristics, security requirements, and compute resources availability. The developed architecture autonomously adapts itself by offloading performance-critical, network processing tasks to hardware threads, which are loaded at runtime using dynamic partial reconfiguration.


## Self-adaptive and self-aware computing systems

Another line of research also leverages the unified software/hardware interface and partial reconfiguration to create self-adaptive and self-aware computing systems that autonomously optimize performance goals under varying workloads. In the EPiCS project funded by the European Commission, researchers of the University of Paderborn even advance the autonomy of computing systems and enable them to optimize for diverse goals such as performance, energy consumption, and chip temperature based on the current quality-of-service requirements, workload characteristics and system state.

<cite>ReconOS – an operating system approach for reconfigurable computing</cite>

# Publications

ReconOS has been developed in the context of several academic research projects since 2006. Below you find a comprehensive list of publications that describe selected aspects of ReconOS.

*Copyright Notice:* This material is presented to ensure timely dissemination of scholarly and technical work. Copyright and all rights therein are retained by authors or by other copyright holders. All persons copying this information are expected to adhere to the terms and constraints invoked by each author's copyright. In most cases, these works may not be reposted without the explicit permission of the copyright holder.

## Journal papers

* Andreas Agne, Markus Happe, Ariane Keller, Enno Lübbers, Bernhard Plattner, Christian Plessl and Marco Platzner. 
  **ReconOS – an Operating System Approach for Reconfigurable Computing**. 
  *IEEE Micro*, 2014. To appear.

* Markus Happe, Enno Lübbers and Marco Platzner. 
  **A Self-adaptive Heterogeneous Multi-core Architecture for Embedded Real-time Video Object Tracking.**
  *International Journal of Real-Time Image Processing*, Volume 8, Issue 1, pp 95-110, Springer Berlin / Heidelberg, 2013. 
  &#91;[PDF](happe13_jrtip.pdf)&#93;

* Enno Lübbers and Marco Platzner.
 **ReconOS: Multithreaded Programming for Reconfigurable Computers.**
 In *Transactions on Embedded Computing Systems (TECS)*, Volume 9 Issue 1, IEEE, 2009.
  &#91;[PDF](luebbers09_tecs.pdf)&#93;

## Refereed Conference Papers

* Markus Happe, Friedhelm Meyer auf der Heide, Peter Kling, Marco Platzner and Christian Plessl.
  **On-The-Fly Computing: A Novel Paradigm for Individualized IT Services (Invited Paper).**
  In *Proc. of the Int. Workshop on Software Technologies for Future Embedded and Ubiquitous Systems (SEUS)*, IEEE, June 2013.
  &#91;[PDF](happe13_seus.pdf)&#93;

*  Ariane Keller, Daniel Borkmann and Stephan Neuhaus 
  **Hardware Support for Dynamic Protocol Stacks**
  In *Proc. of the Symposium on Architectures for Networking and Communications Systems*, ACM/IEEE , October 2012.
  &#91;[PDF](keller12_ancs.pdf)&#93;

* Markus Happe, Andreas Agne, Christian Plessl and Marco Platzner.
  **Hardware/Software Platform for Self-Aware Compute Nodes.**
  In *FPL Workshop on Self-Awareness in Reconfigurable Computing Systems (SRCS)*, September 2012.
  &#91;[PDF](happe12_fpl.pdf)&#93;

* Markus Happe and Enno Lübbers.
  **A Hybrid Multi-Core Architecture for Real-Time Video Tracking.**
  In *FPL Workshop on Computer Vision on Low-Power Reconfigurable Architectures*, 2011.
  &#91;[PDF](happe11_fpl.pdf)&#93;

* Ariane Keller, Bernhard Plattner, Enno Lübbers, Marco Platzner, and Christian Plessl.
  **Reconfigurable nodes for future networks.**
  In *Proc. IEEE Globecom Workshop on Network of the Future (FutureNet)*, pages 372–376. IEEE, December 2010.
  &#91;[PDF](keller10_futurenet.pdf) | [BibTeX](keller10_futurenet.bib)&#93;

* Enno Lübbers, Marco Platzner, Christian Plessl, Ariane Keller, and Bernhard Plattner.
  **Towards adaptive networking for embedded devices based on reconfigurable hardware.**
  In *Proc. Int. Conf. on Engineering of Reconfigurable Systems and Algorithms (ERSA)*, pages 225–231. CSREA Press, July 2010.
  &#91;[PDF](keller10_ersa.pdf) | [BibTeX](keller10_ersa.bib)&#93;

* Markus Happe, Enno Lübbers and Marco Platzner.
  **An Adaptive Sequential Monte Carlo Framework with Runtime HW/SW Repartitioning.**
  In *Proc. Int. Conference on Field-Programmable Technology (FPT)*, Dec. 2009. IEEE.
  &#91;[PDF](happe09_fpt.pdf) | [BibTeX](happe09_fpt.bib) | [Talk](happe09_fpt_slides.pdf)&#93;

* Enno Lübbers and Marco Platzner.
  **Cooperative Multithreading in Dynamically Reconfigurable Systems.**
  In *Proc. Int. Conference on Field Programmable Logic and Applications (FPL)*, Aug/Sep 2009. IEEE.
  &#91;[PDF](luebbers09_fpl.pdf) | [BibTeX](luebbers09_fpl.bib) | [Poster](luebbers09_fpl_poster.pdf)&#93;

* Markus Happe, Enno Lübbers, and Marco Platzner.
  **A Multithreaded Framework for Sequential Monte Carlo Methods on CPU/FPGA Platforms.**
  In *Proc. Int. Workshop on Applied Reconfigurable Computing (ARC)*, Mar 2009. Springer. &#91;[PDF](happe09_arc.pdf) | [BibTeX](happe09_arc.bib)&#93;

* Enno Lübbers and Marco Platzner.
  **A Portable Abstraction Layer for Hardware Threads.**
  In *Proc. Int. Conference on Field Programmable Logic and Applications (FPL)*, Sep. 2008. IEEE.
  &#91;[PDF](luebbers08_fpl.pdf) | [BibTeX](luebbers08_fpl.bib) | [Talk](luebbers08_fpl_slides.pdf)&#93;

* Enno Lübbers and Marco Platzner.
  **Communication and Synchronization in Multithreaded Reconfigurable Computing Systems.**
  In *Proc. Int. Conference on Engineering of Reconfigurable Systems and Algorithms (ERSA)*, Jul 2008. CSREA Press.
  &#91;[PDF](luebbers08_ersa.pdf) | [BibTeX](luebbers08_ersa.bib) | [Talk](luebbers08_ersa_slides.pdf)&#93;

* Enno Lübbers and Marco Platzner.
  **ReconOS: An RTOS supporting Hard- and Software Threads.**
  In *Proc. Int. Conference on Field Programmable Logic and Applications (FPL)*, Aug 2007. IEEE.
  &#91;[PDF](luebbers07_fpl.pdf) | [BibTeX](luebbers07_fpl.bib) | [Talk](luebbers07_fpl_slides.pdf)&#93;


## Non-refereed Publications

* Christian Plessl, Marco Platzner, Andreas Agne, Markus Happe and Enno Lübbers.
  **Programming Models for Reconfigurable Heterogeneous Multi-cores.**
  *Awareness Magazine*, March 2012.
  &#91;[PDF](2012_plessl_awareness_magazine.pdf) | [BibTeX](2012_plessl_awareness_magazine.bib)&#93;

## Tutorials

* Markus Happe, Andreas Agne, Christian Plessl, and Marco Platzner.
  **Getting started with ReconOS.**
  Tutorial at the Int. Conference on Field Programmable Logic and Applications (FPL), Oslo Norway, 2012
