# iOS App Architecture With RxSwift And Redux

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

It's time to consult the **Redux Turtle** to see if indeed we are doing this right (so far).

![](https://i.imgur.com/qk0xAsX.png)

Her name is 













