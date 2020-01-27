---
date: 2020-01-27T09:04:21Z
title: How I do field research for Quantum Resistant Cryptography
tags:
    - PHd
    - PQC
---

After starting my PHd I've had to develop a way to find relevant papers. In
this blog post I'll share this information so that it can be quickly referenced
by me, and all those that might need to do similar research. Some of it could
probably be used by others as well.


## The easiest first

When diving into a new topic the first place I usually go looking is [Google
Scholar][gs] due to the breadth of publications it searches. A similar place I
look at is the [DBLP][dblp] which is the same idea but from a german
university. The DBLP uses a different source of information than Google
Scholar. So if you want to find all possible papers/publications you will need
to have used both.

## Going straight to the Source

After having found the first few papers, I make sure to find all relevant
conferences so that I do not miss anything. This list is very specific to
Quantum Resistant Cryptography so will be completely different for other
subjects.

- [PQCrypto](https://pqcrypto.org/)
- [Asiacrypt](https://www.iacr.org/meetings/asiacrypt/)
- [Crypto](https://www.iacr.org/meetings/crypto/)
- [Eurocrypt](https://www.iacr.org/meetings/eurocrypt/)
- [Latincrypt](https://latincrypt2019.cryptojedi.org/index.shtml)
- [CHES](https://iacr.org/meetings/ches/)
- [Africacrypt](http://africacrypt2020.org/index.html)
- [COSADE](http://www.cosade.org/)

Of course, you also have to check out the previous years. Depending on the
topic you are looking into, this can be quite time consuming. But that's how it
is.

## Following the Trail

After having identified a few papers, it is time to mark them down in your
favorite software. In my case I'm using [Zotero](https://www.zotero.org/) but I
just use it as a quick library. Once you have those saved or otherwise in quick
access it is time to read through them, and find out if they are truly
relevant. If they are, check them out on DBLP/Google Scholar and look at the
"References/Referenced By" lists. They are crucial to understand the web of
knowledge that you want to dive into. Often, there are foundational papers that
will be cited often. Those could be useful to understand to get a deep look
into the topic. Either way, you then continue diving into the references of
those references etc... until you feel like you have a good overview.
Don't forget to add those to your collection, after all you can delete them
later if they are not relevant.

## Conclusion

Using this technique I found that I managed to quickly dive into the subject of
isogeny based quantum resistant cryptography. For different subjects there
might be different steps to take, but I do believe that the core idea could
stay the same. I hope you found it useful!

[gs]: https://scholar.google.com/
[dblp]: https://dblp.uni-trier.de/
