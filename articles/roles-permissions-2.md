## Building a Roles & Permissions System: Part 2

[Part 1](./permission-system-1.md) showed how to implement a `moderator` role with hardcoded permissions. In this post we will implement a more scalable system where roles & permissions are properly decoupled.

Fun fact: This will only increase the number of logical lines from 5 to 14 :)

## The Scenario

In [part 1](./permission-system-1.md) we defined `user` and `club` predicates. This isn't actually necessary for our purposes; as long as the ids (e.g. `alice` and `chess`) match across predicates, we don't need to define `user(alice)` and `club(chess)`.

With that in mind, let's build our program from scratch again, and start with defining club memberships:

```pl
member(alice, boxing).
member(bob,   boxing).
member(carly, boxing).
member(dan,   boxing).

member(alice, chess).
member(bob,   chess).
```

Next, let's define some roles. Last time we defined a `moderator` predicate. This time let's make things more flexible by defining a more general `role` predicate instead:

```pl
role(alice, boxing, admin).
role(bob,   boxing, moderator).
role(bob,   chess,  moderator).
```

For every moderator permission, we want admins to **also** have that permission. Let's define this relationship using another predicate:

```pl
role_inherits(admin, moderator).
```

Note that this doesn't actually define any inheritance *logic*; it only defines a **relationship** that we will take advantage of later.

Lastly, let's assign specific permissions to specific roles:

```pl
permission(admin,     promote_to_mod).
permission(moderator, ban_user).
permission(moderator, ban_protection).
```

**Note 1:** Notice how we *don't* say that admins can ban a user. We are assuming that admins can do everything moderators can do – even though that isn't true yet! The logic for that will come later.

**Note 2:** Notice how these are all facts. Any facts that you define in Prolog can easily be stored and loaded by an external database.

### Query Examples

With our fresh new list of facts, let's run some queries to get a feel for what's possible.

First: What roles does `bob` have, and in which clubs?

```prolog
?- role(bob, C, R).
C = boxing,
R = moderator ;
C = chess,
R = moderator.
```

What permitted actions does `bob` have in the boxing club?

```prolog
?- role(bob, boxing, R), permission(R, A).
R = moderator,
A = ban_user ;
R = moderator,
A = ban_protection ;
false.
```

What permitted actions does `alice` have in the boxing club?

```prolog
?- role(alice, boxing, R), permission(R, A).
R = admin,
A = promote_to_mod.
```

**Uh oh.** Notice how `alice` does not have moderator permissions. She only has admin permissions!

Why is this? Let's look at how the `permission` predicate behaves:

```prolog
?- permission(moderator, A).
A = ban_user ;
A = ban_protection.

?- permission(admin, A).
A = promote_to_mod.
```

Aha, the `permission` predicate is only giving us **direct** relationships! This is actually a good thing. However, logically we want `admin` to inherit *all* permissions from `moderator`, so let's do that next.

## Role Inheritance

This problem brings us to our first two lines of logic: a predicate `role_has_permission` that handles permissions **while also considering role inheritance:**

```pl
role_has_permission(Role, Action) :- permission(Role, Action).
role_has_permission(Role, Action) :-
  role_inherits(Role, Child),
  role_has_permission(Child, Action).
```

This is a recursive predicate, and it also happens to be a common Prolog pattern. Here's the explanation:

- [line 1] A `Role` has permission to do an `Action` if it is specified by the `permission` predicate.
- [line 2] Either that, or a `Role` has permission to do an `Action` if:
  - [line 3] The `Role` inherits from some `Child` role,
  - [line 4] where that specific `Child` role has permission to do the `Action`.

Now we can correctly get all the permitted actions for a given role, and also query them regarding Alice and the boxing club:

```prolog
?- role_has_permission(moderator, A).
A = ban_user ;
A = ban_protection ;
false.

?- role_has_permission(admin, A).
A = promote_to_mod ;
A = ban_user ;
A = ban_protection ;
false.

?- role(alice, boxing, R), role_has_permission(R, A).
R = admin,
A = promote_to_mod ;
R = admin,
A = ban_user ;
R = admin,
A = ban_protection ;
false.
```

Perfect! Now we can correctly ask if a user has a specific permission in a specific club. For example, can Alice and Bob promote other users to moderators?

```prolog
?- role(alice, boxing, R), role_has_permission(R, promote_to_mod).
R = admin ;
false.

?- role(bob, boxing, R), role_has_permission(R, promote_to_mod).
false.
```

The first query says "yes, alice *can* promote_to_mod because she has the admin role". The second query says "no, bob cannot" because bob has no role in the boxing club that also has the `promote_to_mod` permission.

### A Quick Abstraction

In Prolog, it's ideal to encode our real-world questions as predicates. The previous query does not conform to this ideal, so let's write a new predicate to keep our code clean:

```pl
user_has_permission(User, Club, Action) :-
  role(User, Club, Role),
  role_has_permission(Role, Action).
```

Note that this is the same logic as the query we just ran. Here's the explanation:

A `User` has permission to do an `Action` in a `Club` if:

- [line 2] The `User` has some `Role` in `Club`,
- [line 3] such that `Role` has permission to do `Action`.

With these three lines of code, we can now get a yes/no answer to the last question we asked in the previous section:

```prolog
% Instead of:
%   role(alice, boxing, R), role_has_permission(R, promote_to_mod).
% We can now write:
?- user_has_permission(alice, boxing, promote_to_mod).
true ;
false.

% Instead of:
%   role(bob, boxing, R), role_has_permission(R, promote_to_mod).
% We can now write:
?- user_has_permission(bob, boxing, promote_to_mod).
false.
```

Much nicer! We will also reuse this predicate shortly.

## Adding Ad-hoc Permission Logic

Ok, now let's update `ban_user` logic [from part 1](./permission-system-1.md#the-logic) to use our new, more scalable system, and achieve the advertised 14 lines of logic. After that, we will also write a new `promote_to_mod` action for good measure.

First, the new `ban_user`:

```pl
ban_user(Actor, Club, Target) :-
  user_has_permission(Actor, Club, ban_user),
  dif(Actor, Target),
  member(Target, Club),
  \+ user_has_permission(Target, Club, ban_protection).
```

So easy! If you understood part 1, then no further explanation is needed.

Now let's do `promote_to_mod`:

```pl
promote_to_mod(Actor, Club, Target) :-
  user_has_permission(Actor, Club, promote_to_mod),
  member(Target, Club),
  \+ role(Target, Club, _).
```

The only new-ish part is the last line. Logically speaking, it only passes  when `Actor` **does not have a special role**. Programmatically speaking, it gets interesting.

Because of the `\+` operator (remember it means "not"), Prolog first tries to **prove** `role(Target, Club, _)`. If Prolog can prove it, then `\+` flips it to false. If Prolog *can't* prove it, then `\+` will flip it to true, causing `promote_to_mod` to be true as a whole.

In other words, `promote_to_mod` will only pass if `role(Target, Club, _)` is not true. The underscore `_` in that code means "something, anything, I don't care what it is, as long as something is there". So again, the code is effectively saying "fail if Target has any special role at all in Club", which is exactly what we want.

**Note:** Negation is a Prolog fundamental. You have to be careful with what you negate. For example, if you write `\+ something(X)`, and `something(X)` takes a very long time to prove, then your code will be quite inefficient! Don't worry too much though; just like all languages, there are tricks you can do to get around such problems.

## DRYing things up

There's a small amount of [WETness](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself#DRY_vs_WET_solutions) in our action code: each action verifies if the actor has permission to do that action! This is quite redundant, as this behavior is obviously implied in any action we write.

To remove this redundancy, we will use the built-in [call](https://www.swi-prolog.org/pldoc/doc_for?object=call/2) predicate. Here's an example of using it. The following two queries are equivalent:

```prolog
?- member(alice, C).
C = boxing ;
C = chess.

?- call(member, alice, C).
C = boxing ;
C = chess.
```

(For functional programmers, this is similar to a higher-order function. For JavaScripters, this is like Function.prototype.call. For lispers, there's even more to get excited about in Prolog 😉).

`call` is a very useful tool to learn (and use sparingly). It allows us to use an atom as both a **value** and a **predicate**. Behold:

```pl
can(Actor, Club, Action, Target) :-
  user_has_permission(Actor, Club, Action),
  call(Action, Actor, Club, Target).

%
% New action code!
% Notice how we removed the first line of each predicate.
%
ban_user(Actor, Club, Target) :-
  dif(Actor, Target),
  member(Target, Club),
  \+ user_has_permission(Target, Club, ban_protection).

promote_to_mod(_Actor, Club, Target) :-
  member(Target, Club),
  \+ role(Target, Club, _).
```

and its usage:

```prolog
?- can(alice, boxing, ban_user, carly).
true ;
false.

?- can(alice, boxing, ban_user, bob).
false.
```

Wonderful! But how does it work? As long as you understand how `call` works, then the code is straightforward. The key here is realizing `Action` is used as both a value (in `user_has_permission()`) and a predicate (in `call()`).

**Heavy note:** `call` will run literally anything, and SWI Prolog has the capability to do sensitive things like read and write files. If you're putting this code on the web, BE SURE TO SANITIZE YOUR INPUTS!

## Conclusion

And there you have it! A beautiful and scalable roles & permissions system in less than 20 lines of Prolog.

In those few lines of code, we were able to:

- Decoupled roles and permissions
- Implement role inheritance
- Cleanly encoded real-world questions into individual predicates
- Remove redundancy using the higher-order predicate `call`
- Build it in such a way that is super easy to extend!

The next post in this series will extend our system with **error messages**, allowing the system to know **why** a user's action was rejected.

```prolog
%
% List of facts (also known as "the database")
%
member(alice, boxing).
member(bob,   boxing).
member(carly, boxing).
member(dan,   boxing).

member(alice, chess).
member(bob,   chess).

role(alice, boxing, admin).
role(bob,   boxing, moderator).
role(bob,   chess,  moderator).

role_inherits(admin, moderator).

permission(admin, promote_to_mod).
permission(moderator, ban_user).
permission(moderator, ban_protection).

%
% Role & Permissions logic
%
role_has_permission(Role, Action) :- permission(Role, Action).
role_has_permission(Role, Action) :-
  role_inherits(Role, Child),
  role_has_permission(Child, Action).

user_has_permission(User, Club, Action) :-
  role(User, Club, Role),
  role_has_permission(Role, Action).

%
% A dash of metaprogramming for good software design
%
can(Actor, Club, Action, Target) :-
  user_has_permission(Actor, Club, Action),
  call(Action, Actor, Club, Target).

%
% Action-specific validation
%
ban_user(Actor, Club, Target) :-
  member(Target, Club),
  \+ user_has_permission(Target, Club, ban_protection),
  dif(Actor, Target).
```
