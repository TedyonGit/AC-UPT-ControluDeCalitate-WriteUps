# Baofeng

Our radio monitoring agency captured a recording of someone disturbing the radio waves. Due to buget cuts there's nobody to decode the message and who he is. Please help us!
Flag format: `ctf{callsign_cityname}

- On this challenge we had attached a ``.mp3`` file which we need to estract the ``callsign`` and ``cityname``.
- We used a website called [audio-to-text](https://elevenlabs.io/audio-to-text) to extract every word out of the audio file.

![baofang-audio-to-text](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/Baofeng/baofang-audio-to-text.png)

```
CQ, CQ, CQ. This is Yankee Oscar Two Tango Sierra Sierra. My QTH is Kilo November 15 Kilo Sierra. CQ, CQ, CQ. This is Yankee Oscar Two Tango Sierra Sierra.
```

- After we extracted we knew is NATO phonetic alphabet so we decipher every word in order.

```
CQ = General callsign
Yankee = Y
Oscar = O
Tango = T
Sierra = S
Sierra = S

QTH = My current location

Kilo = K
November = N
Kilo = K
Sierra = S

Yankee = Y
Oscar = O
Tango = T
Sierra = S
Sierra = S
```
- The result was ``YO2TSS KN15KS YO2TSS``
- There is one catch! In the flag format they said to specify the call sign which was ``YO2TSS`` and ``cityname`` and ``KN15KS`` isn't a city name.
- In that case, we searched ``KN15KS`` on google.

![google-search](https://github.com/TedyonGit/AC-UPT-ControluDeCalitate-WriteUps/blob/main/Baofeng/Screenshot%202025-09-18%20093626.png)

- And now we knew that code ``KN15KS`` means ``hunedoara``

Flag: ctf{yo2tss_hunedoara}
