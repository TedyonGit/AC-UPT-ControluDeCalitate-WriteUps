# Minecrafty

Join my very own minecraft server and have some fun! Don't forget to visit our tourist spot to get your flag.

- On this challenge we have attached the ``server.zip``. Let's try and search in the files the word ``flag`` and see what comes up.

![find](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/minecrafty/searchinfiles.png)

![results](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/minecrafty/chatflag.png)

- Nice! We found ``flag`` text in ``chat.go``, looks like a command. Let's open it so see more.

![flagcommand](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/minecrafty/flagcommand.png)

- So, the command check if your position is at ``{x: 69420, y: 69420, z: 69420}``, i don't see any checks for fake packets or such.
- If you didn't got to the required positions you will receive the message: `` ctf{try_harder}  Your Position: x y z  Expected Position: 69420 69420 69420 ``.
- If you are you will receive the flag.

- Our solution was to install a hacked client and abuse the fly until we reach the coords, which we did in aprox 2 min.

![flag](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/minecrafty/flagminecraft.png)

- There are many solutions and easier by creating a bot and send a position packet, but oh well our solution work too :D.
- Flag: ctf{72b79a618cd6c995584d8971eba740f8597661a6c3806f6d36a6b59dca110071}

# Notes
- We tried to connect with the Owner's name which was ``Stancium19`` to check if has op or not.
