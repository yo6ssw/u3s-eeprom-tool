---
layout: post
title: The Lonely Green Triangle
excerpt: Your mission should you choose to accept it is to write an application that opens a fullscreen (or not) window onto which a green triangle using OpenGL is drawn; after 3 seconds the application must close.
tags:
    - demoscene
---
### Target

Your mission should you choose to accept it is to write an application that opens a fullscreen (or not) window onto which a green triangle using OpenGL is drawn; after 3 seconds the application must close.

### Design

What I first do is attempt to write the client code that solves the problem. This is pretty much a lengthy process going through numerous iterations until I am satisfied with the way the code looks. Each iteration aims to make the client code shorter, clearer and cleaner. For our objective I ended up with the following code:

{% highlight c++ %}
int main() {
    Window window(800, 600, WindowType::Fullscreen);

    Timer timer;
    while (timer.secondsSinceStart() < 3) {
        drawGreenTriangle();
        window.present();
    }

    return 0;
}
{% endhighlight %}

So what's going on here? We create a `Window` and a `Timer` and while 3 seconds have yet to pass, we draw a green triangle. The code is simple, expressive and hides away all the intricacies of implementation.

This little piece of code involves _design_. As you can see, just by expressing how I would like things to happen caused the discovery of two entities, namely `Window` and `Timer` along with some of their responsibilities.

For example, in a previous iteration I chose `millisecondsSinceStart()` over the current `secondsSinceStart()` but I decided to postpone it until really needed. Why needed? because I happen to know that later on we will need to measure the time taken for a frame to render and that time we will need to express in milliseconds. However, using milliseconds at this stage would have broken the YAGNI principle to which I strive to adhere ([see Jeff Atwood's post](http://blog.codinghorror.com/kiss-and-yagni/)).

Let's sketch out the interfaces of the entities we have discovered. 

{% highlight c++ %}
enum class WindowType {
    Windowed,
    Fullscreen
};

class Window {
public:
    Window(int width, int height, WindowType type);

    void present();
};

class Timer {
public:

    double secondsSinceStart();
};
{% endhighlight %}

We also have a function to implement in the main context: 

{% highlight c++ %}
void drawGreenTriangle() {
    // ... draw a green triangle here with OpenGL ...
}
{% endhighlight %}

Ok, time has now come to implement what we designed.

### Implementation

The github repository can be found [here](https://github.com/benishor/demoscene-lab). The v1.0 tag contains the code for this post. Alternatively you can download a zip with the sources [here](https://github.com/benishor/demoscene-lab/archive/v1.0.zip). For instructions on how to build and run the code, check the `README.md` file.

![The lonely green triangle](/assets/the_lonely_green_triangle.png)