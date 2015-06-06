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

  Add these member function templates.
  (Note that the names are still subject to bikeshedding here.)

  class any
  {
  public:

    ...

    template<class ValueType> const ValueType& ref() const;
    template<class ValueType> ValueType& ref() &;
    template<class ValueType> ValueType&& ref() &&;

    template<class ValueType> const ValueType* ptr() const noexcept;
    template<class ValueType> ValueType* ptr() noexcept;
  };

  Users just write the expected type to be contained in a value of any,
  and both value category and constness are preserved (or possibly just
  constified).


Naming:

  T& of_type(), T* get()

    [ Example:
      any x(5);                                   // x holds int
      assert(x.of_type<int>() == 5);              // cast to reference and read it
      x.of_type<int>() = 10;                      // cast to reference and store to it
      assert(x.of_type<int>() == 10);

      x = "Meow";                                 // x holds const char*
      assert(strcmp(x.of_type<const char*>(), "Meow") == 0);
      x.of_type<const char*>() = "Harry";
      assert(strcmp(x.of_type<const char*>(), "Harry") == 0);

      x = string("Meow");                         // x holds string
      string s, s2("Jane");
      s = move(x.of_type<string>());              // move from any
      assert(s == "Meow");
      x.of_type<string>() = move(s2);             // move to any
      assert(x.of_type<string>() == "Jane");

      string cat("Meow");
      const any y(cat);                           // const y holds string
      assert(y.of_type<string>() == cat);

      y.of_type<string>() = "";                   // error; cannot assign
                                                  //  to const string&
    - end example ]

    [ Example:
      bool is_string(const any& operand) {
        return operand.get<string>(); // != nullptr
      }
    - end example ]

