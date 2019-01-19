For types that endeavour to be Regular:

== defines what 'strong equality' means
<=> == 0  iff == means <=> should return strong ordering
Otherwise it should return weak ordering

Algorithms in general should require (and call) `weak_order`.
If your type does not provide an order, you should still provide an overload of
`strong_order`.

If types obey this, '= default' on both == and <=> work correctly.
