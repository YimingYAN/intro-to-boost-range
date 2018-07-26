# Boost.Range

### An introduction to Boost.Range Library

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
   std::cout << '\n';
   std::cout << "Sum : " << sum << '\n';
}
```

```
Results: 4 8 
Sum : 12
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
   std::cout << '\n';
   std::cout << "Sum : " << sum << '\n';
}
```

@[1-4]
@[9-13]
@[15-19]
@[21-22]
@[25-26]

```
Results: 4 8 
Sum : 12
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

* Littered with iterators
* Violate the principle of respecting levels of abstraction
* Algorithms do not compose well: no easy way to combine ```transform``` and ```copy_if```, and there is no such thing as a “transform_if” 
---

### The Range Library

* Boost.Range
* [range-v3](https://github.com/ericniebler/range-v3)

---

### Range? 

* A range can be iterated over, which means that it has a ```begin``` and  ```end``` 
* ```begin``` and  ```end``` both return something that behaves essentially like an iterator

```cpp
Range {
  begin()
  end()
}
```

* All STL containers are themselves ranges
* Most of the time, what we want is a range, which corresponds better to the level of abstraction
  ```cpp
      boost::range::transform(input, std::back_inserter(output), func);
  ```
  VS
  ```cpp 
      std::transform(input.begin(), input.end(), std::back_inserter(output), f);
  ```

---

### Oh View!!

---

### Adapter?

---

### Algorithms !

---

### Example with Range

```cpp
#include <boost/range/>

#include <iostream>
#include <vector>
 
int main() {
   using boost::range::filtered;

   const auto numbers = std::vector<int>{ 1, 2, 3 ,4, 5 };
 
   // Filter on even numbers and multiply by 2
   auto results = numbers | filtered([](int n){ return n % 2 == 0; }) | transformed([](int n) { return n * 2; });
    
   // Get the sum 
   const auto sum = boost::accumulate(results, 0); 
    
   std::cout << "Results: ";
   boost::copy(results, std::ostream_iterator<int>(std::cout, " "));    
   std::cout << '\n';
   std::cout << "Sum : " << sum << '\n';
}
```


### Learn more and get help?


http://ericniebler.com/ The author of range-v3
https://greek0.net/boost-range/ Boost Range For Humans , Full of examples


### Credit 
https://www.fluentcpp.com/2018/02/09/introduction-ranges-library/
https://www.fluentcpp.com/2017/01/12/ranges-stl-to-the-next-level/
https://www.fluentcpp.com/2017/04/14/understand-ranges-better-with-the-new-cartesian-product-adaptor/

