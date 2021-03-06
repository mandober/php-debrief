# Lists in LISP

http://www.faqs.org/faqs/lisp-faq/part2/section-9.html


## Cons list

https://en.wikipedia.org/wiki/Cons


cons cell:
┌─────┬─────┐
│ CAR │ CDR │
└─────┴─────┘

In normal list structure, each element of the list is represented as a CONS cell, which basically consists of two halfs: two fields, called the CAR and the CDR, with each having the same capacity (capacity to store a pointer, which is 32 bits or 4 bytes @ x86m or 64 bits or 8 bytes @ x86_64).

In a most basic list, the `CAR` points to the payload value; the `CDR `points to the next CONS cell in the list or to the special, terminator value, `NIL`, that marks the end of a list.

## CDR-coding

CDR-coding is a space-saving way to store lists in memory. It is normally only used in Lisp implementations that run on processors that are specialized for Lisp, as it is difficult to implement efficiently in software.

CDR-coding takes advantage of the fact that most CDR cells point to another CONS cell (as opposed to a direct value, in a pair), and further that the entire list is often allocated at once (e.g. by a call to `LIST`).

Instead of using two pointers to implement each CONS cell, the CAR cell contains a pointer and a two-bit "CDR code". The CDR code may contain one of three values: CDR-NORMAL, CDR-NEXT, CDR-NIL.

If the code is `CDR-NORMAL`, this cell is the first half of an ordinary CONS cell pair, and the next cell in memory contains the CDR pointer, as the normal CONS cell does.

If the CDR code is `CDR-NEXT`, the next cell in memory contains the next CAR cell; in other words, the CDR pointer is implicitly `this_address + 1`, where
`this_address` is the memory address of the CAR cell.

If the CDR code is `CDR-NIL`, then this cell is the last element of the list; the CDR pointer is implicitly a reference to the object NIL.


When a list is constructed incrementally using CONS, a chain of ordinary pairs is created; however, when a list is constructed in one step using `LIST` or `MAKE-LIST`, a block of memory can be allocated for all the CAR cells, and their CDR codes all set to CDR-NEXT (except the last, which is CDR-NIL), and the list will only take half as much storage (because all the CDR pointers are implicit).

If this were all there were to it, it would not be difficult to implement in software on ordinary processors; it would add a small amount of overhead to the CDR function, but the reduction in paging might make up for it.

But the problem arises when a program uses RPLACD on a CONS cell that has a CDR
code of CDR-NEXT or CDR-NIL. Normally RPLACD simply stores into the CDR cell of a CONS, but in this case there is no CDR cell - its contents are implicitly specified by the CDR code, and the word that would normally contain the CDR pointer contains the next CONS cell (in the CDR-NEXT case) to which other data structures may have pointers, or the first word of some other object (in the CDR-NIL case).

When CDR-coding is used, the implementation must also provide automatic "forwarding pointers"; an ordinary CONS cell is allocated, the CAR of the original cell is copied into its CAR, the value being RPLACD'ed is stored into its CDR, and the old CAR cell is replaced with a forwarding pointer to the new CONS cell.

Whenever CAR or CDR is performed on a CONS, it must check whether the location contains a forwarding pointer. This overhead on both CAR and CDR, coupled with the overhead on CDR to check for CDR codes, is generally enough that using CDR codes on conventional hardware is infeasible.

There is some evidence that CDR-coding doesn't really save very much memory, because most lists aren't constructed at once, or RPLACD is done on them enough times that they don't stay contiguous. At best, this technique can save 50% of the space occupied by CONS cells. However, the savings probably depends to some extent upon the amount of support the implementation provides for creating CDR-coded lists.

For instance, many system functions on *Symbolics Lisp Machines* that operate on lists have a `:LOCALIZE` option; when `:LOCALIZE T` is specified, the list is first modified and then copied to a new, CDR-coded block, with all the old cells replaced with forwarding pointers. The next time the garbage collector runs, all the forwarding pointers will be spliced out. Thus, at a cost of a temporary increase in memory usage, overall memory usage is generally reduced because more lists may be CDR-coded.

There may also be some benefit in improved paging performance due to increased locality as well (putting a list into CDR-coded form makes all the "cells" contiguous). Nevertheless, modern Lisps tend to use lists much less frequently, with a much heavier reliance upon code, strings, and vectors (structures).
