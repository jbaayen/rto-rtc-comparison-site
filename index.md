## by Ties van der Heijden (HKV, TU Delft) and Jorn Baayen (KISTERS)

<a href="https://tudelft.nl"><img src="images/TUDelft.svg" height="35"></a>
&nbsp;&nbsp;
<a href="https://water.kisters.de/en"><img src="images/KISTERS.svg" height="20"></a>

### Introduction

KISTERS RTO and RTC-Tools 2 both use [homotopy continuation](https://arxiv.org/abs/1801.06507) to find optimal
control strategies for non-linear hydraulic systems.  We were curious to see how the two implementations stack up
against each other.

### Technical comparison

We begin by presenting a summary overview comparing the primary characteristics of the two optimization
solutions:

|                                  | RTC-Tools 2.3.2           | KISTERS RTO  |
| -------------------------------- |:--------------|:------|
| Modelling paradigm               | Modelica | KISTERS Network Store |
| Optimization solver              | IPOPT inside continuation loop     |  KISTERS Gorilla |
| Parallelization on CPU | Basic | Extensive (*patent pending*) |
| Parallelization on GPU | No | Optional (*patent pending*) |
| Globally optimal solutions       | No                                      | Yes |
| Globally optimal goal programming          | No                                                                                    |   Yes (*patent pending*) |
| Convergence guarantees           | No (*bound violations resulting in infeasibilities, IPOPT restoration phase failures*) | Yes  |
| Open source   | Yes | No |


### Benchmark setup

We consider a single river reach, with a fixed upstream inflow boundary condition, and an adjustable downstream boundary condition.  The single river reach is discretized using *n* level nodes.  The number of level nodes is varied on a binary scale between 16 and 256 in this benchmark.

<div align="center">
<img src="images/grid.png" height="100px">
</div>

The characteristics of the reach and the boundary conditions are identical to those in a [previous benchmark](https://publicwiki.deltares.nl/download/attachments/138543226/Baayen_2019-09-13%20Comparison%20Optimization%20Methods.pdf?version=1&modificationDate=1571401624947&api=v2).

The benchmark problem is single-objective.  The objective is to keep the up- and downstream water levels as close as possible to a reference water level of 0 m above datum.

In order to make a fair comparison, GPU acceleration features of RTO / Gorilla were disabled for this benchmark.

### Results

If we plot objective function values against the number of water level discretization points, 
we see straight away that RTC-Tools 2.3.2 does *not* find global optima (lower values are better):

<div align="center">
<img src="images/perf.svg">
</div>

This most likely due to the fact that the RTC-Tools Channel Flow [shallow water discretization](https://gitlab.com/deltares/rtc-tools-channel-flow/-/blob/29906a7f7eb76edabd8d3d9b068374dc0de84a55/src/rtctools_channel_flow/modelica/Deltares/ChannelFlow/Hydraulic/Branches/Internal/PartialHomotopic.mo) is not synchronized with the [theory](https://arxiv.org/abs/1801.06507).
Because of this, constraints may become linearly dependent, at which point regularization heuristics are activated in [IPOPT](https://github.com/coin-or/Ipopt).

Secondly, we note that RTO consistently outperforms RTC-Tools 2 in terms of computation time:

<div align="center">
<img src="images/wall_time.svg">
</div>

For realistic problems, RTO is more than **10 times faster**  than RTC-Tools 2 (note the logarithmic plot axes).   This large difference is mostly due to the use of the KISTERS Gorilla non-convex optimization solver.

### Conclusions

As of this writing, KISTERS RTO is vastly superior to RTC-Tools 2 both in terms of computation time, and in terms
of optimality performance.  The superior solution quality coming from RTO is a consequence of a careful implementation of the [path-stable](https://arxiv.org/abs/1801.06507) shallow water discretization.  The superior solution times are largely due to the use of the highly parallelized KISTERS Gorilla NLP solver.

RTC-Tools 2 could be improved if its shallow water discretizations were to be brought in sync with the theoretical results on global optimality, and would benefit in terms of computation time if IPOPT were to be replaced with KISTERS Gorilla as its underlying optimization engine.

