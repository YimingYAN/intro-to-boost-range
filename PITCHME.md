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
   std::cout << "Sum : " << sum << '\n';
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
#include <iterator>
#include <numeric>
#include <vector>
 
int main() {
   const auto numbers = std::vector<int>{ 1, 2, 3 ,4, 5 };
 
   std::vector<int> evenNumbers;
   const auto isEven = [](int n){return n % 2 == 0;};
   std::copy_if(numbers.cbegin(), numbers.cend(), std::back_inserter(evenNumbers), 
                isEven);
 
   std::vector<int> results;
   const auto multiplyBy2 = [](int n){return n*2;};
   std::transform(evenNumbers.cbegin(), evenNumbers.cend(), std::back_inserter(results), 
                  multiplyBy2);
    
   const auto sum = std::accumulate(results.cbegin(), results.cend(), 0); 
    
   std::cout << "Results: ";
   std::copy(results.cbegin(), results.cend(), 
             std::ostream_iterator<int>(std::cout, " "));    
   std::cout << "Sum : " << sum << '\n';
}
```

@[10-13]
@[15-18]
@[20]
@[23-24]

```
Results: 4 8 Sum : 12
```

---

## What are the @size[1.5em](@color[orange](problems)) with this piece of code?

+++

## What are the @size[1.5em](@color[orange](problems)) with STL algorithms?

+++

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

### The Range Libraries

* [Boost.Range](https://www.boost.org/doc/libs/1_67_0/libs/range/doc/html/index.html)
* [range-v3 by Eric Niebler](https://github.com/ericniebler/range-v3)

---

### Range

+++
#### Honestly, what is Range?

+++
* Can be traversed: ```begin``` and  ```end``` methods
* Two methods both return something that behaves like an iterator

```cpp
Range {
  begin()
  end()
}
```

+++ 

#### All STL containers are themselves ranges

+++

@snap[south]
Better level of abstraction
@snapend

#### Most of the time, what we want is a range
  ```cpp
      boost::range::transform(input, std::back_inserter(output), func);

      std::transform(input.begin(), input.end(), std::back_inserter(output), f);
  ```

---

### Behind the scenes: Smart iterators

+++

### Normal iterators
- Moving along the elements of the collection 
- Accessing the elements of the collection (deference)


+++

### “Smart” iterators

#### Customise one or both of these behaviours 

+++
### Smart iterators
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
- Customises the way it moves
- When advancing by one, advances its underlying iterator until it reaches an element that satisfies the predicate or the end of the collection.


---

### Adaptors

#### Combining ranges and smart iterators

+++ 

### Range ==> Adaptor ==> New range
* Initial adapted range remains unchanged
* Produced range does not contain elements
* No function evaluation involved yet

+++ 

```cpp
std::vector numbers = { 1, 2, 3, 4, 5 };

// function
auto range = boost::adaptors::transform(numbers, multiplyBy2);

// pipe
auto range = numbers | boost::adaptors::transformed(multiplyBy2);

// traverse the elements and applies the multiplyBy2
auto sum = boost::accumulate(range, 0); 
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
#include <iterator>

using boost::adaptors::filtered;
using boost::adaptors::transformed; 

int main() {
   const auto numbers = std::vector<int>{ 1, 2, 3 ,4, 5 };
 
   const auto isEven = [](int n){return n % 2 == 0;};
   const auto multiplyBy2 = [](int n){return n*2;};
   
   auto results = numbers | filtered(isEven) | transformed(multiplyBy2);
   const auto sum = boost::accumulate(results, 0); 
    
   std::cout << "Results: ";
   boost::range::copy(results, std::ostream_iterator<int>(std::cout, " "));
   std::cout << "Sum : " << sum << '\n';
}
```

@[19-20]

---

### Interesting Usages

---

### boost::combine
@snap[south-east]
Similar to python zip, but not quite
@snapend

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

```
(a, 1)
(b, 2)
(c, 3)
(d, 4)
(e, 5)
```

---
### boost::join
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

int main() {
    const auto vec = std::vector<char>{'G', 'P', 'R', 'O', 'M', 'S'};

    for (const auto & elem : boost::adaptors::index(vec, 0)) {
        std::cout << elem.index() << " : " << elem.value() << '\n';
    }
    return 0;
}
```

```
0 : G
1 : P
2 : R
3 : O
4 : M
5 : S
```

---

### boost::adaptors::map_keys and values
```cpp
#include <iostream>
#include <map>
#include <string>

#include <boost/algorithm/string/join.hpp>
#include <boost/range/adaptors.hpp>

const std::map<int, std::string> numbermap = {
    std::make_pair(1, "one"),
    std::make_pair(2, "two")
};

int main() {
    std::cout << "map keys: ";
    for (const auto & key : numbermap | boost::adaptors::map_keys) {
        std::cout << key << " ";
    }
    std::cout << '\n';

    std::cout << "map values: "
              << boost::algorithm::join(numbermap | boost::adaptors::map_values, " ")
              << '\n';

    return 0;
}

```

```
map keys: 1 2  
map values: one two
```

---

### Learn more and get help?

* [Range 2.0](https://www.boost.org/doc/libs/1_67_0/libs/range/doc/html/index.html): The Boost official doc
* [Blog from the author of range-v3](http://ericniebler.com/)  
* [Boost Range For Humans](https://greek0.net/boost-range/) : Full of examples 
* Not sure about the performance? Bench it at [Quick Bench](http://quick-bench.com)

---
### Credits

#### Jonathan Boccara's blogs 
* [Introduction to the C++ Ranges Library](https://www.fluentcpp.com/2018/02/09/introduction-ranges-library/)
* [Ranges: the STL to the Next Level](https://www.fluentcpp.com/2017/01/12/ranges-stl-to-the-next-level/)

#### Christian Aichinger's blog
* [Boost Range For Humans](https://greek0.net/boost-range/)  

---
### What else?

+++

@snap[middle]
[gitpitch](https://gitpitch.com/) is great for code presentation 
@snapend

---

## @size[1.5em](@color[grey](The End))

