---
layout: guide
title: "Mocking with CUTE"
tutorial_index: 3
active: guides
---

# Refactoring Towards Seams
<a name="refactoringtowardsseams"></a>

Unwanted dependencies are a critical problem in software development. We often have to _break existing dependencies_ before we can change some piece of code. Breaking existing dependencies is also an important preliminary to introduce unit tests for legacy code — according to Feathers definition code without unit tests.

Feathers' _seams_ help in reasoning about the opportunities that exist when we have to break dependencies. The goal is to have a place where we can alter the behaviour of a program without modifying it in that place. This is important because editing the source code is often not an option (e. g., when a function the code depends on is provided by a system library).

## What is a Seam?
<a name="whatisaseam"></a>

Feathers characterises a seam as _a place in our code base where we can alter behaviour without being forced to edit it in that place_. This has the advantage that we can inject the dependencies from outside, which leads to both an improved design and better testability. Every seam has one important property: an _enabling point_. This is the place where we can choose between one behaviour or another. There are different kinds of seam types.

## What Kinds of Seams Does C++ Provide?
<a name="whatkindsofseamsdoescppprovide"></a>

C++ offers a wide variety of language mechanisms to create seams. Beside the classic way of using subtype polymorphism which relies on inheritance, C++ also provides static polymorphism through template parameters. With the help of the preprocessor or the linker we have additional ways of creating seams.

## Screencast Introduction to Seams
<a name="screencastintroductiontoseams"></a>

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/p4oM2bEZvAU&hl=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/p4oM2bEZvAU&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>

## Object Seam
<a name="objectseam"></a>

Object seams are probably the most common seam type. To start with an example, consider the following code where the class `GameFourWins` has a hard coded dependency to `Die`:

{% highlight c++ %}
// Die.h
struct Die {
  int roll() const ;
};
// Die.cpp
int Die::roll() const {
  return rand() % 6 + 1;
}
// GameFourWins.h
struct GameFourWins {
  void play(std::ostream& os);
private:
  Die die;
};
// GameFourWins.cpp
void GameFourWins::play(std::ostream& os = std::cout) {
  if (die.roll() == 4) {
    os << "You won!" << std::endl;
  } else {
    os << "You lost!" << std::endl;
  }
}{% endhighlight %}

According to Feathers definition, the call to `play` is not a seam because it is missing an enabling point. We cannot alter the behaviour of the member function `play` without changing its function body because the used member variable `die` is based on the concrete class `Die`. Furthermore, we cannot subclass `GameFourWins` and override `play` because `play` is monomorphic (not virtual).

This fixed dependency also makes `GameFourWins` _hard to test in isolation_ because `Die` uses C's standard library pseudo-random number generator function `rand`. Although `rand` is a deterministic function since calls to it will return the same sequence of numbers for any given seed, it is hard and cumbersome to setup a specific seed for our purposes. The classic way to alter the behaviour of `GameFourWins` is to _inject the dependency from outside_. The injected class inherits from a base class, thus enabling subtype polymorphism. To achieve that, Mockator provides a refactoring called _Extract Interface_ and creates the following code:

{% highlight c++ %}
struct IDie {
  virtual ~IDie() {}
  virtual int roll() const =0;
};
struct Die : IDie {
  int roll() const {
    return rand() % 6 + 1;
  }
};
struct GameFourWins {
  GameFourWins(IDie& die) : die(die) {}
  void play(std::ostream& os=std::cout) {
    // as before
  }
private:
  IDie& die;
};{% endhighlight %}

This way we can now inject a different kind of `Die` depending on the context we need. This is a seam because we now have an enabling point: The instance of `Die` that is passed to the constructor of `GameFourWins`.

### Object Seams Screencast

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/ozhRP0kobuk&hl=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/ozhRP0kobuk&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>

## Compile Seam
<a name="compileseam"></a>

Although object seams are the classic way of injecting dependencies, we think there is often a better solution to achieve the same goals. C++ has a tool for this job providing _static polymorphism_: template parameters. With template parameters, we can inject dependencies at compile-time. We therefore call this seam _compile seam_.

The essential step for this seam type is the application of a the refactoring _extract template parameter_. The result of this refactoring can be seen here:

{% highlight c++ %}
template <typename Dice=Die>
struct GameFourWinsT {
  void play(std::ostream& os = std::cout) {
    if (die.roll() == 4) {
      os << "You won !" << std::endl;
    } else {
      os << "You lost !" << std::endl;
    }
  }
private:
  Dice die;
};
typedef GameFourWinsT<> GameFourWins;{% endhighlight %}

The enabling point of this seam is the place where the template class `GameFourWinsT` is instantiated.

The use of static polymorphism with template parameters has several advantages over object seams with subtype polymorphism. It does not incur the run-time overhead of calling virtual member functions that can be unacceptable for certain systems. Probably the most important advantage of using templates is that a template argument only needs to define the members that are actually used by the instantiation of the template (providing _compile-time duck typing_). This can ease the burden of an otherwise wide interface that one might need to implement in case of an object seam.

### Compile Seams Screencast

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/c4El5wN2SBI&hl=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/c4El5wN2SBI&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>

## Preprocessor Seam
<a name="preprocessorseam"></a>

C and C++ offer another possibility to alter the behaviour of code without touching it in that place using the _preprocessor_. Although we are able to change the behaviour of existing code as shown with object and compile seams before, we think preprocessor seams are especially useful for debugging purposes like tracing function calls. An example of this is shown next where we trace calls to C's `malloc` function with the help of Mockator:

{% highlight c++ %}
// malloc.h
#ifndef MALLOC_H_
#define MALLOC_H_
void* my_malloc(size_t size, const char* fileName, int lineNumber);
#define malloc(size) my_malloc((size), __FILE__ , __LINE__)
#endif

// malloc.cpp
#include "malloc.h"
#undef malloc
void* my_malloc(size_t size, const char* fileName, int lineNumber) {
  // remember allocation in statistics
  return malloc(size);
}{% endhighlight %}

The enabling point for this seam are the options of our compiler to choose between the real and our tracing implementation. We use the option `-include` of the GNU compiler here to include the header file `malloc.h` into every translation unit. With `#undef` we are still able to call the original implementation of `malloc`.

### Preprocessor Seams Screencast

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/uv-AC_-QwKs&hl=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/uv-AC_-QwKs&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>

## Link Seams
<a name="linkseam"></a>

Beside the separate preprocessing step that occurs before compilation, we also have a post-compilation step called linking in C and C++ that is used to combine the results the compiler has emitted. The linker gives us another kind of seam called _link seam_. We show three kinds of link seams here:

* _Shadowing functions_ through linking order (override functions in libraries with new definitions in object files)
* _Wrapping functions_ with GNU's linker option -wrap (GNU Linux only)
* _Run-time function interception_ with the preload functionality of the dynamic linker for shared libraries (GNU Linux and Mac OS X only)

### Link Seams Screencast

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/jlP5lLOOixA&hl=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/jlP5lLOOixA&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>

### Shadow Functions Through Linking Order

In this type of link seam we make use of the _linking order_. The linker incorporates any undefined symbols from libraries which have not been defined in the given object files. If we pass the object files first before the libraries with the functions we want to replace, the GNU linker prefers them over those provided by the libraries. Note that this would not work if we placed the library before the object files. In this case, the linker would take the symbol from the library and yield a duplicate definition error when considering the object file. Mockator helps in _shadowing functions_ and generates code and the necessary CDT build options to support this kind of link seam:

{% highlight c++ %}
// shadow_roll.cpp
#include "Die.h"
int Die::roll() const {
  return 4;
}
// test.cpp
void testGameFourWins () {
  // ...
}{% endhighlight %}

{% highlight bash %}
$ ar -r libGame.a Die.o GameFourWins.o
$ g++ -Ldir/to/GameLib -o Test test.o shadow_roll.o -lGame{% endhighlight %}

The order given to the linker is exactly as we need it to prefer the symbol in the object file since the library comes at the end of the list. This list is the enabling point of this kind of link seam. If we leave `shadow_roll.o` out, the original version of `roll` is called as defined in the static library `libGame.a`. This type of link seam has one big disadvantage: _it is not possible to call the original function anymore_. This would be valuable if we just want to wrap the call for logging or analysis purposes or do something additional with the result of the function call.

### Wrapping Functions With GNU's Linker

The GNU linker _ld_ provides a lesser-known feature which helps us to _call the original function_. This feature is available as a command line option called _wrap_. The man page of _ld_ describes its functionality as follows: "Use a wrapper function for symbol. Any undefined reference to symbol will be resolved to `__wrap_symbol`. Any undefined reference to `__real_symbol` will be resolved to symbol."

As an example, we compile `GameFourWins.cpp`. If we study the symbols of the object file, we see that the call to `Die::roll` — mangled as `_ZNK3Die4rollEv` according to Itanium's Application Binary Interface (ABI) that is used by GCC v4.x — is undefined (`nm` yields `U` for undefined symbols).

{% highlight bash %}
$ gcc -c GameFourWins.cpp -o GameFourWins.o
$ nm GameFourWins.o | grep roll
U _ZNK3Die4rollEv{% endhighlight %}

This satisfies the condition of an undefined reference to a symbol. Thus we can apply a wrapper function here. Note that this would not be true if the definition of the function `Die::roll` would be in the same translation unit as its calling origin. If we now define a function according to the specified naming schema `__wrap_symbol` and use the linker flag `-wrap`, our function gets called instead of the original one. Mockator helps in applying this seam type by creating the following code and the corresponding build options in Eclipse CDT:

{% highlight c++ %}
extern "C" {
  extern int __real__ZNK3Die4rollEv();
  int __wrap__ZNK3Die4rollEv() {
    // your intercepting functionality here
    return __real__ZNK3Die4rollEv();
  }
}{% endhighlight %}

{% highlight bash %}
$ g++ -Xlinker -wrap=_ZNK3Die4rollEv -o Test test.o GameFourWins.o Die.o{% endhighlight %}

To prevent the compiler from mangling the mangled name again, we need to define it in a C code block. Note that we also have to declare the function `__real_symbol` which we delegate to in order to satisfy the compiler. The linker will resolve this symbol to the original implementation of `Die::roll`.

Alas, this feature is _only available with the GNU tool chain on Linux_. GCC for Mac OS X does not offer the linker flag `-wrap`. A further constraint is that it _does not work with inline functions_ but this is the case with all link seams presented here. Additionally, when the function to be wrapped is part of a shared library, we cannot use this option.

### Run-time Function Interception

If we have to _intercept functions from shared libraries_, we can use this kind of link seam. It is based on the fact that it is possible to alter the _run-time linking behaviour_ of the loader `ld.so` in a way that it considers libraries that would otherwise not be loaded. This can be accomplished by the environment variable `LD_PRELOAD` that the loader `ld.so` interprets.

With this we can instruct the loader to prefer our function instead of the ones provided by libraries normally resolved through the environment variable `LD_LIBRARY_PATH` or the system library directories. As an example, consider the following code and the CDT build options which is generated by Mockator to intercept function calls to `Die::roll`:

{% highlight c++ %}
#include <dlfcn.h>
int rand(void) {
  typedef int (*funPtr)(void);
  static funPtr origFun = 0;
  if (!origFun) {
    void* tmpPtr = dlsym(RTLD_NEXT, "rand");
    origFun = reinterpret_cast<funPtr>(tmpPtr);
  }
  int notNeededHere = origFun();
  return 3;
}{% endhighlight %}

{% highlight bash %}
$ LD_PRELOAD=path/to/libRand.so executable{% endhighlight %}

The advantage of this solution compared to the first two link seams is that it _does not require re-linking_. It is solely based on altering the behaviour of `ld.so`. A disadvantage is that this mechanism is _unreliable with member functions_, because the member function pointer is not expected to have the same size as a void pointer.

# Using Test Doubles
<a name="usingtestdoubles"></a>

Although there are already various existing mock object libraries for C++, we believe that creating mock objects is still too complicated and time-consuming for developers. Mockator provides a _mock object library_ and an _Eclipse plug-in_ to create mock objects in a simple yet powerful way. Mockator leverages the new language facilities C++11 offers while still being compatible with C++98/03.

Features include:

* Mock classes and functions with sophisticated IDE support 
* Easy conversion from fake to mock objects that collect call traces
* Convenient specification of expected calls with C++11 initializer lists or with Boost assign including Eclipse linked edit mode support
* Support for regular expressions to match calls with expectations

## Creating Mock Objects
<a name="creatingmockobjects"></a>

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/m_5ZMMmsY3M&hl=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/m_5ZMMmsY3M&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>

## Move Test Double to Namespace
<a name="movetestdoubletonamespace"></a>

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/z3wKo6aE7LI=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/z3wKo6aE7LI&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>

## Converting Fake to Mock Objects
<a name="convertingfaketomockobjects"></a>

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/FsXikqCiPLE=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/FsXikqCiPLE&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>

## Toggle Mock Support
<a name="togglemocksupport"></a>

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/CRX5_wdny_M=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/CRX5_wdny_M&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>

## Registration Consistency
<a name="registrationconsistency"></a>

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/vVxTAFhPRFk=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/vVxTAFhPRFk&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>

## Mock Functions
<a name="mockfunctions"></a>

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/iIlZSJ947GA=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/iIlZSJ947GA&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>

## Using Regular Expressions For Expectations
<a name="usingregularexpressionsforexpectations"></a>

<object width="425" height="344">
  <param name="movie" value="https://www.youtube.com/v/z0hdYPA7gSg=en&fs=1"></param>
  <param name="allowFullScreen" value="true"></param>
  <param name="allowscriptaccess" value="always"></param>
  <embed src="https://www.youtube.com/v/z0hdYPA7gSg&hl=en&fs=1" type="application/x-shockwave-flash" allowscriptaccess="always" allowfullscreen="true" width="425" height="344"></embed>
</object>
