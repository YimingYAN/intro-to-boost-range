# Ranges in C++ 

@color[gray](By Example)

---

### Base example... Ouch!

```cpp
#include <iostream>
#include <vector>
 
int main() {
   const auto numbers = std::vector<int>{ 1, 2, 3 ,4, 5 };
 
   auto results = std::vector<int>{};
   auto sum = 0;
   for(int num : numbers) {
       // Check if even
       if (num % 2 == 0) {
           // If even, multiply by 2
           const auto product = num*2; 
           results.push_back(product);
           // Add to the sum 
           sum += product;
       }
   }
 
 
   std::cout << "Results: ";
   for (int ii : results) {
       std::cout << ii << ' '; 
   } 
   std::cout << " Sum : " << sum << '\n';
}
```

```
Results: 4 8 Sum : 12
```

---
### Of course, STL!

```cpp
#include <algorithm>
#include <iostream>
#include <numeric>
#include <vector>
 
int main() {
   const auto numbers = std::vector<int>{ 1, 2, 3 ,4, 5 };
 
   // Filter on even numbers
   std::vector<int> evenNumbers;
   std::copy_if(numbers.cbegin(), numbers.cend(), 
                std::back_inserter(evenNumbers), 
                [](int n){ return n % 2 == 0; });
 
   // Multiply by 2 for each even number
   std::vector<int> results;
   std::transform(evenNumbers.cbegin(), evenNumbers.cend(), 
                std::back_inserter(results), 
                [](int n){ return n * 2; });
    
   // Get the sum 
   const auto sum = std::accumulate(results.cbegin(), results.cend(), 0); 
    
   std::cout << "Results: ";
   std::copy(results.cbegin(), results.cend(), 
             std::ostream_iterator<int>(std::cout, " "));    
   std::cout << "Sum : " << sum << '\n';
}
```

@[1-4]
@[9-13]
@[15-19]
@[21-22]
@[25-26]

```
Results: 4 8 Sum : 12
```

---

## What are the @size[1.5em](@color[orange](problems)) with this piece of code?

---

```cpp
   std::copy_if(numbers.cbegin(), numbers.cend(), std::back_inserter(evenNumbers), [](int n){ return n % 2 == 0; });
   std::transform(evenNumbers.cbegin(), evenNumbers.cend(), std::back_inserter(results), [](int n){ return n * 2; });
   std::accumulate(results.cbegin(), results.cend(), 0); 
   std::copy(results.cbegin(), results.cend(), std::ostream_iterator<int>(std::cout, " "));    
```

* Littered with iterators (```begin``` and ```end```)
* Violate the principle of respecting levels of abstraction
* Algorithms do not compose well: no easy way to combine ```transform``` and ```copy_if```, and no such thing as “transform_if” 
---

### The Range Library

* [Boost.Range](https://www.boost.org/doc/libs/1_67_0/libs/range/doc/html/index.html)
* [range-v3 by Eric Niebler](https://github.com/ericniebler/range-v3)

---

### Range? 

* Can be traversed: ```begin``` and  ```end``` methods
* ```begin``` and  ```end``` both return something that behaves like an iterator

```cpp
Range {
  begin()
  end()
}
```

--- 

### Range

#### All STL containers are themselves ranges

+++

#### Most of the time, what we want is a range, which corresponds better to the level of abstraction
  ```cpp
      boost::range::transform(input, std::back_inserter(output), func);

      std::transform(input.begin(), input.end(), std::back_inserter(output), f);
  ```

---

### Behind the scenes: Smart iterators

+++

### Normal iterator
- Moving along the elements of the collection 
- Accessing the elements of the collection (deference)


+++

### “Smart” iterators: customise one or both of these behaviours 
* Iterator ```itr```  and a function (or function object) ```func```
* For instance:
  - ```transform_iterator```
  - ```filter_iterator```

+++
### ```transform_iterator``` 
- Customises the way it accesses elements
- When dereferenced, applies ```func``` to ```*itr``` and returns the result.

+++
### ```filter_iterator``` 
- Customises the way its moves
- When advancing by one, advances its underlying iterator until it reaches an element that satisfies the predicate or the end of the collection.


---

### Adaptors

@snap[south-east]
Range ==> Adaptor ==> New range
@snapend

* Initial adapted range remains unchanged
* Produced range does not contain elements
* No function evaluation involved yet

+++ 

```cpp
std::vector numbers = { 1, 2, 3, 4, 5 };

// function syntax
auto range = boost::adaptors::transform(numbers, multiplyBy2);

// pipe syntax
auto range = numbers | boost::adaptors::transformed(multiplyBy2);

// traverse the elements and applies the multiplyBy2
auto sum = boost::accumulate(numbers, 0); 
```

---


### Back to the Example with Range

```cpp
#include <boost/range/adaptor/filtered.hpp>
#include <boost/range/adaptor/transformed.hpp>
#include <boost/range/algorithm/copy.hpp>
#include <boost/range/numeric.hpp>

#include <iostream>
#include <vector>

using boost::adaptors::filtered;
using boost::adaptors::transformed; 

int main() {
   const auto numbers = std::vector<int>{ 1, 2, 3 ,4, 5 };
 
   const auto isEven = [](int n){return n % 2 == 0;};
   const auto multiplyBy2(int n){return n*2;};
   
   auto results = numbers | filtered(isEven) | transformed(multiplyBy2);
   const auto sum = boost::accumulate(results, 0); 
    
   std::cout << "Results: ";
   boost::range::copy(results, std::ostream_iterator<int>(std::cout, " "));
   std::cout << '\n';
   std::cout << "Sum : " << sum << '\n';
}
```

@[18-19]

---

### Interesting Usages

---

### boost/range/combine.hpp

```cpp
#include <iostream>
#include <vector>
#include <string>

#include <boost/range/combine.hpp>

int main() {
    const auto str = std::string("abcde");
    const auto vec = std::vector<int>{1, 2, 3, 4, 5};

    for (const auto & zipped : boost::combine(str, vec)) {
        char c; int i;
        boost::tie(c, i) = zipped;

        std::cout << "(" << c << ", " << i << ")\n";
    }

    return 0;
}
```
+++ 

Output: 

```
(a, 1)
(b, 2)
(c, 3)
(d, 4)
(e, 5)
```

---
### boost/range/join.hpp
```cpp
#include <iostream>
#include <vector>
#include <string>

#include <boost/range/join.hpp>


int main() {
    const auto str = std::string("abcde");
    const auto vec = std::vector<char>{'A', 'B', 'C', 'D', 'E'};

    // join() takes two ranges and concatenates them to a single range.
    for (const auto & c : boost::join(str, vec)) {
        std::cout << c;
    }
    std::cout << '\n';

    return 0;
}
```
Output:
```
abcdeABCDE
```

---

### boost::adaptors::index
@snap[south-east]
Similar to python enumerate
@snapend

```cpp
#include <iostream>
#include <string>
#include <vector>

#include <boost/range/adaptor/indexed.hpp>

const auto vec = std::vector<char>{'G', 'P', 'R', 'O', 'M', 'S'};

int main() {
    for (const auto & elem : boost::adaptors::index(vec, 0)) {
        std::cout << elem.index() << " : " << elem.value()
                  << '\n';
    }
    return 0;
}
```

+++ 

Output:
```
0 : G
1 : P
2 : R
3 : O
4 : M
5 : S
```

---

### Learn more and get help?

* [Range 2.0](https://www.boost.org/doc/libs/1_67_0/libs/range/doc/html/index.html): The Boost official doc
* [Blog from the author of range-v3](http://ericniebler.com/)  
* [Boost Range For Humans](https://greek0.net/boost-range/) : Full of examples 
* Not sure about the performance? Bench it at [Quick Bench](http://quick-bench.com)

---
### Credits

#### Jonathan Boccara's blog 
* [Introduction to the C++ Ranges Library](https://www.fluentcpp.com/2018/02/09/introduction-ranges-library/)
* [Ranges: the STL to the Next Level](https://www.fluentcpp.com/2017/01/12/ranges-stl-to-the-next-level/)
* [Boost Range For Humans from Christian Aichinger](https://greek0.net/boost-range/)  

