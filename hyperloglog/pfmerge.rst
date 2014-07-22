.. _pfmerge:

PFMERGE
===============

**PFMERGE destkey sourcekey [sourcekey ...]**

Merge multiple HyperLogLog values into an unique value that will approximate the cardinality of the union of the observed Sets of the source HyperLogLog structures.

The computed merged HyperLogLog is set to the destination variable, which is created if does not exist (defauling to an empty HyperLogLog).

**可用版本：**
    >= 2.8.9

**时间复杂度：**
    O(N) to merge N HyperLogLogs, but with high constant times.

**返回值：**
    Simple string reply: The command just returns OK.

::

    redis> PFADD hll1 foo bar zap a
    (integer) 1

    redis> PFADD hll2 a b c foo
    (integer) 1

    redis> PFMERGE hll3 hll1 hll2
    OK

    redis> PFCOUNT hll3
    (integer) 6
