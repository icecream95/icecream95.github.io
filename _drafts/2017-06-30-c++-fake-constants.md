---
layout: post
title: C++ Fake Constants
---

One thing I never like is the need to append a pair of brackets to so many things in the C++ world.

I have had enough of having to write `vec.length()`. Those brackets irritate me, especially for short names.

So I looked at the idea of fake constants. These are variables that can be manipulated by parts of the class implementation, but not users.

Also, this can be used for other ideas. Imagine this:

You're making a function which uses a lot of random numbers.
Even using `bind()`, you are driven crazy by having to call `r()` all the time. Using the similar techniques to fake constants, you can reduce 
this down to just `r`, and set a new seed with `r = x`. That reduces all the random number calls to one third of their original space.

Let's get into it, starting with the random number example:

{% highlight c++ %}
#include <iostream>
#include <random>
#include <utility>

using namespace std;

class Easy_rand
{
public:
    Easy_rand(int low, int high, int seed = 0):
        generator{seed}, distribution{low,high} {}
    Easy_rand& operator=(int e) {generator=default_random_engine{e};}
    operator int() {return distribution(generator);}
private:
    default_random_engine generator;
    uniform_int_distribution<int> distribution;
};

int main()
{
    Easy_rand r {1,6,10};
    cout << r << ' ' << r << ' ' << r << ' ' << r << ' ';
    cout << (r=10,r) << ' ' << r << ' ' << r << ' ' << r << '\n';
}
{% endhighlight %}

I get this result:

`1 2 4 4 1 2 4 4`

Now we have this working, let's look at a simple example of our original topic, fake constants:

{% highlight c++ %}
#include <iostream>

using namespace std;

class Fake_const_ex
{
public:
    Fake_const_ex() {}
    Fake_const_ex& add(int i) {l += i;return *this;}
    struct Impl_const
    {
        
    } length;
private:
    int l {0};
};

int main()
{
    Fake_const_ex ex;
    cout << ex.length << ' ';
    ex.add(10);
    cout << ex.length << ' ';
    cout << ex.add(15).length << ' ';

    Fake_const_ex::Impl_const t; // oops!
    cout << t;
}
{% endhighlight %}

