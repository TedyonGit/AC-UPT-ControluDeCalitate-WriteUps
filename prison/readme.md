# prison

You're going to prison kid.There's no way around that. Take a good look and learn your betters. Your life will depend on them now. This ain't no scandinavian heaven where you're handed everything on a silver platter.
Flag format: either CTF{server_host(ChunkX,ChunkY,ChunkZ)} (case insensitive and ascii characters only)-Careful, these are chunk coordinates, not block coordinates. or if you don't have a minecraft account: CTF{server_host:owner_username}
Example: CTF{mc.hypixel.net(-69,420,10)} or CTF{mc.hypixel.net:hypixel}

- In this challenge we need to find a specific chunk and a minecraft server.
- In that case we extract unique features that can help us in finding it.

- First, we see some custom rank names, let's strat with that. Open up google and search for:

```
what minecraft server has server ranks "srwarden", "srguard"
```

- Hmmm.. we didn't find it, let's add "reddit" to the search, because there needs to be a post or similar posts

```
what minecraft server has server ranks "srwarden", "srguard" reddit
```

- We got a bunch of results and we the name "The Pen", we might be on the right track. Let's open the first reddit post.

![]()

```
Guard - Ensures safety of Prisoners, enforces rules, and takes Contraband

SrGuard - Well seasoned Guard with Moderator Privileges

Warden - Administrator Position

SrWarden - Senior Administrator Position
```

- So far so good. This server has rank names ``srguard`` and ``srwarden`` and is a classic prison.
- Let's enter on the server ``play.thepen-mc.net`` with version 1.21.8.

- Direct connection

![]()

- Join server

![]()

- We connected to the server and we need to do a tutorial.

![]()

- Is not much of a tutorial, all you need to do is just move forward and click on the NPC to ``complete the tutorial``. But we see something familiar. The ``warden`` rank.

![]()

- Let's proceed forward to see if we found the room or not.

- Go right and leave the bus

![]()

- Go forward to enter the gate

![]()

- Go right again to enter the building

![]()

- On the left side we find ``staff room``

![]()

- And there we go! We found it! With minor changes, but we found it :D

![]()

- All we have to do is just to format the flag and that's it! Flag: ctf{play.thepen-mc.net(-215,4,35)}

![happyending]()
