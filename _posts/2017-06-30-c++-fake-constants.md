---
layout: post
title: C++ Fake Constants
---

One thing I never like is the need to append a pair of brackets to so many
things in the C++ world.

I have had enough of having to write `vec.length()`. Those brackets irritate me,
especially for short names.

So I looked at the idea of fake constants. These are variables that can be
manipulated by parts of the class implementation, but not users.

Also, this can be used for other ideas. Imagine this:

You're making a function which uses a lot of random numbers.
Even using `bind()`, you are driven crazy by having to call `r()` all the time.
Using the similar techniques to fake constants, you can reduce this down to just
`r`, and set a new seed with `r = x`. That reduces all the random number calls
to one third of their original space.

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
    cout << r << ' ' << r << ' ' << r << ' ' << r << '\n';
    cout << (r=10,r) << ' ' << r << ' ' << r << ' ' << r << '\n';
}
{% endhighlight %}

I get this result:

```1 2 4 4
1 2 4 4```

(you might get something else, but the last four numbers will always equal the
first four)

Now we have this working, let's look at a simple example of our original topic,
fake constants:

{% highlight c++ %}
#include <iostream>

using namespace std;

struct No_copy
{
    No_copy(No_copy&) = delete;
    No_copy() = default;
};

class Fake_const_ex
{
public:
    Fake_const_ex() {}
    Fake_const_ex& add(int i) {length.l += i;return *this;}
    struct : No_copy
    {
        operator int() {return l;}
        const int* operator& () {return &l;}
    private:
        friend class Fake_const_ex;
        int l {0};
    } length;
};

int main()
{
    Fake_const_ex ex; 
    cout << ex.length << ' ';
    ex.add(10);
    cout << ex.length << ' ';
    cout << ex.add(15).length << ' ';

    int t = ex.length;  // Unfortunatly, auto doesn't work. Any ideas?
    t += 2;
    cout << t << ' ';
    
    const int* p = &ex.length;
    ex.add(5);
    cout << *p << '\n';
}
{% endhighlight %}

The only problem with this technique is that, as you can see, creating an auto
variable from the fake constant gives an error, so you need to explicitly use int.

Also, this can be used to control writes to an object:

{% highlight c++ %}
#include <iostream>

using namespace std;

struct Argument_error {};

class Chocolate_bar
{
public:
    void operator=(int i) {if (i > n || i < 0) throw Argument_error{};
    // We don't want more chocolate magically appearing!
                           if (i == 0) cout << "Yum!\n"; if (n-i>5)
                           cout << "Greedy!\n";n = i;}
    operator int() {return n;}
private:
    int n {40};
};

int main()
{
    Chocolate_bar c;
    cout << c << ' ';
    c = 37; cout << c << ' ';
    c = 30; cout << c << ' ';
    auto c2 = c;
    c = 0; cout << c << ' ';
    c2 = 2; cout << c2 << ' ';
    c2 = 0; cout << c2 << ' ';
    try
    {c = 50;}
    catch (...)
    {cout << "oops!";}
    try{c2 = -1;}
    catch (...)
    {cout << "oops!";}
}
{% endhighlight %}

This outputs "Yum!" and "Greedy!" etc. in the right places, and triggers
and exception in both the try blocks.

All the code in this post are free for your use (though you'll probably want
to change some of it first).

Any questions or improvements are welcome. Either send an email, add an issue
on Github or submit a pull request.
