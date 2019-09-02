# Identifying Incorrect or Zero Memory Allocation Bugs in C

One of the common cases of heap related bugs are cases where arithmetic operations are performend on parameter of memory allocation functions such as malloc(). In certain cases, the arithmetic operations can lead to integer overflow (resulting in less memory being allocated) or could outrightly be zero (cases where the arithmetic operation computes to zero). These may lead to vulnerabilities that could be exploited. In this example, we use Ocular to decipher if such a condition exists using 3 allocation scenarios.
