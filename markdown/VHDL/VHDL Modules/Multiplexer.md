# Multiplexer

In reality, its unlikely we'd make this an entity in itself in a real design as it can very easily be implemented "inline" within another module, but I've included this as a module in its own right because we can learn a useful lesson here.  Below there is code showing a multiplexer created "from scratch" within a design as well as code instantiating a multiplexer from the entity we have created.  The point to takeaway here is that there is an overhead associated with including modules within our design.  Dividing our design into submodules to reduce complexity and improve encapsulation and reuse is valuable, but it isn't always worth it.  There are no hard and fast rules on when a system should be divided into modules and much of this will come down to your own requirements and preferences.

!!! TODO
    Ideally come up with a rule of thumb here

Below I've written all the code required to instantiate and use this module within a design.  I've also written how to simply create a multiplexer

!!! TODO
    Examine the code required to instantiate vs the code required to just implicitly make a multiplexer
