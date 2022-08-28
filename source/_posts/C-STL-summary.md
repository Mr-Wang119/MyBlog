---
title: C++ STL summary
date: 2022-08-26 15:13:03
tags: C++
---

# Summary

The idea of STL (Standard Template Library) is to develop the interface for such a container once, and then use everywhere for data of any type.

The type of a container is the tempate parameter. Template parameters are specified with the '<' '>' brackets in code.

```c++
vector<int> N;
```

<!--more-->

# Vector

Vector is an array with extended functionality.

If you want to know whether the container is empty, you're better off using empty() function:

```c++
bool is_nonempty = (v.size() >= 0); // Try to avoid this
bool is_nonempty = !v.empty();
```

This is because not all the containers can report their size in O(1), and you should not require counting all elements in a double-linked list just to ensure that it contains at least one.

When using  ```push_back()``` ,vector will not allocate just one element each time. Instead, vector allocates more memory then it actually needs when adding new elements.

The ```resize()``` function makes vector contain the required number of elements. If you require less elements than vector already contain, the last ones will be deleted.

If you use ```push_back()``` after ```resize()```, it will add elements AFTER the newly allocated size, but not INTO it.

When vector is passed as a parameter to some function, a copy of vector is actually created. It may take a lot of time and memory to create new vectors when they are not really needed. Using reference instread.

# Pairs

The greate advantage of pairs is that they have built-in operations to compare themselves. Pairs are compared first-to-second element.

# Iterators

In STL iterators are the most general way to access data in containers. Objects with take value (unary *), comparison, and increment/decrement(++/--) that are associated with containers are called iterators. Any STL container may be traversed by means of an iterator.

The following operations are defined for iterators:

- get value of an iterator, int x = *it;
- increment and decrement iterators it1++, it2--;
- compare iterators by '!=' and by '<'
- add an immediate to iterator it += 20; <=> shift 20 elements forward
- get the distance between iterators, int n = it2 - ti1;

# Set

Set can add, remove and check the presence of particular element in O(log N), where N is the count of objects in the set. A count of the elements in the set, N, is returned in O(1).

There are several `find()`s in STL. There is a global algorithm `find()`, which takes two iterators, element, and works for O(N). It is possible to use it for searching for element in set, but why use an O(N) alogrithm while there exists an O(log N) one? Another algorithm that works for O(log N) while called as member function is `count()`.

# Map

While `map::find()` will never change the contents of map, operator [] will create an element if it does not exist. This is why operator [] may not be used if map is passed as a const reference parameter to some function.

# Internally

Internally map and set are almost always stored as red-black trees. The elements of map and set are always sorted in ascending order while traversing these containers. 

# Algorithms

Set ans map have the member functions `find()` and `count()`, which works in O(log N), while `std::find()` and `std::count()` take O(N).

# String Streams

C++ prvides two objects for some string processing/input/output: `istringstream` and `ostringstream`. They are both declared in `#include <sstream>`.

Object istringstream allows you to read from a string like you do from a standart input.

```c++
void f(const string & s) {
  // Construct an object to parse strings
  istringstream is(s);
  
  // vector to store data
  vector<int> v;
  
  // Read integer while possible and add it to the vector
  int tmp;
  while (is >> tmp) {
    v.push_back(tmp);
  }
}

string f(const vector<int> & v) {
  // Construct an object to do formatted output
  ostringstream os;
  
  // Copy all elements from vector<int> to string stream as text
  tr(v, it) {
    os << ' ' << *it;
  }
  
  // Get string from string stream
  string s = os.str();
  
  // Remove first space character
  if (!s.empty()) {
    s = s.substr(1);
  }
  
  return s;
}
```

# Nontrivial Sorting

`sort()` uses the same technique as all STL:

- All comparison is based on `operator <`

```c++
struct fraction {
  int n, d;
  bool operator < (const fraction & f) const {
    return n*f.d < f.n*d;
  }
}
```

or

```c++
typedef pair<double,double> dd;
const double epsilon = 1e-6;

struct sort_by_polar_angle {
  dd center;
  // Constructor of any type
  // Just find and store the center
  template<typename T> sort_by_polar_angle(T b, T e) {
    int count = 0;
    center = dd(0, 0);
    while (b != e) {
      center.first += -> first;
      center.second += -> second;
      b++;
      count++;
    }
    double k = count ? (1.0 / count) : 0;
    center.first *= k;
    center.second *= k;
  }
  
  // Compare two points, return true if the first one is earlier than the second one looking by polar angle
  // When writing comparator, you should override not 'operator <' but 'operator ()'
  bool operator() (const dd & a, const dd & b) const {
    double p1 = atan2(a.second - center.second, a.first - center.first);
    double p2 = atan2(b.second - center.second, b.first - center.first);
    return p1 + epsilon < p2;
  }
};

vector<dd> points;
sort(all(points), sort_by_polar_angle(all(points)))
```

`operator <` should always return false for equal objects.

Imagine you are going to make the `struct point`. We want to interset some line segments and make a set of intersection points. Due to finite computer precision, some points will be the same while their coordinates differ a bit.

```c++
comst double epsilon = 1e-7;
struct point {
  double x, y;
  
  // Declare operator < taking precision into account
  bool operator < (const point & p) const {
    if (x < p.x - epsilon) return true;
    if (x > p.x + epsilon) return false;
    if (y < p.y - epsilon) return true;
    if (y > p.y + epsilon) return false;
    return false;
  }
};
```

# Time Complexity

## Priority Queue

Priority Queue is the implementation of Max Heap by default.

| Function  | Time Complexity | Space Complexity |
| --------- | --------------- | ---------------- |
| Q.top()   | O(1)            | O(1)             |
| Q.push()  | O(log n)        | O(1)             |
| Q.pop()   | O(log n)        | O(1)             |
| Q.empty() | O(1)            | O(1)             |

## Map

The `map<int, int> M` is the implementation of self-balancing Red-Black Trees.

The `unordered_map<int,int> M` is the implementation of Hash Table which makes the complexity of operations like insert, delete and search to Theta(1).

The `multimap<int,int> M` is the implementation of Red-Black Trees which are self-balancing trees making the cost of operations the same as the map.

the `unordered_multimap<int, int> M` is the implemented same as the unordered map is implemented which is the Hash Table.

| Function                     | Time Complexity | Space Complexity |
| ---------------------------- | --------------- | ---------------- |
| M.find(x)                    | O(log n)        | O(1)             |
| M.insert(pair<int,int>(x,y)) | O(log n)        | O(1)             |
| M.erase(x)                   | O(log n)        | O(1)             |
| M.empty()                    | O(1)            | O(1)             |
| M.clear()                    | Theta(n)        | O(1)             |
| M.size()                     | O(1)            | O(1)             |

## Set

The `set<int> s` is the implementation of Binary Search Trees.

The `unordered_set<int> s` is the implementation of Hash Table.

The `multiset<int> s` is the implementation of Red-Black trees.

The `unordered_multiset<int> s` is implemented the same as the unordered set but uses an extra variable that keeps track of the count.

The complexity becomes Theta(1) and O(n) when using unordered_set the ease of access becomes easier due to Hash Table implementation.

| Function    | Time Complexity | Space Complexity |
| ----------- | --------------- | ---------------- |
| s.find()    | O(log n)        | O(1)             |
| s.insert(x) | O(log n)        | O(1)             |
| s.erase(x)  | O(log n)        | O(1)             |
| s.size()    | O(1)            | O(1)             |
| s.empty()   | O(1)            | O(1)             |

## Stack

It is implemented using the linked list implementation of a stack.

| Function  | Time Complexity | Space Complexity |
| --------- | --------------- | ---------------- |
| s.top()   | O(1)            | o(1)             |
| s.pop()   | O(1)            | O(1)             |
| s.empty() | O(1)            | O(1)             |
| s.push(x) | O(1)            | O(1)             |

## Queue

Queue in STL is implemented using a linked list.

| Function  | Time Complexity | Space Complexity |
| --------- | --------------- | ---------------- |
| q.push(x) | O(1)            | O(1)             |
| q.pop()   | O(1)            | O(1)             |
| q.front() | O(1)            | O(1)             |
| q.back()  | O(1)            | O(1)             |
| q.empty() | O(1)            | O(1)             |
| q.size()  | O(1)            | O(1)             |

## Vector

Vector is the implementation of dynamic arrays and uses new for memory allocation in heap.

| Function                    | Time Complexity | Space Complexity |
| --------------------------- | --------------- | ---------------- |
| sort(v.begin(), v.end())    | Theta(nlog(n))  | Theta(log n)     |
| Reverse(v.begin(), v.end()) | O(n)            | O(1)             |
| v.push_back(x)              | O(1)            | O(1)             |
| v.pop_back(x)               | O(1)            | O(1)             |
| v.size()                    | O(1)            | O(1)             |
| v.clear()                   | O(n)            | O(1)             |
| v.erase()                   | O(n)            | O(1)             |

