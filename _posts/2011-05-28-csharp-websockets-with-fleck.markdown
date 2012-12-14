---
layout: post
title: C# Websockets with Fleck
---

In honor of Fleck, a C# websocket library, becoming Mono compatible, I'm
finally giving it some exposure.

[Fleck][fleck] was written in response to a lack of lightweight websockets support in
C#. It was forked from the [Nugget][nug] project, and stripped down to the core.
The result is an API similar to [em-websockets][em-ws] that stays out of your way.

Look familiar?
---

<div id="alphabet"></div>

**If you're in a browser that supports WebSockets** (Chrome, Safari, Firefox
trunk), go ahead and type a bit. Your keystrokes will be captured, and
broadcasted to any other users on the page. If this page is lonely, you can
open a second browser window to test it out.

Fleck in Action
---

Get rolling with Fleck by creating a new `WebSocketServer` with the address to
bind.  Call `Start()` to start the server and configure callbacks for `OnOpen`,
`OnClose`, and `OnMessage`.

    var allSockets = new List<IWebSocketConnection> ();
    var server = new WebSocketServer ("ws://localhost:8081");
    server.Start (socket =>
    {
      socket.OnOpen = () => allSockets.Add (socket);
      socket.OnClose = () => allSockets.Remove (socket);
      socket.OnMessage = message =>
      {
        foreach (var s in allSockets.ToList())
          s.Send (message);
      };
    });
    var input = Console.ReadLine ();

In the example, connected sockets are added to an array and broadcasted to when
`OnMessage` is triggered.

Client Side
---

The client side script listens for keypresses and sends them to the websocket.
On receiving a message, which is just a single letter in this case, the
corresponding letter will be lit up.


    var socket;
    var alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

    function animateCharacter(letter)
    {
      var upper = letter.toUpperCase();
      $('#character_' + upper)
        .stop(true,true)
        .animate({ opacity: 1}, 100)
        .animate({ opacity: .2}, 100);
    }

    function setup()
    {
      var target = $('#alphabet');
      for(var i = 0; i <=alphabet.length; i++)
      {
        var char = alphabet.charAt(i);
        target.append('<span id="character_' + char +'">' + char + '</span');
      }
      connect();

      $(document).keypress(function(e){
        var char = String.fromCharCode(e.keyCode);
        socket.send(char);
      });
    };

    function connect(){
      socket = new WebSocket('ws://localhost:8081');
      socket.onmessage = function(mess) {
        animateCharacter(mess.data);
      };

    };

    window.onload += setup();

With it all put together, WebSockets makes for a simple way to connect users
together through the browser without plugins, polling, or other ugly frameworks.

[nug]: http://nugget.codeplex.com/
[fleck]: https://github.com/statianzo/Fleck
[em-ws]: http://github.com/igrigorik/em-websocket

<script src="http://ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script> 
<script src="/js/fleck-websocket-example.js"></script>

