# Array

## 31. next permutation

核心思想是：从后往前找到第一个减小了的数（如果不存在直接reverse），如果存在记下标为mid；从后往前找第一个比它大的数，记下标为 new_mid，交换位于 mid 和 new_mid 的值，并将 mid 开始**后面**的数逆序。



