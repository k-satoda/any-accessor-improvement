DRAFTING


Preface:

  The latest TS.
  http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4529.html#any.nonmembers

Problem:

  - While pointer version is used with ValueType without qualifications
    of pointer type and constness, reference version needs care of
    qualifications for reference type and constness.
    This inconsitency leads to missing qualification with reference
    version and it can result in unnecessary copy. (Or, some
    compile-time errors about unmatching qualification with operand
    which are just irritating but are not problem at runtime.)

    In practice, an occurence of any_cast<Heavyweight> catch my eyes,
    but it is OK if the function argument is a pointer. Then it is left
    in the code and increase mixture with qualified uses of reference
    version.

  - (hypothetical)
    In reverse, with knowledge that reference version should be used
    with qualification like all of built-in cast operators, it is
    possible that pointer version is mis-used with qualification. In
    this case, it can results in silent failure; getting a null pointer
    and it is treated as the case where operand was null or *operand
    didn't contain the expected type. Dedicated test cases are needed to
    ensure no such errors exist.

Proposal:

  Change return types of reference versions of any_cast, from:

    template<class ValueType>
      ValueType any_cast(const any& operand);
    template<class ValueType>
      ValueType any_cast(any& operand);
    template<class ValueType>
      ValueType any_cast(any&& operand);

  to:

    template<class ValueType>
      const ValueType& any_cast(const any& operand);
    template<class ValueType>
      ValueType& any_cast(any& operand);
    template<class ValueType>
      ValueType&& any_cast(any&& operand);


Sidenotes:

  - Adding new interface (member function template get<> or such) was
    considered but turned down because any new interface to make a cast
    spoils grep-ability of any_cast, and misses obvious relation with
    bad_any_cast.
