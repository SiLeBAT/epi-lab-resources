# NgRx - Actions

**Style guide for defining actions**
* https://ngrx.io/guide/store/actions

## Overview of using actions

Actions are mainly used to describe the UI-Flow and corresponding state transitions of an app. They can be triggered by different sources:
* The user interacting with the app
* The app reacting to a server event
* The app reacting to other actions and their side effects

For a developer there are two main places where to use actions:

**An action can be dispatched by a source**

An action can be dispatched by different sources anywhere in the code. This could be e.g. a component, an effect or a service.

**A target can react to an action**

There are two patterns to react to an action:
* You can reduce an action with a reducer to change the state of the store. This can trigger a state change of some component using a selector from this state.
* You can also react to the action by using an effect and trigger state changes by calling services or dispatching new actions in the effect.

Both patterns change the state of the app, so these are both referenced as a target reacting to an action in this document.

![Simple action chart](./images/NgRx-Actions-Simple.png)

## Problems with using actions

By using actions it is easy to produce bugs that are difficult to trace. These bugs are mainly produced if an action has different sources and targets. So it is recommended by NgRx to use only actions with a single unique source. We decided to use also actions with different sources, but take special precautions to avoid undesired behaviour.

### Different sources
![Simple multi source action chart](./images/NgRx-Actions-Simple-Multi-Source.png)

### Different targets
![Simple multi target action chart](./images/NgRx-Actions-Simple-Multi-Target1.png)
![Simple multi target action chart](./images/NgRx-Actions-Simple-Multi-Target2.png)

The problems mainly occur if different developers, or the same developer at different times uses an already defined action. The following use cases try to explain these problems. In all examples a second developer uses actions with an already defined action-target flow by a first developer.

### Example 1: A developer dispatches new action from source B, unaware of target B
The developer dipatches an already defined action from a new component. He is aware of Target A and the corresponding side effect. He is not aware of Target B, but target B produces an undesired side effect not intended by the developer for the new component.

![New source problem chart](./images/NgRx-Actions-Problem-New-Source.png)

### Example 2: A developer adds a new target B, unaware of source B
The developer writes a new target B with a side effect for an action. He is targeting source A of that action. He is unaware of source B. The new target B produces a side effect with undesired results for source B.

![New target problem chart](./images/NgRx-Actions-Problem-New-Target.png)

## Style Guide

To reduce bugs from the described problems we define rules for using actions by defining three semantic types of actions. The types are mainly defined by the number of sources dispatching these actions.

### Event action

This is an action as recommended by NgRx. It has a single unique source, that means there is only one line of code where it is dispatched.
* A developer is not allowed to reuse this action. He can change the action to one of the other two types, check all targets and side-effects and then reuse the action.
* A developer is allowed to add new targets for the action.

![Event action chart](./images/NgRx-Actions-Type-Event.png)

**Naming convention**

As for all action types the name of the action describes the intent / next state of the application. The action.type describes the source and the event that triggered the action.

    <Intent>.type = '[<Module>/<Source component or source sub module>] <Description of event>'
    
    Login.type = '[User/LoginPage] Login button clicked'
    SendSamples.type = '[Samples/ValidateSamples]  Samples successfully validated'
    
### Multi action

This is an action with multiple sources. It has no information about the specific source it was dispatched from. All target effects are desired for all sources.
* A developer can reuse this action. He must check all sources and targets of the action for undesired side-effects when dispatching it again.
* A developer can add a new target for the action. He must check all sources and targets of the action for undesired side-effects produced by the new target.

![Action action chart](./images/NgRx-Actions-Type-Action.png)

**Naming convention**

See event action. The action.type describes the action itself and where the action is defined or handled.

    <Intent>Action.type = '[<Module>/<Action sub module>] <Description of action>'
    
    LoginAction.type = '[User/Login] Login user'
    SendSamplesAction.type = '[Samples/SendSamples] Send samples to server'
    
### Command actions

The command action is an action with multiple sources. It has information about the specific source it was dispatched from. Targets react only to specified sources of the action.
* A developer can reuse this action. He must add the new source to the desired targets of the action.
* A developer can add a new target for the action. He must specify the sources the new target shall listen for.

![Command action chart](./images/NgRx-Actions-Type-Command.png)

**Naming convention**

See multi action.

    <Intent>Command.type = '[<Module>/<Action sub module>] <Description of action>'
    
    LoginCommand.type = '[User/Login] Login user'
    SendSamplesCommand.type = '[Samples/SendSamples] Send samples to server'
    
To use command actions the developer must implement a technique to store the source of a command. A command/response pattern is already implemented and can be used. It is described in the corresponding DevGuide document.
