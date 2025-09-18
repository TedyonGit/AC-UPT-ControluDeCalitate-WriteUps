# lolchat

lol

- This one was a bit tricky and we didn't find much about it. We enter the site and we are welcome by this interface:

![welcome](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/lolchat/welcome.png)

- Nothing much, but we can work with this.
- First we just checked the website to see what has to offer. We found a ``script.js`` containing socket connection.

![script](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/lolchat/scriptjs.png)

- We played around with it, like changing rooms, usernames, etc.

```js
function connect() {
    const chosenName = names[Math.floor(Math.random() * (names.length - 1))];
    document.getElementById("username").innerHTML = `You are: ${chosenName}`;
    const socket = io({ auth: { name: chosenName } });

    socket.on("connect", () => {
        console.log("Connected to server");

        socket.emit("joinRoom", "general");
    });

    socket.on("message", (userId, msg) => {
        addMessage(userId, msg);
    });

    socket.on("activeUsers", (count) => {
        console.log(`Active users: ${count}`);
        document.getElementById(
            "activeUsers"
        ).innerHTML = `Active users: ${count}`;
    });

    socket.on("disconnect", () => {
        console.log("Disconnected from server");
    });

    window.socket = socket;
}
// To make the chatbox work, without modifying this u will only type in "hall" room :D
function send(event) {
    event.preventDefault();
    const input = document.getElementById("messageInput");
    const message = input.value;
    if (message && window.socket) {
        window.socket.emit("sendMessage", { room: "general", message });
        addMessage("you", message);
        input.value = "";
    }
}
```

- Until we changed the room to ``party``. After trying to join party we were greeted with a message from system telling us that ``system: sorry john, you are not allowed in the room party``.

![notallowed](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/lolchat/not_allowed.png)
  
- In that case we made a python script with socketio that tries every name from the list we gathered from ``script.js`` to see which names are allowed to enter ``party room``.

```python
import socketio
import time

url = 'http://ctf.ac.upt.ro:9583'

usernames = [
    "john", "charlie", "anne", "dave", "frank", "grace", "bob"
]

room = "party"

clients = []

def CreateClient(username):
    sio = socketio.Client()

    @sio.event
    def connect():
        print(f'Connected as: {username}')
        sio.emit('joinRoom', room)

    @sio.event
    def message(userId, msg):
        if "sorry" in msg:
            sio.disconnect()
            return

    @sio.event
    def disconnect():
        print(f'Failed to connect as: {username} to room PARTI PARTI PARTI!')

    try:
        sio.connect(url, auth={'name': username})
    except Exception as e:
        print(e)

    return sio

for uname in usernames:
    client = CreateClient(uname)
    clients.append(client)

try:
    for client in clients:
        client.wait()
        
except KeyboardInterrupt:
    for client in clients:
        client.disconnect()

```

![partiparti](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/lolchat/parti%20parti%20check.png)

- Jackpot! We found user ``bob`` being able to join party room!!!


- Now the next step is to connect to user ``bob`` and join room ``party``.

```js
function connect() {
    const chosenName = "bob";
    document.getElementById("username").innerHTML = `You are: ${chosenName}`;
    const socket = io({ auth: { name: chosenName } });

    socket.on("connect", () => {
        console.log("Connected to server");

        socket.emit("joinRoom", "party");
    });

    socket.on("message", (userId, msg) => {
        addMessage(userId, msg);
    });

    socket.on("activeUsers", (count) => {
        console.log(`Active users: ${count}`);
        document.getElementById(
            "activeUsers"
        ).innerHTML = `Active users: ${count}`;
    });

    socket.on("disconnect", () => {
        console.log("Disconnected from server");
    });

    window.socket = socket;
}

function send(event) {
    event.preventDefault();
    const input = document.getElementById("messageInput");
    const message = input.value;
    if (message && window.socket) {
        window.socket.emit("sendMessage", { room: "party", message });
        addMessage("you", message);
        input.value = "";
    }
}
```

![connectedasbob](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/lolchat/connect_bob.png)

- Now we are connected as ``bob``, but nothing is happening.
- Ok, now we need to remember that the challenge name is ``lolchat``, so has something to do with ``randomchat``. Let's see what happens if we type ``hello``.

![hello](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/lolchat/hello.png)

- Oh, we got a response from 2 users ``josh`` and ``alice``. We are on the right track, now we know the users are activating after a certain word. Let's try what are we looking for which is going to be ``flag``.

![flag](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/lolchat/flagf.png)

- Hmm... Let's try the word ``game``, what is going to happen?

![game](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/lolchat/game.png)

- Wow! Let's try ``blue``.

![final](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/lolchat/finish.png)

- GG! We found the flag ``ctf{00d1594338bdd8dd9f01a0e6e999217732b64d95cccbe54ae97c67e6b854845a}``

# Note:

- If you guess wrong the color the user ``josh`` is going to leave the chat without any reply and you need to type ``game`` again to open the "payload".
- I think there are many words you can use to trigger the users, but we didn't find much only like: ``hat``, ``gaming``
- We had some encounters with users talking to each other and not responding to the inputs, but with a restart worked fine after.
