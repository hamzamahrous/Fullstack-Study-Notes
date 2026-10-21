## Fundamentals

### 1. What does TypeScript add on top of JavaScript?
- A **static type system** checked at compile time .. then the types are **erased** and plain JS ships.
- Note that TS adds **no runtime checks** .. data from an API can still break your "typed" code.

### 2. What is type erasure, and what does it mean for you?
- Types, interfaces and `import type` vanish from the output .. you **can't check a type at runtime**.
- Runtime checks only work on JS values .. `typeof`, `instanceof`, `in`, or a validator like Zod at the boundary.

### 3. What is type inference, and when should you annotate?
- TS infers types from values and returns .. annotating everything is noise.
- Annotate **function parameters** and **public API boundaries** (exported functions, service methods) to lock the contract .. infer the rest.

### 4. What is structural typing?
- Compatibility is decided by **shape, not name** .. two unrelated types with the same members are interchangeable.
- Note that **excess property checks** only run on an object literal written **directly** in the assignment .. `const u: User = { name, age }` errors, but passing the same object through a variable first compiles fine.

### 5. `type` vs `interface`?

| | `interface` | `type` |
| --- | --- | --- |
| object shapes | ✓ | ✓ |
| unions, primitives, tuples, mapped/conditional | ✗ | ✓ |
| declaration merging | ✓ | ✗ |
| extending | `extends` .. clearer errors, cached checks | `&` intersection |

- Rule of thumb .. `interface` for object contracts, `type` for everything else.

### 6. `any` vs `unknown` vs `never`?
- `any` .. **turns the checker off**, spreads silently.
- `unknown` .. the safe **top type** .. anything is assignable to it, but you must **narrow before using** it.
- `never` .. the **bottom type** .. no value can exist (impossible branches, functions that always throw).

### 7. `void` vs `undefined` vs `never` as a return type?
- `void` .. the return value **shouldn't be used**. `undefined` .. it literally returns `undefined`. `never` .. it **never returns** (throws or loops).
- Note that a callback typed `() => void` can still be given a function that returns something.
### 8. What does `strict: true` turn on?
- `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, `strictPropertyInitialization`, `useUnknownInCatchVariables`, and more.
- The big one is `strictNullChecks` .. without it `null` / `undefined` fit into every type.

### 9. `null` vs `undefined`, and `?` vs `| undefined`?
- `null` .. there is a value and it's nothing. `undefined` .. no value yet.
- `prop?: T` .. the key may be **missing**. `prop: T | undefined` .. the key **must exist**, value may be `undefined`.

## Composing Types

### 10. Union vs intersection?
- `A | B` .. **either** one .. you can only use members common to both until you narrow.
- `A & B` .. **both at once** .. members combined.

### 11. What are literal types and widening?
- A literal type is one exact value .. `'left'`, `42`, `true`.
- `const x = 'a'` infers `'a'`, `let x = 'a'` **widens** to `string`.

### 12. What does `as const` do?
- Infers the **narrowest literal types** and makes everything `readonly`, deeply.
- Note that it's **compile-time only** .. nothing is frozen at runtime.

### 13. Tuples vs arrays?
- Array .. any length, one element type. Tuple .. **fixed length, type per position** (`[string, number]`).
- Use `readonly [..]` for tuples you don't mutate .. otherwise `push` still compiles.

### 14. Index signatures vs `Record`?
- `{ [key: string]: number }` and `Record<string, number>` are the same for open-ended keys.
- `Record<'a' | 'b', number>` .. **requires** every key of the union .. an index signature can't do that.

### 15. Enums vs literal unions?
- Enums are a real JS object at runtime; unions are types only, so they’re erased (zero code).
- With unions you can pass plain strings directly: `setStatus('active')`. A string enum forces `Status.Active`.
- Prefer unions by default.

### 16. `readonly` vs `Readonly<T>` vs `Object.freeze`?
- `readonly` .. one property. `Readonly<T>` .. all top-level properties. Both are **compile-time and shallow**.
- `Object.freeze` .. real runtime immutability, but also **shallow**.

## Narrowing

### 17. What is narrowing, and what triggers it?
- Control-flow analysis refines a union inside a branch.
- Triggers .. `typeof`, `instanceof`, `in`, `===` checks, truthiness, a discriminant check, a type predicate.

### 18. What is a type predicate (custom type guard)?
- A function returning `value is T` .. a `true` result narrows the caller to `T`.
- Note that TS **trusts it blindly** .. a wrong predicate is a lie the compiler believes. 

### 19. What is an assertion function?
- `function assert(x): asserts x is T` .. it throws if wrong, and **narrows everything after the call**.

### 20. What is a discriminated union?
- Discriminated union = a union of object types, each with the same property holding a different literal value.
- Checking the tag (`if` / `switch`) narrows to that member, so its fields are guaranteed.
- `never` in `default` gives an exhaustiveness check.

### 21. How do you make a `switch` exhaustive?
- In `default`, assign the value to `never` .. adding a new member to the union then becomes a **compile error** there.

```TS
// ✗ errors the moment a case is missing
default: const _x: never = state; 
```

### 22. `as` vs `satisfies` vs an annotation?
- Annotation (`x: T`) .. checks and **widens** the variable to `T`.
- `as T` .. **overrides** the checker .. no safety.
- `satisfies T` .. **checks** against `T` but **keeps the narrower inferred type**.

### 23. When is a type assertion okay?
- When you know more than the compiler (DOM queries, tests) .. never to silence a real error.
- `!` is the same idea for nullish values. Note that `as unknown as T` is a **red flag** in code review.

## Functions & Classes

### 24. What are function overloads?
- Several signatures above **one implementation** .. the caller sees only the overloads, never the implementation signature.
- Order them **specific to general** .. TS picks the first match. Often a union or a generic is simpler.

### 25. How do you type `this`?
- A fake first parameter .. `function f(this: HTMLElement)`, erased in the output.
- Arrow functions take `this` from the enclosing scope, so they can't declare it.

### 26. `private` vs `#private`?
- `private` .. **compile-time only** .. still readable at runtime and through `obj['x']`.
- `#field` .. real JS private, **enforced at runtime**.

### 27. Abstract class vs interface?
- Abstract class .. **exists at runtime**, can hold shared implementation and a constructor. Interface .. a contract that's **erased**.
- Note for Angular .. an interface **can't be a DI token** (erased) .. use an `InjectionToken` or an abstract class.

### 28. What are parameter properties?
- `constructor(private readonly http: HttpClient)` .. declares and assigns the field in one go.
- Note that they're not allowed under `erasableSyntaxOnly` (Node's type stripping) .. because they emit code.

## Generics

### 29. What are generics, and why not just `any`?
- **Type parameters** that let one function or type work with many types **while keeping the relation** between input and output.
- `any` loses that link .. `identity<T>(x: T): T` keeps it.

### 30. What is a generic constraint?
- `T extends Something` .. limits what `T` can be, so you can use its members.
- The classic one is `get<T, K extends keyof T>(obj: T, key: K): T[K]` .. key-safe property access.

### 31. Generic defaults and `const` type parameters?
- `<T = string>` .. a default when nothing is passed or inferred.
- `<const T>` (TS 5.0) .. infers literals as if the caller wrote `as const`.

### 32. When should you **not** use a generic?
- If the type parameter appears **only once**, it adds nothing .. use the concrete type.
- A generic should connect at least two places (param ↔ return, param ↔ param).

## Type Operators

### 33. `keyof`, `typeof`, and indexed access?
- `keyof T` .. union of `T`'s keys. `typeof x` (in a type position) .. the **type of a value** .. not the JS runtime `typeof`.
- `T['prop']` .. that property's type. `T[number]` .. an array's element type.

### 34. What are conditional types?
- `T extends U ? X : Y` .. a type-level `if`.

### 35. What are distributive conditional types?
- On a **naked type parameter**, a conditional runs **once per union member** and unions the results .. that's how `Exclude` works.

### 36. What does `infer` do?
- Captures a piece of a type inside a conditional .. `T extends Promise<infer U> ? U : T`.

### 37. What are mapped types?
- Loop over keys to build a new type .. `{ [K in keyof T]: ... }`.
- Modifiers .. `+/-readonly`, `+/-?`. Key remapping .. `as`, and remapping to `never` **filters** a key out.

### 38. What are template literal types?
- String types built from other types .. `` `on${Capitalize<Event>}` `` gives `'onClick' | 'onFocus'`.
- Unions **multiply out** across every combination.

## Utility Types

### 39. The utility types you must know?

| type                        | does                             |
| --------------------------- | -------------------------------- |
| `Partial` / `Required`      | all optional / all required      |
| `Readonly`                  | all `readonly`, shallow          |
| `Pick` / `Omit`             | keep / drop some keys            |
| `Record<K, V>`              | object with keys `K`, values `V` |
| `Exclude` / `Extract`       | remove / keep union members      |
| `NonNullable`               | drop `null` / `undefined`        |
| `ReturnType` / `Parameters` | from a function type             |
| `Awaited`                   | unwrap promises, recursively     |

### 40. Implement `Partial`, `Pick` and `ReturnType` yourself.

```TS
type MyPartial<T> = { [K in keyof T]?: T[K] };
type MyPick<T, K extends keyof T> = { [P in K]: T[P] };
type MyReturnType<F> = F extends (...args: any[]) => infer R ? R : never;
```

### 41. How would you write `DeepPartial` or `DeepReadonly`?
- A mapped type that **recurses** into object properties .. `[K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K]`.
- Note that functions are objects too .. a real version special-cases them.

### 42. What's the gotcha with `Omit`?
- Its key isn't constrained to `keyof T` .. a **typo is silently accepted**.
- It's **not distributive** .. `Omit` on a union keeps only the common keys.

## Advanced

### 43. What is variance?
- **Covariant** .. follows the subtype direction (return types, arrays). **Contravariant** .. reversed (function parameters).
- Note that **method** parameters stay **bivariant** even under `strictFunctionTypes` .. only function-typed properties are checked strictly. Arrays being covariant is a known unsoundness.

### 44. Declaration merging and module augmentation?
- Same-named `interface`s **merge** into one.
- `declare module 'lib' { ... }` adds to a library's types .. `declare global { ... }` adds to the global scope (e.g. `Window`).

### 45. What are `.d.ts` files, and how do you use an untyped library?
- Types-only files describing JS that already exists .. no output.
- Order to try .. `@types/lib`, then write a `.d.ts`, then `declare module 'lib';` as a last resort (everything becomes `any`).

### 46. Decorators .. what does an Angular dev need to know?
- Functions that annotate or replace classes and members .. `@Component`, `@Injectable`.
- Note that TS 5.0 added **standard** decorators, while Angular is built on the older `experimentalDecorators` flavor .. the two have **different signatures**.

### 47. What are branded types?
- Fake **nominal typing** on top of structural .. `type UserId = string & { readonly __brand: 'UserId' }`.
- Stops passing an `OrderId` where a `UserId` is expected, even though both are strings.

### 48. How do you type mutually exclusive props?
- A union where the other side's keys are `?: never` .. `{ href: string; onClick?: never } | { onClick: () => void; href?: never }`.

## tsconfig

### 49. `target` vs `module` vs `lib`?
- `target` .. which JS **syntax** to emit. `module` .. which **module system** to emit. `lib` .. which built-in **API types** exist (`DOM`, `ES2022`).
- `moduleResolution: 'bundler'` .. matches how Vite / the Angular CLI resolve imports.

### 50. `import type`, `isolatedModules` and `verbatimModuleSyntax`?
- `import type` is **always erased** .. safe for type-only imports.
- `isolatedModules` .. every file must compile alone (how esbuild / SWC work). `verbatimModuleSyntax` .. forces you to mark type-only imports explicitly.

***

Sources used to pick the questions:
- [GreatFrontEnd .. TypeScript Interview Questions for Senior Developers](https://www.greatfrontend.com/blog/typescript-interview-questions-for-senior-frontend-developers)
- [GreatFrontEnd .. TypeScript Interview Questions for 3 Years Experience](https://www.greatfrontend.com/blog/typeScript-Interview-Questions-for-3-Years-Experience)
- [TechInterview .. Types, Generics, and Advanced Patterns](https://www.techinterview.org/post/3233460485/typescript-interview-questions/)
- [CodeForGeek .. Top 50 TypeScript Interview Questions](https://codeforgeek.com/typescript-interview-questions/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) .. for checking the answers
