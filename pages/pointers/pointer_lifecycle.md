---
title: Pointer Lifecycle
keywords: engage, about
tags: [pointer]
sidebar: overview_sidebar
permalink: pointer_lifecycle.html
summary: NRLS Pointer Lifecycle
---

{% include important.html content="This site is under active development by NHS Digital and is intended to provide all the technical resources you need to successfully develop the NRLS API. This project is being developed using an agile methodology so iterative updates to content will be added on a regular basis." %}

{% include warning.html content="
The current version of the NRLS API does not give Providers the ability to update the properties of an existing Pointer. 
The only property that can be modified is the status and that is done in a very specific way only allowing a Provider to replace one 
pointer with another [Managing Pointers](pointer_maintenance.html#managing-pointers-to-static-content).
<br/>
This means that a number of scenarios outlined below around updating a Pointer can not yet be achieved though the NRLS intends to support these features in a future release. 
<br/>
Specifically the NRLS API does not currently support- <br/>

- The following status property transitions:<br/>

	&emsp; o **current** to **entered-in-error**<br/>
	&emsp; o **entered-in-error** to **current**<br/>
	&emsp; o **current** to **superseded** where the current Pointer is not being replaced.<br/>
	
- Modification of other properties<br/>
" %}


## Pointer Lifecycle ##

A Pointer is a reference to some content. From the perspective of NRLS that content is held on a remote system. 
It has its own lifecycle that is managed by a third-party (the Record owner). The state of the Pointer should reflect the 
state of the referenced content so that Consumers are able to make informed decisions around whether or not a Pointer is worth 
following or given the choice of more than one Pointer which is the most appropriate to follow.

## Pointer Status ##

The Pointer has the concept of a status. It can have one of three possible values - 
- Current - The status provide a means for a Consumer to understand whether the Pointer references the “current” Record. The definition of “current” is under the control of the Provider but a Consumer should be confident that by selecting the latest Pointer they will be presented with what the Provider considers to be the most appropriate for Consumers to use. 
- Superseded – This is an overloaded term in NRLS. It can have one of two meanings:
	1. Indicates that this Pointer has been replaced by another. Consumers should be aware that there is another 
	Pointer referencing a more [current content](pointer_maintenance.html#managing-pointers-to-static-content).
	2. Indicates that this Pointer has been deprecated but has not been replaced by another DocumentReference. 
	There is no newer Pointer or content that can be retrieved but the use of this Pointer and its content is discouraged. See [Status transition](#pointer-status-transition-worked-examples) for more information.
- Entered-in-error – Indicates that the Pointer should not really have been entered into the NRLS however in keeping with the principle of deleting only by exception [link to principles](overview_principles.html) this status allows a Provider to mark a Pointer as erroneous without needing to delete it.

## Pointer status: legal transitions ##

Not only is the value of a Pointer’s status constrained but the transition from one status to another is also tightly controlled.

<img src="images/pointers/pointer_transitions.png">

***Fig 1: status transitions: The NRLS controls that transition from one status to another. It is not possible to transition from one to all states.***

All Pointers begin life with a status of current. From there it is possible to move into a deprecated state or to an entered-in-error state.

Once in a deprecated state the Pointer cannot transition anywhere else. From the entered-in-error state the Pointer can move back to current. 
One cannot build a chain of Pointers on top of a Pointer whose status is entered-in-error. Only the current Pointer can be used in this 
way by superseding it and replacing it with a new version that becomes the current Pointer. See the Pointer status transition: 
worked examples section that details how to the NRLS allows a Provider to transition the status of their Pointers.

## Pointer status: making transitions ##

When a Pointer is first created it will always have a status of current. 
From there it is possible to supersede that Pointer or to mark it as entered-in-error. Fig 2 below illustrates the NRLS functions that must be invoked in order to trigger the transition from one state to another (legal) state.

<img src="images/pointers/pointer_transitions2.png">

***Fig 2: How to transition between statuses: The NRLS allows the Provider to transition their Pointers state using a combination 
of the create and update actions.***

One transition that is worth expanding on is the transition from current to superseded. In this case the existing Pointer (P1) whose status is current is to be replaced or superseded by a new Pointer (P2) which will become the current Pointer and P1 will become superseded. 

In order to supersede there must always be a replacement. Therefore to ensure the transactional integrity of this activity which spans 
two Pointers (P1 and P2 in our example) the action is wrapped up into the CREATE of P2. More details on the mechanics of this are provided 
in the section named Managing Pointers to static content below.

## Pointer status transition: worked examples ##

Note that in the diagrams below three properties from the Pointer data model are referenced. One of them is the version. 
This is the version of the Pointer and not the version of the content that the Pointer references. 
Each time a particular instance of a Pointer is modified the NRLS service will increment the version by one as can be seen in several of the worked examples.

***Null to Current***

As a Provider I want to create a new Pointer on NRLS so that Consumers are aware of a resource that I own.

<img src="images/pointers/pointer_transitions3.png">

The Provider CREATEs a brand new Pointer transitioning from the null state (no Pointer exists) to a newly minted Pointer whose status can only be current.

***Current to Superseded (replaced)***

As a Provider I want to create a new Pointer on NRLS that supersedes one of my existing Pointers so that Consumers have access to the latest 
information regarding my resource.

<img src="images/pointers/pointer_transitions4.png">

As part of the CREATE of the new Pointer that will replace the existing one the Provider describes the relationship between the 
new and existing Pointer. The NRLS uses this relationship to create a new Pointer marking it as current and to deprecate the existing 
Pointer marking it as superseded and also modifying its version to reflect the change to that Pointer.

***Current to Superseded (deprecated)***

As a Provider I want to mark one of my existing pointers as superseded to discourage its use. 
In this case I do not have anything to replace that Pointer with.

<img src="images/pointers/pointer_transitions8.png">

The Provider must UPDATE the existing resource changing its status from current to superseded. In doing so the NRLS will modify its version to reflect the change to that Pointer.

***Current to Entered in error***

As a Provider I want to mark an existing Pointer as entered in error so that Consumers know not to base any clinical decisions 
on the Pointer or the resource that it references.

<img src="images/pointers/pointer_transitions5.png">

The Provider must UPDATE the existing resource changing its status from current to entered-in-error. In doing so the NRLS will modify its version to reflect the change to that Pointer.


***Entered in error to Current***

As a Provider I want to revert an existing Pointer from entered in error to current so that Consumers know that the 
Pointer and its referenced resource can be used in clinical decision making processes. 

<img src="images/pointers/pointer_transitions6.png">

The Provider must UPDATE the existing resource changing its status from entered-in-error to current. 
In doing so the NRLS will modify its version to reflect the change to that Pointer.

***Entered in error to Superseded***

As a Provider I want to transition a Pointer from entered-in-error to superseded so that Consumers know that the Pointer 
and its referenced resource can be used in clinical decision making processes but that there is a newer Pointer available that 
holds more up to date information. 

<img src="images/pointers/pointer_transitions7.png">

In order to make this transition the Provider must invoke two separate actions on the NRLS. 
The first is to UPDATE the entered-in-error Pointer marking it as current. Without this the next step will not be possible. 
Step two is to CREATE a new Pointer and as part of that supersede the existing Pointer. 
This scenario is really a combination of the Entered in error to current and current to superseded scenarios detailed earlier. 
Note also how the version number of the existing Pointer changes as its status transitions from entered-in-error to current to superseded. 

