<pre class='metadata'>
Title: Everyone Deserves a Little Order TODO
Status: D
Audience: LEWG, LWG
Editor: Gašper Ažman <gasper.azman@gmail.com>
Editor: Jeff Snyder <jeff-isocpp@caffeinated.me.uk>
Shortname: P0891
Abstract: The specification of ordering algorithms at the end of [[P0768R1]] does not provide the ability to provide a default order for user-defined types (since they are specified in such a way that they are not intended to be customization points), and yet mixes in such a customization for iec559 floating point types. This paper suggests splitting that capability out into a separate customization point.
Group: WG21
Date: 2018-10-27
Markup Shorthands: markdown yes
Revision: 1
ED: https://github.com/atomgalaxy/a-little-order/strong-ordering.bs
</pre>

Design guidelines
=================

- `*_order` algorithms are just a `<=>` with another name
- they behave consistently with `<=>`
- as `<=>`, they are customization points (in walter's paper meaning) (p0551)
- in the absence of their definition, the standard provides a fallback that participates in overload resolution:
  - if the result of `<=>` is convertible to the particular ordering
  - or if a stronger ordering function is defined (it falls back to it).
- fallback functions are not cool enough to have these names - the names should belong to the actually useful cust points.
- THERE IS MORE FUTURE THAN THERE IS PAST
  - so we should cater to the future while not leaving the past hanging
- We infer weak ordering from JUST `<` so we can always do that.

- and we provide overloads for `strong_order` for iec559 types.

- fallbacks are a separable section
- suite of overloads for standard types

Fallbacks:
- when are they useful:
  - when you are assuming that (old ops) have the specified order and you preferentially use <=> if available.
- `assume_*_order(T c&, T c&)` -  means: if `*_order` defined, use that, otherwise assume `<` and `==` do the right thing, because you are *asserting* that.

Bikeshed for fallbacks:
- `*_3way_compare`

TODO:
- naming of fallbacks
- wording after barry
- provide overloads of `*_order` for all containers.
- provide all overloads for all the standard types:
  - (this is separable)
  - `optional`
  - `expected`
  - `tuple`
  - `pair`
  - `variant`
  - `vector`
  - `deque`
  - `list`
  - `forward_list`
  - `set`
  - `map`
  - `multiset`
  - `multimap`
  - `array`
  - `stack`
  - `queue`

- bikeshedding section on naming fallbacks

- take wording for fallbacks from Jeff's p0863
- integrate with Barry's ordering fallback

```cpp
template <typename T>
struct __XXX_ordering_wrapper /* exposition only */ {
  T const& __value; // exposition only
  XXX_order operator<=>(__cmp_fallback_wrapper const&) = default; // d1186r1 gens the 'asserted' comparison category from < and =
};
template <typename T>
XXX_order XXX_ordering(T const& __x, T const& __y) requires () {
  return __XXX_ordering_wrapper{x} <=> __XXX_ordering_wrapper{y};
}
```

New papers:
- `strong_order(complex)`
- Relax the stupid restriction for `complex<T>` for T to be an iec type. I WANT MY FUCKING GAUSSIAN INTEGERS
- all library <=> (now separable section of this paper), also look 
- NO: synthesize `strong_order` iff members have it
  - provide magic library function that uses reflection to do this.

Propose common ordering objects:

```
namespace std {
  struct strong {
    static constexpr auto less_v = [](auto const& x, auto const& y) {
        return is_lt(strong_order(x, y));
    };
    using less = decltype(less_v);

    struct less {
      template <typename T, typename U> bool operator()(T c& x, U c& y) {
      }
    };
    struct greater { template <typename T> bool op(T c&, T c&); };
    struct equal { template <typename T> bool op(T c&, T c&); };
    struct neq { template <typename T> bool op(T c&, T c&); };
    struct cmp { ... <=>; }
  };
}

std::set<T, std::strong::less>

```

