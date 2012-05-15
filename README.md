# Stategraph — evented data structure for untangling state!

Stategraphs are a great tool for managing state, it can be used to simplify
things such as game logic, or UI flow.  This library provides evented states,
and nested graphing!  Woah!

## Documentation

### Constructing

To begin using stategraph, simply require it, and instantiate an object!

    var StateGraph = require('stategraph');

    var graph = new StateGraph();

You can also extend the prototype; using plain old prototypal OOP, or with your
favorite OOP sugar, such as [Self](https://github.com/munro/self).

    function Game() {
        StateGraph.call(this);

        // `this` is now a stategraph!
    }
    Game.prototype = Object.create(StateGraph.prototype);

### State creation & movement

To create `State` nodes in your graph, call the
`.state(String name, Function callback)` method with the name of the state, and
a callback to run when the graph has entered that state.  The first argument of
the callback is the `State` node.  So you may attach state to that variable, and
it will still be there when you come back!

    graph.state('home', function (home) {
        assertTrue(home === this);

        home.count = (home.count || 0) + 1;
    });

To move between nodes, call the `.go(String name)` method with the name of the
state.

    graph.go('home');

You can also call `.go(String name, args...)` with extra arguments that will be
passed to the state callback.

    graph.state('game', function (game, players, map) {
        game.players = players;
    });

    graph.go('game', ['player 1', 'player 2'], 'some_map');

When you're all done with your graph, you can end it by calling `.end()`.
Though the graph can be started back up at any time with `.go(String name)`.

    graph.end();

### Evented state

When the graph leaves a state, you can bind a callback to the state's `end`
event.  Like the state callback, the first argument of the `end` event is
the `State` object.

    graph.state('hello', function (hello) {
        end.timer = setTimeout(function () {
            alert('Hello world!');
        }, 1000);
    }).on('leave', function (hello) {
        // Let's remove the trigger if the graph leaves the state before it
        // triggers.
        clearTimeout(hello.timer);
    });

You can also create & bind your own custom events.

    graph.state('world', function (world) {
        world.emit('test', 'just because');
    }).on('test', function (message) {
        alert('testing ' + message); // testing just because
    });

### Nested state

To nest state, simply define your first state by calling
`.state(String name, Function callback)` on your `State` object, and a
`StateGraph` will be lazily mixed into your state!  Giving you access to all
the methods in the `StateGraph` prototype.

    graph.state('lobby', function (lobby) {
        lobby.state('host', function (host) {
            // host can kick
        });

        lobby.state('player', function (player) {
            // can't do much
        });

        // players can chat
    });

To move about nested graphs, you can use the `.go(String name)` method to go to
the lobby state, which returns the lobby `State` object (which is now also a
`StateGraph`!)  So you can chain your nested graphs by calling `go` again!

    graph.go('lobby').go('host');

If you want to switch sub-states while in the lobby, simply call `state` without
and arguments to return the current state.  Then call go on our lobby `State`
object.

    graph.state().go('player');
    graph.state().go('host');

## Methods

* `StateGraph`
    * `state(String name, Function callback) -> State` — Define a state with a callback
        to be called when entered.  Remember the `State` context is the first
        argument of the callback!  This function returns the newly created `State`
        object.
    * `state() -> State` — Return the current state.
    * `go(String name[, args...]) -> State` — Switch states, passing any args to the state
        callback.  Returns the newly entered state.
    * `end()` — Leave the current state, ending the graph.  The graph can be
        reentered at any time!

* `State`
    * Inherits [events.EventEmitter](http://nodejs.org/api/events.html#events_class_events_eventemitter)
    * Lazily inherits `StateGraph` when the `state` method is called.
    * Events 
        * `leave` — Triggered when the graph leaves this state.  The first argument
            is the `State` object.

## Example

It is very useful to use a stategraph when handling the state of a user!  For
security purposes, you don't want to user to be able to trigger events when
they're not in that state.

For example, if you have a game where only hosts can kick other players, you
don't want a user to trigger the kick event if they're not a host!  So instead
of having a state check at the beginning of your events, you can simply remove
them!

    graph.on('host', function (host) {
        // Give host ability to kick
        host.on_kick = function (other_player) {
            game.kick(other_player);
        };
        player.on('kick', host.on_kick);
    }).on('leave', function (host) {
        // No more kick for you!
        player.removeListener('kick', host.on_kick);
    });

    graph.on('player', function (player) {
        // Poor player can't do anything!
    });

## License

(The MIT License)

Copyright (C) 2012 Ryan Munro <munro.github@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
