# How to write unmaintainable Haskell code

1. Use rewrite rule pragmas to alter functions. The compiler doesn't check whether the rewrite rule makes sense at all, so feel free to sprinkle your code with silly rewrites.
```haskell
    {-# RULES "reverse map/append"
        forall f xs ys. map f (xs ++ ys) = map f ys ++ map f xs #-}
```

2. The IO monad is your friend! Take the restrictivity of Haskell away, when you're done use QuickCheck to show  it's referentially transparent and wrap it in unsafePerformIO.
```haskell
    len xs = unsafePerformIO $ do
        count <- newIORef 0
        let getLength ys = case ys of
            []      -> return ()
            (y:ys') -> modifyIORef' count (+1) >> getLength ys'
        getLength xs
        readIORef' count
```
(Make sure to use `modifyIORef'` or your code may be unsafe.)

3. `Writer` < `State`

4. `State` < `IORef`

5. `IORef` < `RWST IO`

6. A suitable type to write something is `Writer`. A suitable type to modify state is `State`. `IORefs` are good in general. I'm sure you already guessed how to combine this with the previous points.

7. QuickCheck is a statistical test that generates false sense of security, and should be avoided for that reason.

8. All laws in Haskell are meant to be broken. Got a monad, but `m >>= return` just won't be `m` again? A `-- hack, careful` should be sufficient documentation.

9. Haskell's expressiveness allows coding in a very concise style. For example it renders comments redundant and makes you able to call all your types `T` and typeclasses `C` and them import them qualified.

10. Never import a qualified module under the same name twice. Combines well with the previous point.

11. Boilerplate code should be avoided; GHC will complain when it requires explicit type annotations.

12. If you want your program to be run just as you've written it, disable compiler optimization rewrites. The nicer way of doing this is by using the `-fno-enable-rewrite-rules` flag when compiling, however it is more effective to define a rule that makes GHC go into an infinite loop when compiling, forcing the compilation to be done like this.
```haskell
    loop a b = ()
    {-# RULES "loop" forall x y. loop x y = loop y x #-}
```
Note that you have to use loop somewhere so it's not optimized away. A good way is having `return $ loop 1 2` as the last function in main.

13. Naming conventions can help making code more readable. For example in `(xs:x)`, `xs` stands for "x singular", and `x` contains the rest of the x.

14. Use built-in functions as identifiers. Make sure to mention the name in the docs multiple times. Then create a base case that doesn't work for that operator.
```haskell
    times 0  _  _ = 1
    times n (+) x = x + power (n-1) (+) x
    -- 2*3 = ?
```

15. ```haskell
    import Prelude hiding ((+))
    import qualified Prelude
    a + b = a +. b +. c
          where (+.) = (Prelude.+)
                c = signum $ b `rem` 13
```