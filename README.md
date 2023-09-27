# Approach
A small c++ module supporting the "approach" operation: the act of converging one value toward another in a stateless, frame rate independent way, where the speed may depend on the distance. A common strategy is exponential decay, which is similar to a spring, but other strategies such as simple linear speed can be used as well.

Examples:
```c++
float getUpdatedValue(float currentValue, float targetValue, float dt)
{
    return approach(currentValue, targetValue, dt, Linear(4.0f));
}
```

```c++
Quaternion getUpdatedValue(Quaternion currentValue, Quaternion targetValue, float dt)
{
    return approachDirection(currentValue, targetValue, dt, Exponential(2.0f));
}
```

```c++
SpringState<Vec2> getUpdatedValue(SpringState<Vec2> currentValue, Vec2 targetValue, float dt)
{
    return approach(currentValue, targetValue, dt, Spring(2.0f, 0.1f));
}
```

# Motivation
I've often seen code like the following somewhere in an update loop in a game project:
```c++
float updatedValue = lerp(someValue, someTargetValue, 0.1f);
```

This code is assumes a fixed frame rate. A bit better is this:
```c++
float updatedValue = lerp(someValue, someTargetValue, blendSpeed * dt);
```

This behaves somewhat uniformly for some combinations of blendSpeed and dt, but not all. But what is the actual mathematical operation that is being attempted? To begin with we can see the velocity of the convergence will depend on the distance between the values. To simplify we can pretend we are always converging towards zero. That would look something like this:

$y'(x)=-Ay(x)$

The solution to this differential equation is $y(x)=c_1e^{-Ax}$ which is a simple exponential decay! So if we are willing to take the performance burden of computing an exponential we can easily make this bit of code perfectly frame rate independent with a little bit of rearranging to account for blending toward a target instead of zero:

```c++
float updatedValue = lerp(someValue, targetValue, 1.0f - expf(-rate * dt);
```

So that's great! I've done just that in code refactors, including writing settings conversion code so that the exponential version will run with a rate exactly equivalent to the old lerp version at a specific frame rate (which is a fun exercise for the reader!) Because of the exponential decay, this type of convergence tends to feel very spring-like. In fact, if you want this feel, and you are able to track more state, you may just want to use a spring anyway. But for flexibility it would be nice to be able to swap among a variety of different strategies to accomplish this convergence task. 

Some examples of strategies:
- exponential decay, but put a speed limit on it so it doesn't go too fast at large distances.
- linear convergence, but never let the distance become greater than a specific radius
- converge _faster_ as you get closer, like a magnet
- and so on...

Another desire is to be able to use approach on as many types as possible, and in both linear and angular contexts. The basic requirement for types that want linear space approach is just that they can do enough arithmetic to support a lerp function (+,-,* w/ float), and compute a distance magnitude, either through a dot product that yields a floating point scalar, or by using abs on a scalar. For types that operate in an angular context the lerp math and dot product are required.

# Requirements
- c++20 concepts and modules
- tested in msvc++ only so far

I've been using variants of this code personally for years, however, this particular project is mostly an excuse to experiment with newer c++ features, such as concepts and modules. The module currently depends only on cmath (for sqrt, exp, and trigonometric functions)
