# conan-ci
Document the design of conan-ci

# Introduction
During the development of software sdk which is composed of many libraries, there are several possible version varying or shifting scenarios for any particular library in sdk, which would drive the rebuilding of series of libraries that have a "dependency" relationship with that library. Whenever anything to do with a library in sdk has changed, the library needs to be rebuilt and packaged with a new version tag along with those "dependency" libraries. Hence, in the following sections, version varying or shifting scenarios would be analyzed more detailedly so that version shifting regular for any library would be defined in section "**Version Shifting Regular**". Besides, "dependency" relationship would organize all the libraries in sdk as "tree topology", which would be described in section "**Requirements Topological Graph**". Conan -the C/C++ Package Manager- provides the method to abstract the "dependecny" information from the generated conan package. In section "**Construction of Requirements Topological Graph**", not only that "dependency"-gathering method would be explored in depth, but also "tree topology"-construction process would be explained completely. Finally, the general process of CI based on "tree topology"(i.e. Requirements Topological Graph) would be covered in section "**CI based on Requirements Topological Graph**"

# Version Shifting Regular
*
# Requirements Topological Graph
# Construction of Requirements Topological Graph
# CI based on Requirements Topological Graph