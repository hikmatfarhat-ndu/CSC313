Many of the data structures that we will be studying and implementing are already implemented in the Standard Template Library (STL). Our strategy is to use and familiarize ourselves with the provided containers and algorithms and then implement them ourselves.
# Vector container
A vector is basically a dynamic array. We can add and remove elements from it and it auto resizes itself. Since we would like to store any type of objects in a vector the implementation uses templates to pass any type to the vector. A vector has the all usual syntax of arrays, specifically the indexing.

```cpp
#include <iostream>
#include <vector>
int main(){
    //declare a vector of int. creates an empty vector
    std::vector<int > v;
    //declare a vector of strings
    std::vector<std::string > sv;
    //add elements to the end
    v.push_back(1);
    v.push_back(2);
    sv.push_back("one");
    sv.push_back("two");
    //print all elements similar to arrays
    for(int i=0;i<sv.size();++i)
        std::cout<<sv[i]<<",";
    std::cout<<"\n";
    //print all elements using range-based for loop
    for(auto x:v)
        std::cout<<x<< ",";
    std::cout<<"\n";   
}
```

You can try the code [here](https://repl.it/@hfarhat/vector-ex1)
The ```push_back``` member function adds an element at the end of the vector. Above we have used two features from c++11: auto and range-based for loops. As you can see we can create a vector of any type (note the syntax). 
**Note**: since the type of an auto variable is inferred by the compiler it cannot be used with _uninitialized_ variables.

```cpp
auto x;//error
auto y=1;//OK
```
## Automatic allocation

In a typical use case when we use ```push_back``` we don't know in advance the size of the data. If memory allocated to the vector is "full" then the vector needs to allocate more space **and** copy the existing data to the new storage space before adding the new values.
Let us look an example using arrays

```cpp
  //create an array of two elements
        int size = 2;
        int* p = new int[2];
        for (int i = 0; i < 10; ++i) {
            if (i == size) {//array is full
                std::cout << "copying\n";
                int* old = p;
                p = new int[size + 1];
                for (int j = 0; j < size; ++j)
                    p[j] = old[j];
                size++;
            }
            p[i] = i;

        }
        std::cout << "------content-------\n";
        for (int i = 0; i < size; ++i)
            std::cout << p[i] << ",";
        std::cout << "\n";
```
It is obvious from the example that there is a lot of copying. In fact if we add _n_ numbers there will be _n<sup>2</sup>_ operations. A smarter strategy would be to overallocate. The overallocation is STL implementation dependent. A good rule of thumb, used by the g++ STL, is to double the size every time.

```cpp
 {
        //create an array of two elements
        int size = 2;
        int* p = new int[2];
        for (int i = 0; i < 10; ++i) {
            if (i == size) {//array is full
                std::cout << "copying\n";
                int* old = p;
                p = new int[2*size];
                for (int j = 0; j < size; ++j)
                    p[j] = old[j];
                size*=2;
            }
            p[i] = i;

        }
        std::cout << "------content-------\n";
        for (int i = 0; i < size; ++i)
            std::cout << p[i] << ",";
        std::cout << "\n";
    }
```

But as you can see in the output, after the 10th element the values are arbitrary. This is because we didn't differentiate between __size__ and __capacity__. This is exactly how a vector handles allocated memory.

```cpp
 std::vector<int> v;
for (int i = 0; i < 100; i++) {
    std::cout << "i= " << i << ", capacity=" << v.capacity() << std::endl;
    v.push_back(i);
}
```

If you run the above you will see the effect of preallocating memory. **Note**: it seems the msc++ uses a different preallocation strategy. From what i see it looks that the size is incremented 50% every time instead of 100%.
### Iterators
Iterators are generalization of pointers and present a common interface to all STL containers and algorithms. For an array a pointer is sufficient since the elements of an array form a contiguous location in memory. What if the elements are not stored contiguously? Since every container stores the elements differently it implements its own methods to _iterate_ over its elements. This has the added value that the user does not need to know the internal workings of the container in order to be able to use it.
 Given a container **c** an iterator **itr** points at an element stored in **c**. Therefore dereferencing an iterator **(*itr)** will return the element itself. Also iterators can be incremented and decremented like pointers: (itr++) and (itr--). Furthermore, every container **c** has a **begin** and **end()** method. As an example, suppose that we have a container that stores an integer sequence from 0 to n-1 but even numbers first then odd numbers. For example, 0,2,4,6,8,1,3,5,7,9. Below is a code that allow us to iterate over the values in order

```cpp
class Container {
    int* p, * _begin;
protected:
    int _size;
    int *_end;
public:
    class Iterator {
        int* current;
        int _size;
    public:
        Iterator(int* p,int size) :current(p),_size(size) {}
        Iterator operator++() {
            
            if (*current == _size-1)current = current + 1;
            else if (*current % 2 == 0)
                current = current + _size / 2;
            else
                current = current-_size/2+1;
            return *this;
        }
        int operator *() {
            return *current;
        }
        bool operator!=(const Iterator& rhs) {
            return current != rhs.current;
        }
        
    };
    Container(int size) :_size(size) {
        p = new int[_size];
        _begin = p;
        _end = p + size;
        for (int i = 0; i < _size; ++i) {
            if (i % 2 == 0)
                p[i / 2] = i;
            else
                p[_size / 2 + i / 2] = i;
        }
    }
    Iterator begin() {
        return Iterator(_begin,_size);
    }
    Iterator end() {
        return Iterator(_end,_size);
    }
};

```

so we can print the values in order as follows:

```cpp
 Container c(10);
    for (Container::Iterator itr = c.begin(); itr != c.end(); ++itr)
        std::cout << *itr << ",";

```

Note the syntax for a dependent type (we could have used auto) because Iterator is a nested class. Similarly to the above we can use iterators with any STL container. In particular with vectors.

```cpp
#include <vector>
#include <iostream>
int main(){
  std::vector<std::string> sv;
  sv.push_back("one");
  sv.push_back("two");
  for(auto itr=sv.begin();itr!=sv.end();itr++){
      std::cout<<(*itr)<<std::endl;
  }
}  
```

The auto keyword is useful since otherwise we have to write down the long type of the iterator: (since it is an iterator to container of type ```std::vector<std::string>```)

```cpp
std::vector<std::string>::iterator itr;
```

vectors can be used similarly to arrays.

```cpp
std::vector<int> iv;
iv.push_back(1);
iv.push_back(2);
for(int i=0;i<iv.size();i++)
  iv[i]=i;
```

Since vectors are required by the c++ standard to use contiguous memory 
it is best to add and remove(as opposed to change) from the end of a vector.
We still can insert and erase elements at arbitrary position using iterators but it is a _costly_ operation.

## Pre allocation

In what follows we will use a TestClass.

```cpp
template <int nodebug=0>
struct TestClass {
    int _x, _y;
    TestClass(int x = 0, int y = 0) :_x(x), _y(y) {
       if(!nodebug) std::cout << "ctor\n";

    }
    TestClass(const TestClass& rhs) {
        if(!nodebug)std::cout << "copy ctor\n";
        _x = rhs._x;
        _y = rhs._y;
    };
    TestClass& operator=(const TestClass& rhs) {
        if(!nodebug)std::cout << "assignment\n";
        _x = rhs._x;
        _y = rhs._y;
        return *this;
    }
     std::pair<int, int> val() {
        return std::pair<int, int>(_x, _y);
    }
    int& x() {
        return _x;
    }
    int& y() {
        return _y;
    }
    ~TestClass() {
        if(!nodebug)std::cout << "dtor\n";
    }
};
```
Let us see what happens when we add objects of type TestClass to a vector.

```cpp
std::vector<TestClass<0>> v;
TestClass a(1,2);
TestClass b(3,4);
v.push_back(a);
v.push_back(b);
```

if we run the above we get the following output

```cpp
ctor
ctor
copy ctor
copy ctor
copy ctor
...
```

You can try the code [here](https://repl.it/@hfarhat/vector-ex2)
Obviously there are two calls for the constructor for a and b. The method push_back saves a __copy__ of the input hence the two calls for the copy constructor. The third call for the copy constructor is because the vector was resized to accommodate b.

Sometimes it is useful to preallocate memory to minimize the number of copy operations when the vector is resized. There are two ways of doing this.
1. Specifying the size when the vector is created
1. Using the vector::reserve method

```cpp
{
std::vector<TestClass<0>> v(2);
TestClass a(1, 2);
TestClass b(3, 4);
v[0] = a;
v[1] = b;
std::cout << "size= " << v.size() << std::endl;
std::cout << "----------------\n";
}
{
std::vector<TestClass<0>> v;
v.reserve(2);
TestClass a(1, 2);
TestClass b(3, 4);
v.push_back(a);
v.push_back(b);
std::cout << "size= " << v.size() << std::endl;
std::cout << "----------------\n";
}
```

The output of the above code is 

```cpp
ctor
ctor
ctor
ctor
assignment
assignment
size= 2
----------------
ctor
ctor
copy ctor
copy ctor
size= 2
----------------
```

This is because not only the vector constructor reserves space for two objects but it will also call the default constructor of the object to initialize the reserved space. In this case we use the index operator to change the values. Whereas the member function reserve does not create objects when it reserves space.
Removing elements from the __end__ of the vector is done using ```pop_back()```

```cpp
std::vector<TestClass<0>> v;
v.push_back(TestClass<0>(1, 2));
v.push_back(TestClass<0>(3, 4));
std::cout << v.size() << std::endl;
v.pop_back();
std::cout << v.size() << std::endl;

```

## insertion and deletion
So far we added and removed elements from the __end__ of the vector. We can do the same operations at **arbitrary positions** using iterators even though if these operations are to be done repeatedly a **vector is not the best data structure to use**.
### insertion
The code below uses the _insert_ function to add an element between __a__ and __b__ of the vector.

```cpp
{
std::vector<TestClass<0>> v;
v.reserve(4);
TestClass<0> a(1,2);
TestClass<0> b(3,4);
TestClass<0> c(5,6);
TestClass<0> d(7,8);
v.push_back(a);
v.push_back(b);
v.push_back(c);

//insert d between a and b
 auto itr=v.insert(v.begin()+1,d);
 //find where d is inserted
    for (auto i = v.begin(); i != v.end(); ++i) {
        if (i == itr)std::cout << "element inserted here: ";
        std::cout << i->x() << "," << i->y() << std::endl;

    }
}
```

If you inspect the output you will see the following happening 
1. ctor is used to create a,b,c,d.
1. copy ctor is used to to copy the values of a,b,c to the vector (push_back)
1. copy ctor is used to make a copy of d
1. copy ctor is used to copy the value of c the 4th place that was reserved in the vector.
1. Assignment is used to overwrite the 3rd place by the value of b
1. Assignment is used to overwrite the 2nd place by the value of d

This means that **inserting** a value in a vector any place **other than the end** will cause order of __n__ copy/assignments. If such insertions are done often then a vector is not the optimal data structure to use. Note that the return value of insert is an iterator to the element that was inserted.
Similarly, we can erase values from a vector at arbitrary positions

```cpp
{
std::vector<TestClass<0>> v(5);
TestClass<0> a(1, 2);
TestClass<0> b(3, 4);
v[0] = a;
v[1] = b;
std::cout << "size= " << v.size() << std::endl;
std::cout << "----------------\n";
v.erase(v.begin());
std::cout << "-------done----\n";
}
```

Since the deleted element is at the beginning of the vector it also triggers an order of __n__ assignments to move values to the left.
Because of that, if you need to remove multiple items meeting a certain criterion, it is better to use the **remove/erase idiom**. We illustrate with two different ways of removing elements whose __y__ value is 2 from a vector. The first, goes through a loop and when it finds an element with y==2 it erases it. Note that erase returns an iterator to the element __after__ the erased one.  

```cpp
{
std::vector<TestClass<0>> v;
v.reserve(4);
TestClass<0> a(1, 2);
TestClass<0> b(3, 4);
TestClass<0> c(5, 2);
TestClass<0> d(5, 6);
v.push_back(a); v.push_back(b); v.push_back(c); v.push_back(d);
std::cout << "---searching--\n";
for (auto itr = v.begin(); itr != v.end();) {
    if (itr->y() == 2) {
        itr=v.erase(itr);
    }
    else itr++;
}
std::cout << "----done searching\n";
  }


```
The second, more efficient way is to use the **std::remove_if then erase**. The function std::remove_if doesn't actually remove elements, it "moves all elements meeting the criterion towards the end" of the vector and returns an iterator to the first one.

```cpp
{
std::vector<TestClass<0>> v;
v.reserve(4);
TestClass<0> a(1, 2);
TestClass<0> b(3, 4);
TestClass<0> c(5, 2);
TestClass<0> d(5, 6);
v.push_back(a); v.push_back(b); v.push_back(c); v.push_back(d);
std::cout << "---searching--\n";
auto itr=std::remove_if(v.begin(), v.end(), [](auto& t) { return t.y() == 2; });
v.erase(itr, v.end());
std::cout << "----done searching\n";
}
```

### accumulate

The standard fold operation in functional languages is implemented using the std::accumulate function. It takes a range (i.e. start and end iterators) and an initial value (usually zero). By default accumulate adds all the numbers in the range. so

```cpp
std::accumulate(start,end,init);
```
is equivalent to

```cpp
std::accumulate(start,end,init,std::plus{});
```
This means we can change the default behavior by supplying our own function. Accumulate works by repeatedly calling (default case plus) the function on the current element and the accumulated value starting with *start and init. The example below multiplies all the elements of the vector.

```cpp
std::vector<int> v {1,2,3,4};
auto res=std::accumulate(v.begin(),v.end(),1,std::multiplies{});
```

A more complicate example is shown below where we add the even and odd numbers separately. 

```cpp
 {
     std::random_device e;
     std::uniform_int_distribution<> dist(1, 10);
     const int n = 10;
     std::vector<int> v(n);
     std::generate(v.begin(), v.end(), [&]() {return dist(e); });
     for (auto& x : v)
         std::cout << x << ",";
     std::cout << std::endl;
    
     const auto result = std::accumulate(v.begin(), v.end(), std::make_pair(0, 0),
         [](std::pair<int,int> sum,int n) {
             n % 2 == 1?sum.first += n:sum.second += n;
             return sum;
         });
     auto [x, y] = result;
     std::cout << x << "," << y << std::endl;

 }
 ```

 
## Extra
We give here a more complicated example of the remove/erase idiom. Suppose that we have a vector of strings and some of them are empty. This typically occurs when reading some delimited data from a file in which some of the fields are empty. Suppose further that we want to remove all __all_blank__ strings from the vector. Below is the code to do just that.
First note the definition of the filter lambda: it returns true when the input is a blank character. The filter is used in the lambda res which returns an iterator to the first character that is __not__ blank. If all characters are blank res returns the end() iterator.

```cpp
    std::vector<std::string> vs{ "hi","  ","there","hello","  ","welcome","  ","end"};
    auto filter = [](auto c) {return c == ' ' ? true : false; };
    auto res = [&](auto& x) {return std::find_if_not(x.begin(), x.end(), filter); };

    std::remove_if(vs.begin(), vs.end(), [&](auto& is) { return res(is)== is.end(); });
  
    for (auto& c : vs)
        std::cout << c << ",";
    std::cout<<std::endl;
```
You can run the above code [here](https://repl.it/@hfarhat/remove-blank-strings). Notice how some of the all blank strings become empty. This is the result of using move semantics.
For example

```cpp
std::string s {"test"};
std::string u=std::move(s);
std::cout<<"("<<u<<")";
std::cout<<"("<<s<<")";

```

You can try this code [here](https://repl.it/@hfarhat/move-string). As you can see _u_ "steals" the resources of _s_, i.e. its characters. When we don't use std::move then _u_ would be a copy of _s_. Actually, std::move is just static cast to an rvalue reference.

### transform

One of the most useful STL functions is std::transform. It is similar to the map function in functional languages (and Python). It takes
1. A source range, defined by start and end iterators
1. An iterator to the beginning of the destination range. Note it is your responsibility to make sure that the destination range is large enough to fit the input
1. A function that takes an element from the input and transforms it to another element which be stored in the destination range. In the example below, we use transform to convert each TestClass element to another with _x and _y swapped.

```cpp
   {
    std::vector<TestClass<1>> u;
    u.reserve(4);
    u.emplace_back(1, 2); u.emplace_back(3, 4); u.emplace_back(5, 2); u.emplace_back(5, 6);
    std::vector<TestClass<1>> v;
    v.resize(u.size());
    std::transform(u.begin(), u.end(), v.begin(), [](auto& t) {
        int tmp = t.x();
        t.x() = t.y();
        t.y() = tmp; 
        return t; }
        );
    for (auto& a : v)
        std::cout << a.x() << "," << a.y() << std::endl;
    }

```
The STL iterators interface allows us to apply many STL algorithms to almost any container. Below is an example of a few of those algorithms.

### Counting

For the purpose of these examples we will use a simple vector of integers. Note we don't use the reserve function because in that case no element is created and therefore begin==end.

```cpp
#include <iostream>
#include <random>
#include <algorithm>

#include <vector> {
    int n = 20;
    std::random_device e;
    std::uniform_int_distribution<> dist(1, 10);
    std::vector<int> v;
    v.resize(n);
    std::generate(v.begin(), v.end(), [&]() {return  dist(e); });
    for (auto& x : v)
        std::cout << x << ",";
    int m = dist(e);
    std::cout << "\n The number of " << m << " is " << std::count(v.begin(), v.end(), m);
    }

```
A different version allow us to use a predicate. For example below we count the even numbers in the input as well as print the min and max in the range.

```cpp
#include <iostream>
#include <random>
#include <algorithm>

#include <vector> {
    int n = 20;
    std::random_device e;
    std::uniform_int_distribution<> dist(1, 10);
    std::vector<int> v;
    v.resize(n);
    std::generate(v.begin(), v.end(), [&]() {return  dist(e); });
    for (auto& x : v)
        std::cout << x << ",";
    int m = dist(e);
    std::cout << "\n The # of evens is "
    << std::count(v.begin(), v.end(),[](int i) {return i%2==0;});
    std::cout << "min element is " << *std::min_element(v.begin(), v.end())<<std::endl;
    std::cout << "max element is " << *std::max_element(v.begin(), v.end())<<std::endl;
    }

```

Many of the algorithms we are using require that the destination has enough space to copy elements to it that is why we resize the destination vector before running the algorithm.
There is a convenient back_insert_iterator that automatically calls the push_back method of the container so we don't need to resize it beforehand.

```cpp
 {
    std::vector<int> v;
    std::back_insert_iterator itr(v);
    *itr = 1;
    *itr = 2;
    *itr = 3;
    for (auto& x : v)
        std::cout << x << ",";
    std::cout << std::endl;
    }
```
Let us use a back_insert_iterator to filter all even numbers from a container. Note that vector __d__ has 0 size and is not resized beforehand because we don't know how many even numbers there are in the input.

```cpp
 {
        int n = 20;
        std::random_device e;
        std::uniform_int_distribution<> dist(1, 10);
        std::vector<int> v;
        v.resize(n);
        std::generate(v.begin(), v.end(), [&]() {return  dist(e); });
        std::vector<int> d;
        std::copy_if(v.begin(), v.end(), std::back_insert_iterator(d),
            [](int i) { return i % 2 ==0; });
        for(auto& x:d)
            std::cout << x << ",";
        std::cout << std::endl;
    }

```
You can see the code [code](https://repl.it/@hfarhat/stdaccumulate)


