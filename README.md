# Approach
A small c++ module supporting the "approach" operation: the act of converging one value toward another in a frame rate agnostic way, where the speed may depend on the distance, similar to a spring, but may use other strategies such as simple linear speed to accomplish the goal as well.

Example:
```c++
float getUpdatedValue(float currentValue, float targetValue, float dt)
{
    return approach(currentValue, targetValue, dt, Linear(4.0f));
}
```

# Motivation
I've often seen code like the following somewhere in an update loop in a game project:
```c++
float updatedValue = lerp(someValue, someTargetValue, 0.1f);
```

This code is very much not frame rate agnostic! The next tier of sophistication looks like this:
```c++
float updatedValue = lerp(someValue, someTargetValue, blendSpeed * dt);
```

This behaves reasonably for some combinations of blendSpeed and dt, but not all. So the question becomes: what is the actual mathematical operation that is being attempted here? To begin with we can see the velocity of the convergence will be dependent on the distance between the values. To simplify we can pretend we are always converging towards zero. That would look something like this:

$y'(x)=-Ay(x)

The solution to this differential equation is $y(x)=c_1e^{-Ax}$ which is a simple exponential decay! So if we are willing to take the performance burden of computing an exponential we can easily make this bit of code perfectly frame rate independent as follows:
```c++
float updatedValue = someValue * expf(-rate * dt);
```

So that's great! I've done just that in code refactors, including writing settings conversion code so that the exp version will run with a rate exactly equivalent to the old lerp version at a specific frame rate (which is a fun exercise for the reader!) Because of the exponential decay, this type of convergence tends to feel very spring-like. In fact, if you want this feel, and you are able to track more state, then you may just want to use a spring anyway. But I like flexibility, I'd like to be able to swap among a variety of different strategies to accomplish this convergence task. A few templates later and here we are.




