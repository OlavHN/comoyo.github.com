---
layout: post
author: bakksjo
title: The inverse of IoC is Control
category: Java
---
##Boy, everyone is stupid except me##

Sometimes, I look at where the Java developer community is heading and I think
to myself: “They’re all doing it wrong!”. This leads naturally to the follow-up
thought: “Or am I the only one not getting it?”

This time, I’m surprised by people’s affection of IoC containers. Spring, for
example. I don’t get it. What problem is that solving? Before, we used to wire
our application together in the main() method. Now, apps are wired using long,
unreadable XML files, and if there’s a typo in it, you won’t know about it
until run time. What was wrong with main()?  

##Use the compiler, Luke##
The same principles spread out to other Java software as well. Take Jersey, for
instance. You don’t give the Jersey server a set of resources to host. Oh no.
You give it a Java package name - a string -- and then Jersey searches that
package for classes that presumably are resources it should instantiate and
host. Jersey finds them (or not) at run time. Hopefully, your program works
when you launch it.

To quote the Wikipedia article on IoC: *“[...] object coupling is bound at run
time by an assembler object and is typically not known at compile time using
static analysis.”* So, what reasons are there to do the coupling at run time,
and why do people accept losing the static analysis of a compiler? I can think
of one such case: Plugins. When you are developing a system where you want to
be able to add or remove plugins without restarting your application. So if
you’re developing a browser or an IDE, you might need support for loading code
into your runtime dynamically (although this has little to do with IoC,
really). But how many of the world’s Spring-based projects have this property?
My Jersey-based REST service certainly won’t support adding new resource
classes at run time. If I need to add a resource class to it, I’ll statically
build a new release and redeploy the binary. I know up-front what parts should
make up my application, so why would I want to do the coupling at run time?

When I browse code in a new project, I want to see how stuff is wired together.
“Who instantiates this class, and when?” The IoC container approach leads to
code that is “detached”; you can see a class being active at run time, but you
can’t see a single code line in your project actually mentioning that class.
You don’t know the instantiation order, the lifecycle, nothing. You simply
don’t have control.  

##Wisdom of the crowds##
Googling the web for answers, it seems to me that people confuse IoC with DI -
Dependency Injection. It’s all over stackoverflow if you look. People ask what
IoC is, and they’re shown an example of DI. People are taught that to do DI,
you need an IoC container.  

##Don’t get me wrong##
I’m all in favor of DI. I use it all the time. I easily inject mocks, test
units of code in isolation, with minimal plumbing and setup time - all without
a container in sight. And I wire the program together in the main() method. You
can figure out what my program does from start to end solely by reading Java
code, and you don’t have to switch contexts or make assumptions to understand
how it works. So I’m not against the concept of inversion of control *in
programming* as such. I’m against the *container* approach that people tag onto
the concept.  

##There’s no configuration like code##
I’d argue not only that IoC containers are *unnecessary* for DI and testability,
they’re even *harmful*: Most of the out-of-the-box container injection I’ve seen
is inherently *static*, meaning they have a one-instance-fits-all ideology.
Dependencies are only declared by *type*, so there is this assumption that
different instances is not an issue. If `Foo` needs an instance of `Bar`, the
container might use a `BarProvider` or `BarFactory` (that it also locates at run
time - bleh!) to create an instance of `Bar` and give it to `Foo`. But what if
there’s a `Baz` that also needs a `Bar`, but not the same `Bar` that `Foo` got, but one
that is configured differently? BarProvider doesn’t know much about the context
in which it is being called - whether it is asked to make a Bar for Foo or Baz.
Now suddenly the looks-good-on-a-slide IoC container usage example doesn’t look
quite so beautiful anymore. Enter ugly config files with something that looks
suspiciously like code with conditional logic encoded into XML. It’s a DSL,
another special-purpose language and framework you have to learn. What was
wrong with Java as a programming language? What’s more flexible than main() as
a place to declare initialization order and inject dependencies?

Sure, IoC containers may be viewed as a tool to turn *imperative* setup into a
*declarative* setup. And declarative is better than imperative, right? Many times
I agree. But a Java program is unavoidably imperative, and when you introduce a
framework to make a small (but important) subset of your program declarative,
you’re not only increasing complexity, you’re mixing paradigms. This adds a lot
of WTF-ness to your program.
