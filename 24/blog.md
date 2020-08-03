Hello again! For this post I don't have any specific topic in particular I feel like digging deep into and rambling on about, so what about if I just briefly talk about seven different random things I've worked on in the last few weeks? Let's do it.

# 1: Physics Body Framework

Recently I reworked our physics system a bit to make it easier to implement new kinds of mechanics in the game. Previously, it was required for objects in the game (including the normal stage meshes which the ball interacts with) to perform collision detection with the ball themselves; this becomes problematic however when one object in the game requires multiple kinds of collision. For example, goals in the game have an outer frame mesh which must act like a hard mesh collision, as well as an invisible "trigger" mesh used to detect when the ball rolls through the goal; these have separate collision detection needs, and making the goal itself implement collision detection for itself makes this trickier. In addition, the old collision framework generally assumed that object collision would usually be with a hard triangle mesh, which isn't always the case: detecting an overlap with an invisible "trigger" mesh can be done in less steps than a "hard" mesh, and detecting collisions with other shapes such as spheres must be done in an entirely different way.

So, I decided to completely factor out the process of collision detection from objects in the game into the physics system, and have game objects only respond to collisions detected by said physics system. Game objects tell the physics system that they have "collision bodies", such as a hard meshes, trigger meshes, or spheres, and the physics system notifies the game objects when these are collided with. Going back to the example of the goal, the goal just needs to specify that it has a "collision mesh physics body" that represents the hard outer frame mesh and a "trigger mesh physics body" representing the inner goal trigger, and the physics system reports to the goal when either of these are interacted with.

This is more conceptually similar to Unreal Engine's collision detection system, which allows you to attach hitbox components to your actors and receive collision notifications; our system is a little simpler though since we don't use a full-blown component system.

# 2: Line Trace Robustness

Some of our mechanics rely on a _line trace_ to function properly, which generally involves drawing a line from one place in space to another point and seeing what it intersects with along the way. For example, to detect whether the ball is inside of a _wind volume_ (more on those later), we must detect whether the ball's center has passed through a set of triangles, or not. If the ball is moving slowly enough, the distance between the starting and ending point of the line trace may be too small, and we might not detect this small line intersecting with a triangle. To solve this, I added a line which follows behind that ball which only updates after the ball has moved a certain minimum distance. This way, the line we use to detect whether the ball has crossed a triangle is guaranteed to be a minimum length, and so this computation should be more robust.

# 3: Goal Loading

Goals are now showing up in the game again! For now it's just an ugly debug placeholder model (don't worry, Brandon designed it to look ugly himself!!), but you can collide with the goalpost and we can detect when you cross through the middle. Loading goals needs to function a bit differently internally than loading normal stage meshes, since the same model needs to be loaded on different stages, so we need a separate database for objects like this.

# 4: Small Gravity Floor and Touch Platform Tweaks

For gravity floors, I've made it possible to disable line traces for a certain gravity floor. Gravity floors normally work by changing your gravity if you touch them, _or_ if a line trace below the ball intersects them; this is what allows you to orbit around curved gravity surfaces! But sometimes this is undesirable, and it ultimately should be up to the level designers whether to enable this or not.

In a similar vein, I've allowed "disabling touches" on a touch platform in a touch platform group. That might seem to defeat the purpose of the mechanic, but when you touch one object in a group, _all_ other objects in the group will start to animate. Sometimes, you'd like one object in this group to _not_ cause the group to start animating, but still be affected if _another_ object in the group is touched.

# 5: Wind Volume Stuff

Wind volumes are one among several other new mechanics I've been working on for the game. When the ball enters a wind volume, "wind" travelling in a certain direction should be applied to the ball. This is a little different than just applying a force to the ball; the force should be applied based on the ball's speed relative to the wind. In addition, it's common to approximate the force applied to something in a moving fluid based on the relative velocity _squared_, as well as apply more force when the fluid is more dense; in the case of air, it's not very dense, so in order to feel realistic, it makes sense to use a _small_ density constant so the ball doesn't trudge slowly through wind volumes even when there's no wind, similar to normal air.

In addition, I've found that it feels pretty natural to apply _partial_ wind to the ball if it's partially intersecting a wind volume. The way we're determining _how much_ the ball is partially intersecting the wind volume at the moment is a simple matter of finding the triangle of the wind volume which the ball is intersecting the least, and figuring out how far the ball's center is from said triangle.

# 6: World Transform Caching

One overdue optimization we've finally implemented is world transform caching: previously, whenever the game needed to know where an object was in world space (aka, _very often..._), it would need to recompute that object's world transform by walking up the scene graph while combining parent nodes' local transforms together. This sort of math can get pretty expensive, and it's silly to do when it only needs to be done once per frame. So now, that is the case.

Some mechanics cause objects to modify where they are (or where they "were") in space when the ball collides with them, meaning some world transforms would become incorrect upon ball collision. Instead of recomputing all world transforms whenever this happens, we just mark world transforms as either "valid" or "invalid" depending on whether they're correct or incorrect. Whenever something needs a world transform, we check if it's valid, and if not, we recompute it. This is called being _lazy_ in programmer-speak, but don't worry, it can actually be a really good thing.

# 7: Additional Mechanics

Some of the other mechanics I've been working on include sticky floors, quicksand floors, springs, and conveyers. Looks like you'll have to wait a little longer to see them fully in action though!

# This is the end of the list.

Farewell until next time!