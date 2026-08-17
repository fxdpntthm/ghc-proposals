
Uniform Template Haskell and Rebindable Syntax
===============================================

.. author:: Apoorv Ingle (@fxdpntthm)
.. date-accepted:: 
.. ticket-url::
.. implemented::
.. highlight:: haskell
.. header:: This proposal is `discussed at this pull request <https://github.com/ghc-proposals/ghc-proposals/pull/0>`_.
            **After creating the pull request, edit this file again, update the
            number in the link, and delete this bold sentence.**
.. sectnum::
.. contents::

This proposal makes the behaviour of typed and untyped template haskell with rebindable syntax uniform.

Motivation
----------
The behaviour of typed template haskell and untyped template haskell is not uniform when used with rebindable syntax.

Consider the following code split across two modules. 
The `Lib` modules declares `int_QuoteTTH` and `int_QuoteUTH`. Both quote the same if-then-else expression with a local 
definition of `ifThenElse`.

::

  {-# Language TemplateHaskell #-}
  {-# Language RebindableSyntax #-}
  module Lib where 

  import Language.Haskell.TH.Syntax

  ifThenElse :: Bool -> Int -> Int -> Int
  ifThenElse _ a b = a + b

  int_QuoteTTH :: Code Q Int
  int_QuoteTTH = [|| if True then 10 else 15 ||]

  int_QuoteTTH :: Quote m => m Exp
  int_QuoteUTH = [| if True then 10 else 15 |]

The second module `Client` splices in the two declarations defined in the `Lib` module. 

::

  {-# Language TemplateHaskell #-}
  module Client where
  import Lib 

  t1 :: Int 
  t1 = $$(int_QuoteTTH)

  t2 :: Int
  t2 = $(int_QuoteUTH)

  main :: IO ()
  main = do 
    print t1 -- 25
    print t2 -- 10


The current behaviour is justified as follows: 
For typed template haskell quotes (`t1`), the type checking of the expression is performed at definition site. Thus, the expression 
`[|| if True then 10 else 15 ||]`` is typechecked after renaming at the definition (in `Lib` module) thus, with the 
rebindable syntax flag enabled, the expression is transformed to `ifThenElse True then 10 else 15` while splicing in `Client` module.

On the other hand, for untyped template haskell quotes (`t2`) the expression `[| if True then 10 else 15 |]` is first spliced in `Client` module
and then typechecked. As rebindable syntax is turned off in the `Client` module, the splice is evaluated to `if True then 10 else 15`.


Proposed Change Specification
-----------------------------

Rebindable syntax disambiguation happens at splice time for both typed template haskell and untyped template haskell.

In the above `Client` module, `t1` will be spliced in as `if True then 10 else 15` just as `t2`. They will both have the same value at evaluation.

Proposed Library Change Specification
-------------------------------------

No changes.

Examples
--------
This section illustrates the specification through the use of examples of the
language change proposed. It is best to exemplify each point made in the
specification, though perhaps one example can cover several points. Contrived
examples are OK here. If the Motivation section describes something that is
hard to do without this proposal, this is a good place to show how easy that
thing is to do with the proposal.

Effect and Interactions
-----------------------

The proposal will also make the interaction between `OverloadedLists` and `TemplateHaskell` uniform wrt typed and untyped quotes.


Costs and Drawbacks
-------------------
It is possible for typed template haskell quotes fail to type check during splice time. (Check this)


Backward Compatibility
----------------------
The following change is backwards incompatible and may change the behaviour of the programs
However, `TemplateHaskell` and `RebindableSyntax` are two language features that rarely used together hence 
breakage is expected to happen very rarely.

The advantae is that the proposed change significantly simplifies the compiler implementation and also makes the behaviour uniform reducing
overall complexity for the user and the compiler implementor.

Alternatives
------------


 
List alternative designs to your proposed change. Both existing
workarounds, or alternative choices for the changes. Explain
the reasons for choosing the proposed change over these alternative:
*e.g.* they can be cheaper but insufficient, or better but too
expensive. Or something else.

The PR discussion often raises other potential designs, and they should be
added to this section. Similarly, if the proposed change
specification changes significantly, the old one should be listed in
this section.

Unresolved Questions
--------------------
Explicitly list any remaining issues that remain in the conceptual design and
specification. Be upfront and trust that the community will help. Please do
not list *implementation* issues.

Hopefully this section will be empty by the time the proposal is brought to
the steering committee.


Implementation Plan
-------------------
(Optional) If accepted who will implement the change? Which other resources
and prerequisites are required for implementation?

Endorsements
-------------
(Optional) This section provides an opportunity for any third parties to express their
support for the proposal, and to say why they would like to see it adopted.
It is not mandatory for have any endorsements at all, but the more substantial
the proposal is, the more desirable it is to offer evidence that there is
significant demand from the community.  This section is one way to provide
such evidence.

Acknowledgments
---------------

(Optional) This section provides an opportunity to say thanks
to third parties for their contributions to the proposal.
