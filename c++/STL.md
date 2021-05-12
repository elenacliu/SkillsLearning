用到过的一些 STL 用法。

# algorithm

1. upper_bound 和 lower_bound

   原数组必须是顺序递增的

   ! lower_bound 并不是求下界！

   返回值是**迭代器**！

   ```c++
   // 返回指向范围 [first, last) 中首个不小于（即大于或等于） value 的元素的迭代器，或若找不到这种元素则返回 last 。
   
   // comp	- 若第一参数小于（即先序于）第二个则返回 ​true 的二元谓词。
   // 谓词函数的签名应等价于如下：
   
   // bool pred(const Type1 &a, const Type2 &b);
   
   // 虽然签名不必有 const & ，函数也不能修改传递给它的对象，而且必须接受（可为 const 的）类型 Type1 与 Type2 的值，无关乎值类别（从而不允许 Type1 & ，亦不允许 Type1 ，除非 Type1 的移动等价于复制 (C++11 起)）。
   
   template< class ForwardIt, class T >
   constexpr ForwardIt lower_bound( ForwardIt first, ForwardIt last, const T& value );
   
   template< class ForwardIt, class T, class Compare >
   constexpr ForwardIt lower_bound( ForwardIt first, ForwardIt last, const T& value, Compare comp );
   
   template< class ForwardIt, class T >
   constexpr ForwardIt upper_bound( ForwardIt first, ForwardIt last, const T& value );
   
   template< class ForwardIt, class T, class Compare >
   constexpr ForwardIt upper_bound( ForwardIt first, ForwardIt last, const T& value, Compare comp );
   ```

2. nth_element

   期望复杂度是 O(n) 的，但是最差可以达到 O(n^2)

   ```c++
   // nth_element 是部分排序算法，它重排 [first, last) 中元素，使得：
   
   // nth 所指向的元素被更改为假如 [first, last) 已排序则该位置会出现的元素。
   // 这个新的 nth 元素前的所有元素小于或等于新的 nth 元素后的所有元素。
   
   template< class RandomIt >
   constexpr void nth_element( RandomIt first, RandomIt nth, RandomIt last );
   ```

3. partition

   其实就是对谓词 P，返回 true 的在返回 false 的前面。不保证稳定性。

   返回值：指向第二组元素首元素的迭代器。

   ```c++
   template< class ForwardIt, class UnaryPredicate >
   constexpr ForwardIt partition( ForwardIt first, ForwardIt last, UnaryPredicate p );
   ```

   ```c++
   template <class ForwardIt>
    void quicksort(ForwardIt first, ForwardIt last)
    {
       if(first == last) return;
       auto pivot = *std::next(first, std::distance(first,last)/2);
       ForwardIt middle1 = std::partition(first, last, 
                            [pivot](const auto& em){ return em < pivot; });
       ForwardIt middle2 = std::partition(middle1, last, 
                            [pivot](const auto& em){ return !(pivot < em); });
       quicksort(first, middle1);
       quicksort(middle2, last);
    }
   ```

4. next_permutation & prev_permutation

   O(n)

   至多 `(last-first)/2` 次交换。

   典型实现在排列的整个序列上，平均每次调用使用约 3 次比较和 1.5 次交换。

   ```c++
   template< class BidirIt >
   constexpr bool next_permutation( BidirIt first, BidirIt last );
   
   template< class BidirIt, class Compare >
   constexpr bool next_permutation( BidirIt first, BidirIt last, Compare comp );
   
   template< class BidirIt >
   constexpr bool prev_permutation( BidirIt first, BidirIt last);
   
   template< class BidirIt, class Compare >
   constexpr bool prev_permutation( BidirIt first, BidirIt last, Compare comp);
   ```

# string

1. 查找子串

   ```c++
   // 寻找等于 str 的首个子串
   constexpr size_type find( const basic_string& str,
                             size_type pos = 0 ) const noexcept;
   // 寻找等于范围 [s, s+count) 的首个子串。此范围能含空字符
   constexpr size_type find( const CharT* s,
                             size_type pos, size_type count ) const;
   
   // 返回值
   // 找到的子串的首字符位置，或若找不到这种子串则为 npos 。注意返回的是 size_t 而不是 iterator
   // std::string::npos == -1
   ```

2. 截取子串

   ```c++
   // 返回子串 [pos, pos+count) 。若请求的子串越过 string 的结尾，或若 count == npos ，则返回的子串为 [pos, size()) 
   constexpr basic_string substr( size_type pos = 0, size_type count = npos ) const;
   ```

3. 字符串比较

   ```c++
   constexpr int compare( const basic_string& str ) const noexcept;
   constexpr int compare( size_type pos1, size_type count1,
                          const basic_string& str ) const;
   constexpr int compare( size_type pos1, size_type count1,
                          const basic_string& str,
                          size_type pos2, size_type count2 = npos ) const;
   // 1) 比较此 string 与 str 。
   // 2) 比较此 string 的 [pos1, pos1+count1) 子串与 str 。若 count1 > size() - pos1 则子串为 [pos1, size()) 。
   // 3) 比较此 string 的 [pos1, pos1+count1) 子串与 str 的子串 [pos2, pos2+count2) 。若 count1 > size() - pos1 则第一子串为 [pos1, size()) 。类似地若 count2 > str.size() - pos2 则第二子串为 [pos2, str.size()) 。
   
   // 返回值
   // 若 *this 在字典序中先出现于参数所指定的字符序列，则为正值。
   
   // 若两个序列比较等价则为零。
   
   // 若 *this 在字典序中后出现于参数所指定的字符序列，则为负值。
   
   // 异常
   // 接收名为 pos1 或 pos2 的参数的重载若参数在范围外则抛出 std::out_of_range 。
   ```

4. insert

   ```c++
   // Inserts string str at the position index
   constexpr basic_string& insert( size_type index, const basic_string& str );
   // Inserts a string, obtained by str.substr(index_str, count) at the position index
   constexpr basic_string& insert( size_type index, const basic_string& str,
                                   size_type index_str, size_type count = npos);
   ```

5. to_string

   ```c++
   std::string to_string( int value );
   (1)	(since C++11)
   std::string to_string( long value );
   (2)	(since C++11)
   std::string to_string( long long value );
   (3)	(since C++11)
   std::string to_string( unsigned value );
   (4)	(since C++11)
   std::string to_string( unsigned long value );
   (5)	(since C++11)
   std::string to_string( unsigned long long value );
   (6)	(since C++11)
   std::string to_string( float value );
   (7)	(since C++11)
   std::string to_string( double value );
   (8)	(since C++11)
   std::string to_string( long double value );
   (9)	(since C++11)
   ```

   

# map

面试官：stl的map是基于红黑树实现的，那么为什么要基于红黑树？

其余的选择：跳转表、avl、BST、Splay 等等

## hashmap & treemap

