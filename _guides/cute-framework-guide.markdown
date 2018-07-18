---
layout: guide
title: "CUTE Framework Guide"
tutorial_index: 1
active: guides
---

# How Things Work
<a name="howthingswork"></a>

I tried to create a simple, orthogonal and thus easy to extend and adapt testing framework that stays easy to use. I avoided some complexity for users of CUTE by exploiting modern C++ library features in the Boost library that is part of the `std::tr1` standard.

Note that all classes presented below are in namespace cute, which is omitted for the sake of brevity.

## CUTE Test
<a name="cutetest"></a>

The core class stores test functions using `std::function`. With `std::function`, any parameterless function or functor can be a test. In addition, each `cute::test` has a name for easier identification. That name is given either during construction or derived from a functor's `typeid`. The GNU g++ compiler requires that you demangle the name given by the `type_info` object, while VC++ provides a human readable `type_info::name()` result directly.

{% highlight c++ %}
struct test{
    void operator()()const{ theTest(); }
    std::string name()const{ return name_;}

    template <typename VoidFunctor>
    test(VoidFunctor const &t, std::string sname = demangle(typeid(VoidFunctor).name()))
        :name_(sname),theTest(t){}

    template <typename VoidFunctor>
    test(std::string sname,VoidFunctor const &t)
        :name_(sname),theTest(t){}

    private:
        std::string name_;
        std::function<void()> theTest;
};{% endhighlight %}

As you can see, there is no need to inherit from class `test`.

For simple functions, or when you want to name your tests differently from the functor's type, you can use the `CUTE()` macro:

{% highlight c++ %}
#define CUTE(name) cute::test((&name),(#name)){% endhighlight %}

`CUTE` is a function-like macro that takes the name of a test function and instantiates the test class with the address of that test function and its name.

Using a template constructor allows you to use any kind of functor that can be stored in a `std::function<void()>`, but this means that the functor can take no parameters. To construct with functions, functors or member functions with parameters, use `std::bind()` as shown below.

## Sweet Suites
<a name="sweetsuites"></a>

Running a single test with `cute::runner` is not very interesting. You might as well just call that function directly and check the results. The power of unit testing is realized when you have a larger collection of test cases that run after every compile and on a build server after every check-in. Thus there is a need for running many tests at once.

In contrast to other unit testing frameworks (including JUnit) I refrained from applying the Composite design pattern [GoF] for implementing the test case container. I love Composite and it is handy in many situations for tree structures, but it comes at the price of strong coupling by inheritance and lower cohesion in the base class, because of the need to support the composite class' interface. The simplest solution I came up with is to simply represent the test suite as a `std::vector<cute::test>`. Instead of a hierarchy of suites, you just run a sequence of tests. When the tests run, the hierarchy plays no role. You still can arrange your many tests into separate suites, but before you run them, you either concatenate the vectors or you run the suites individually in your `main()` function using the runner.

Tests can be added to the suite using `vector::push_back()`, but to make it really easy to fill your suite with tests, CUTE also provides an overloaded `operator+=` that will append a test object to a suite:

{% highlight c++ %}
typedef std::vector<test> suite;
suite &operator+=(suite &left, suite const &right);
suite &operator+=(suite &left, test const &right);{% endhighlight %}

This idea is blatantly stolen from `boost::assign`.

So this is all it takes to build a test suite:

{% highlight c++ %}
suite s;
s += TestFunctorA{};
s += CUTE(testFunctionB);
// and so on    ...{% endhighlight %}

If you really want to organize your test as a sequence of test suites, CUTE provides a `suite_test` functor that will take a test suite and run it through its call operator. However, if any test in a `suite_test` fails, the remaining tests will not be run.

CUTE's Eclipse plug-in eases the construction of test suites by providing automatic code generation and adjustment for registering test functions in suites. You can have standalone CUTE executables for a single suite, or test multiple suites, each in a separate library project.

## Assertions and Failures

A unit testing framework would not be complete without a way to actually check something in a convenient way. One principle of testing is to fail fast, so any failed test assertion will abort the current test and signal the failure to the top-level runner. You might have already guessed that this is done by throwing an exception. Later on, we will want to know where that test failed, so I introduced an exception class `test_failure` that takes the source file name and line number in the source file. Java does this automatically for exceptions, but as C++ programmers we must obtain and store this information ourselves. We rely on the preprocessor to actually know where we are in the code. Another `std::string` allows sending additional information from the test programmer to the debugger of a failing test.

This is how `cute_base.h` looks without the necessary `#include` guards and `#include <string>`:

{% highlight c++ %}
struct test_failure {
        std::string reason;
        std::string filename;
        int lineno;
        test_failure(std::string const &r,char const *f, int line)
        :reason(r),filename(f),lineno(line)
        { 	}
        char const * what() const { return reason.c_str(); }
};{% endhighlight %}

For actually writing test assertions, I provided macros that will throw if a test fails:

{% highlight c++ %}
#define ASSERTM(msg,cond) do { if (!(cond)) \
    throw cute::test_failure( \
        CUTE_FUNCNAME_PREFIX+cute::cute_to_string::backslashQuoteTabNewline(msg), \
        __FILE__,__LINE__); \
    } while(false)
#define ASSERT(cond) ASSERTM(#cond,cond)
#define FAIL() ASSERTM("FAIL()",false)
#define FAILM(msg) ASSERTM(msg,false){% endhighlight %}

This is all you need to get started. However, some convenience is popular in testing frameworks. Unfortunately, convenience often tends to be over-engineered and I am not yet sure if the convenience functionality I provided is yet simple enough. Therefore I ask for your feedback on how to make things simpler or confirmation that it is already simple enough.

## Testing for Equality

Testing two values for equality is probably the most popular test. Therefore, all testing frameworks provide a means to test for equality. JUnit, for example, provides a complete set of overloaded equality tests. C++ templates can do that as well with less code. For more complex data types, such as strings, it can be difficult to see the difference between two values, when they are simply printed in the error message.

{% highlight c++ %}
void anotherTest(){
    ASSERT_EQUAL(42,lifeTheUniverseAndEverything);
}{% endhighlight %}

One means to implement `ASSERT_EQUAL` would be to just `#define` it to map to `ASSERT((expected)==(actual))`. However, from my personal experience of C++ unit testing since 1998, this gives too little information when the comparison fails. This is especially true for strings or domain objects, where seeing the two unequal values is often essential for correcting the programming mistake. In my former life, we had custom error messages for a failed string comparison that allowed us to spot the difference easily. Therefore, CUTE provides a template implementation of `ASSERT_EQUAL`. This is of course called by a macro to enable file position reporting.

I speculated (perhaps wrongly) that it would be useful to specify your own mechanism to create the message if two values differ, which is implemented as a to-be-overloaded interface in the namespace `cute::cute_to_string`:

{% highlight c++ %}
namespace cute_to_string {
    template <typename T>
    std::string to_string(T const &t) {
        std::ostringstream os;
	to_stream(os,t);
	return os.str();
    }
    // common overloads of interface that work without an ostream
    static inline std::string to_string(char const *const &s){
        return s;
    }
    static inline std::string to_string(std::string const &s){
	return s;
    }
}{% endhighlight %}

Your overloaded `to_string` function is then called in `diff_values` which composes the standard message for your failed test case...

{% highlight c++ %}
template <typename ExpectedValue, typename ActualValue>
std::string diff_values(ExpectedValue const &expected
					, ActualValue const & actual
					, char const *left="expected"
					, char const *right="but was"){
    // construct a simple message...to be parsed by IDE support
    std::string res;
    res += ' ';
    res += left;
    res += ":\t" + cute_to_string::backslashQuoteTabNewline(cute_to_string::to_string(expected))+'\t';
    res += right;
    res +=":\t"+cute_to_string::backslashQuoteTabNewline(cute_to_string::to_string(actual))+'\t';
    return res;
}{% endhighlight %}

...and which is called in case your `ASSERT` throws a `test_failure`.

{% highlight c++ %}
template <typename ExpectedValue, typename ActualValue>
void assert_equal(ExpectedValue const &expected
			,ActualValue const &actual
			,std::string const &msg
			,char const *file
			,int line) {
    typedef typename impl_place_for_traits::is_integral<ExpectedValue> exp_integral;
    typedef typename impl_place_for_traits::is_integral<ActualValue> act_integral;
    if (cute_do_equals::do_equals(expected,actual,exp_integral(),act_integral()))
        return;
    throw test_failure(msg + diff_values(expected,actual),file,line);
}
#define ASSERT_EQUALM(msg,expected,actual) cute::assert_equal((expected),(actual), \
    CUTE_FUNCNAME_PREFIX+cute::cute_to_string::backslashQuoteTabNewline(msg),__FILE__,__LINE__)
#define ASSERT_EQUAL(expected,actual) ASSERT_EQUALM(#expected " == " #actual, (expected),(actual)){% endhighlight %}

As of version 1.5, CUTE allows all kinds of types to be compared by `ASSERT_EQUAL`. While earlier versions allowed only types where `operator<<(ostream &,TYPE)` was defined, some template meta-programming tricks now allow also other types, as long as `operator==(expected,actual)` is defined and delivers a bool compatible result. For integer types, meta-programming ensures that no signed-unsigned comparison warning is issued anymore. Comparing two floating point values without specifying a delta, automatically selects a delta that masks the least significant decimal digit, based on the size of expected. Floating point comparison subtracts actual and expected and sees if the absolute value of the difference is less than delta, by using `std::abs()`.

## Exception Testing

Another good unit testing practice is to verify that things go wrong as intended.

To embed a piece of code (an expression, or anyhting that can be passed as a macro parameter) that should throw a specific exception type, you can use the macro...

{% highlight c++ %}
ASSERT_THROWS(code,exception_type);{% endhighlight %}

...within your test function. For example:

{% highlight c++ %}
void test_that_something_throws() {
    ASSERT_THROWS(should_throw_std_exception(),std::exception);
}{% endhighlight %}

This test will fail if `should_throw_std_exception()` does not throw an exception of type `std::exception`. Any other exception will lead to an error, in contrast to failure.

There is no need to implement the try-catch again by hand to test error conditions. What is missing is the ability to expect a runtime error recognized by the operating system such as an invalid memory access. Those are usually signaled instead of thrown as a nice C++ exception.

You might need parenthesis around the code in the macro parameter to disambiguate commas, particularly commas in a parameter list.

## Listening Customization

You have already seen that the runner class template can be specialized by providing a listener. The `runner` class is an inverted application of the Template Method design pattern [GoF]. Instead of implementing the methods called dynamically in a subclass, you provide a template parameter that acts as a base class to the class @runner@, which holds the template methods `runit()` and `operator()`.

{% highlight c++ %}
template <typename Listener=null_listener>
struct runner{
    Listener &listener;
    std::vector<std::string> args;
    runner(Listener &l, int argc = 0, const char *const *argv = 0):listener(l){
        if(needsFiltering(argc,argv)){
            args.reserve(argc-1);
            std::remove_copy_if(argv + 1, argv + argc,back_inserter(args),std::logical_not<char const *>());
        }
    }
    bool operator()(const test & t) const
    {
        return runit(t);
    }

    bool operator ()(suite const &s, const char *info = "") const
    {
        runner_aux::ArgvTestFilter filter(info,args);
        bool result = true;
        if(filter.shouldrunsuite){
            listener.begin(s, info,
                count_if(s.begin(),s.end(),boost_or_tr1::bind(&runner_aux::ArgvTestFilter::shouldRun,
                    filter,boost_or_tr1::bind(&test::name,_1))));
            for(suite::const_iterator it = s.begin();it != s.end();++it){
                if (filter.shouldRun(it->name())) result = this->runit(*it) && result;
            }
            listener.end(s, info);
        }
        return result;
    }
private:
    bool needsFiltering(int argc, const char *const *argv) const
    {
        return argc > 1 && argv ;
    }

    bool runit(const test & t) const
    {
        try {
            listener.start(t);
            t();
            listener.success(t, "OK");
            return true;
        } catch(const cute::test_failure & e){
            listener.failure(t, e);
        } catch(...) {
            listener.error(t,"unknown exception thrown");
        }
    return false;
    }
};{% endhighlight %}

If you look back to `runner::runit()`, you will recognize that if any reasonable exception is thrown, it would be hard to diagnose the reason for the error. Therefore, I included catch clauses for `std::exception`, string and char pointers to get information required for diagnosis. The demangling is required for GNU g++ to get a human-readable information from the exception's class name.

{% highlight c++ %}
} catch(const std::exception & exc){
    listener.error(t, demangle(exc.what()).c_str());
} catch(std::string & s){
    listener.error(t, s.c_str());
} catch(const char *&cs) {
    listener.error(t,cs);
}{% endhighlight %}

Again I ask you for feedback if doing this seems over-engineered. Are you throwing strings as error indicators?

As you can see, there are a bunch of methods delegated to the base class given as `runner`'s template parameter `(begin, end, start, success, failure, error)`. The default template parameter `null_listener` applies the Null Object design pattern and provides the concept all fitting Listener base classes.

{% highlight c++ %}
struct null_listener{ // defines Contract of runner parameter
    void begin(suite const &, char const * /*info*/, size_t /*n_of_tests*/){}
    void end(suite const &, char const * /*info*/){}
    void start(test const &){}
    void success(test const &,char const * /*msg*/){}
    void failure(test const &,test_failure const &){}
    void error(test const &,char const * /*what*/){}
};{% endhighlight %}

Whenever you need to collect the test results or you want to have a nice GUI showing progress with the tests, you can create your own custom listener.

Again you can stack listeners using an inverted version of a Decorator design pattern [GoF]. Here is an example of an inverted Decorator using C++ templates that counts the number of tests by category:

{% highlight c++ %}
template <typename Listener=null_listener>
struct counting_listener:Listener{
    counting_listener()
    :Listener()
    ,numberOfTests(0),successfulTests(0)
    ,failedTests(0),errors(0),numberOfSuites(0){}

    counting_listener(Listener const &s)
    :Listener(s)
    ,numberOfTests(0),successfulTests(0)
    ,failedTests(0),errors(0),numberOfSuites(0){}

    void begin(suite const &s, char const *info, size_t size){
        ++numberOfSuites;
        Listener::begin(s,info, size);
    }
    void start(test const &t){
        ++numberOfTests;
        Listener::start(t);
    }
    void success(test const &t,char const *msg){
        ++successfulTests;
        Listener::success(t,msg);
    }
    void failure(test const &t,test_failure const &e){
        ++failedTests;
        Listener::failure(t,e);
    }
    void error(test const &t,char const *what){
        ++errors;
        Listener::error(t,what);
    }
    int numberOfTests;
    int successfulTests;
    int failedTests;
    int errors;
    int numberOfSuites;
};{% endhighlight %}

From the above schema, you can derive your own stackable listener classes, such as a listener that displays in a GUI the progress and results of tests as they run. If you do so, please share your solution.

## Member Functions as Tests

With `std::bind()` at your disposal, it is easy to construct a functor object from a class and its member function. Again this is canned in a macro that can be used like this:

{% highlight c++ %}
CUTE_MEMFUN(testobject,TestClass,test1);
CUTE_SMEMFUN(TestClass,test2);
CUTE_CONTEXT_MEMFUN(contextObject,TestClass,test3);{% endhighlight %}

The first version uses object `testobject`, an instance of `TestClass`, as the target for the member function `test1`. The second version creates a new instance of `TestClass` to then call its member function `test2` when the test is executed. The last macro provides a means to pass an additional object to `TestClass`' constructor when it is incarnated. The idea of incarnating the test object and thus have its constructor and destructor run as part of the test comes from Kevlin Henney and is implemented in Paul Grenyer's testing framework Aeryn.

The macro `CUTE_MEMFUN` delegates its work to a template function as follows:

{% highlight c++ %}
template <typename TestClass>
test makeMemberFunctionTest(TestClass &t,void (TestClass::*fun)(),char const *name){
    return test(boost_or_tr1::bind(fun,boost_or_tr1::ref(t)),demangle(typeid(TestClass).name())+"::"+name);
}
#define CUTE_MEMFUN(testobject,TestClass,MemberFunctionName) \
    cute::makeMemberFunctionTest(testobject,\
        &TestClass::MemberFunctionName,\
        #MemberFunctionName){% endhighlight %}

When the template function `makeMemberFunctionTest` is called, it employs `std::bind` to create a functor object that will call the member function fun on object `t`. Again we can employ C++ reflection using `typeid` to derive part of the test object's name. We need to derive the member function name again using the preprocessor with a macro. In order to also allow const member functions, the template function comes in two overloads, one using a reference (as shown) and the other using a const reference for the testing object.

## Test Object Incarnation

I will spare you the details, and just present the mechanism of object incarnation and then calling a member function for the case where you can supply a context object:

{% highlight c++ %}
template <typename TestClass,typename MemFun, typename Context>
struct incarnate_for_member_function_with_context_object {
    MemFun memfun;
    Context context;
    incarnate_for_member_function_with_context_object(MemFun f,Context c)
    :memfun(f),context(c){}
    incarnate_for_member_function_with_context_object(incarnate_for_member_function_with_context_object const &other)
    :memfun(other.memfun),context(other.context){}

    void operator()(){
        TestClass t(context);
        (t.*memfun)();
    }
};
template <typename TestClass, typename MemFun, typename Context>
test makeMemberFunctionTestWithContext(Context c,MemFun fun,char const *name){
    return test(incarnate_for_member_function_with_context_object<TestClass,MemFun,Context>(fun,c),
        demangle(typeid(TestClass).name())+"::"+name);
}{% endhighlight %}

This allows you to use test classes with a constructor to set up a test fixture and a destructor for cleaning up after the test. This eliminates need to for explicit `setUp()` and `tearDown()` methods, as in JUnit.

### Example

{% highlight c++ %}
struct TestClass{
    static int callcounter;
    int i;
    TestClass():i(1){} // for incarnation; setUp
    TestClass(int j):i(j){} // for incarnation; setUp
    ~TestClass(){...} // for destruction; tearDown
    void test1(){
        ++callcounter;
        ASSERT_EQUAL(1,i++);
    }
    void test2() const {
        ++callcounter;
        ASSERT(true);
    }
    void test3() {
        ++callcounter;
        ASSERT_EQUAL(2,i++);
        ++i;
    }
    void test_incarnate(){
        ++callcounter;
        ASSERT_EQUAL(42,i++);
    }
    void test_incarnate_const() const {
        ++callcounter;
        ASSERT_EQUAL(43,i);
    }
};

....

cute::suite s3;
s3 += CUTE_SMEMFUN(TestClass,test1);
s3 += CUTE_SMEMFUN(TestClass,test2);

TestClass context{2};
s += CUTE_CONTEXT_MEMFUN(context, TestClass, test3);{% endhighlight %}
