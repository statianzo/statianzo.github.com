---
layout: post
title: Websockets using Ruby Eventmachine
---

HTML5 is loaded with a lot of features that will make the lives of developers
much easier and the experience for end users more pleasant as well. Lets take a
look one of the new features: WebSockets. We'll make the magic happen with
Ruby's EventMachine gem.

Mash Some Keys!
---

<div id="alphabet"></div>

**If you're in a browser that supports WebSockets** (Chrome, Safari, Firefox
trunk), go ahead and type a bit. Your keystrokes will be captured, and
broadcasted to any other users on the page. If this page is lonely, you can
open a second browser window to test it out.

Setting Up EventMachine
---

Using [EventMachine][em] for WebSockets is simple with the
[em-websockets][em-ws] gem. Start an `EventMachine.run` block and
`EventMachine::WebSocket.start` will do the rest. It provides a socket object
that resembles the javascript WebSocket spec with callbacks for *onopen*, *onmessage*,
and *onclose*.

    require 'eventmachine'
    require 'em-websocket'

    @sockets = []
    EventMachine.run do
      EventMachine::WebSocket.start(:host => '0.0.0.0', :port => 8080) do |socket|
        socket.onopen do
          @sockets << socket
        end
        socket.onmessage do |mess|
          @sockets.each {|s| s.send mess}
        end
        socket.onclose do
          @sockets.delete socket
        end
      end
    end

In the example, connected sockets are added to an array and broadcasted to when
*onmessage* is triggered.

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
      socket = new WebSocket('ws://h.jxs.me:8080');
      socket.onmessage = function(mess) {
        animateCharacter(mess.data);
      };

    };

    window.onload += setup();

With it all put together, WebSockets makes for a simple way to connect users
together through the browser without plugins, polling, or other ugly frameworks.

[em]: http://rubyeventmachine.com/
[em-ws]: http://github.com/igrigorik/em-websocket 

<script src="http://ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script> 
<script src="/js/websocket-example.js"></script>

