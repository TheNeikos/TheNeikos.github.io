---
date: 2014-10-16T13:46:21Z
title: 'Going into Resources (Games #2)'
url: /Going_into_Resources_(Games_-2).html
---

I have talked with a friend of mine (who is rather knowleadgable) about an
implementation of a resource system in my game. He offered me interesting
insight into two ways of calculating and keeping resources up to date.

Absolute
--------

This was what I thought I would end up implementing. In pseudocode this could
look like this:

```
    for each building
        for each resource of the building
            add/remove resource of given type
        end
    end
```

This means that for each update a transaction is created that calculates the
current demand and output and works through it. This has several limitations,
it doesn't scale well. Imagine dozens of buildings in hundreds of villages.
That is a lot to calculate *every second* (the time of one turn). This is when
he suggested the alternative.

Streaming
---------

The pseudocode for this could look like this:

On adding/removing a building
```
    resource table
    for each building
        for each resource of the building
            add/remove resource and insert into resource table
        end
    end
    save resource table
```
Then, each update loop do this:

```
    add resource table to current resources
```

This means that it is a really fast operation as everything has been calculated
in advance. It also means that you can have non instant construction, and for
example build for 1 minute a large house. Then pause and finish building
something else and then go back to the house. As this means that you only have
to do time consuming calculations if things change (rather than every second)
you are saving on time and thus have lesser problems scaling.



Resources
---------

These are the units that make the game work. Some have a limited supply, others
are plentiful but sometimes hard to acquire.

### Wood

Wood is the most basic element. It's used pretty much everywhere. It burns hot
and is pretty sturdy. So it is used in refining resources, and buildings. Wood
is gotten from the local forests.

### Stone

Stone is also one of the basic element. Mainly used for buildings, you need it
for larger buildings as stone foundations are a must in that regard.

### Iron

Iron is the first of the lesser metals to be thorougly understood. Once you can
forge it a whole new series of advancements open up to you.

### Copper

Copper is also one of the lesser metals. It is not yet well understood, however
magical uses have been reported throughout the world. Spending effort on
researching this might be worth it.
