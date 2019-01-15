# conan-ci
Document the design of conan-ci

# Introduction
During the development of software sdk which is composed of many libraries, there are several possible version varying or shifting scenarios for any particular library in sdk, which would drive the rebuilding of series of libraries that have a "dependency" relationship with that library. Whenever anything to do with a library in sdk has changed, the library needs to be rebuilt and packaged with a new version tag along with those "dependency" libraries.

Hence, in the following sections, version varying or shifting scenarios would be analyzed more detailedly so that version shifting regular for any library would be defined in section "**Version Shifting Regular**". Besides, "dependency" relationship would organize all the libraries in sdk as "tree topology", which would be described in section "**Requirements Topological Graph**". Conan -the C/C++ Package Manager- provides the method to abstract the "dependecny" information from the generated conan package. In section "**Construction of Requirements Topological Graph**", not only that "dependency"-gathering method would be explored in depth, but also "tree topology"-construction process would be explained completely. Finally, the general process of CI based on "tree topology"(i.e. Requirements Topological Graph) would be covered in section "**CI based on Requirements Topological Graph**".

# [Version Shifting Rule](semver)
Before the version shifting rule to be defined, the scenarios that cause software version shifting would be covered completely and detailedly. Generally, there are two tpyes of software library contained in sdk, i.e. open source software(OSS) and personal development library(PDL).

OSS usually has a version format like "MAJOR.MINOR[.MICRO][.BUILD][-[rc,alpha,beta]SEQ]", where *MAJOR*, *MINOR*, *MICRO* and *BUILD* are nonnegative integers, *SEQ* is positive integer, *square brackets []* means optional, and rc, alpha, beta denote pre-release. Software pre-release may be divided into three phases as [*phase-rc*, *phase-alpha* and *phase-beta*](release), where *phase-alpha* is the first phase to begin testing utilizing white-box, black-box or gray-box techniques, *phase-beta* is the second phase for testing that focuses on reducing impacts to software users, and *phase-rc* (rc, i.e. release candidate with potential to be a final product) is the final phase just before release. PDL can use the same version format as OSS. A common explanation for changing these numbers is that increase the:
1. *MAJOR* when incompatible API changes are made to library,
2. *MINOR* when new features or functionalities are added to library in a backwards-compatible manner,
3. *MICRO* when backwards-compatible bug fixes are made(For that reason, *MICRO* is sometimes called *PATCH*),
4. *BUILD* when a rebuild is made to library because of version shifting of dependency library, and
5. *SEQ* when backwards-compatible bug fixes are made to pre-release version of library.

The version shifting scenarios are analyzed detailedly as above. Next, several version shifting rules are made as follows (The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.):
1. Any normal version number MUST take the form *MAJOR.MINOR.MICRO* where *MAJOR*, *MINOR* and *MICRO* are non-negative integers, and MUST NOT contain leading zeroes. *MAJOR* is the major version, *MINOR* is the minor version, *MICRO* is the patch version. Each element MUST increase numerically. For instance: 1.9.0 -> 1.9.1 -> 1.10.0 -> 2.0.0.
2. Once a version has been released, the contents of that version MUST NOT be modified any more. Any modifications MUST be released as a new version.
3. Major version zero (0.MINOR.MICRO) is for initial development. Anything may change at any time. Version 1.0.0 is for initial release.
4. Micro version *MICRO* (MAJOR.MINOR.MICRO|MAJOR>0) MUST be incremented if only backwards compatible bug fixes are introduced into library. A bug fix is defined as an internal change that fixes incorrect behavior to library.
5. Minor version *MINOR* (MAJOR.MINOR.MICRO|MAJOR>0) MUST be incremented if new backwards compatible functionality is introduced into library API. It MUST be incremented if any library API is marked as deprecated. It MAY be incremented if substantial new functionality or improvement is introduced within the private code. It MAY include micro level changes. Micro version *MICRO* MUST be reset to be 0 when minor version is incremented.
6. Major version *MAJOR* (MAJOR.MINOR.MICRO|MAJOR>0) MUST be incremented if any backwards incompatible changes are introduced into library API. It MAY include minor and micro level changes. Minor version *MINOR* and micro version *MICRO* MUST be reset to be 0 when major version is incremented.
7. Build version *.BUILD* MAY be appended immediately following the micro version and SHOULD be incremented if the version of any dependency library has made any change, but the target library to be rebuilt has not made any change.
8. A pre-release version MAY be denoted by appending *-[rc,alpha,beta]-SEQ* immediately following the micro version or the build version, where *SEQ* is positive integer.
9. Precedence of version refers to how versions are compared to each other when ordered. Precedence is determined by the first difference compared numerically from left to right as follows: MAJOR, MINOR, MICRO, BUILD and SEQ.


# Requirements Topological Graph
# Construction of Requirements Topological Graph
# CI based on Requirements Topological Graph

[semver]: https://semver.org/spec/v2.0.0.html
[release]: https://en.wikipedia.org/wiki/Software_release_life_cycle