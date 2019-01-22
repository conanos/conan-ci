# conan-ci
Document the design of conan-ci

# Introduction
During the development of software sdk which is composed of many libraries, there are several possible version varying or shifting scenarios for any particular library in sdk, which would drive the rebuilding of series of libraries that have a "dependency" relationship with that library. Whenever anything to do with a library in sdk has changed, the library needs to be rebuilt and packaged with a new version tag along with those "dependency" libraries.

Hence, in the following sections, version varying or shifting scenarios would be analyzed more detailedly so that version shifting regular for any library would be defined in section "**Version Shifting Regular**". Besides, "dependency" relationship would organize all the libraries in sdk as "tree topology", which would be described in section "**Requirements Topological Graph**". Conan -the C/C++ Package Manager- provides the method to abstract the "dependecny" information from the generated conan package. In section "**Construction of Requirements Topological Graph**", not only that "dependency"-gathering method would be explored in depth, but also "tree topology"-construction process would be explained completely. Finally, the general process of CI based on "tree topology"(i.e. Requirements Topological Graph) would be covered in section "**CI based on Requirements Topological Graph**".

# [Version Shifting Rule](semver)
Before the version shifting rule to be defined, the scenarios that cause software version shifting would be covered completely and detailedly. Generally, there are two tpyes of software library contained in sdk, i.e. open source software(OSS) and personal development library(PDL).

OSS usually has a version format like "MAJOR.MINOR[.MICRO][.BUILD][-[rc,alpha,beta]-SEQ]", where *MAJOR*, *MINOR* and *MICRO* are nonnegative integers,  *BUILD* and *SEQ* is positive integer, *square brackets []* means optional, and rc, alpha, beta denote pre-release. Software pre-release may be divided into three phases as [*phase-rc*, *phase-alpha* and *phase-beta*](release), where *phase-alpha* is the first phase to begin testing utilizing white-box, black-box or gray-box techniques, *phase-beta* is the second phase for testing that focuses on reducing impacts to software users, and *phase-rc* (rc, i.e. release candidate with potential to be a final product) is the final phase just before release. PDL can use the same version format as OSS. A common explanation for changing these numbers is that increase the:
1. *MAJOR* when incompatible API changes are made to library,
2. *MINOR* when new features or functionalities are added to library in a backwards-compatible manner,
3. *MICRO* when backwards-compatible bug fixes are made(For that reason, *MICRO* is sometimes called *PATCH*),
4. *BUILD* when a rebuild is made to library because of version shifting of dependency library, and
5. *SEQ* when backwards-compatible bug fixes are made to pre-release version of library.

The version shifting scenarios are analyzed detailedly as above. Next, several version shifting rules are made as follows (The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.):
1. Any normal version number MUST take the form MAJOR.MINOR.MICRO.BUILD* where *MAJOR*, *MINOR* and *MICRO* and *BUILD* are non-negative integers, and MUST NOT contain leading zeroes. *MAJOR* is the major version, *MINOR* is the minor version, *MICRO* is the patch version, *BUILD* is the build version that denotes the sequence nubmer of build caused by the changes of dependency libraries. Each element MUST increase numerically. For instance: 1.9.0.0 -> 1.9.0.1 -> 1.9.1.0 -> 1.10.0.0 -> 2.0.0.0
2. Once a version has been released, the contents of that version MUST NOT be modified any more. Any modifications MUST be released as a new version.
3. Major version zero (0.MINOR.MICRO) is for initial development. Anything may change at any time. Version 1.0.0 is for initial release.
4. Build version *BUILD* MUST be incremented if the version of any dependency library has made some change, but the building library does not make any change.
5. Micro version *MICRO* (MAJOR.MINOR.MICRO|MAJOR>0) MUST be incremented if only backwards compatible bug fixes are introduced into library. A bug fix is defined as an internal change that fixes incorrect behavior to library. It MAY include build level change. Build version *BUILD* MUST be reset to be 0 when micro version is incremented.
6. Minor version *MINOR* (MAJOR.MINOR.MICRO|MAJOR>0) MUST be incremented if new backwards compatible functionality is introduced into library API. It MUST be incremented if any library API is marked as deprecated. It MAY be incremented if substantial new functionality or improvement is introduced within the private code. It MAY include micro level change. Micro version *MICRO* MUST be reset to be 0 when minor version is incremented.
7. Major version *MAJOR* (MAJOR.MINOR.MICRO|MAJOR>0) MUST be incremented if any backwards incompatible changes are introduced into library API. It MAY include minor and micro level changes. Minor version *MINOR* and micro version *MICRO* MUST be reset to be 0 when major version is incremented.
8. A pre-release version MAY be denoted by appending *-[rc,alpha,beta]-SEQ* immediately following the build version, where *SEQ* is positive integer.
9. Precedence of version refers to how versions are compared to each other when ordered. Precedence is determined by the first difference compared numerically from left to right as follows: MAJOR, MINOR, MICRO, BUILD and SEQ. Besides that, pre-release version has lower precedence than the normal version with the same MAJOR, MINOR, MICRO and BUILD. For example: 1.0.0.1-alpha-1 < 1.0.0.1-alpha-2 < 1.0.0.1-beta-1 < 1.0.0.1-rc-1 < 1.0.0.1-rc-2 < 1.0.0.1 < 1.0.0.2-rc-1 < 1.0.0.2 < 1.0.1.0 < 1.1.0.0 < 2.0.0.0

# Requirements Topological Graph
Requirements topological graph shows the dependency relationship among the libraries that form a software sdk. Obviously, all the libraries in a software sdk and the dependency relationships among them form a directed acyclic graphic (DAG). Next graph is a typical requirements topological graph.

![DAG](.\DAG.jpg)

In terms of graph theory, any library in sdk can be represented by a vertice in DAG, and the dependency relationship between two libraries can be represented by a directed edge. So there are nine vertices in this DAG as A-I and directed edges(B->A, E->C, E->D, F->D, G->B, G->E, H->D, H->F, I->C, I->D, I->F, I->H).

A conclusion can be drawn that any requirements topological graph can be splited into several subgraphs as shown in this DAG. Each subgraph has exactly one vertice that only has incoming edges. That kind of vertice may be called root vertice for convience. So the example DAG is consisted of three subgraphs, and the root vertices are A, C and D.

![DAG-Subgraphs](.\DAG-Subgraphs.jpg)

Why should spliting requirements topological graph in this way be analyzed so detailedly? That's because that these subgraphs each with exactly one root vertice are the basis of CI. The intuitive steps when 
building software sdk by the way of CI are summarized as follows:
1. Construct the DAG subgraph whatever library contained in requirements topological graph has made changes in version. The root vertice of the DAG subgraph is the library that has made changes in version.
2. Calculate the sequence of library building according to the DAG subgraph. The algorithm will be elaborated in next section.
3. Build the library one after another in the order got in last step.

Before the algorithm be elaborated, some definitions are given as follows:
1. V(X) represents vertice X where X is a library contained in sdk.
2. D(X,Y) represents the distance between vertice X and Y if there is a directed path from X to Y. D(X,Y)==1 if vertice X connects to vertice Y directly.
3. Dmax(X,Y) represents the maximum distance between vertice X and Y if there are many directed paths present from X to Y.
4. T(G) represents the adjacent table of DAG G. For example, the adjacent table of DAG above is shown in figure below. Each element of T(G) is a vertice set, each of which is a adjacent vertice of some vertice in DAG.

![DAG-AdjacentTable](.\DAG-AdjacentTable.jpg)

5. M(G) represents the adjacent matrix of DAG G. For example, the adjacent matrix of DAG above is shown in figure below. Each element of M(G) denotes that the row vertice connects to the column vertice if the value equals to 1.

![DAG-AdjacentMatrix](.\DAG-AdjacentMatrix.jpg)

CI of building the software sdk is consists of three kinds of job. Job one is to build the software sdk from scratch. Job two is to build the libraries that form a DAG subgraph, which is triggered by the version change of the root vertix library. Job three is to add new libraries to existing sdk or develop sdk one library after another. For job one, the adjacent matrix of DAG can be used to calculate the building sequence of all libraries. For job two, the adjacent table can be used to calculate the building sequence of DAG subgraph.For job three, just build the target library one by one. Following are the two algorithms for job one and two. 

![Job-one](.\Job-one.jpg)

![Job-two](.\Job-two.jpg)

# Construction of Requirements Topological Graph With Conan
According to [the documentation of conan](conan), the requirements topological graph of a software library has been contained in the corresponding conan recipe.

Every conan recipe file "conanfile.py" has two member methods "requirements()" and "build_requirements()", separately corresponding to attributes "requires" and "build_requires". The method "requirements()" and attribute "requires" are used to declare the library dependencies, while "build_requirements()" and "build_requires" are used to declare dependencies that are only needed to be rebuilt from source if the binaries of them don't exist, such like dev tools, compilers, build systems, code analyzers, testing libraries, etc.

Conan has provided such a command as "conan info" to extract graph of dependencies (i.e. requirements topological graph) from conan recipe. Take library "gstreamer" for example.
1. Open the command line interface of Windows
2. Change the current working directory to where conan recipe of library "gstreamer" is
3. Run command "conan info . --graph=file.dot" and "type .\file.dot"

![Graph-of-dependencies-gstreamer](.\Graph-of-dependencies-gstreamer.jpg)

According to the file, requirements topological graph of library "gstreamer" is as following DAG.
![DAG-gstreamer](.\DAG-gstreamer.jpg)

Until now, construction of requirements topological graph for any software sdk with conan tools can be easily implemented. For each library contained in sdk, a graph of dependencies can be gathered by executing the command "conan info . --graph=XXX.dot". With these graphs of dependencies, adjacent matrix of sdk can be constructed, based on which then CI of software sdk development can be realized.

# CI based on Requirements Topological Graph

[semver]: https://semver.org/spec/v2.0.0.html
[release]: https://en.wikipedia.org/wiki/Software_release_life_cycle
[conan]: https://docs.conan.io/en/latest/