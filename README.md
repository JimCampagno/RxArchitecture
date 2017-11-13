# iOS App Architecture Using Redux & RxSwift

![](https://i.imgur.com/yCjzYVL.jpg)

## Redux & Swift

Redux is a predictable state container for JavaScript apps.

It helps you write applications that behave consistently, run in different environments (client, server, and native), and are easy to test.

The whole state of your app is stored in an object tree inside a single store. The only way to change the state tree is to emit an action, an object describing what happened. To specify how the actions transform the state tree, you write pure reducers.

### Actions

Actions describe the fact that something happened, but don't specify how the application's state changes in response. The following is an `enum` that will represent _what happened_.

```swift
enum Action {

    case run
    case walk
    case sit

}
```

Throughout our application, there will be parts of our code-base that will need to generate `Action`s. As it stands, lets imagine that a user in our app can tap three separate buttons that will _generate_ an action.

![](https://i.imgur.com/lFDbscX.png)

If I were to then hand this `action` constant to another part of our code-base, we can inspect the type by switching on it and then _react_ to it, like so:

```swift
// action constant created as a result of a button tap
let action: Action = .run

// somewhere else in our code-base
switch action {

case .run:
    print("Run Forrest, run.")
case .walk:
    print("Walk slowly my dear Watson")
case .sit:
    print("One letter away from 💩")

}

// 'Run Forrest, run.' prints to the console
```

This `Action` type describes the state of our app.

What is the _state_ of our app?

```swift
typealias Energy = Double

struct State {

    var energy: Energy = 50

}
```

Running, walking, and sitting will affect the `energy` level of our `State` instance. Lets create the following key which we can reference:

Action | State (energy level)
--- | ---
Sit | +10 
Run | -15
Walk | -5

The next move might then be to add some new methods to our `State` type which can handle these various actions, like so:

```swift
struct State {

    var energy: Energy = 50

    mutating func handle(action: Action) {
        switch action {
        case .sit:
            energy += 10
        case .run:
            energy -= 15
        case .walk:
            energy -= 5
        }
    }

}
```

This then allows us to use our `State` type as follows:

```swift
var initialState = State()

initialState.handle(action: .run)
initialState.handle(action: .walk)
initialState.handle(action: .sit)

print(initialState.energy)
// Prints "40.0"
```

It's time to consult the **Redux Turtle** to see if indeed we are doing this right. Her name is Barbara, but she goes by Barb and really knows her stuff.

![](https://i.imgur.com/qk0xAsX.png)

After glancing at our code, she begins to whisper something, "Reducer. You must use a reducer". Thanks Barb.

### Reducers

Barbs word of advice was:

> Reducers provide pure functions, that based on the current action and the current app state, create a new app state

Lets remove the `mutating func` from `State`, which would leave us with the `State` type looking like this:

```swift
struct State {

    var energy: Energy = 50

}
```

Lets now create a **reducer**:

```swift
func energyReducer(action: Action, state: State = State()) -> State {
    var copy = state

    switch action {
    case .run:
        copy.energy += -15
    case .sit:
        copy.energy += 10
    case .walk:
        copy.energy -= 5
    }

    return copy
}
```

Note that using `copy` here as a variable name within this function is probably not the best name for it (who knows, maybe Barb knows best). I'm doing this because I want to stress that indeed the `copy` variable is a _copy_ because the `State` type is a struct and when using the assignment operator this way, then a copy of that instance is made and assigned to the `copy` variable. If a user of this function did not provide a value for the `state` parameter, then `copy` would be assigned a new value of `State` where the default `energy` stored property would have a value of `50`. Technically a shallow copy is made versus a dee copy so be conscious of the stored properties on your `State`, looking out if they are reference types versus value types. Barb can chew your ear off if you want to get into a discussion about Classes vs Structs.

Now instead of allowing the developers of our team (and ourself) the ability to mutate (change) any stored properties of the instance of our `State`, they are left _having_ to go through our reducer no matter what. We can ensure this by instead making a constant (using the `let` keyword) versus `var` of our `State`, like so:

```swift
let initialState = State()
```

This protects us from code that can be written like this:

```swift
initialState.energy = 50 // won't compile
```

In fact, that won't even compile. You will be given the following error:
> note: change 'let' to 'var' to make it mutable

There's _one_ place that can handle an `Action` in our code-base, and that's through the reducer function we made. Lets put it to use. Before reading the explanation of the following code, step through it line by line to make sure you understand and can follow along.

```swift
let initialState = State()

let stateAfterRun = energyReducer(action: .run, state: initialState)
let stateAfterWalk = energyReducer(action: .walk, state: stateAfterRun)
let stateAfterSit = energyReducer(action: .sit, state: stateAfterWalk)

print(stateAfterSit.energy)
// Prints "40.0"
```

It's not that we're mutating our initial instance of `State` which is stored in our `initialState` constant. Each call to `energyReducer(action:state:)` will create a _new_ instance of `State` and return it back to the caller of the function. 

Rules we should follow:

- NEVER mutate the arguments of the reducer function
- NEVER perform side effects like API calls within the reducer function

So far, so good. In consulting with Barb, she had recommended that we seek further advice from the wisest turtle in town. In fact, that's the nick-name the species has decided to bestow upon their great one.

Her name is Maryann, the first of her name and she commands the most respect upon any question. Rumor has it that she's so big, that there's a castle on her back which she refers to as "the store".

![](https://i.imgur.com/IG9vrbM.jpg)




















