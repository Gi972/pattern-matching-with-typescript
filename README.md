# Pattern Matching with TypeScript

[TypeScript](https://www.typescriptlang.org/index.html) does not have any pattern matching functionality built in. This article shows several ways how you can replicate the core of a simple pattern matcher using a few simple structures and functions within TypeScript.

Resulting code will have improved maintainability and better runtime type safety when done right.

## What is Pattern Matching?

Pattern matching is a fundamental and powerful building block to many functional programming languages like [Haskell](http://learnyouahaskell.com/syntax-in-functions) or [Scala](http://docs.scala-lang.org/tutorials/tour/pattern-matching.html).

*TODO: Disclaimer: Language-Level Pattern matchers go further than what we build here… Though it's a nice abstraction pattern to improve code maintainability*

A *pattern* can contain one or more *cases*. Each *case* describes *behavior* which has to be applied once the *case* matches.

You might think: *"Hey! That sounds like a `switch` statement to me!"*. And you are right indeed:

## Match with Switch Statement

```typescript
function matchNumber(n: number): string {
  switch (n) {
    case 1:
      return 'one';
    case 2:
      return 'two';
    case 3:
      return 'three';
    default:
      return `${n}`;
  }
}

function randomNumber(): number {
  return Math.floor(Math.random() * (10 - 1) + 1); // Random number 1...10
}

console.log(matchNumber(randomNumber()));
```

We can use a `switch` statement to map `number`s to its desired `string` representation.

Doing so is straightforward, though we can make out flaws for `matchNumber`:

1. The *behavior* for each case is baked into the `matchNumber` function. If you want to map `number`s too, let's say `boolean`s, you have to reimplement the complete `switch` block in another function.
2. Requirements can be misinterpreted and behavior for a case gets lost. What about `4`? What if a developer forgets about `default`?
   The possibility of bugs multiplies easily when the `switch` is reimplemented several times as described under point 1.

Trying to solve these flaws outlines requirements for an improved solution:

1. Separate matching a specific *case* from its *behavior*
2. Make reuse of matcher simple to prevent bugs through duplicated code
3. Implement matcher once for different types

## Separation of Concerns

Let's define an interface containing functions for each *case* we want to be able to match. This allows separating behavior from actual matcher logic later.

```typescript
interface NumberPattern {
  One: () => string;
  Two: () => string;
  Three: () => string;
  Other: (n: number) => string;
}
```

Having `NumberPattern`, we can rebuild `matchNumber`:

```typescript
function matchNumber(p: NumberPattern): (n: number) => string {
  return (n: number): string => {
    switch (n) {
      case 1:
        return p.One();
      case 2:
        return p.Two();
      case 3:
        return p.Three();
      default:
        return p.Other(n);
    }
  };
}
```

The new implementation consumes a `NumberPattern`. It returns a function which uses our `switch` block from before with an important difference: It does no longer map a `number` to a `string` on its own, it delegates that job to the pattern initially given to `matchNumber`.

Applying `NumberPattern` and the new `matchNumber`  to the task from the previous section results in the following code:

```typescript
const match = matchNumber({
  One: () => 'One',
  Two: () => 'Two',
  Three: () => 'Three',
  Other: (n) => `${n}`
});

console.log(match(randomNumber()))
```

We clearly separated *case behaviors* from the matcher. That first point can be ticked off. Does it further duplicating code and improve maintainability of the matcher?

```typescript
const matchGerman = matchNumber({
  One: () => 'Eins',
  Two: () => 'Zwei',
  Three: () => 'Drei',
  Other: (n) => `${n}`
});

console.log(matchGerman(randomNumber()))
```

Another tick! Because we have split concerns by introducing `NumberPattern`, changing behavior without reimplementing the underlying matcher logic is straightforward.

## Truly Reusable

Map a `number` to something different than a `string` still needs reimplementation of `matchNumber`. Can we solve this without doing so for each target type over and over again? Sure! [Generics](https://www.typescriptlang.org/docs/handbook/generics.html) provide an elegant solution:

```typescript
interface NumberPattern<T> {
  One: () => T;
  Two: () => T;
  Three: () => T;
  Other: (n: number) => T;
}

function matchNumber<T>(p: NumberPattern<T>): (n: number) => T {
  return (n: number): T => {
    // ...
  };
}
```

Introducing the generic type parameter `T` makes `NumberPattern` and `matchNumber` truly reusable: It can map a `number` to any other type now. For example a `boolean`:

```typescript
const isLargerThanThree = matchNumber({
  One: () => false,
  Two: () => false,
  Three: () => false,
  Other: n => n > 3
});

console.log(isLargerThanThree(100));
// results in true
console.log(isLargerThanThree(1));
// results in false
```

This fulfills the last point in our requirement list to implement the matcher once for different types. The final example will probably never make it to production code, though it demonstrates the basic mechanic how a pattern and a corresponding matcher can be implemented in TypeScript.

## Matching Union Types

