---
title: "Unity Network Baking Paras"
description: "But why did we have 2 different projects in the first place?"
date: "2020-05-09T11:16:16.301Z"
categories: []
published: false
---

But why did we have 2 different projects in the first place?

To make things seamless between the two different projects (One building the asset bundle and the one loading the asset bundle) I created a script called LoadScript that was present in both the projects. The scene producing the asset bundle had an empty script but the script was attached to the root game object in the scene, while the scene loading the asset bundle had the script with the actual logic.

When you build an asset bundle, Unity stores the name of the scripts attached to each game object but does not actually store the script in the asset bundle. So when we load an asset bundle who’s root game object has LoadScript attached to it, Unity locates the file in the project and attaches it to the root game object not caring if its different from the script attached while building. This trickery allows us to have an entry point in the loaded scene even though the entire scene is cleared and reloaded.
