---
layout: post
title:  "Firebase advanded data modeling and role based authentication / authorization"
description: "Using firebase for the first time, one might have questions about how to model data & do authorization. Here's my idea of how to do it (fingers crossed!)."
date: 2016-08-24 09:10:30+01:00
size: 12
sizeDesktop: 12
cover: "assets/firebase-advanced-data-modeling-and-role-based-authentication-authorization/firebase-data-modeling-and-auth.png"
textColor: grey-50
metaColor: grey-500
shadow: true
comments: true
author: dan
tags: [firebase, front-end, back-end, database, db, json, auth, bolt]
---

It's been almost 5 years since Firebase was founded. It looked cool from the beginning, but I never got to use it in a project. <br/> That's going to change today, because I am set on using it for a new project. 

I have no experience with it, so this is just my idea of how to do things. If you have a better idea, feel free to share so we can all become better at Firebasing!


## Project requirements
Before diving deep into how I modelled my data, it's important to understand what are the entities involved and how they relate to each other.

I'll try not to go into details, they won't make much difference anyway.

### Entities
The general rule I followed when designing these entities: avoid deep nesting (saw it [here](https://www.youtube.com/watch?v=9gnvC-xIDgs), by [Chris Esplin](http://twitter.com/chrisesplin))

```javascript
Auth: { /* Model designed by firebase, we can't touch this tanana */ } 

Users: {
  /* I extend firebase Auth with this object */
  $uid: { /* Additional user fields */ }
} 

Projects: {
  $pid: { /* Metadata */ }
}

Files: {
  $pid: {}
}

Tasks: {
  $pid: {}
}

Messages: {
  $pid: {}
}
```

<br/>

### User roles & relationships
Once more, I might be omitting some details here and there, but that shouldn't change my decisions by much.

- Admin

    * **Everything**: Create, Read, Update, Delete 


- Manager
    
    * **Projects**, **Tasks**, **Files**: Create, Read, Update, Delete 
    * **Messages**: Create, Read, Update, Delete 
    * **Messages**: Read 
    * **Tasks**: Create, Read, Update, Delete 
    * **Users**: Create 
    * **Self** Read, Update, Delete (Users -> $ownUid)


- User 

    * **Projects**, **Tasks** Read 
    * **Files** Create, Read 
    * **Own Files** Update, Delete 
    * **Messages** Read 
    * **Own Messages** Create, Update, Delete 
    * **Tasks** Read, Update 
    * **Self** Read, Update, Delete (Users -> $ownUid)


- External users / collaborators

    * **Own Projects** Read (projects that she is part of)
    * **Own Tasks** Read 
    * **Files** Create, Read 
    * **Own Files** Update, Delete 
    * **Self** Read, Update, Delete (Users -> $ownUid)


### General approach
As you can see, it's quite a mess. Expressing this with conditionals (i.e. `if(user.admin === true){...}`) would be a terrible idea. Here's why:

- hard to document
- hard to maintain
- hard to change / scale
- code coupling
- a pain to express as firebase's JSON rules

I ditched this idea quickly after sketching out the system. In fact, I can't see when this type of role-check approach can work. Frankly, it looks a lot like code smell & quick way to acquire technical debt from the beginning of your project. <br/>
What I do want to go for instead is an activity-based authorization system. 

The way that will work is I make a new entity in my real-time database, called **Permissions**.

```javascript
Permissions: {
  $uid: {
    Users: {},
    Projects: {},
    Files: {},
    Tasks: {},
    Messages: {}
  }
}
```

Each user has it's own permission object for all of the entities. <br/>
Here's how that'll look:

```javascript
Permissions: {
  $uid: {
    Projects: {
      create: {}, // { any: true } OR { $pid: true }
      read: {},
      update: {}, 
      delete: {}
    }
    ...
  }
}
```

A small note here, my permissions are a mirror of CRUD, but it can be anything really. For my application this would be good enough, but you might choose to have more fine-grained control in yours.

The way this will work is that upon creation, users get a set of permission. As users create entities in the system, their permission list will be extended.

Now that's fine and dandy, but how to actually read these? And how is this better?<br/>
Grab some popcorn, sit tight, we got this.


### Bolt FTW
If you didn't come across it yet, [Firebase Bolt](https://github.com/firebase/bolt) is a compiler for Firebase rules (it might be illegal, but we could even call it a Firebase rules transpiler?!). It's supposedly still in Beta, so make sure to double-check the output after compiling.

Now, even with Bolt, I've noticed I'd be in trouble already. I want to have CRUD rules, but Firebase only understands Read / Write rules. This forced me to make an important decision, either I:

+ **A.** modify objects to have `{ public: {/* what I have so far */ }, private: {} }` paths & add a field named 'deleted' on `private`, for which I add stricter write rules, i.e. admin only (NB: this way data will never actually be deleted) 
+ **B.** use `newData.exists()` together with my permissions, which will prevent deletion for users that are not authorized to delete

I choose to go with **B.**, but it wasn't an easy decision. After I'll deploy the first version of the app, the only way to change my decision is via (probably) a complicated migration script & plenty of changes to the business logic. <br/>
I'll have a similar issue with 'create'. There's no create rule in Firebase, but that could be translated to *allowed to write on a parent node*. 

There's some good news though, Bolt actually allows splitting the `write` rule into `create`, `update`, `delete` ([see here](https://github.com/firebase/bolt/blob/master/docs/language.md#write-aliases)). It's a matter of preference in the end, but going for either of the styles will work.


`<detour>` *If you do wish to set more fine-grained rules, I can only see it happening by modifying the data structure. For example, if you wanted to have a 'canChangeStatus' permission that only applies to admins, I'd do something like this:*

```javascript
{
  Users: {
    $uid: {
      read: {...} // i.e. joinedDate, lastLoginDate, ...
      write: {...} // i.e. firstName, lastName, ...
      status: {...} // i.e. statusChangedDate, statusName, ...
    }
  }
}
```

*In this case, there will be an additional write rule on **status**. If status is a string, we might be able to do it without changing the data structure. However, that seems to add complexity & increase the number of rules you have to set (from case to case), in which case grouping seems to be a cleaner alternative. <br/>*
*If you have an even better way to handle this, I would love to see it.* `</detour>`


### Bolt implementation
Finally, the fun part. I'm going to sketch out how to implement this in Bolt, but take it with a pinch of salt. I'll be a while until I deploy the first version; initially I'll build a prototype without any permission logic. Most likely I'll write up a quick new post on what changes I did to it before deploying. For now, here goes:

```javascript
TODODODODODOOOTODO
---------
isAllowed(entity, entityId, permission) {
  auth != null && auth.uid &&
  root.permissions[auth.uid][entity][permission].any === true ||
  root.permissions[auth.uid][entity][permission][entityId] === true
}

/* Nobody can access anything by default */
path / {
  read() { false }
  write() { false }
}

type Project {
  read() { isAllowed('project', $pid, 'read') }
  create() { isAllowed('project', $pid, 'create') }
  update() { isAllowed('project', $pid, 'update') }
  delete() { isAllowed('project', $pid, 'delete') }
}

/*
 * ...
 * other types omitted, they would look roughly the same
 * ...
 */

path /projects/{$pid} is Project {}
// path /tasks/{$pid}/{$tid} is Task {}
// path /files/{$pid}/{$fid} is File {}
// path /messages/{$pid}/{$mid} is Message {}
// path /users/{$uid} is User {}
// path /permissions/{$uid} is Permission {} // For now, no user can do CRUD on permissions

```

I omitted fields & validation rules, but that shouldn't matter.

# TODOODODODOOOTOOOO