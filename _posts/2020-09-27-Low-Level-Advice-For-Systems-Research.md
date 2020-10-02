---
layout: post
title: ! 'Low-level advice for Systems PhD students'
type: post
---


There's no shortage of "how to do research advice" on the Internet for graduate students. 
Such advice, while inspiring, is extremely hard to translate into a means
of staying productive as a systems PhD student on a daily or weekly basis. 

I think this unfortunate, because I believe a lot of the stress associated with
systems research can be offset by following good practices and methodology. Having
found myself discussing this topic often with students, I think it's time to blog
about this topic.

I'll cover two broad topics: how to effectively prototype artifacts and how to
be systematic about experimentation. 

On the prototyping side, we'll cover
the *tracer bullet methodology* to gather incremental feedback about an idea,
and the importance of systematically testing your
code. 

On the topic of experiments, I'll argue for setting up
your experiment infrastructure as early as possible in a project, share some tips on
automating your experiments, avoiding tunnel vision, how to make your system experimentation-friendly, and end with
 some guidelines on how to understand the results of your experiments.  

<br>

### Disclaimer

As with any advice from people who landed research positions after
their PhDs, remember that survivor bias is a thing. I can only say that
following these guidelines work well for me, there is no guarantee that it
will work for you.

<br>

### Who is this article for?

This article might help you if you're a systems PhD student, and one or more of the following bullets 
resonate with you:

* You're concerned that you're spending a lot of time building your system, with little idea of whether
  it will be worth it at all.
* There are many components of your system left to implement, and you're not sure what
  you should prioritize over others.
* You find setting up, understanding and debugging experiments overwhelming.

<br>

### The premise

The premise for this article is the following comic I drew a while ago.

<p align="center">
<img src="http://lalith.in/img/codequality.png" alt="drawing" width="500"/>
</p>

You have a finite amount of time, there's a lot of systems work to do, there's a submission deadline looming, 
and that worries you. As you get closer to the deadline, things will inevitably get chaotic.
This article will hopefully help you effectively plan and
alleviate some of that chaos.

<br>

### Effective Prototyping

Your research prototype's goal is to exercise a certain hypothesis. 
To validate
that hypothesis, you will eventually subject your prototype to a set of experiments.
More often than not, the experiments will compare your system against one or more baseline
systems too.

Your goal through this process is to:
* prototype in a way that you get continuous, incremental feedback about your idea
* test for correctness during development time, so you don't have to during experiments;
  debugging performance problems and system behavior over an experiment is hard enough
* build the experiment setup and infrastructure as early as possible, and preferably
  even before you build the system (rather than building the system first, and 
  *then* thinking about experiments)
* Save time by automating the living crap out of your experiment workflow 

<br>

#### The Tracer Bullet Methodology

I picked up this term from [The Pragmatic Programmer](https://www.amazon.com/Pragmatic-Programmer-Journeyman-Master/dp/020161622X)
 book.

Here's the problem. Let's say you have multiple components to build before your system will work end-to-end. 
Each of these components might be complex by themselves, so building them one after the other in sequence and stitching 
them together might take a lot of time. That's bad if you're not even sure if this system is worth building:
you don't want to spend a year building a system, only to find out you've hit a dud.

Instead, focus on getting a working end-to-end version early that might even only work
for one example input, taking as many shortcuts as you need to get there 
(e.g., an interpreter that can only compile the program "1 + 1" and nothing else). 
From there evolve the system to work with increasingly complex inputs, filling in gaps in your skeleton code
and eating away at hard coded assumptions and shortcuts as you go along.
The benefit of this approach is that you always see the system work end-to-end, for more and more
examples, and you receive continuous, incremental feedback on a variety of factors throughout the process.
This strategy applies at any granularity of your code: the entire system
itself, specific modules within them, or even specific functions.

For example, in the [DCM](github.com/vmware/declarative-cluster-management) project, 
our hypothesis was that rather than having developers hand-craft fragile heuristics for cluster management
problems, a compiler could synthesize the required implementation from an SQL specification. Doing so would make it
easier to build cluster managers and schedulers that perform well, are flexible, and compute high-quality decisions.
  
When we started working on DCM, we weren't even sure if this idea was practical and whether SQL was an expressive
enough language for the task-at-hand. Building a fully functional compiler that generated fast-enough code 
would have taken a while, and we wanted to validate the core hypothesis early. So to get started, we took a cluster 
management system, expressed some of its logic in SQL, and had the compiler emit code that we'd mostly written by hand. 
We tried this for several cluster management policies, and received steady feedback about the feasibility
 of our design along the way. Importantly, even before we had filled in the compiler's gaps, we had a working
 end-to-end demo running within a real system, which gave us the required confidence to double down on the idea.

<br>
 
#### Testing

Test your code thoroughly before you run experiments. The additional time you invest
in writing tests and running them on every build, will be more than made up by *not* having to debug your system for
correctness when you run experiments. Debugging performance issues and system behavior during
an experiment is daunting enough as it is: you don't want to compound this by debugging correctness issues
at the same time. 

When working on a system with multiple authors, tests are essential to slowing down the rate at 
which bugs and regressions creep in over time. When you find a new bug in your system, add a test case that reproduces
it and add it to the test suite. 

Only use a given commit/build of your system for an experiment if it has passed all tests.
 
Test every build and commit using a continuous integration (CI) infrastructure.
A CI system monitors a repository for new commits and runs a series of prescribed test workflows. 
There are free solutions like [Travis](https://travis-ci.org/) you can easily use, or you can also host your own
infrastructure using [Gitlab CI](https://docs.gitlab.com/ee/ci/). 

A common pushback I've heard about being rigorous with testing research prototypes is 
"Hey! We're trying to write a paper, not production code!". That statement however, is non sequitur. 
A research prototype with tests is
not a production system! I'm not suggesting that you add metrics, log retention, compliance checks, rolling upgrades,
backwards compatibility and umpteen other things that production systems need (unless they're relevant
to your research, obviously).

Use any automated tooling available to harden your code. For example, I configure all my Java projects to use
 [Checkstyle](https://github.com/checkstyle/checkstyle) to enforce a coding style, and both 
 [SpotBugs](https://github.com/spotbugs/spotbugs) and [Google ErrorProne](https://errorprone.info/)
 to run a suite of static analysis passes on the code. I use [Jacoco](https://github.com/jacoco/jacoco) and
  tools like [CodeCov](https://codecov.io) for tracking code coverage of my tests. 
  Every build invokes these tools and fails the build if there are problems. The 
 CI infrastructure also runs these checks whenever I push commits to a git repository. 

<br>

### Experiments

A common trap that students fall into is to not think about experiments until their system is "ready". 
This leads to the same problem listed earlier about not getting incremental feedback.

<br>

#### Set up the experiment infrastructure early

Most systems papers often have one or more baselines to compare against. If so, set up the experiment infrastructure
to measure the baselines as early as possible, even *before* you start formulating a hypothesis
about something you'd like to improve. Use the data collected from studying the baselines to formulate
your hypothesis. 

From my own experience, you're likely to find problems others haven't noticed
themselves (you'd be surprised at how flimsy prior art can be sometimes). You'll also learn whether what
you think is a problem is even real at all. Either way, you'll get valuable ammunition towards formulating 
a good hypothesis.

<br>

#### Automate your experiments like a maniac

The main goal for your experiment setup should be to completely automate the entire workflow. And I mean
**the entire workflow**. For example, have a single script that given a commit ID from your system's git repository,
 checks out that commit ID, builds the artifacts, sets up or refreshes the infrastructure, 
 runs experiments with all the parameter combinations for the system and the baselines, cleans up between runs, 
 downloads all the relevant logs into an archive with the necessary metadata, processes the logs, 
 generates a report with the plots and sends that report to a Slack channel.

When running experiments, make sure to not only collect log data and traces, but also the experiment's metadata. 
Common bits of metadata that I tend to record include a timestamp for when I triggered the experiment
, the timestamps for the individual experiment runs and repetitions, the relevant git commit IDs, metadata
about the environment (e.g., VM sizes, cluster information) and all the parameter combinations that were run. 
Save all the metadatato a file in a machine-friendly format (like CSV or JSON). 
Avoid encoding experiment metadata into file or folder names ("experiment_1_param1_param2_param3"). This makes it hard
to introduce changes over time (e.g. adding a new parameter will likely break your workflow).
Propagate the experiment metadata all the way to your graphs, to be sure you know what commit ID and parameter
 combinations produced a given set of results. Aggressively pepper the workflow with sanity checks (e.g,
 a graph should never present data obtained from two different commits of your system).
 
Never, ever, configure your infrastructure manually (e.g., bringing up VMs on AWS using the EC2 GUI, followed by logging
 into the VMs and
installing software libraries and configuring them). Instead, always script these workflows up 
(I like using [ansible](https://www.ansible.com/) for such tasks).
The reason being, that accumulating ad-hoc tweaks and commands with side-effects (like changing the OS configuration) 
impairs reproducibility. Instead, be disciplined about only introducing changes to the infrastructure via a set of well 
maintained scripts. They come in handy especially when unexpected failures happens and you need to migrate to a new
infrastructure (almost every project I've worked on had to go through this!).
 
The sooner you start with the above workflows, the better. Once it's up, you'll soon hit a point
where just like your tests, you're able to continuously run experiments as part of your development process, to
make sure your system is on track as far as your evaluation goals go. Every change you make gives you a report
full of data about how that change affects various metrics of concern for your system. They become part of 
your testing and regression suites.  
 
The tracer bullet methodology applies to your experiment setup just as much as it does to the system you're
building. You'll find your experiment setup evolving in tandem with your system: you'll add new logging and 
tracing points, you'll add new parameter combinations you want to test, and new baselines.

I have a fairly standard workflow I use for every project. It starts with using 
[ansible](https://www.ansible.com/) to set up the infrastructure (e.g., bring up and configure VMs on EC2), 
deploy the artifacts, run the experiments, and collect the logs. I use python to parse the raw logs and produce 
an [SQLite](https://sqlite.org/index.html) database with the necessary traces and experiment metadata. 
I then use [R](https://www.r-project.org/) to analyze the 
traces, [ggplot](https://ggplot2.tidyverse.org/) for plotting, and use [RMarkdown](https://rmarkdown.rstudio.com/) to 
produce a report that I can then send to a Slack channel. All of these steps are chained together using a single 
top-level bash script.

<br>

#### Avoiding tunnel-vision 

A common risk with system building is to fall into the experiment tunnel-vision trap: 
for example, frantically trying
to improve performance for a benchmark or metric that may not matter, while ignoring those that are
important to assess your system. 

There are two things you can do to avoid this trap.

First, try to sketch out the first few paragraphs of your paper's evaluation section as early as possible.
This will make you think carefully about the main theses you'd like your evaluation to support.
I've always that this simple trick helps me stay focused when planning experiments. You'll find yourself
refining both the text for the evaluation section and the experiment workflow over time.

The second  is to use what I recommended in the previous section: wait to see a report with data about
 all your experiments before iterating on a change to your system. Otherwise, you'll be playing experiment
  whack-a-mole for a while! Make sure your report contains the
 same graphs and data that you plan to add to the evaluation section. 

<br>

#### Experimentation-friendly prototyping

Build your prototype to be experimentation-friendly. Always use feature flags and configuration parameters 
for your system to toggle different settings (including log levels). If you find yourself modifying and recompiling 
your code only to enable/disable a certain feature, you're doing it **completely wrong**.
 
 Here's one scenario where feature flags come in handy. Often, a paper proposes a collection of techniques that when 
 combined, improve over the state-of-the-art (e.g., five new optimization passes in a compiler). 
 If your paper fits that description, always make sure to have
  the quintessential "look how X changes as we turn on
 these features, one after the other" experiment. If you only test the combination of these features, and not their
  individual contributions, you (and the readers) won't know if some of your features negatively impact your system! 

And no, for the majority of systems projects, a few additional `if` statements are hardly going to affect your 
performance (actual pushback I've seen!). 
If production-ready JVMs can ship with hundreds of such flags, your research prototype can too.

<br>

#### Measure, measure, measure

Systems are unfortunately way too complex.
If you want to understand what happened over an experiment, there is no substitute to measuring aggressively. 
Log any data that you think will help you understand what's going on, even if it is data that you will not present in a 
paper. If in doubt, always over-measure rather than under-measure.

To borrow a quote from [John Ousterhout](https://web.stanford.edu/~ouster/cgi-bin/sayings.php): 
"Use your intuition to ask questions, not answer them".   
Often, when faced with a performance problem,
it's tempting to assume you know why (your intuition) and introduce changes to fix that problem. Don't!
Treat your intuition as a hypothesis, and dig in further to confirm exactly why a certain performance
problem occurs. For example, expand your workflow with more logs or set up additional experiments to control
for the suspected factors.

A particularly dangerous example I've seen is to declare victory the moment one sees their system beat the baseline 
in end-to-end metrics. For example, "we have 800x higher transaction throughput than baseline Y" 
(ratios that are quite fashionable these days!).
Again, don't assume, but follow up with the required lower-level analysis to confirm you can
 thoroughly explain *why* the performance disparity exists. 
Some basic questions you can ask your data, based on unfortunate examples I've encountered in the wild: 
* are you sure your database is not faster on reads because all reads return `null`?
* does your "Big Data" working set fit in an L3 cache? 
* is your system dropping requests whereas the baseline isn't?
* are you measuring latency differently for the baseline vs your system (round-trip vs one-way)? 
* are you introducing competition side effects [1] (section 1.5.3)?
* are the baselines actually faster than your system, but you've driven them to congestion collapse?
 
There are several aspects of measurement methodology that I think every systems student should know. For example,
the relationship between latency and throughput, 
open vs closed loop workload generation, how to understand bottlenecks, 
how to summarize performance data, the use of confidence intervals, and much more. 
I highlight recommend these books and articles to learn more:

[1] ["Performance Evaluation of Computer and Communication Systems"](https://perfeval.epfl.ch/), Jean-Yves Le Boudec.  
[2] ["The Art of Computer Systems Performance Analysis: Techniques for Experimental Design, Measurement, Simulation, and Modeling"](https://www.cse.wustl.edu/~jain/books/perfbook.htm), Raj Jain.  
[3] ["Mathematical foundations of computer networking"](https://www.informit.com/store/mathematical-foundations-of-computer-networking-9780321792105), Srinivasan Keshav.  
[4] ["Systems Benchmarking Crimes"](http://gernot-heiser.org/benchmarking-crimes.html), Gernot Heiser