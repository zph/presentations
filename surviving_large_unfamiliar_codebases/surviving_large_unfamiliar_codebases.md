% Surviving Large Unfamiliar Codebases
% Zander Hill (@_ZPH)
% July 10, 2014

# A Harrowing Tale of Legacy Apps and Oracle Databases

# My Background

## Zander

- Polyglot. Ruby by day, statically typed by night. Remote work evangelist.

- Use vim for coding and Emacs for writing this presentation.

- <3 modal editing and longs walks through documentation.

## Where I Work

- Work at a product company on team with 20 Engineers. Around 80 Engineers when including `Ruby/Js, Clojure, .NET, and Ops` teams.

- We've had our primary website since 1999 (.NET).

- Currently support two primary sites and ~ 10 secondary services.

# Legacy Code

## We've got history :).

- Started in 1999 (date of first website on wayback machine)

- 2 Apps still on `ruby-1.8.7`.

- At least one still on `rails-2.3.x`.

## Proof

![Old Site](images/1999-apartment-guide.jpg)

## RESPECT

![](images/ali-g-respect.jpg)

## alt Respect
![](images/honor-legacy-code.jpg)

## Mess
![](images/legacy-code-mess.jpg)

## Layers

## 
![](images/tcp-ip-cat-layers.jpg)

## 
![](images/osi-network-layer-cats.jpg)

## 
![](images/cat-pile.jpg)

# Remainder
## What am I calling Large

- Prelude doesn’t encode uncertainty in the API where it’s possible to do so for partial functions

- `head :: [a] -> a` — is a damn lie, surpassed in damnation only by statistics and politicians

- Should be — `head :: [a] -> Maybe a`

## SLOC and costs of system

- length cannot and should not exist for a type that has the capability to be coinductive. 

- Nuts.

- `length [1..]` -> bottom  — this is gross. We’re betraying new people by leaving stuff like this in Prelude.

## Discuss Ruby vs. Javascript Code (TDD importance on Js side)

- What made head dangerous was the empty list case. There’s another problematic case - codata.

- Coinductive lists are things like: [1..]

- Coinduction guarantees productivity but not termination

## Crafting with negative space

- Part of the value in learning Haskell is learning to craft with negative space

- Most dyn-langs give you nothing *but* positive space to work with, but you have to no ability to *eliminate* possibilities.

- This forces imprecision and implicitness in your APIs. This is unacceptable in 2014.

## Crafting with negative space

- The problem is that Prelude [a] conflates inductive and coinductive use-cases. Sometimes this is convenient, but unwise as a universal data structure. Is so for historical/simplicity reasons.

- Use Vector or Sequence

- and don’t use head dammit

## Domain and codomain

- String is a ridiculous type to have in your signatures

- Probably lying. Is your function meant for *ALL POSSIBLE* Strings in the universe?

- String being [Char] under the hood, this also conflates induction & conduction and making this the default text-y type is icky.

## Domain and codomain

- Int has a domain of  18446744073709551615 values.

- Are you sure that’s what your API means when it says “Int”? Seeing String and Int in types is an API smell

- Do you mean a hardware Int, size implicit to the machine int?

## newtype to the rescue!

- ```haskell
data Webserver = Webserver String
```

- Nuke the String from orbit. You don’t mean all possible strings!

- ```haskell
newtype Host = Host String
data Webserver = Webserver Host
```

- A modest improvement, but not compelling.
Could still construct:
Webserver (Host “GOBBLEBLUH”)

## newtype to the rescue!

- ```haskell
newtype Host = Host String
data Webserver = Webserver Host
```

- The problem is that our data isn’t being validated.

- ```haskell
-- hide the Host constructor
newtype Host = Host String
data Webserver = Webserver Host
mkHost :: String -> Maybe Host
mkHost hstr = -- …validation code…
```

## newtype to the rescue!

- ```haskell
newtype Host = Host String
data Webserver = Webserver Host
mkHost :: String -> Maybe Host
```

- But how do we use this? Webserver doesn’t expect Maybe Host.

- Control.Applicative! Or if you have just one argument, Functor.

## newtype to the rescue!

- ```haskell
newtype Host = Host String
newtype Port = Port Int
data Webserver = Webserver Host Port
mkHost :: String -> Maybe Host
mkPort :: Int -> Maybe Port
```

- ```haskell
server :: Maybe Webserver
server = Webserver
  <$> mkHost “google.com”
  <*> mkPort 80
```

- Thanks to Applicative, we don’t need to break out intermediate but independent uncertainty WRT Maybe values. Now we’re being explicit about uncertainty for possibly bad inputs.

## But why is newtype important?

- newtype drops all the baggage associated with a type, but getting to use its representation for free. It’s erased at compile-time. This means it’s effectively free WRT performance.

- Not exporting a value constructor on a newtype or data type also gives you data hiding that doesn’t get violated nearly as often as in other languages (Scala, Java, Ruby, Python)

- This also means you have no real excuse for not using it.

## But why is newtype important?

- Baggage dropped includes typeclass instances

- This is how we maintain canonicity with typeclasses with types that share a representation

- As a different language community would put it, we don’t *complect* the object with its representation

## Record syntax is also dangerous

- Uh, with sums of products that is. It’s okay otherwise.

- ```haskell
-- assume appropriate deriving
data Blah = Woot | Alt { access :: Int }
λ> :t access
access :: Blah -> Int
```

- but what if it’s Woot instead of Alt?

## Record syntax is sorta dangerous

- ```haskell
λ> blah Woot
*** Exception: No match in record
selector blah
```

- Well that’s not okay

## Record syntax is dangerous if you’re being silly

- You have two solutions to this sum-of-products & record syntax problem.

- 1: Don’t use record syntax

- 2: Split the record type out

## Record syntax isn’t so bad

```haskell
newtype AltProxy = AltProxy { access :: Int }
data Blah = Woot | Alt AltProxy
λ> :t access
access :: AltProxy -> Int

λ> blah Woot
-- this is now a type error
-- as Go^H^H SPJ intended.
```

## Record syntax isn’t so bad

```haskell
newtype AltProxy = AltProxy { access :: Int }
```

^^ doesn’t have to be a newtype and can’t be if there’s more than one field anyway

Remember newtypes are just a cost-free way to create single value constructor data types.

## Food for thought

- Consider how splitting out more granular types can allow you to circumscribe value inhabitants of types in a way reminiscent of what dependently typed languages will allow.

- If audience is curious and time allows, I can bring up an example demonstrating the (trivial) idea.

