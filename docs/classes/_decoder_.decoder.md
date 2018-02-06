[json-type-validation](../README.md) > ["decoder"](../modules/_decoder_.md) > [Decoder](../classes/_decoder_.decoder.md)



# Class: Decoder


Decoders transform json objects with unknown structure into known and verified forms. You can create objects of type `Decoder<A>` with either the primitive decoder functions, such as `boolean()` and `string()`, or by applying higher-order decoders to the primitives, such as `array(boolean())` or `dict(string())`.

Each of the decoder functions are available both as a static method on `Decoder` and as a function alias -- for example the string decoder is defined at `Decoder.string()`, but is also aliased to `string()`. Using the function aliases exported with the library is recommended.

`Decoder` exposes a number of 'run' methods, which all decode json in the same way, but communicate success and failure in different ways. The `map` and `andThen` methods modify decoders without having to call a 'run' method.

Alternatively, the main decoder `run()` method returns an object of type `Result<A, DecoderError>`. This library provides a number of helper functions for dealing with the `Result` type, so you can do all the same things with a `Result` as with the decoder methods.

## Type parameters
#### A 
## Index

### Methods

* [andThen](_decoder_.decoder.md#andthen)
* [map](_decoder_.decoder.md#map)
* [run](_decoder_.decoder.md#run)
* [runPromise](_decoder_.decoder.md#runpromise)
* [runWithException](_decoder_.decoder.md#runwithexception)
* [anyJson](_decoder_.decoder.md#anyjson)
* [array](_decoder_.decoder.md#array)
* [boolean](_decoder_.decoder.md#boolean)
* [constant](_decoder_.decoder.md#constant)
* [constantFalse](_decoder_.decoder.md#constantfalse)
* [constantNull](_decoder_.decoder.md#constantnull)
* [constantTrue](_decoder_.decoder.md#constanttrue)
* [dict](_decoder_.decoder.md#dict)
* [fail](_decoder_.decoder.md#fail)
* [lazy](_decoder_.decoder.md#lazy)
* [number](_decoder_.decoder.md#number)
* [object](_decoder_.decoder.md#object)
* [oneOf](_decoder_.decoder.md#oneof)
* [optional](_decoder_.decoder.md#optional)
* [string](_decoder_.decoder.md#string)
* [succeed](_decoder_.decoder.md#succeed)
* [union](_decoder_.decoder.md#union)
* [valueAt](_decoder_.decoder.md#valueat)
* [withDefault](_decoder_.decoder.md#withdefault)



---


## Methods
<a id="andthen"></a>

###  andThen

► **andThen**B(f: *`function`*): [Decoder](_decoder_.decoder.md)`B`






Chain together a sequence of decoders. The first decoder will run, and then the function will determine what decoder to run second. If the result of the first decoder succeeds then `f` will be applied to the decoded value. If it fails the error will propagate through. One use case for `andThen` is returning a custom error message.

Example:

    const versionDecoder = valueAt(['version'], number());
    const infoDecoder3 = object({a: boolean()});

    const decoder = versionDecoder.andThen(version => {
      switch (version) {
        case 3:
          return infoDecoder3;
        default:
          return fail(`Unable to decode info, version ${version} is not supported.`);
      }
    });

    decoder.run({version: 3, a: true})
    // => {ok: true, result: {a: true}}

    decoder.run({version: 5, x: 'abc'})
    // =>
    // {
    //   ok: false,
    //   error: {
    //     ...
    //     at: 'input',
    //     message: 'Unable to decode info, version 5 is not supported.'
    //   }
    // }


**Type parameters:**

#### B 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| f | `function`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`B`





___

<a id="map"></a>

###  map

► **map**B(f: *`function`*): [Decoder](_decoder_.decoder.md)`B`






Construct a new decoder that applies a transformation to the decoded result. If the decoder succeeds then `f` will be applied to the value. If it fails the error will propagated through.

Example:

    number().map(x => x * 5).run(10)
    // => {ok: true, result: 50}


**Type parameters:**

#### B 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| f | `function`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`B`





___

<a id="run"></a>

###  run

► **run**(json: *`any`*): `Result.Result`.<`A`>,.<[DecoderError](../interfaces/_decoder_.decodererror.md)>






Run the decoder and return a `Result` with either the decoded value or a `DecoderError` containing the json input, the location of the error, and the error message.

Examples:

    number().run(12)
    // => {ok: true, result: 12}

    string().run(9001)
    // =>
    // {
    //   ok: false,
    //   error: {
    //     kind: 'DecoderError',
    //     input: 9001,
    //     at: 'input',
    //     message: 'expected a string, got 9001'
    //   }
    // }


**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| json | `any`   |  - |





**Returns:** `Result.Result`.<`A`>,.<[DecoderError](../interfaces/_decoder_.decodererror.md)>





___

<a id="runpromise"></a>

###  runPromise

► **runPromise**(json: *`any`*): `Promise`.<`A`>






Run the decoder as a `Promise`.


**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| json | `any`   |  - |





**Returns:** `Promise`.<`A`>





___

<a id="runwithexception"></a>

###  runWithException

► **runWithException**(json: *`any`*): `A`






Run the decoder and return the value on success, or throw an exception with a formatted error string.


**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| json | `any`   |  - |





**Returns:** `A`





___

<a id="anyjson"></a>

### «Static» anyJson

► **anyJson**(): [Decoder](_decoder_.decoder.md)`any`






Decoder identity function. Useful for incremental decoding.

Example:

    const json: any = [1, true, 2, 3, 'five', 4, []];
    const jsonArray: any[] = Result.withDefault([], array(anyJson()).run(json));
    const numbers: number[] = Result.successes(jsonArray.map(number().run));

    numbers
    // => [1, 2, 3, 4]




**Returns:** [Decoder](_decoder_.decoder.md)`any`





___

<a id="array"></a>

### «Static» array

► **array**A(decoder: *[Decoder](_decoder_.decoder.md)`A`*): [Decoder](_decoder_.decoder.md)`A`[]






Decoder for json arrays. Runs `decoder` on each array element, and succeeds if all elements are successfully decoded.

To decode a single value that is inside of an array see `valueAt`.

Examples:

    array(number()).run([1, 2, 3])
    // => {ok: true, result: [1, 2, 3]}

    array(array(boolean())).run([[true], [], [true, false, false]])
    // => {ok: true, result: [[true], [], [true, false, false]]}


**Type parameters:**

#### A 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| decoder | [Decoder](_decoder_.decoder.md)`A`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`[]





___

<a id="boolean"></a>

### «Static» boolean

► **boolean**(): [Decoder](_decoder_.decoder.md)`boolean`






Decoder primitive that validates booleans, and fails on all other input.




**Returns:** [Decoder](_decoder_.decoder.md)`boolean`





___

<a id="constant"></a>

### «Static» constant

► **constant**A(value: *`A`*): [Decoder](_decoder_.decoder.md)`A`






Decoder primitive that only matches on exact values.

Note that `constant('string to match')` returns a `Decoder<string>` which fails if the input is not equal to `'string to match'`. In many cases this is sufficient, but in some situations typescript requires that the decoder type be a type-literal. In such a case you must provide the type parameter, which looks like `constant<'string to match'>('string to match')`.

One place where this happens is when a type-literal is in an interface:

    interface Bear {
      kind: 'bear';
      isBig: boolean;
    }

    const bearDecoder1: Decoder<Bear> = object({
      kind: constant('bear'),
      isBig: boolean()
    });
    // Type 'Decoder<{ kind: string; isBig: boolean; }>' is not assignable to
    // type 'Decoder<Bear>'. Type 'string' is not assignable to type '"bear"'.

    const bearDecoder2: Decoder<Bear> = object({
      kind: constant<'bear'>('bear'),
      isBig: boolean()
    });
    // no compiler errors

Another is in type-literal unions:

    type animal = 'bird' | 'bear';

    const animalDecoder1: Decoder<animal> = union(
      constant('bird'),
      constant('bear')
    );
    // Type 'Decoder<string>' is not assignable to type 'Decoder<animal>'.
    // Type 'string' is not assignable to type 'animal'.

    const animalDecoder2: Decoder<animal> = union(
      constant<'bird'>('bird'),
      constant<'bear'>('bear')
    );
    // no compiler errors


**Type parameters:**

#### A 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| value | `A`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`





___

<a id="constantfalse"></a>

### «Static» constantFalse

► **constantFalse**(): [Decoder](_decoder_.decoder.md)`false`






Decoder primitive that only matches the value `false`, and returns the decoder `Decoder<false>` instead of `Decoder<boolean>`. It is a more ergonomic alias for `constant<false>(false)`.




**Returns:** [Decoder](_decoder_.decoder.md)`false`





___

<a id="constantnull"></a>

### «Static» constantNull

► **constantNull**(): [Decoder](_decoder_.decoder.md)`null`






Decoder primitive that only matches the value `null`. It is equivalent to `constant(null)`, but is provided for completeness.




**Returns:** [Decoder](_decoder_.decoder.md)`null`





___

<a id="constanttrue"></a>

### «Static» constantTrue

► **constantTrue**(): [Decoder](_decoder_.decoder.md)`true`






Decoder primitive that only matches the value `true`, and returns the decoder `Decoder<true>` instead of `Decoder<boolean>`. It is a more ergonomic alias for `constant<true>(true)`.




**Returns:** [Decoder](_decoder_.decoder.md)`true`





___

<a id="dict"></a>

### «Static» dict

► **dict**A(decoder: *[Decoder](_decoder_.decoder.md)`A`*): [Decoder](_decoder_.decoder.md)`object`






Decoder for json objects where the keys are unknown strings, but the values should all be of the same type.

Example:

    dict(number()).run({chocolate: 12, vanilla: 10, mint: 37});
    // => {ok: true, result: {chocolate: 12, vanilla: 10, mint: 37}}


**Type parameters:**

#### A 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| decoder | [Decoder](_decoder_.decoder.md)`A`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`object`





___

<a id="fail"></a>

### «Static» fail

► **fail**A(errorMessage: *`string`*): [Decoder](_decoder_.decoder.md)`A`






Decoder that ignores the input json and always fails with `errorMessage`.


**Type parameters:**

#### A 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| errorMessage | `string`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`





___

<a id="lazy"></a>

### «Static» lazy

► **lazy**A(mkDecoder: *`function`*): [Decoder](_decoder_.decoder.md)`A`






Decoder that allows for validating recursive data structures. Unlike with functions, decoders assigned to variables can't reference themselves before they are fully defined. We can avoid prematurely referencing the decoder by wrapping it in a function that won't be called until use, at which point the decoder has been defined.

Example:

    interface Comment {
      msg: string;
      replies: Comment[];
    }

    const decoder: Decoder<Comment> = object({
      msg: string(),
      replies: lazy(() => array(decoder))
    });


**Type parameters:**

#### A 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| mkDecoder | `function`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`





___

<a id="number"></a>

### «Static» number

► **number**(): [Decoder](_decoder_.decoder.md)`number`






Decoder primitive that validates numbers, and fails on all other input.




**Returns:** [Decoder](_decoder_.decoder.md)`number`





___

<a id="object"></a>

### «Static» object

► **object**A(decoders: *[DecoderObject](../modules/_decoder_.md#decoderobject)`A`*): [Decoder](_decoder_.decoder.md)`A`






An higher-order decoder that runs decoders on specified fields of an object, and returns a new object with those fields.

The `optional` and `constant` decoders are particularly useful for decoding objects that match typescript interfaces.

To decode a single field that is inside of an object see `valueAt`.

Example:

    object({x: number(), y: number()}).run({x: 5, y: 10})
    // => {ok: true, result: {x: 5, y: 10}}


**Type parameters:**

#### A 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| decoders | [DecoderObject](../modules/_decoder_.md#decoderobject)`A`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`





___

<a id="oneof"></a>

### «Static» oneOf

► **oneOf**A(...decoders: *[Decoder](_decoder_.decoder.md)`A`[]*): [Decoder](_decoder_.decoder.md)`A`






Decoder that attempts to run each decoder in `decoders` and either succeeds with the first successful decoder, or fails after all decoders have failed.

Note that `oneOf` expects the decoders to all have the same return type, while `union` creates a decoder for the union type of all the input decoders.

Examples:

    oneOf(string(), number().map(String))
    oneOf(constant('start'), constant('stop'), succeed('unknown'))


**Type parameters:**

#### A 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| decoders | [Decoder](_decoder_.decoder.md)`A`[]   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`





___

<a id="optional"></a>

### «Static» optional

► **optional**A(decoder: *[Decoder](_decoder_.decoder.md)`A`*): [Decoder](_decoder_.decoder.md)`undefined`⎮`A`






Decoder for values that may be `undefined`. This is primarily helpful for decoding interfaces with optional fields.

Example:

    interface User {
      id: number;
      isOwner?: boolean;
    }

    const decoder: Decoder<User> = object({
      id: number(),
      isOwner: optional(boolean())
    });


**Type parameters:**

#### A 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| decoder | [Decoder](_decoder_.decoder.md)`A`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`undefined`⎮`A`





___

<a id="string"></a>

### «Static» string

► **string**(): [Decoder](_decoder_.decoder.md)`string`






Decoder primitive that validates strings, and fails on all other input.




**Returns:** [Decoder](_decoder_.decoder.md)`string`





___

<a id="succeed"></a>

### «Static» succeed

► **succeed**A(fixedValue: *`A`*): [Decoder](_decoder_.decoder.md)`A`






Decoder that ignores the input json and always succeeds with `fixedValue`.


**Type parameters:**

#### A 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| fixedValue | `A`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`





___

<a id="union"></a>

### «Static» union

► **union**A,B(ad: *[Decoder](_decoder_.decoder.md)`A`*, bd: *[Decoder](_decoder_.decoder.md)`B`*): [Decoder](_decoder_.decoder.md)`A`⎮`B`

► **union**A,B,C(ad: *[Decoder](_decoder_.decoder.md)`A`*, bd: *[Decoder](_decoder_.decoder.md)`B`*, cd: *[Decoder](_decoder_.decoder.md)`C`*): [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`

► **union**A,B,C,D(ad: *[Decoder](_decoder_.decoder.md)`A`*, bd: *[Decoder](_decoder_.decoder.md)`B`*, cd: *[Decoder](_decoder_.decoder.md)`C`*, dd: *[Decoder](_decoder_.decoder.md)`D`*): [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`⎮`D`

► **union**A,B,C,D,E(ad: *[Decoder](_decoder_.decoder.md)`A`*, bd: *[Decoder](_decoder_.decoder.md)`B`*, cd: *[Decoder](_decoder_.decoder.md)`C`*, dd: *[Decoder](_decoder_.decoder.md)`D`*, ed: *[Decoder](_decoder_.decoder.md)`E`*): [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`⎮`D`⎮`E`

► **union**A,B,C,D,E,F(ad: *[Decoder](_decoder_.decoder.md)`A`*, bd: *[Decoder](_decoder_.decoder.md)`B`*, cd: *[Decoder](_decoder_.decoder.md)`C`*, dd: *[Decoder](_decoder_.decoder.md)`D`*, ed: *[Decoder](_decoder_.decoder.md)`E`*, fd: *[Decoder](_decoder_.decoder.md)`F`*): [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`⎮`D`⎮`E`⎮`F`

► **union**A,B,C,D,E,F,G(ad: *[Decoder](_decoder_.decoder.md)`A`*, bd: *[Decoder](_decoder_.decoder.md)`B`*, cd: *[Decoder](_decoder_.decoder.md)`C`*, dd: *[Decoder](_decoder_.decoder.md)`D`*, ed: *[Decoder](_decoder_.decoder.md)`E`*, fd: *[Decoder](_decoder_.decoder.md)`F`*, gd: *[Decoder](_decoder_.decoder.md)`G`*): [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`⎮`D`⎮`E`⎮`F`⎮`G`

► **union**A,B,C,D,E,F,G,H(ad: *[Decoder](_decoder_.decoder.md)`A`*, bd: *[Decoder](_decoder_.decoder.md)`B`*, cd: *[Decoder](_decoder_.decoder.md)`C`*, dd: *[Decoder](_decoder_.decoder.md)`D`*, ed: *[Decoder](_decoder_.decoder.md)`E`*, fd: *[Decoder](_decoder_.decoder.md)`F`*, gd: *[Decoder](_decoder_.decoder.md)`G`*, hd: *[Decoder](_decoder_.decoder.md)`H`*): [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`⎮`D`⎮`E`⎮`F`⎮`G`⎮`H`






Combines 2-8 decoders of disparate types into a decoder for the union of all the types.

If you need more than 8 variants for your union, it's possible to use `oneOf` in place of `union` as long as you annotate every decoder with the union type.

Example:

    type C = {a: string} | {b: number};

    const unionDecoder: Decoder<C> = union(object({a: string()}), object({b: number()}));
    const oneOfDecoder: Decoder<C> = oneOf(object<C>({a: string()}), object<C>({b: number()}));


**Type parameters:**

#### A 
#### B 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| ad | [Decoder](_decoder_.decoder.md)`A`   |  - |
| bd | [Decoder](_decoder_.decoder.md)`B`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`⎮`B`






**Type parameters:**

#### A 
#### B 
#### C 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| ad | [Decoder](_decoder_.decoder.md)`A`   |  - |
| bd | [Decoder](_decoder_.decoder.md)`B`   |  - |
| cd | [Decoder](_decoder_.decoder.md)`C`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`






**Type parameters:**

#### A 
#### B 
#### C 
#### D 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| ad | [Decoder](_decoder_.decoder.md)`A`   |  - |
| bd | [Decoder](_decoder_.decoder.md)`B`   |  - |
| cd | [Decoder](_decoder_.decoder.md)`C`   |  - |
| dd | [Decoder](_decoder_.decoder.md)`D`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`⎮`D`






**Type parameters:**

#### A 
#### B 
#### C 
#### D 
#### E 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| ad | [Decoder](_decoder_.decoder.md)`A`   |  - |
| bd | [Decoder](_decoder_.decoder.md)`B`   |  - |
| cd | [Decoder](_decoder_.decoder.md)`C`   |  - |
| dd | [Decoder](_decoder_.decoder.md)`D`   |  - |
| ed | [Decoder](_decoder_.decoder.md)`E`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`⎮`D`⎮`E`






**Type parameters:**

#### A 
#### B 
#### C 
#### D 
#### E 
#### F 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| ad | [Decoder](_decoder_.decoder.md)`A`   |  - |
| bd | [Decoder](_decoder_.decoder.md)`B`   |  - |
| cd | [Decoder](_decoder_.decoder.md)`C`   |  - |
| dd | [Decoder](_decoder_.decoder.md)`D`   |  - |
| ed | [Decoder](_decoder_.decoder.md)`E`   |  - |
| fd | [Decoder](_decoder_.decoder.md)`F`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`⎮`D`⎮`E`⎮`F`






**Type parameters:**

#### A 
#### B 
#### C 
#### D 
#### E 
#### F 
#### G 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| ad | [Decoder](_decoder_.decoder.md)`A`   |  - |
| bd | [Decoder](_decoder_.decoder.md)`B`   |  - |
| cd | [Decoder](_decoder_.decoder.md)`C`   |  - |
| dd | [Decoder](_decoder_.decoder.md)`D`   |  - |
| ed | [Decoder](_decoder_.decoder.md)`E`   |  - |
| fd | [Decoder](_decoder_.decoder.md)`F`   |  - |
| gd | [Decoder](_decoder_.decoder.md)`G`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`⎮`D`⎮`E`⎮`F`⎮`G`






**Type parameters:**

#### A 
#### B 
#### C 
#### D 
#### E 
#### F 
#### G 
#### H 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| ad | [Decoder](_decoder_.decoder.md)`A`   |  - |
| bd | [Decoder](_decoder_.decoder.md)`B`   |  - |
| cd | [Decoder](_decoder_.decoder.md)`C`   |  - |
| dd | [Decoder](_decoder_.decoder.md)`D`   |  - |
| ed | [Decoder](_decoder_.decoder.md)`E`   |  - |
| fd | [Decoder](_decoder_.decoder.md)`F`   |  - |
| gd | [Decoder](_decoder_.decoder.md)`G`   |  - |
| hd | [Decoder](_decoder_.decoder.md)`H`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`⎮`B`⎮`C`⎮`D`⎮`E`⎮`F`⎮`G`⎮`H`





___

<a id="valueat"></a>

### «Static» valueAt

► **valueAt**A(paths: *(`string`⎮`number`)[]*, decoder: *[Decoder](_decoder_.decoder.md)`A`*): [Decoder](_decoder_.decoder.md)`A`






Decoder that pulls a specific field out of a json structure, instead of decoding and returning the full structure. The `paths` array describes the object keys and array indices to traverse, so that values can be pulled out of a nested structure.

Example:

    const decoder = valueAt(['a', 'b', 0], string());

    decoder.run({a: {b: ['surprise!']}})
    // => {ok: true, result: 'surprise!'}

    decoder.run({a: {x: 'cats'}})
    // => {ok: false, error: {... at: 'input.a.b[0]' message: 'path does not exist'}}

Note that the `decoder` is ran on the value found at the last key in the path, even if the last key is not found. This allows the `optional` decoder to succeed when appropriate.

    const optionalDecoder = valueAt(['a', 'b', 'c'], optional(string()));

    optionalDecoder.run({a: {b: {c: 'surprise!'}}})
    // => {ok: true, result: 'surprise!'}

    optionalDecoder.run({a: {b: 'cats'}})
    // => {ok: false, error: {... at: 'input.a.b.c' message: 'expected an object, got "cats"'}

    optionalDecoder.run({a: {b: {z: 1}}})
    // => {ok: true, result: undefined}


**Type parameters:**

#### A 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| paths | (`string`⎮`number`)[]   |  - |
| decoder | [Decoder](_decoder_.decoder.md)`A`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`





___

<a id="withdefault"></a>

### «Static» withDefault

► **withDefault**A(defaultValue: *`A`*, decoder: *[Decoder](_decoder_.decoder.md)`A`*): [Decoder](_decoder_.decoder.md)`A`






Decoder that always succeeds with either the decoded value, or a fallback default value.


**Type parameters:**

#### A 
**Parameters:**

| Param | Type | Description |
| ------ | ------ | ------ |
| defaultValue | `A`   |  - |
| decoder | [Decoder](_decoder_.decoder.md)`A`   |  - |





**Returns:** [Decoder](_decoder_.decoder.md)`A`





___

