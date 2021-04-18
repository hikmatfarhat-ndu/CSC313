# Introduction

Before discussion template parameter deduction it is useful to review variable declarations
in c++. Consider the the following example:

```cpp
int main(){
int x=2;
const int y=7;
int& rx=x;
int& ry=y;//error ry should be const int&

const int & rpy=y;//ok

int& rl=8;//error cannot bind a lvalue reference to a temp

int&& rpl=8;//ok rl is an rvalue reference
}

}
```
You can test the above code  [here](https://godbolt.org/z/7rT4a1)
From the above we see that when declaring a variable and assign it to another expression the
type of the two sides should match. This kind of reasoning carries over to template parameter
matching.

# Template Deduction

Typically a template function looks like
```cpp
template <typename T>
void f(t x);
```
Clearly **t** and **T** are related. And the call to the function would look like

```cpp
int main(){

    f(expr);
}
```
Where **expr** is some expression.  The compiler uses **expr** to deduce two types: one for
**t** and one for **T**. As explained in the previous section it is useful to analyze
the situation _as if_ we had ```t=expr``` and then deduce the types. 
Before proceeding we give an example to clarify.
```cpp
template <typename T>
void f(const T& x);
```
So in that case ```t=const T&```. So if we call the function 

```cpp
int main(){
    int y=12;
    f(y);
}
```
If we analyze ```const T& =int y``` we can deduce that ```T=int```.```t=const int &```. In practice it is a bit more complicated than that. There are three cases:

1. **t** is a reference or pointer. If ```expr``` is a a reference or pointer ignore it then
pattern match to **t**. For example

```cpp
template <typename T>
void f(T& x);
template<typename T>
void g(T* x);

int main(){
    int x=27;
    const int ax=x;
    const int& bx=x;
    const int* p=&x;
    f(x);// T is int and t=int& can be deduced
         // by looking at T&=int
    f(ax);// T is const int and t=const int &
         // by looking at T&=const int
    f(bx);// T is const int and t=const int &
        //by looking at T&=const int&
    g(p);// T is const int and t=const int *
        //by looking at T*=const int *
    g(&x);// T is int and t=int *
        //by looking at T*=int *
}
```

1. **t** is a **universal reference**. For example
```cpp
template<typename T>
void f(T&& x);
```
Note that the symbol "&&" usually denotes an **rvalue** reference but in the context of
templates it denotes a universal reference meaning it can match any reference.

* if _expr_ is an lvalue then both **T** and **t** are revalue references

* if _expr_ is an rvalue then the usual deduction applies

Example.

```cpp
template<typename T>
void f(T&& x);
int main(){
    int x=2;// both T and t are int&
            // deduced from T&&=int
    const int y=8;// both T and t are const int&
    int& rx=x;// both T and t are int&
    const int& ry=y;// both T and t are const int &
    int&& z=18;// T is int and t is int&&
}

```
You can test the code [here](https://godbolt.org/z/119TK7)

