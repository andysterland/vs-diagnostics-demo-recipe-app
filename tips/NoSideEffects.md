## No Side Effect Expression Evaluation
Adding an expression to the watch window(s) will cause that expression to be evaluated in the context of where the debugger is paused. This is great, it allows you to inspect anything you want, or even execute code, the downside is that it can cause 'side effects' which in this case means you could alter the state of your application. Which is not so great, now normally this doesn't matter as most of the time the expressions you will be evaluating will typically, at worse, just result in a getter getting invoked. As getters should be idempotent and calling them should thus never change the state of your application side effects shouldn't be a problem, but if they are... You can use the `nse` format specifier after the expression, like so:

`<expression>, nse`

### Example: Executing Code in Watch Expressions
In this example we'll be executing some code in a watch expression that will change the state of a variable to illustrate the effect of `,nse` on watches.

1. Navigate to **Recipe.Service** Set a breakpoint at **line 24** in **Controllers/RecipesController.cs**.

![Breakpoint at line 24 in RecipeController.cs](NoSideEffect-SetBreakpoint1.png)

2. Launch the application and navigate to the root page. This should cause the controller above to run and the breakpoint to hit.

3. Back in VS go to the **Watch Window** and enter the expression `limit`.

![Watch 'limit' added with a value of 10](NoSideEffect-AddFirstWatch1.png)

4. Add a new watch with the expression `++limit` this will cause the value of `limit` to increment.

![Watch '++limit' added with a value of 11](NoSideEffect-AddWatchWithSideEffect1.png)

5. Add a new watch with the expression `++limit, nse`.

![Watch '++limit, nse' added with a value of 11](NoSideEffect-AddWatchWithNoSideEffect1.png)

6. Note how the value of limit is not increment again and stays at it's previous value.

**Bonus Tip**: When the last watch of `++limit, nse` was added the watch for `limit` was updated but the watch value for `++limit` was not updated. This was because `++limit` would have side effects and wasn't auto-evaluated you need to use the **refresh** icon on the right of the watch value to re-evaluate it. 


### Example: Getters with Side Effects
In this example, we have some crazy code that is using an IEnumerator in a getter property to return an item. The problem with this approach is that every time the property is accessed, the getter will cause the iterator to move to the next item. 

1. In the **Recipe.Service** project, navigate to **Models/RecipeManager.cs**. Add the code shown below inside the `RecipeManager` class definition.
    ```
    private IEnumerator<long?> keysEnumerator;
    public Recipe NextRecipe
    {
        get
        {
            if (!keysEnumerator.MoveNext())
            {
                keysEnumerator.Reset();
            }

            return Recipes[keysEnumerator.Current];
        }
        set { }
    }
    ```

    **Warning: The above is an insane property - getters should be idempotent.**

2.  In the `RecipeManager` constructor after the `foreach` loop, add the following code:

    `keysEnumerator = Recipes.Keys.GetEnumerator();`

3. In the **Recipe.Service** project, set a breakpoint at **line 24** in **Controllers/RecipeControllers.cs**.

4. Run the application, which should cause the breakpoint to hit.

5. In the **Watch Window** add the expression `RecipeManager.Singleton.NextRecipe, nse`.

6. Inspect the watch window item. 

This illustrates how `, nse` works. You can think of `,nse` as executing the code involved but without writing back to your application. So in this case when you look an item with `,nse` you get the next item every time you inspect the object but your application isn't changed. 

7. Stop the application and delete the code you added during this exercise.