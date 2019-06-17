================
CBOR omitted tag
================

Overview
========

This document specifies a CBOR [1] tag applied to CBOR Undefined (0xf7)
for indicating an omitted value (such as a gap in an array)::

  Tag: FIXME
  Data item: Undefined (0xf7)
  Semantics: Omitted value (e.g. gap in array)
  Point of contact: Sami Vaarala <sami.vaarala@iki.fi>
  Description of semantics: https://github.com/svaarala/duktape/blob/master/doc/cbor-omitted-tag.rst

NOTE: Ideally the assigned tag would fall in 32-255 because the tag may apply
to a lot of CBOR data (e.g. sparse arrays).

Semantics
=========

A CBOR Undefined (0xf7) tagged with the "omitted tag" defined here
indicates a normal CBOR Undefined value with the hint that the value
was omitted, e.g. there was a gap in an array being encoded.

Example
=======

The following ECMAScript array has two gaps in the middle::

  var arr = [ 'foo', , , 'bar' ]

One possible CBOR encoding for the array uses CBOR Undefined (0xf7) to
represent the gaps::

  84 63 66 6f 6f f7 f7 63 62 61 72
                 == ==

The gaps are encoded as CBOR Undefined (0xf7).  This encoding would decode
back into the following ECMAScript array, with ECMAScript ``undefined`` values
replacing the gaps::

  [ 'foo', undefined, undefined, 'bar' ]

With the "omitted" tag applied, the array could be encoded as::

  84 63 66 6f 6f FIXME f7 FIXME f7 63 62 61 72
                 ======== ========

and if the decoder understood the tag, it would decode back into the original
value containing the gaps.

Rationale
=========

Some programming languages such as ECMAScript differentiate between an
``undefined`` value and an omitted value.  For example, consider the following
ECMAScript arrays::

  var arr1 = [ 'foo', , , 'bar' ];
  var arr2 = [ 'foo', undefined, undefined, 'bar' ];

Both arrays have length 4.  The first array has two omitted values, at
indices 1 and 2, while the second array has explicit ECMAScript ``undefined``
values in these indices.  For the most part the two arrays behave the same;
for example::

  > print(arr1.length);
  4

  > print(arr2.length);
  4

  > print(arr1[1]);
  undefined

  > print(arr2[1]);
  undefined

However, there are also differences::

  > print(1 in arr1);
  false

  > print(1 in arr2);
  true

  > arr1.forEach(function (v, k) { print(k, v); });
  0 foo
  3 bar

  > arr2.forEach(function (v, k) { print(k, v); });
  0 foo
  1 undefined
  2 undefined
  3 bar

In some situations it may be preferable to preserve the omitted vs. undefined
difference so that an encoded value can be decoded without loss of information.
This allows, for example, better roundtripping of ECMAScript values.

A tag applied to CBOR Undefined (0xf7) seems to be the cleanest solution:

* It is backwards compatible with no need to update encoders/decoders.
  Adding a new simple value would affect all encoders and decoders, which
  seems disproportionate because the semantic difference between omitted
  and undefined only matters in certain environments.

* When the tag is ignored, the ordinary CBOR Undefined semantics remain
  useful, and in many situations the lost information is not significant.

References
==========

* [1] C. Bormann and P. Hoffman. "Concise Binary Object Representation (CBOR)".
  RFC 7049, October 2013.

Author
======

Sami Vaarala ``<sami.vaarala@iki.fi>``
