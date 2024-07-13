# C++ Notes

## `virtual` keyword
Let's say you have a base class function `a()` that calls a private base class function `b()`. In your derived class, you only redefine `b()` and not `a()`.

If you do not use the `virtual` keyword when defining `b()` in the base class, the derived class's `b()` will never be called! You do not need `override` in the derived class, as far as I can tell, and using `virtual` in the base class is fine even if the base class is not an abstract class! 