References: WG5/N2147, J3/18-110r1

# Introduction

In WG5/N2147, "Fortran 202X Feature Survey Results -- Final", by Steve
Lionel, the proposed F202X feature that had the most interest and the
most comments was "Generic Programming or Templates". This feature had a
score of 6.21, compared to the next highest score of 5.04 for "Automatic
Allocation on READ into ALLOCATABLE character or array". It also had
almost four pages of comments, while no other feature had as much as a
full page of comments. In spite of the obvious demand for this feature,
its complexity made it inappropriate to include in Fortran 202X. Instead
it was planned that preliminary work on this feature would be fitted
into the schedule of work on Fortran 202X in the hope that this would
allow the completion of the work during the development of Fortran 202Y.

This paper explores use cases for generics in order to identify
requirements for a Fortran generic facility. The identification of
requirements is expected to allow detailed work on generics. The use
cases divide into two broad categories: generic algorithms and generic
containers. Generic algorithms are typically single procedures with a
focus on one task. Generic containers are the equivalent of a derived
type with accompanying procedures. In many cases, the generic algorithm
is intended to iterate over the elements of a container, in which case
it is a great convenience for the generic container to define a
syntactically consistent means of iterating over the container and
extracting individual elements. Such a construct is termed an iterator.
The discussions of generic algorithms and containers will often mention
iterators.

## Organization of document

# Use cases

## Generic Algorithms

A common justification for providing generics/templates for Fortran is
that they would enable the development of a library of algorithms that
would simplify the development of code. Typically an algorithm can be
implemented as a single procedure. As generics, they could be
implemented as one procedure per generic module, multiple procedures per
generic module, or as individual generic procedures. Their usage would
be simplified if the generics allowed template parameters to be applied
to a procedure invocation as opposed to at the type definition or module
declaration level. (ref. M. Haveraaen) It appears that, for the
algorithmic use cases below, the template parameters can be inferred
from the arguments to the procedure, further simplifying their usage. As
a result, while not a requirement for a generic algorithm, almost all
generic algorithms would benefit from the capabilities:

- the ability to define and instantiate a single generic procedure;
- the ability to use type inference in instantiating a single generic
  procedure; and
- instantiation in the same specification part that defines the data
  type.

The number of algorithms of potential interest is large. For example,
the C++ 20 Library defines about 200 algorithms in fourteen categories:

- Non-modifying sequence operations,
- Modifying sequence operations,
- Partitioning operations,
- Sorting operations,
- Binary search operations (on sorted ranges),
- Other operations on sorted ranges,
- Set operations (on sorted ranges),
- Heap operations,
- Minimum/maximum operations,
- Comparison operations,
- Permutation operations,
- Numeric operations,
- Operations on uninitialized memory, and
- the C library.

Not all of these are of interest to a Fortran "template" library, i.e.,
"Heap operations" and "Operations on uninitialized memory" appear to be
of little interest to the Fortran community, and the "C library"
consists of two non-templated algorithms. Further, many of the
algorithms are near duplicates of one another, differing only in which
form of iterator is used: the old style iterator or the newer ranges.
Still it appears that well over 50 algorithms are of likely interest. To
illustrate the capabilities of generic algorithms we have tried to
select one representative algorithm from each of the eleven categories
of interest.

### Use case: Extending intrinsic algorithms (0-0-0)

A somewhat common pattern is to desire to reproduce the algorithm of an
intrinsic procedure in the context of a custom derived type. `FINDLOC` is
perhaps the most frequently cited example. Here, the user wishes to find
the first entry in an array that equals a given value. The algorithm
itself is the same for any type that supports tests for equality `==`,
and can be generalized to any container that supports iterators.

The generic `FINDLOC` procedure has the following requirements:

- the ability to have a general type as a generic parameter either as
  an explicit parameter or as implicit in the iterators;
- the ability to associate an equality operation with a generic type
  parameter, either implicitly as a property of the type definition,
  or as an explicit parameter to the generic with a defined interface;
- the ability to specialize for different ranks of arrays; and
- the ability to take one or more iterators as instantiation
  parameters.

### Use case: Swap (0-0-0)

A common task in code is to swap the values or targets of two variables.
The swap can involve different forms of variables:

- Swap non-polymorphic (`TYPE`) scalars,
- Swap polymorphic (`CLASS`) scalars,
- Swap scalars with `LEN` parameters,
- Swap pointers,
- Swap allocatables (via `MOVE_ALLOC`),
- Swap arrays of the same shape, and
- Swap ranges of elements of two containers.

Ideally the generics facility could provide means to allow a single
template to work for all these variants. The variants may require
additional parameters to indicate the argument's attributes.

The generic swap procedure has the following requirements:

- the ability to have a general type as a generic parameter;
- the ability to associate an assignment operation with a type
  parameter, either implicitly as a property of all types, implicitly
  as a property of the type definition, or as an explicit parameter to
  the generic with a defined interface; and
- the ability to specialize depending on whether
  - an instantiation type parameter is polymorphic or not;
  - an instantiation type has a `LEN` parameter or not;
  - the arguments are pointers;
  - the arguments are allocatables; or
  - the arguments are arrays.

### Use case: Partition (0-0-0)

It is sometimes useful to divide a collection of items into two sets
selected according to the results of a logical predicate. Such a
division is termed a partition. The partition procedure needs to be able
to iterate through the collection, apply the predicate, sort the results
into two parts, and return a marker for the start of the second portion
of the partition.

A generic partition procedure has the following requirements:

- the ability to have a general type as a generic parameter either as
  an explicit parameter or as implicit in the iterators;
- the ability to associate an assignment operation with a type
  parameter, either implicitly as a property of all types, or as a
  property of the type definition, or as an explicit parameter to the
  generic with a defined interface;
- the ability to associate a unary logical predicate with a generic
  type parameter, either implicitly as a property of the type
  definition, or as an explicit parameter to the generic with a
  defined interface; and
- the ability to take one or more iterators as instantiation
  parameters.

### Use case: Sorting (0-0-0)

While the need to sort a list of items is somewhat rarer in typical
Fortran applications than in wider software communities, it nonetheless
arises often enough to be a problem. It is typically applied to a rank
one array, but can be generalized to any container with appropriate
iterators, e.g. Lists and Vectors. A given sorting algorithm can
generally be applied to any type that provides a comparison (`<` or
`>`) operation, or equivalent binary logical predicate. With current
Fortran capabilities, the algorithm must be re-implemented for each type
- with some possible simplification through the use of include files.

A generic sorting procedure has the following requirements:

- the ability to have a general type as a generic parameter either as
  an explicit parameter or as implicit in the iterators;
- the ability to associate an assignment operation with a type
  parameter, either implicitly as a property of all types, or as a
  property of the type definition, or as an explicit parameter to the
  generic with a defined interface;
- the ability to associate a comparison operation with a generic type
  parameter, either implicitly as a property of the type definition,
  or as an explicit parameter to the generic with a defined interface;
  and
- the ability to take one or more iterators as instantiation
  parameters.

### Use case: Searching (0-0-0)

Searching is the complement of sorting. One typically sorts an array so
that it can be searched for specific elements. A binary search algorithm
allows the searching of a sorted array with O(ln(n)) cost. For the
search to be efficient, it must use the same comparison operation used
in the sorting. With current Fortran capabilities, the algorithm must be
re-implemented for each type - with some possible simplification through
the use of include files. The requirements for the generic search are
essentially the same as for the corresponding sort algorithm.

### Use case: Merging (0-0-0)

It is sometimes useful to merge two sorted collections into one sorted
collection. The MERGE procedure must take two collections sorted using
the same comparison operation, iterate through both of them in the same
direction comparing current elements using the same comparison, and
enter them into a third collection according to the results of the
comparison.

A generic MERGE procedure has the following requirements:

- the ability to have a general type as a generic parameter either as
  an explicit parameter or as implicit in the iterators;
- the ability to associate an assignment operation with a type
  parameter, either implicitly as a property of all types, or as a
  property of the type definition, or as an explicit parameter to the
  generic with a defined interface;
- the ability to associate a comparison operation with a generic type
  parameter, either implicitly as a property of the type definition,
  or as an explicit parameter to the generic with a defined interface;
  and
- the ability to take two iterators as instantiation parameters.

### Use case: Set intersection (0-0-0)

A sorted container has properties similar to a set, and can easily be
converted to a set by removing adjacent duplicate values. As a result it
is useful to define set operations, such as `SET_INTERSECTION`, for pairs
of sorted containers. In `SET_INTERSECTION`, the two collections are
scanned in sequence and for each value with m copies in the first set
and n copies in the second `MIN(M,N)` copies are copied to the container
holding the intersection.

A generic `SET_INTERSECTION` procedure has the following requirements:

- the ability to have a general type as a generic parameter either as
  an explicit parameter or as implicit in the iterators;
- the ability to associate an assignment operation with a type
  parameter, either implicitly as a property of all types, or as a
  property of the type definition, or as an explicit parameter to the
  generic with a defined interface;
- the ability to associate a comparison operation with a generic type
  parameter, either implicitly as a property of the type definition,
  or as an explicit parameter to the generic with a defined interface;
  and
- the ability to take two iterators as instantiation parameters.

### Use case: Max element (0-0-0)

It is often useful to know the maximum or minimum elements of a
collection. The function `MAX_ELEMENT` would return the maximum element of
a collection according to a comparison operation assumed to be a less
than operation, `<`, perhaps supplemented by an equality operation
`==` to deal with NaNs. In `MAX_ELEMENT`, the elements of a collection
are scanned in sequence, using an iterator, replacing the current
maximum element with the new one if the new one compares false when the
first argument in a comparison with the current maximum, and true when
the second.

A generic `MAX_ELEMENT` procedure has the following requirements:

- the ability to have a general type as a generic parameter either as
  an explicit parameter or as implicit in the iterator;
- the ability to associate an assignment operation with a type
  parameter, either implicitly as a property of all types, or as a
  property of the type definition, or as an explicit parameter to the
  generic with a defined interface;
- the ability to associate a comparison operation with a generic type
  parameter, either implicitly as a property of the type definition,
  or as an explicit parameter to the generic with a defined interface;
  and
- the ability to take an iterator as an instantiation parameter.

### Use case: Lexicographical Comparison (0-0-0)

It is sometimes useful to perform a lexicographical comparison of two
collections using a binary logical predicate. The comparison should
return -1 if the first collection is less than the second, 1 if the
first collection is greater, and 0 if they are equal. The collections
should be compared element by element, with the first mismatching
element determining which collection is less or greater than the
other, otherwise the shorter range is less than the other, otherwise if
the collections are the same length and the elements are equivalent, the
collections are considered equal.

A generic `LEXICOGRAPHICAL_COMPARISON` procedure has the following
requirements:

- the ability to have a general type as a generic parameter either as
  an explicit parameter or as implicit in the iterators;
- the ability to associate an assignment operation with a type
  parameter, either implicitly as a property of all types, or as a
  property of the type definition, or as an explicit parameter to the
  generic with a defined interface;
- the ability to associate a comparison operation with a generic type
  parameter, either implicitly as a property of the type definition,
  or as an explicit parameter to the generic with a defined interface;
  and
- the ability to take two iterators as instantiation parameters.

### Use case: Permutation (0-0-0)

It is sometimes useful to perform a permutation of a collection or
verify that one collection is considered a permutation of the other
under a binary logical predicate. As an example we will consider the
inquiry function `IS_PERMUTATION` that returns true if the first
collection is considered a permutation of the second. To do this it
first iterates over each collection generating a sorted collection and
then compares each sorted collection element by element to see if the
sorted collections are identical.

A generic `IS_PERMUTATION` procedure has the following requirements:

- the ability to have a general type as a generic parameter either as
  an explicit parameter or as implicit in the iterators;
- the ability to associate an assignment operation with a type
  parameter, either implicitly as a property of all types, or as a
  property of the type definition, or as an explicit parameter to the
  generic with a defined interface;
- the ability to associate a comparison operation with a generic type
  parameter, either implicitly as a property of the type definition,
  or as an explicit parameter to the generic with a defined interface;
  and
- the ability to take two iterators as instantiation parameters.

### Use case: Inner Product (0-0-0)

It is sometimes useful to perform numerical operations on one or two
collections. The operations can take several forms, e.g., inner product,
accumulate, adjacent difference, etc. We will use the `INNER_PRODUCT` as
an example. The `INNER_PRODUCT` iterates over two collections
simultaneously, takes the "product" of corresponding elements and sums
the resulting products.

A generic `INNER_PRODUCT` procedure has the following requirements:

- the ability to have a general type as a generic parameter either as
  an explicit parameter or as implicit in the iterators;
- the ability to associate an assignment operation with a type
  parameter, either implicitly as a property of all types, or as a
  property of the type definition, or as an explicit parameter to the
  generic with a defined interface;
- the ability to associate a "product" operation with a generic type
  parameter, either implicitly as a property of the type definition,
  or as an explicit parameter to the generic with a defined interface;
  and
- the ability to associate a "summation" operation with a generic type
  parameter, either implicitly as a property of the type definition,
  or as an explicit parameter to the generic with a defined interface;
  and
- the ability to take two iterators as instantiation parameters and
  synchronize them.

## Generic Containers

As scientific models become more complex, an increasing fraction of the
lines of code are infrastructure as-opposed to direct numerical
calculation. Examples of infrastructure include software layers that
couple independently developed subsystems, frameworks for managing
distributed parallelism, advanced I/O (e.g., checkpoint/restart via
NetCDF), etc. Generally these infrastructure layers evolve as an attempt
to avoid code duplication as multiple parts of the system require
similar functionality.

Software "containers" are abstractions that enable aggregating groups
of related entities for convenient and efficient access. Many categories
of software containers have been defined, with each category generally
tailored to a common design issue. By providing a "standard" and
reliable means to perform commonly occurring operations, containers can
greatly simplify design and implementation of complex algorithms.

Fortran provides just one category of software container - Array. Array
containers are designed to optimize random access to a *fixed*
collection of elements, and generally requires all contained elements to
be of the same dynamic type. Fortran provides excellent mechanism for
declaring and constructing arrays of any type as well as for accessing,
storing, and modifying array members (elements) with succinct, tailored
syntax: tuples of indices within parens:

```Fortran
a(i,j) = x
x = a(i,j)
a(i,j) = a(i,j)**2
```

Other commonly used container categories include List, Vector, Map (or
Associative Array, Dictionary), Set, Stack, Queue, etc. Unlike Array
containers, these others generally are more dynamic - growing as
necessary when new elements are added to the container. Here we briefly
describe some of the most common categories of containers.

### Use case: List (0-0-0)

Lists are sequentially accessed containers. They are particularly useful
when the data needs to be modified, as they are O(1) complexity in
prepending, appending, inserting, and deleting an element. They come in
several forms of which the most prominent are singly linked lists
(providing inexpensive sequential access in one direction), doubly
linked lists (providing slightly more expensive sequential access in two
directions); and flattened lists (basically lists of rank one arrays).

All the types of lists have similar requirements:

- a generic construct with scope equivalent to that of a module
  encompassing types and associated procedure definitions;
- the ability to have a type as a parameter;
- the ability to associate an assignment operation with a type, either
  implicitly as a property of all types, implicitly as a property of
  the type definition, or as an explicit parameter to the generic with
  a defined interface;
- the ability to iterate over the active elements of the structure;
- instantiation of a generic module with a derived type with the same
  parameters should define the same type, or the same instantiation
  should be capable of defining different entities in different
  contexts; and
- instantiation of a module with a derived type with different
  parameters should define different types;

While not required, a generic list would benefit from the following
capabilities:

- the ability to specialize depending on whether an instantiation type
  parameter has a `LEN` parameter or whether it is to be treated as
  polymorphic;
- the ability to modify elements of the structure while iterating over
  it;
- instantiation in the same specification part that defines the types
  of the data;
- a language defined iteration construct that can go in two
  directions;
- a language defined iteration construct that allows modification of
  the structure; and
- a way of defining a type so that objects of that type can be indexed
  to access elements of that type.

### Use case: Vector (0-0-0)

Vectors are somewhat similar to 1-D Arrays in that they provide
efficient random access. But whereas the size of an Array is fixed at
the time of its construction, a Vector can grow as elements are
appended. (For those unfamiliar with containers, there is no magic here.
Internally arrays are reallocated with copies, but by, say, doubling the
size when reallocating, appending elements is of O(1) complexity on
average.) The requirements for this generic container are very similar
to those for Lists.

### Use case: Set (0-0-0)

Set containers provide efficient mechanisms for searching and inserting
distinct elements. One implementation (ordered sets) uses a balanced
binary search tree data structure to store the elements (keys) providing
O(log(n)) cost for searching and inserting distinct elements. This
structure requires the availability of an ordering operation on the key
type. Another implementation (unordered sets) uses a hash table to store
the elements providing O(1) cost for searching and inserting distinct
elements. This structure requires the availability of hash and equality
operations on the key type.

A generic Set has the following requirements:

- a generic construct with scope equivalent to that of a module
  encompassing types and associated procedure definitions;
- the ability to have a type as a parameter;
- the ability to associate an assignment operation with a type
  parameter, either implicitly as a property of all types, implicitly
  as a property of the type definition, or as an explicit parameter to
  the generic with a defined interface;
- the ability to iterate over the active elements of the structure;
- instantiation of a generic module with a derived type with different
  parameters should define different types; and
- instantiation of a generic module with a derived type with the same
  parameters should define the same type, or the same instantiation
  should be capable of defining different entities in different
  contexts; and
- the ability to specialize the code based on the rank of the data of
  interest, the presence or absence of the pointer attribute, and the
  presence or absence of a `LEN` parameter;

In addition, the ordered Set has the following requirement:

- the ability to associate an ordering operation with a type
  parameter, either implicitly as part of the type definition or as an
  explicit parameter to the generic with a defined interface;

while the unordered Set has the following requirements:

- the ability to associate an equality operation with a type
  parameter, either implicitly as part of the type definition or as an
  explicit parameter to the generic with a defined interface; and
- the ability to associate an arbitrary hashing procedure with a type
  parameter, either implicitly as part of the type definition or as an
  explicit parameter to the generic with a defined interface.

While not required a generic Set would benefit from the following
capabilities:

- instantiation in the same specification part that defines the types
  of the key and data;
- a language defined iteration construct; and
- the ability to specialize the code based on the rank of the entities
  with a given instantiation type parameter;

While not required a generic Set might benefit from the following
capabilities:

- the ability to define an interface that is independent as to whether
  the procedure arguments are polymorphic and whether the procedure is
  type bound;
- the ability to specialize the code based on the presence or absence
  of the pointer attribute associated with a given instantiation type
  parameter;
- the ability to specialize the code based on the presence or absence
  of a `LEN` or `KIND` parameter associated with a given instantiation
  type parameter; and
- the ability to instantiate with scalar parameters of type logical,
  integer, or real.

### Use case: Map (0-0-0)

Maps (also called associative arrays or dictionaries) are similar to
Sets, except that the contained elements are key-value pairs. Typically
the keys are either integers or strings, but can be any data type that
can be either ordered (an ordered Map) or hashed (an unordered Map). A
simple example would be a Map whose keys are the names of students in a
classroom, and the values are their grades. A more relevant example
would be a Map that provides efficient access to a sparse array where
the keys are the indices for the non-zero array elements.

The requirements for generic Maps are similar to those for generic sets
except that, with the addition of the value attribute, they require the
ability to have multiple types as parameters.

### Use case: Bitset (0-0-0)

The bitset, also known as the bit string, bit map, bit vector, or bit
array, optimizes memory storage by densely packing binary information.
As it also reduces memory traffic it has the potential of improving run
time performance. As a result both C++ and Java define bitsets in their
standard libraries. However much of this functionality would be met by
the BITS data type proposed for Fortran 202X.

A generic bitset has one instantiation parameter, the number of bits in
a specific set. This number is treated as a run time invariant and is
fixed at compilation time. This number is mapped onto an array of
integers such that the array is the minimum size necessary to represent
that number of bits. For these bits the bitset type defines a number of
"unary" operations that involve only one bitset, and "binary"
operations that involve two bitsets. The binary operations only "make
sense" if the two bitsets have the same number of bits. By treating
different numbers of bits as representing different types this
constraint is enforced by the type system. The fact that parameterized
derived types treat `LEN` parameters as runtime resolved makes them
impractical to enforce this type safety.

A generic bitset has the following requirements:

- a generic construct with scope equivalent to that of a module
  encompassing types and associated procedure definitions;
- the ability to instantiate with a scalar parameter of type integer;
- instantiation of a generic module with a derived type with different
  parameter values should define different types;
- instantiation of a generic module with a derived type with the same
  parameters should define the same type, or the same instantiation
  should be capable of defining different entities in different
  contexts; and
- the ability to specialize the code based on the value of an integer
  parameter.

### Use case: Iterators (0-0-0)

Iterators provide a simple/consistent mechanism to efficiently loop
through all of the elements in a container. For containers such as
Vector and Array, these are relatively simple, but for containers such
as Set and Map that are implemented with binary trees, the iterator
abstraction is extremely beneficial to the user of the container.
Iterators simultaneously hide complexity and encourage safe coding
styles. In pseudo-code iterator usage is typically something like:

```
< declare container > C
< declare container iterator > I
< declare element > e

I = begin(C)
loop while I is not end(C)
  e = get(I)
  < do something with e >
  next(I)
end loop
```

Concrete use cases:

Atmospheric tracers metadata in the NASA GISS climate model. Each
species (CO2, CH4, NO3, ...) in the atmosphere is associated with a
variety of metadata. These include the molar mass, the radioactive decay
rate, diffusivity, a category label (dust, aerosol, etc.), and so on.
There are O(30) such fields in the model currently. Most are required to
have a value for all tracers, but some fields are only added for tracers
where they are relevant. The types of these metadata are `LOGICAL`,
`INTEGER`, `REAL`, and all but one are scalars. The natural representation
of this is a Map container where the keys are the property name and the
values are the various items of metadata.

Collection of tracers in NASA GISS climate model. Different
configurations of the model utilize different subsets of the available
tracers in the source code. The number ranges from 0 to over 100
tracers. Each tracer has a natural name, usually the chemical formula,
but sometimes regular names like 'terpene' or 'dust'. A natural
mechanism to manage data associated with each tracer is again a Map
container where the keys are the tracer names and the values are a
derived type containing all tracer data. For historical/conservative
reasons only the metadata dictionaries are actually managed this way.
Other data are in dynamically allocated Arrays (Array containers) whose
size can be computed after all of the metadata has been processed.

Regriding registry in NASA GEOS5 model. This model must represent data
on varying grids that discretize the Earth's atmosphere. To transfer
data between grids, largish interpolation tables are generated and
stored. Because of the expense of computing them, they are cached at the
time they are first computed. The mechanism used to manage this is again
a Map where the keys are a pair of specifiers associated with the two
grids that determine the regriding.

A new I/O client-server layer in NASA GEOS5 model.

1. Each server process manages I/O requests from a varying number of
   application processes. This is managed as a Vector of clients that
   grows as needed.
2. To encode/decode messages, a Map container with integer keys is used
   to store message prototypes. This allows each subclass of message to
   decode itself and while the top-level processing only needs to
   handle the integer label.
3. The server processes accumulate data requests from the client in a
   Map where they key is a message ID and the details are in a data
   type.
4. NetCDF metadata itself uses multiple containers. NetCDF variables
   are represented as a Map with the variable names as keys and values
   are Variable derived type. The Variable derived type itself has a
   Map where the keys are Attribute names and the values are Attribute
   values. The Variable derived type also has a Vector of "dims" that
   relate the variable dimensions to a global list for the file.

Next generation pFUnit unit-testing framework for Fortran. The framework
maintains a global list of "exceptions" that are thrown by user tests.
This is naturally represented as a vector of entities of derived type.

New configurable logging utility (pFlogger). This framework manages
collections of abstract data types: Handlers (abstractions of Files),
Loggers, Filters, and Formatters. To enable runtime configurability each
such entity is associated with a name. Map's are then used to manage the
associations. E.g., each Logger can have multiple Handlers. Loggers and
Handlers can both have multiple Filters. These Maps are all polymorphic.
The package also uses a number of containers (Vector and Map) involving
intrinsic types.

### Containers and Fortran

Infrastructure layers for complex Fortran applications often
implicitly/unknowingly implement crude versions of containers like
Vectors and Maps. The Vector pattern arises naturally when the total
number of instances of some collection cannot be determined via a simple
formula or when elements are added sporadically through different
drivers. Instead the size is "discovered" through some iterative
process. A common implementation in the simplest case is a two-pass
algorithm. The passes are nearly identical except that the first pass
just accumulates the number of elements. An allocation is then made and
the 2nd pass fills in the allocated structure.

The need for Map containers arises when we need to find entities through
some registry. E.g., if an ocean component needs the surface winds from
an atmospheric component, an abstract coupler enables this by allowing
both components to use the names `U-wind` and `V-wind` to retrieve the
associated arrays. This avoids hardcoding indices, which is important
when considering independently developed components.

Typical Fortran implementations of these patterns are ad-hoc. Even
within the same application two different layers may implement the same
pattern in rather different ways. The layers also often contain latent
bugs that are not exercised in the current component, but prevent their
adoption in other components.

### Obstacles to clean/robust implementation of containers in Fortran

One large obstacle to development and use of containers in Fortran is
the lack of support for generic coding. Developers must either duplicate
the logic of a container for each potential type of element, or must
resort to non-Fortran means (preprocessors) to have a single
implementation that is applicable to all cases. E.g., one may wish to
have Vectors of integers, reals, logicals, and/or various derived types,
leading to many almost identical implementations and all the associated
dangers of long-term maintenance. With Maps, the problem just grows
combinatorially as one considers variant types of keys and values.

An obstacle to the development of generic containers in Fortran is the
lack of regularity in its type system. The usage of types is complicated
by variants in the syntax due to the distinctions made between
polymorphic (`CLASS`) and non-polymorphic (`TYPE`) entities, entities with
and without a `LEN` parameter, and with and without a `KIND` parameter.
Fortran is also unusual in that it makes array and pointer attributes
independent of type rather than part of the type, and that size, rank,
and shape can be declared or assumed. Unless Fortran's type system is
changed to allow a more general syntax, either the developer of the
container will have to use a large number of specializations, or he will
limit the allowed syntax and the user will have to use workarounds for
those limitations, e.g., wrapper types for polymorphic types for types
with a `LEN` parameter.

Another obstacle, albeit less severe, is the inability to overload
suitable operators for container accessors. With Fortran arrays, parens
can be used to access individual elements on both the left and right
hand sides of assignment:

```
x = a(i,j,k)
b(i,j,k) = y
```

Ideally, Vector and Map containers would have similar mechanism for
accessing and modifying elements. Note that containers generally return
pointers to contained elements rather than copies of those elements.

### State of the art

With aggressive and complicated use of CPP/FPP preprocessing it is
possible to write robust generic Vector and Map containers in Fortran. I
and my colleague Doron Feldman have produced such a package which was
recently released as open source: gFTL. Another similar package, FTL,
has also been independently released. (They beat us to the cooler name.)
These implementations are a nice step forward, and have been of enormous
benefit in the development of new powerful software layers. However,
these containers are still not as simple to use as one could hope for
number of reasons:

1. A separate Fortran class must be constructed for each contained
   type. In languages with stronger support for containers, this step is
   performed by the compiler.
2. Even if two separate projects use the same container package (e.g.,
   gFTL), they cannot assume that the other is providing a common container
   and must redeclare for themselves. One easily ends up with many
   all-but-identical modules for say IntegerVector.
3. Iterators are handled via Fortran's `DO WHILE` construct. Users then
   put a call to `iter` in simple cases, but is dangerous in the presence of
   `CYCLE` because the user must remember to invoke `iter`. Languages that have
   an `increment` component to their loop constructs (e.g., C, C++) can
   handle iterators more robustly by putting the iterator increment into
   the header of the loop. This concern is probably more general than just
   iterators and should perhaps be a paper of its own.
4. As mentioned above, the supported accessors are clunky. The
   inability to use syntax analogous to that of the tuples of indices for
   arrays is unfortunate. It is a minor annoyance for references on the RHS
   of an assignment, but much more so for references that would otherwise
   be on the LHS. E.g., if we have a container for a sparse array and want
   to modify an element:
```
CALL sparse
```
   compared to
```
sparse(i,j) = x ! not implementable in Fortran 2018
```
5. Containers of pointers are problematic. Fortran lacks a robust
   mechanism to order pointers via some `COMPARE` method. This prevents any
   standard-conforming implementation of a Set of pointers or a Map that
   has keys that are pointers. Note that usual trick wrapping the pointers
   in a derived type does not help with this issue.

   This is a multi-faceted use case and there are a number of fronts on
   which the language could be improved to support containers. Any of the
   following would be of some value even without the others:

   1) Generic programming to facilitate user-implementation of containers
      that could be used with arbitrary types. (Ideally, community supported
      packages would then emerge a la C++ STL).
   2) Widen the class of overloaded "punctuation" to allow some form of
      paren/bracket notation for specifying container elements on both LHS and
      RHS of assignments.
   3) Make specific types containers first class language entities, a la
      existing arrays.
   4) Provide an intrinsic `COMPARE` that arbitrarily (but consistently)
      orders two pointers with the same declared type. Here there are a
      variety of Map and Vector containers that arise. In many cases, the
      entities in the container are polymorphic. I.e. the container allows
      entities that are any type that extends some base type. (And in at least
      one case the container entities are of unlimited polymorphic type.) Here
      are some specific containers in the package that are easier to explain:

### Use case: Block matrix linear algebra (0-0-0)

T. Clune

### Use case: Adaptive mesh refinement (0-0-0)

???

### Use case: ... ()

# Mini use cases

Mini use cases are narrow aspects derived from the full use cases in the
previous section. A given use case may provide numerous mini use cases,
and it is possible that only a subset of these will receive backing from
a majority of the committee and thereby feed into the requirements.

# Requirements

#### R:

This is a requirement

#### R:

This is another requirement

# Rejected Use cases
