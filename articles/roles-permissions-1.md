## Building a Roles & Permissions System: Part 1

Logic programming is something all developers should learn. It isn't actually that hard, and simply knowing **what's possible** will positively influence you in choosing better tools, for now and the future.

In this post I'm going to show you how easy it is to write a role-based permissions system in Prolog. In our first iteration, you'll only need **5 lines of logic**, and easily extendable to boot. This will give you a sample taste of the full power of Prolog.

**Note:** This is not a full introduction to Prolog. That said, you should still be able to follow along, because **logic programming is easy**. When you're ready to dive deeper, you can bookmark [this post](https://dev.to/matchilling/introduction-to-logic-programming-with-prolog-cdh) for later.

## The Scenario

Let's say we have users and clubs:

```pl
user(alice).
user(bob).
user(carly).
user(dan).
user(elli).

club(boxing).
club(chess).
```

with a [many-to-many relationship](https://en.wikipedia.org/wiki/Many-to-many_(data_model)) between the two:

```pl
member(alice, chess).
member(dan,   chess).
member(elli,  chess).

member(alice, boxing).
member(bob,   boxing).
member(carly, boxing).
```

And lastly, we want to specify which user is allowed to **moderate** which club:

```pl
moderator(dan,   chess).

moderator(alice, boxing).
moderator(bob,   boxing).
```

Note that we haven't yet written a single line of logic. All code so far has been **facts** – simple statements of truth. And yet, with only this, Prolog grants us great, expressive power! Allow me to demonstrate.

Here is how we ask Prolog for all the clubs `alice` is a member of:

```pl
?- member(alice, C).
C = chess ;
C = boxing.
```

Note how Prolog gives us two solutions.

How about all clubs that `alice` is *also* a moderator of?

```pl
?- member(alice, C), moderator(alice, C).
C = boxing.
```

Or how about all non-moderator members of all clubs?

```pl
?- member(U, C), \+ moderator(U, C).
U = alice,
C = chess ;
U = elli,
C = chess ;
U = carly,
C = boxing.
```

Note how Prolog gives us **three** solutions when there are *six* memberships. Also note that `\+` is the operator for "not".

## The Logic

Given the incredible expressiveness Prolog grants us, it's very easy to begin adding logic for protected moderator actions.

Let's say we want a moderator to be able to **ban a club member**. A little harsh, I know, but it's a concept we all understand!

Here's how you can do it – in the advertised 5 lines – to be explained shortly afterwards:

```pl
can(User, Club, ban_user, Target) :-
  moderator(User, Club),
  member(Target, Club),
  \+ moderator(Target, Club),
  dif(User, Target).
```

and here's how you use it:

```pl
% Alice can ban a member
?- can(alice, boxing, ban_user, carly).
true.

% But she cannot ban another moderator
?- can(alice, boxing, ban_user, bob).
false.

% Nor can she ban herself
?- can(alice, boxing, ban_user, alice).
false.

% And non-moderators cannot ban anyone
?- can(elli, chess, ban_user, alice).
false.
```

That's right, we now have fully working permission logic for banning a user. All moderators now have the ability to ban. To add another moderator, just add a new `moderator` clause.

Here's an explanation of the `can` rule:

- `can(User, Club, ban_user, Target)` - For a given user, club, and target, we are defining logic specifically for `ban_user`.
- `moderator(User, Club)` - The given user must be a moderator of the given club.
- `member(Target, Club)` - The given target user must be a member of *that same* club. You can't ban someone who's not even in the club!
- `\+ moderator(Target, Club)` - The target user must not be a moderator.
- `dif(User, Target)` - And lastly, the user performing the action cannot target themselves (`dif` is built into Prolog).

And that's all it takes. Gaze upon the beauty of Prolog.

## Going Further

- The facts we wrote are hard-coded. In practice, these can be loaded from a database.
- [SWI Prolog](https://www.swi-prolog.org/pldoc/doc_for?object=manual) has HTTP and Docker support, so it's possible to host a "logic server" that answers questions for the other web services in your network.
- The concept of roles and permissions can be decoupled and abstracted further, making our data even more serialization-friendly. This is covered in [Part 2](./permission-system-2.md).


## Full Code

```pl
user(alice).
user(bob).
user(carly).
user(dan).
user(elli).

club(boxing).
club(chess).

member(alice, chess).
member(dan,   chess).
member(elli,  chess).

member(alice, boxing).
member(bob,   boxing).
member(carly, boxing).

moderator(dan, chess).
moderator(alice, boxing).
moderator(bob,   boxing).

can(User, Club, ban_user, Target) :-
  moderator(User, Club),
  dif(User, Target),
  member(Target, Club),
  \+ moderator(Target, Club).
```
