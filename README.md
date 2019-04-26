# Totem
A graph processing engine for hybrid CPU and GPU platforms
Totem – A Beginner’s Guide
 
[Intended audience:  the main audience for this document are the members of the Totem project at UBC.  Others might find it useful as well].
 
# Introduction

This document will guide you to get started with the Totem framework. Totem is a graph-processing framework that leverages CPU-GPU hybrid platforms. At the time of writing, the Totem framework allows you to write graph processing algorithms, e.g., Single Source Shortest Path, in a shared-memory environment. You can write an algorithm that utilizes either the CPU or the GPU or both. The framework takes care of tasks like reading the input graph from a file and memory management and includes a unit testing framework.
 
# Setting up the Workspace

The development environment for Totem is Linux and the workstation is required to be equipped with at least one CUDA supported GPU. NetSysLab cluster has several GPU equipped nodes (ws020, node240 and node250). You can read about how to access a cluster node here. These nodes have CUDA Toolkit (SDK, compiler, runtime environment, utilities etc.) already installed on them. You are only required to checkout the Totem framework source from the GitHub (https://github.com/netsyslab/Totem) to your workspace. At this stage, you need not to worry about defining any environment variable.
 
# Compiling Totem

### Requirements:
- 1）gcc and g++ 4.7.x or higher (basically c++11 support).
- 2）CUDA toolkit: the make files expects the toolkit to be installed under “/usr/local/cuda”. If the toolkit is installed somewhere else, then just modify the “CUDA_INSTALL_PATH” variable in the make.defs file to point to the toolkit path.
- 3）TBB (Intel Threading Building Blocks):
On ubuntu, it can be installed via apt-get as follows: “sudo apt-get install libtbb-dev”
On Fedora, it can be installed via yum as follows: “sudo yum install tbb-devel”
On the Jetson TK1 board running L4T R21.1, the repository which contains libtbb-dev is disabled by default.  To enable the repository, uncomment the following two lines in /etc/apt/sources.list:
 deb http://ports.ubuntu.com/ubuntu-ports/ trusty universe
 deb-src http://ports.ubuntu.com/ubuntu-ports/ trusty universe
After enabling the repository, run “sudo apt-get install libtbb-dev”
- 4）GTEST (Google C++ Testing Framework): This is necessary only if you want to compile and run the unit tests. GTEST can be obtained from “https://code.google.com/p/googletest/”. An environment variable named GTEST that points to the installation path of the google test framework (e.g., “export GTEST=/local/gtest-1.5.0/”)
### Binaries:
Multiple binaries can be compiled. While under the “src” directory:
- 1）“make benchmark”
Compiles the benchmark tool that can be used to experiment with Totem.
run “./build/bin/benchmark -h” to get a detailed help message.
- 2）“make generator”
Compiles a tool that generates synthetic graphs (e.g., RMAT graphs) or modifies existing ones (e.g., sort vertices by degree).
run “./build/bin/generator -h” to get a detailed help message.
- 3）“make test”
Compiles all the unit tests. The unit tests uses the sample graphs that exist under the “data” directory.
run “./build/bin/all.test” to execute all unit tests at once.
- 4）“make test TEST=test_name”
Compiles a specific unit test.
run “./build/bin/test_name.test” to execute the specific unit test.
### Compilation Options:
The following compilation options can be used when compiling any of the above binaries.
- 1）“make target EID=64”: compiles for 64-bit edge ids
- 2）“make target GPU_ARCH=20”: compiles for compute capability 2.0. This  option is necessary for GPUs that do not support compute capability 3.5 and above.
- 3）“make target VERBOSE=YES”: configures Totem to report detailed timing information, in particular the timing of each superstep and each phase within the superstep (e.g., GPU compute time, CPU compute time, communication time etc.).
Example: make benchmark VERBOSE=YES EID=64 // compiles the benchmark for 64-bit ids
                                                                                     //  and reporting detailed timing.
Note that the Jetson TK1 board only supports compute capability 3.2.  Therefore, when performing compilations on the Jetson, “make target GPU_ARCH=20” must be used.
### Examples using the benchmark:
- 1）Run BFS on the both the CPU and the GPU, keeps 30% of the edges of the high degree vertices on the CPU:
./build/bin/benchmark -b0 -p2 -i1 -a30 graph.totem
- 2）Run BFS on the both the CPU and two GPUs, keeps 30% of the edges of the high degree vertices on the CPU, the remaining 70% of the edges are distributed randomly between the two GPUs:
./build/bin/benchmark -b0 -p2 -i1 -a30 -g2 -o graph.totem
 
# Totem Graph Text File Format

The following is the totem graph file format template:                                                                                                                        
                                                                                                                                                                               
#### NODES: vertex_count
#### EDGES: edge_count
#### DIRECTED|UNDIRECTED
[EDGE LIST]
                                                                                                                                                                               
- The first three lines specify the vertex and edge counts and whether the graph is directed or not. The vertices should have numerical IDs that ranges from 0 to vertex_count - 1.
- An edge list represents the edges in the graph. Each line describes a single edge, optionally with a weight as follows: "SOURCE DESTINATION [WEIGHT]". If the weight does not exist, it will be assigned a default value.
- The edge list must be ordered by the source vertex id (i.e., edge 0 -> 1 must appear before edge 10 -> 11).
- An undirected graph is represented as a directed graph with an edge placed each direction.

Example directed graph: 0 → 1, 1 → 2, 1 → 3

Totem graph text file should contain the following:

Nodes:4                                                                                                                                                                        
Edges:3                                                                                                                                                                        
Directed
0 1
1 2
1 3  
Example undirected graph: 0 -- 1, 1 -- 2, 1 -- 3

Totem graph text file should contain the following:

Nodes:4
Edges:6
Undirected
0 1
1 0
1 2
1 3
2 1
3 1

If the graph is very big, it is advisable to change the format to binary using the generator tool as the binary format is much faster load:

./build/bin/generator ALTER BINARY path/to/graph.totem
This will generate the binary graph file under the same directory as the text one with an extension “tbin”.
# Writing a New Algorithm

While writing a new algorithm in Totem framework, you are required to remember a number of things. You begin with creating a “.cu” file in the “trunk/src/alg” directory. File naming convention is the following: totem_name_of_your_algorithm.cu. If it is a hybrid implementation, the naming convention is totem_name_of_your_algorithm_hybrid.cu. In this document, we are not going to discuss algorithm implementation in details. You are encouraged to look at totem_clustering.cu, totem_dijkstra.cu and totem_bfs.cu for independent CPU and GPU implementations, and totem_bfs_hybrid.cu as an example of hybrid implementation. Please take a note of the naming conventions for methods as well. In all cases, you are required to update the “trunk/src/alg/totem_alg.h” file by declaring each method which implements your algorithm.

# Writing and Executing Unit Tests

The next step is to write unit tests. You are required to create a “.cu” file in the “/trunk/src/test” directory. File naming convention is the following: totem_name_of_your_algorithm_unittest.cu. You are encouraged to look at totem_clustering_unittest.cu and totem_dijkstra_unittest.cu to learn more about writing unit tests. The graph files (e.g., complete_graph_300_nodes.totem) used by the unit tests are usually placed in the “trunk/data” directory. Please refer to the README file to read more about these graphs. Look here to read more about the unit testing framework.
 
Once you are finished writing unit tests for your new algorithm, you are required to build the framework before you can execute your algorithm. Assuming your present working directory is “/trunk/src/test”, you can build the framework by just executing the command “make”. This will build the entire framework and create an executable called “all.test” in “/trunk/src/test”. Executing the command “./all.test” will run all the unit tests for the all the available algorithms. If you want to execute unit tests for an individual algorithm (e.g., bfs) you can build the framework with the command “make TEST=bfs”. This will create an executable called “bfs.test” in “/trunk/src/test”. Executing the command “./bfs.test” will run only the unit tests for the bfs algorithm. You can cleanup the workspace (i.e., deleting all the object files and executables) by executing the command “make clean-all”. The GPUs installed on ws020 and node240 are of different architecture than the ones on node250. On these two machines, you are required to build the framework using the following command: make GPU_ARCH=20 (or make GPU_ARCH=20 TEST= name_of_your_algorithm).          
 
# Running Benchmarks

If you want to run an algorithm in the Benchmark mode, you are required to update the source files in the “trunk/src/benchmark” directory. Please refer to an existing algorithm in order to understand how to update these files if you want to add a new algorithm. The file totem_benchmark.cu contains the “main” function. Assuming your present working directory is “/trunk/src/benchmark”, you can build the framework by just executing the command “make”.  This will build the entire framework and create an executable called “benchmark” in “/trunk/src/ benchmark”. Executing the command “./benchmark” will display the available options. If you want to execute an algorithm, say GPU-only implementation of bsf, you just execute the following command: ./benchmark –b 0 –p 1 <path to graph file>. The benchmark application produces several results such as algorithm execution time and execution rate. You are required to understand how to interpret these results. Again, on ws020 and node240, you are required to build the framework using the following command: make GPU_ARCH=20. It is recommend that before you run benchmark always do a clean build; cleaning up the workspace by executing the command “make clean-all”. Another important thing to remember, some very large graphs have 64-bit edge ids. In that case, you are required to build the framework by executing the command “make EID=64”. Please consult with a fellow lab member regarding graphs for benchmarking an algorithm.      
 
# Coding style

You should follow the coding style available here.
We use cpplint to check for code style errors, it is installed on all GPU machines. To use the tool:
- 1) Add the following alias to your .bash_profile or .bashrc:
alias cpplint="cpplint.py --filter=-legal/copyright,-build/include,-build/header_guard"
- 2) To check for code style errors:
cpplint file1.cpp file2.h file3.cpp
Note that the tool should report zero errors/warnings before you commit any file. 
# Development Workflow on GitHub

The following steps guides you through the process of sending patches for review and committing code to the main repository:
1) While signed in to your github account, go to https://github.com/netsyslab/Totem and fork a totem repository under your account by clicking on the "Fork" button on the far right corner.
2) (optional, but extremely useful) On your machine, to show current git branch as part of the command line, place the following bash script lines into your .bash_profile or .bashrc file:
/# Show current git branch as part of the command line                                                                  
function parse_git_branch () {
  git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
}
RED="\[\033[0;31m\]"
YELLOW="\[\033[0;33m\]"
GREEN="\[\033[0;32m\]"
NO_COLOR="\[\033[0m\]"
PS1="$GREEN\u@\h$NO_COLOR:\w$YELLOW\$(parse_git_branch)$NO_COLOR\$"
3) On your machine, clone the repository you just forked (replace YourUserName with your username):
git clone https://github.com/YourUserName/Totem.git
4) For each feature you work on, create a branch (assume a feature called optimize-loading):
git checkout -b optimize-loading
4.1) Note that this branch is created locally on your machine, and it does not yet exist under your github account. To do so, you need to "push" it and link it with a remote branch on your github:
git push --set-upstream origin optimize-loading
Note that "origin" is an alias to your github repository that was created for you in the first step after cloning. It is actually equivalent to https://github.com/YourUserName/Totem.git
5) Develop your feature, and commit incrementally. Note that commits are local, only when you push them they will show up on the corresponding branch under your github account.
git commit -m"meaningful message" file1.cc file2.cu
git push
6) Once the feature is ready for review, create a "pull request" to the Totem’s main repository (the one under netsyslab), you can do this via the web interface. Note that the "pull request" should be created from the main repository page. You can also create the pull request via command line using GitHub’s “hub” tool as follows:
hub pull-request -b netsyslab:master
7) Once the pull request (i.e., the review) is approved, the branch can be merged with the main repository by one of Totem project‘s owners via the web interface.
8) You need to keep your forked repository synchronized with the main one each time you start working on a new feature and create a new branch, you can do it via the github web interface, but also from the command line as follows:
8.1) Add the main repository as a remote (needs to be done once):
git remote add upstream https://github.com/netsyslab/Totem.git
8.2) Switch to the master branch of your clone:
git checkout master
8.3) Merge the changes in the mother repository into the local one:
git pull upstream master
8.4) Push them back to your forked repository:
git push
# Replicating Our Published Results

This section presents the options we used to obtain the various results we published.
**NOTE: While most options should be applicable irrespective of the machine characteristics being used, the percentage of offloaded edges (options -a and -l) may change. In particular, if one has a better CPU than the one we used, then it may be beneficial to offload less to the GPUs and visa versa.
# Technical Report

This section details the options we used to produce the results published in our technical report. The characteristics of the machine we used are detailed in Table 1 in the technical report.
NOTE1: There are two main run times reported by the benchmark, “total” and “exec”. total reports the total run time including the algorithm’s buffer allocation and setup times. exec reports the run time without the algorithm’s buffer allocation and setup times. Typically, “exec” is the time that is reported.
NOTE2: All the configurations listed here do not use the sorting by degree optimization. If used, this optimization should give a good performance boost. To enable vertex sorting by degree, add the -q option.  
For background: essentially what is happening when we partition, is that each element will maintain a local vertex ID list. This is to enable each partition to index their local vertices from 0 to N, the number of vertices in their partition. However, there remains this mapping from local ID to global ID. (So for example, local Vertex ID 0 on the GPU could be global ID 123, and VID 0 on the CPU could be global ID 321, and this would be perfectly valid.) (There is some explanation from our SC poster [link][poster])
When we sort, we actually sort the partitions local ID's (The ones from 0 to N). We can sort in different ways, but we find the most effective to be by vertex degree, due to locality and caching effects (e.g., we end up hitting the bitmap index of the high degree vertices a lot, so it is beneficial if they are next to each other). Since we have the map from local to global as mentioned above, we still know what the global VID was, so during the final steps we can ensure that the correct global ID gets the result.
There are also other options as well (-e and -d I think), that improve or change the method/direction of sorting, but I believe the direction is still initially based off of the partitioning strategy (eg, low or high also implies the sorting direction, lowest to highest or highest to lowest).
NOTE3: The PageRank code executes five rounds. If you want to measure the execution time of a single round, then just change the constant defined here: https://github.com/netsyslab/Totem/blob/master/src/alg/totem_alg.h#L60
NOTE4: The betweenness centrality code performs the computation for a single seed.
NOTE5: The connected components algorithm assumes undirected graphs.
NOTE6: When using more than one GPU (e.g., -g2), using the -o option is necessary to obtain good performance. This option makes sure that when having more than one GPU, that the GPUs get similar-sized partitions.

# Graph500 Submissions

./build/bin/graph500 -s30 -t"-c -d -i1 -q -e -p2 -o -g2 -a92 -l5"
./build/bin/graph500 -s29 -t"-c -d -i1 -q -e -p2 -o -g2 -a90 -l13"
# Notes/Warnings
This information will be useful to those running Graph500 benchmarks. I suggest those of you working on Graph500 or BFS (stepwise) have a quick look through this discussion.
----
Okay, there are two problems (I had thought these problems were fixed, but apparently not):
The Totem output are calculating two things differently. First; the Graph500 code requires init + exec time, not just exec time. You can see this here:
https://github.com/netsyslab/Totem/blob/master/src/thirdparty/graph500-2.1.4-energy/totem/totem_graph500.cu#L236
Second; the Totem output be base the GTEPS as directed TEPS.
Whereas Graph500 considers our two undirected edges as one. Note the difference in edges as reported by the Graph500 end result: it is half the amount reported by Totem.
Effectively, this reduces the value of our benchmark-reported GTEPS by half.
