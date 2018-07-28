---
layout: post
title: "Software Engineering: Introduction to GUI Programming (Incomplete)"
date: 2018-06-13
---
A brief overview of GUI programming, mostly resolved around the tooling available on the JVM platform.
For lower-level concepts, I'll be using Linux and X Window System as examples.

## Input handling

_(Alternatively, what happens when you press a key)_

The keyboard sends a signal to the interrupt controller, which is forwarded to the CPU.
In response to the interrupt raised by the CPU, the kernel runs a device-specific function -- an interrupt handler.

In case of Linux, it writes the input to a special _device file_, which can be read from
using [open/read syscalls](https://stackoverflow.com/a/23317086/1726690).
The data is raw, and needs some processing before it can be used in GUI applications -- e.g.
translating keyboard codes into symbols with respect to user's layout, keeping track of pointer coordinates, and so on.

This part of input handling is usually done by the display server (X11), which has multiple clients (applications) connected to it.
The clients are waiting for events, such as key presses/releases and mouse motions,
and the server has to decide which client to notify about an occurred event.

In windowing systems, each window is associated with a specific application,
and input events are sent to the one that is currently _in focus_, e.g. is under the mouse pointer.

Higher up the chain are toolkits, libraries which encapsulate lower-level events
and graphics primitives and provide a set of UI components -- widgets.

## Java GUI toolkits

The _Abstract Windowing Toolkit_ (AWT) is the oldest one. It provides a platform-independent set of widgets
with platform-specific implementations (peers).

Of particular interest to us is how it dispatches events: event sources (components)
keep a list of event listeners and notify them of state changes.
There are different kinds of events (key event, mouse event, etc.), and
subscribing to them is done via different methods (`addMouseListener`, `addActionListener`, etc.).

The problem with AWT is that its components are native and may have a
different look and feel across platforms (and other subtle differences as well.

This is where _Swing_ comes in: its widgets are lightweight: they're implemented in Java, with
consistent and customizable look and feel across different platforms.
Some controls exist in both Swing and AWT (`JButton` vs `Button`),
while some higher level ones are unique to Swing (`JTree`).

Swing components paint themselves, while AWT delegates this to native tools.
This can lead to performance issues, which, however, are negligible nowadays.

## Class hierarchy

`Component` is the abstract parent of AWT components, with an abstract method `paint`
that defines how the object is painted, and a `repaint` method, which can be invoked by user
to requrest a repaint. This class is extended by a number of widgets, such as `Button`, `Label`,
`TextField`, `TextArea`, `Checkbox`, and so on.

`Container` is a generic container class that can contain other AWT components, which are
put in a list in the order of addition. The order defines the stacking order (the last added
component is added to the bottom of the stacking order). There are two common varieties of `Container`s,
a `Frame` -- a top-level window, -- and a `Panel`, which must be inside another container such as `Frame`.

_to be continued..._
