---
layout: post
title:  "Firebase advanced data modelling and role based authorization"
description: "Using Firebase for the first time, one might have questions about how to model data & do authorization. Here's my idea of how to do it (fingers crossed!)"
date: 2016-08-25 15:10:30+01:00
size: 12
sizeDesktop: 6
cover: "assets/firebase-advanced-data-modelling-and-role-based-authentication-authorization/firebase-data-modelling-and-auth.png"
textColor: grey-50
metaColor: grey-500
shadow: true
comments: true
author: dan
tags: [firebase, front-end, back-end, data-modelling, database, db, json, auth, bolt]
---

It's been almost 5 years since Firebase was released. It looked cool from the beginning, but I never got to use it in a project. <br/> That's going to change today, because I am set on using it and there's no turning back. 

I'm not experienced with it, so this is just my idea of how to handle things. If you have a better idea, feel free to share so we can all become better at Firebasing!


## Project requirements
Before diving deep into how I modelled my data, it's important to understand what are the entities involved, how they relate to each other & what kind of roles I have to deal with.

I'll try not to go into details; they shouldn't affect my general approach (the stupid devil is **\*always\*** in the detail).


#### 1. Entities
I called the main objects in my real-time database entities: name them as you wish, they are just top-level creatures. 
The main rule I followed when designing these entities: avoid deep nesting ([related](https://www.youtube.com/watch?v=9gnvC-xIDgs), by [Chris Esplin](http://twitter.com/chrisesplin)).

```javascript
Auth: { /* Model designed by Firebase, we can't touch this tanana */ } 

Users: {
  /* $uid from Firebase Auth */
  $uid: {
    role: String, // i.e. 'admin', more on that later
  }
} 

Projects: {
  $pid: { /* Metadata */ }
}

Files: {
  $pid: {
    $fid: { /* File */}
  }
}

Tasks: {
  $pid: {
    $tid: { /* Task */}
  }
}

Messages: {
  $pid: {
    $mid: { /* Message */}
  }
}
```

<br/>

#### 2. User roles & relationships
Once more, I might be omitting some details here and there. Here are the main roles & their permissions:

- Admin

    * **Everything**: Create, Read, Update, Delete 


- Manager
    
    * **Projects**, **Tasks**, **Files**: Create, Read, Update, Delete 
    * **Files** Create, Read, Update, Delete 
    * **Messages**: Create, Read, Update, Delete 
    * **Tasks**: Create, Read, Update, Delete 
    * **Users**: Create, Read
    * **Self** Read, Update, Delete (users/{$ownUid})


- User 

    * **Projects**, **Tasks** Read 
    * **Files** Create, Read, Update, Delete 
    * **Messages** Read 
    * **Own Messages** Create, Update, Delete 
    * **Tasks** Read, Update 
    * **Self** Read, Update, Delete (users/{$ownUid})


- External users / collaborators

    * **Some Projects** Read
    * **Some Tasks** Read 
    * **Files** Create, Read 
    * **Own Files** Update, Delete 
    * **Self** Read, Update, Delete (users/{$ownUid})


## General approach
As you can see, it's quite messy. Expressing this with conditionals (i.e. `if(user.admin === true){...}`) would be a terrible idea. Here's why:

- hard to document
- hard to maintain
- hard to change / scale
- code coupling
- a pain to express as Firebase's JSON rules

I'd like to avoid all this so I ditched the idea quickly. In fact, I can't see how this approach would work with most tools / systems, not only with Firebase, but that's another story. <br/>
What I do want to go for instead is an activity-based authorization system. 

Here's how that'll work: By default no user has any permission. Then, I create a new entity in my real-time database, called **Permissions** where I can define roles (that give certain permissions). I'll key by role name and write all permissions for a certain role. Example: <br/>

```javascript
Permissions: {
  user: { // User role
    users: {
      own: { // can read, update, delete own data
        r: true,
        u: true,
        d: true
      },
      other: {} // can't read/write 'other' (not owned) data; we can even omit this field
    },
    /* 
     * ... 
     * skipped some entities because 'messages' are more interesting
     * ...
     */
    messages: {
      c: true, // can create messages
    },
    message: {
      own: { // can update & delete own messages
        r: true,
        u: true,
        d: true
      },
      other: { // can read other messages
        r: true
      }
    }
  }
}
```

A small note here, my permissions mirror CRUD, but they can be anything really. I also grouped them in 'own' & 'other' for readability, but it might as well be `canReadOwn` or `canReadOther`. For my use case the above is good enough, but you might choose to have more fine-grained control in yours.

The way this will work is that upon creation, users are assigned a role. It's important to note that only admins can add users to this app (there's no open registration... kind of). Because I don't want to use a back-end, there's an immediate problem with this: there's actually no way to restrict Firebase registration through the console. With some nifty scripts clever people could create an account for themselves, but the joke's on them. <br/>
Firstly, I'll make sure a `changeRole` permission exists for writing to `/users/$uid/role`, which only admins have (they'll have permission to do everything, including this). <br/>
Secondly, if there's no `/users/$uid/role` set, I restrict all read & write access. That should keep those pesky clever users from messing with my database. 

Now that's fine and dandy, but how to actually implement this?<br/>
Don't worry friend, we got this.


## Bolt FTW
If you didn't come across it yet, [Firebase Bolt](https://github.com/firebase/bolt) is a compiler for Firebase rules (it might be illegal, but could we call it a Firebase rules transpiler?!). It's supposedly experimental, so make sure to double-check the output after compiling (unit test!).

Now, even with Bolt, I've noticed I'd be in trouble already. I want to have CRUD-like rules, but Firebase only knows about Read / Write. How to restrict deletion then? <br/>
I had to choose between (I believe) two options:

+ **A.** build objects such that they have `{ public: {/* what I have so far */ }, private: { deleted: Boolean } }` paths & add a field named 'deleted' on `private`, for which I add stricter write rules, i.e. only admins will have permissions to write (NB: this way data will never actually be deleted). Then, I'd tweak the rules to prevent access for items with `private.deleted === true` 
+ **B.** use `newData.exists()` together with my permissions, which will prevent deletion for users that are not authorized to delete

I went with **B.**, but some might find **A.** to be suitable: that way data can be restored later if needed. After I'll deploy the first version of the app, the only way to change my decision is via (probably) a complicated migration script.

I'll have a similar issue with 'create'. There's no create rule in Firebase, but there are ways to make that work too.<br/>
Long story short, Bolt supports splitting the `write` rule into `create`, `update`, `delete` ([see here](https://github.com/firebase/bolt/blob/master/docs/language.md#write-aliases)). It's a matter of preference in the end, but I'll stick with it for the sake of simplicity. You can achieve the same in a more explicit manner.


## Bolt implementation
Finally, the fun part. I'm going to sketch out how to implement this in Bolt, but take it with a pinch of salt. Most likely I'll write up a quick new post on what changes I did to it before actually deploying. For now, here goes:

```javascript
/*
 * Checks if a given action is allowed.
 *
 * @param {object} obj The object to check.
 * @param {string} entity The entity to check.
 * @param {string} permission The permission to check for.
 * @param {boolean} ownResources A flag that disables 'own' data checks if set (if false, users can perform an action if they own the resource)
 *
 */
isAllowed(obj, entity, permission, ownResources) {
  auth != null && auth.uid && // has to be authenticated
  root.users[auth.uid].role && // has to have a role set
  (
    auth.uid !== obj.owner // user is not owner of the entity
    ? 
    root.permissions[root.users[auth.uid].role][entity].all === true || // allow everything
    /* entity w/o owner: doesn't require extra nesting in the permissions obj (.other) */
    root.permissions[root.users[auth.uid].role][entity][permission] === true ||
    root.permissions[root.users[auth.uid].role][entity].other[permission] === true
    : 
    (!ownResources ? root.permissions[root.users[auth.uid].role][entity].own[permission] === true : false)
  )
}

/* Nobody can access anything by default */
path / {
  read() { false }
  write() { false }
}

/* Example of permission check: Projects & Project */
type Projects {
  create() { isAllowed(this, 'projects', 'c', false) }
}

type Project {
  read() { isAllowed(this, 'project', 'r', false) }
  update() { isAllowed(this, 'project', 'u', false) }
  delete() { isAllowed(this, 'project', 'd', false) }
}

path /projects is Projects {}
path /projects/{$pid} is Project {}
path /users/{$uid}/role { /* Only admins should be able to write the role. */
  read() { auth.uid == $uid } /* Users can read their own role */
  write() { isAllowed(this, 'user', 'changeRole', true) }
}

path /projects/{$uid}/owner { /* Users can only write to their own 'owner' fields. */
  write() { auth.uid == $uid }
}

/*
 * ...
 * Other entities omitted; they would look similar.
 * Each entity with an owner will have a `/owner` rule. 
 * That prohibits anybody from changing it, like we have for `/projects/{$uid}/owner`
 *//
```

That's about it. It compiles via Bolt, so it should work... maybe with some tweaks here & there. With proper unit tests, this baby's gonna be able to handle whatever you throw at it.

To give one more example, here's how the admin role would look:

```javascript
Permissions: {
  admin: { // Admin role: allow everything
    users: {
      all: true
    },
    projects: {
      all: true
    },
    project: {
      all: true
    },
    tasks: {
      all: true
    },
    task: {
      all: true
    },
    files: {
      all: true
    },
    file: {
      all: true
    },
    messages: {
      all: true
    },
    message: {
      all: true
    }
  }
  /* ... */
}
```

## Front-end implementation
Firebase doesn't require a back-end, but you can use one if you want to (or absolutely need it). For this project, I chose not to use a back-end. That means that I'll handle the UI rendering 100% in the front-end. In turn, that will require that I know what a certain user is allowed to do upon authentication. This way I can show / hide relevant parts of the UI (i.e. a delete button).

No matter if you are spinning up a React, Angular, etc. app or even cowboying your way through with a Vanilla JS app, the implementation shouldn't differ much. <br/>
One way to do it is load permissions when a resource is accessed (in the context of the current auth. user). Similarly to the Bolt implementation, a generic `isAllowed()` method should check if certain actions can be performed. 
Another way is to make use of the real-time database and keep an eye out for permission changes. Of course, that means all permissions will be loaded upon login.

## Caveats
<small>(yes, I used this word because I don't like pitfalls. do you?!)</small>

1. Don't take unit testing lightly, you need to do it properly (Bolt == experimental)
2. It's a bit hard to see the full picture; there might still be some security issues, but hey, it's a complex app with many user roles (see 1.).
3. Not being able to restrict registration makes me feel a bit uneasy, so perhaps a back-end would be necessary somewhere down the road.

I want to believe that this strategy will work all the way & I won't need any server logic. Is that maybe too ambitious?<br/>
Would love to get some feedback from experienced Firebasing guru-ninjas.
