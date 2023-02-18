title: Algebraic Data Types in TypeScript
date: 2023-02-18 11:18:54
tags:
  - functional programming
  - typescript
  - ADT
---

So, what's algebraic data type?
> In computer programming, especially functional programming and type theory, an algebraic data type (ADT) is a kind of composite type, i.e., a type formed by combining other types.

No worries if you have a hard time to understand the definition above, we start with storytelling.

<!-- more  -->
# A login function

One day, you, as a programmer(more specifically, a TypeScript developer), got a task from your manager, you need to build a function(or API in another word) that allows user to login to the system with username and password. "That's just a piece of cake.", you thought and built the login function like this,

```typescript
type Credentials = {
  username: string;
  password: string;
}
function login(c: Credentials) {
  return doLogin(c.username, c.password);
}

```

The next day, your manager came back to you, "well, we need to support login with email together with password as well". Easy, easy, you eneded up with the following code with a little extension.

```typescript
type Credentials = {
  username?: string;
  password: string;
  email?: string;
}
function login(c: Credentials) {
  // validation
  if (c.username && c.email
    || !c.username && !c.email) {
    throw new Error('validation error');
  }
  if (c.username) {
    return loginByName(c.username, c.password);
  }
  if (credential.email) {
    return loginByEmail(c.email, c.password);
  }
}

```
This time, you marked the username and email as optional fields, carefully added the validation, and splitted the login process into two small functions, everything seems to be perfect!

As you know, change is the only constant, a few days later, your manager told you, "Our system needs to support login with one-time-passcode(OTP) as it mitigates the weak password problem.".

Okay, adding a `otp` to `Crendential` is trivial but now every field is optional, you probabbly also consider to add a Enum type to the Credentials so you can use switch-case to reduce the `if-else` clausese. 
```typescript
enum LoginType {
  USERNAME = 'username',
  EMAIL = 'email',
  OTP = 'otp',
}

type Credentials = {
  type: LoginType; 
  username?: string;
  password?: string;
  email?: string;
  otp? : string;
}
function login(c: Credentials) {
  // TODO validation, how?
  switch (c.type) {
      case LoginType.USERNAME:
        return loginByName(c.username, c.password);
      case LoginType.EMAIL:
        return loginByEmail(c.email, c.password);
      case LoginType.OTP:
        return loginByOTP(c.otp);
  }
}

```

No bad, but how to do the validation, there are so many possibilities as most of fields are optional, you definitely don't allow the api to be feed with both `otp` and `username` at the same time for example. 
Probably you have realized that there is something wrong, you're right, actually the way that expands the `Credentials` is so called building a **product type**.

# Product 

As aforementioned, a product is a composition of types, it's normally annotated as `*`(*asterisk*), so `C = A * B` means C is the product of A and B. Let me give an example, let's say A is type Bool, B is a type with 3 elements, [1,2,3], you got C as a set of tuples.


| A/B     |           1 |           2 |            3|
|---------|-------------|-------------|-------------|
| True    | (True, 1)   | (True, 2)   | (True, 3)   |
| False   | (False, 1)  | (False, 2)  | (False, 3)  |

Notice there is an important property, 
> the cardinality of the product is the product of cardinalities.
```
Cardi(A * B)  = Cardi(A) * Cardi(B)

```
Given our login case above and simplify the cardinalities of optional fields to 2: exist(X)/non-exist(-).

| Type     |    username |    password |        email|   otp |
|----------|-------------|-------------|-------------|-------|
| EMAIL    | -           |  X          | X           |     - | 
| USERNAME | X           |  X          | -           |     - |
| OTP      | -           |  -          | -           |     X |

You can see from all the combinations we only want to handle those cells with `X`, this reminder me another concept: **total function**. A total function is defined for all inputs of the right type, i.e. for all of a domain. E.g.  `(x: Int) => { return x + 1 }` is a total function as it accepts all integers, but `(x: Int) => { return 1 / x } ` is not as it fails to map to a result with zero on the set of real numbers.

We now see as we used the wrong type(Product) for the argument of login function, it's disqualified as a total function, which leads to tricky validation handlings.
  
# Sum 

A better way to define the argument type of login is using Sum type. Sum is the dual of Product, it's used to hold a value that could take on several different, but fixed types. In TypeScript it's named tagged union types.
Accordingly there is also a quality property for Sum type,
> the cardinality of the sum is the sum of cardinalities

```
Cardi(A + B) = Cardi(A) + Cardi(B)

```

# Improvement with Sum type
Let's rewrite the Credentials type with Sum type as follows,

```TypeScript
type Email = {
  type: LoginType.EMAIL;
  email: string;
  password: string;
}
type UserName = {
  type: LoginType.USERNAME;
  username: string;
  password: string;
}
type OTP = {
  type: LoginType.OTP;
  otp: string;
}
type Credentials = Email | UserName | OTP
function login(c: Credentials) {
  switch (c.type) {
      case LoginType.USERNAME:
        return loginByName(c.username, c.password);
      case LoginType.EMAIL:
        return loginByEmail(c.email, c.password);
      case LoginType.OTP:
        return loginByOTP(c.otp);
  }
}

```

You see now you don't need extra validation as type checking prevents any malformed data as input.
if you call `login` with `{ type: LoginType.EMAIL, otp: 'xxx', email: 'foo@bar.com', password: 'xxx'}`, you will get a compiler error like this:

> error TS2345: Argument of type '{ type: LoginType.EMAIL; otp: string; email: string; password: string; }' is not assignable to parameter of type 'Credentials'.
>  Object literal may only specify known properties, and 'otp' does not exist in type 'Email'.

# When should I use which

Both Sum and Product are the composition of types. If you see the types as the components are independent, then Product is a fit. e.g. `type Calender = [Year, Month, Date]`, if you see the components are dependent when implementing as a Product, Sum will be a better choice. 

# Further improvement with functional style

One thing I'd like to improve for the `login` function is that there is switch-case inside, it's usually used for control flow distribution but there is not a really one, it was misused to map the type to the handlers.
It can be revamped with a data driven pattern.

```typescript
function loginByEmail(c: Email) {
  // ...
}
function loginByName(c: UserName) {
  // ...
}
function loginByOTP(c: OTP) {
  // ...
}
const handlers = {
  [LoginType.EMAIL]: loginByEmail,
  [LoginType.USERNAME]: loginByName ,
  [LoginType.OTP]: loginByOTP,
}
function login(c: Credentials) {
  return handlers[c.type](c); // compile error
}
```
Pretty concise and easy to extend with other handlers but there is a compiler error, oops,

> Argument of type 'Credentials' is not assignable to parameter of type 'never'.
>  The intersection 'Email & UserName & OTP' was reduced to 'never' because property 'type' has conflicting types in some constituents.
>    Type 'Email' is not assignable to type 'never'.
  
You might be curious why there is an intersection, this is because TypeScript type inference uses intersection instead of union when inferring an argument for serveral functions. see details [here](https://stackoverflow.com/a/54936519/4550665).

A quick fix is just adding explicit type for the functions,
```
const handlers = {
  [LoginType.EMAIL]: loginByEmail as (c: Credentials) => string,
  [LoginType.USERNAME]: loginByName as (c: Credentials) => string,
  [LoginType.OTP]: loginByOTP as (c: Credentials) => string,
}
```
  
# Conclusion

The Product type and Sum type are two common classes of algebraic types, understanding them and select the correct one would greatly affect the design of system modeling.


