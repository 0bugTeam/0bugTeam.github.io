---
title: Flag Command 
date: 2023-12-10T13:37:49+02:00
author: 1byteBoy
tags:
  - Web
draft: false
---

```
Flag Command

Embark on the "Dimensional Escape Quest" where you wake up in a mysterious forest maze that's not quite of this world. Navigate singing squirrels, mischievous nymphs, and grumpy wizards in a whimsical labyrinth that may lead to otherworldly surprises. Will you conquer the enchanted maze or find yourself lost in a different dimension of magical challenges? The journey unfolds in this mystical escape!
```

Inspecting `main.js` file in the *Debugger* tab, we can read the main.js file, which has the following function at the end 

```js
const fetchOptions = () => {
    fetch('/api/options')
        .then((data) => data.json())
        .then((res) => {
            availableOptions = res.allPossibleCommands;

        })
        .catch(() => {
            availableOptions = undefined;
        })
}
```

now visiting the endpoitn gives us the following output 

```bash
curl -v http://83.136.251.145:34484/api/options 
```

```
.......
      "BUILD A RAFT AND SAIL DOWNSTREAM"
    ],
    "secret": [
      "Blip-blop, in a pickle with a hiccup! Shmiggity-shmack"
    ]
  }
}
```

using that in the command section of the game will give us the flag.

