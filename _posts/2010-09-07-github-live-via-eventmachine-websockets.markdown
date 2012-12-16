---
layout: post
title: GitHub Live via Eventmachine Websockets
---

Keeping track of commits that have been pushed to GitHub can wind up in many
wasteful F5 keystrokes or polling scheme. Using the Post-Receive hook that
GitHub provides can turn the tables having it notify us when a change is made.
Adding on Websockets can create a browser based display for calls to that hook.

The Hook
---

Setting up the hook is simple after glancing at [post-receive docs][postreceive].
Simply enter the *Admin* section of your own personal GitHub repository and set
up the required service hook. Point it at a url of a web server you own.

Ruby Side
---

For this, I went with [Sinatra][sinatra] because it's simple to get rolling. Also, I use
the [em-websockets][emw] gem for the purpose of simple web sockets. For further
reference on usage see my [previous post][previous] about them. 

{% highlight ruby %}
require 'bundler'
Bundler.setup
require 'sinatra/base'
require 'yaml'
require 'em-websocket'
require 'open-uri'

GITHUB_URI = 'http://github.com/api/v2/yaml/commits/list/statianzo/emptyrepo/master
SOCKETS = []
EventMachine.run do

  class LivePush < Sinatra::Base
    set :public, File.dirname(__FILE__) + '/public'
    get '/' do
     open(GITHUB_URI) {|s| @commits = YAML::load(s.read)['commits']} unless @commit
     erb :show
    end

    post '/receive' do
      puts "incoming!!"
      if params[:payload]
        @commits = nil
        SOCKETS.each {|s| s.send params[:payload]}
      end
      'Thanks'
    end
  end

  EventMachine::WebSocket.start(:host => '0.0.0.0', :port => 8080) do |ws|
    ws.onopen do
      puts "hello!"
      SOCKETS << ws
    end
    ws.onclose { SOCKETS.delete ws}
  end

  LivePush.run!({:port => 8081})
end
{% endhighlight %}

The Sinatra app provides routes for */* and */receive*. The first is for
displaying of the commits and the second takes the content GitHub sends and
broadcasts it to all sockets connected.

Javascript Side
---

The javascript is all about parsing the data and slapping it on the page to make
it visible. Using the json parser from [json.org][json] makes this painless.

{% highlight javascript %}
function connect(){
  var socket = new WebSocket('ws://yourserver.com:8080');
  socket.onmessage = function(mess) {
    if(mess && mess.data){
      var table = $('#commits');
      var result = JSON.parse(mess.data);
      for(var i = 0; i < result.commits.length; i++){
        var commit = result.commits[i];
        table.append('<tr><td>'+ commit.id + '</td><td>'+
            commit.author.name + '</td><td>' + commit.message + '</td></tr>')
      }
    }
  };
};
$(function(){
  connect();
});
{% endhighlight %}

In this example, the results are appended to a `<table>` on the current page.
Very basic, but you can get as fancy as you'd like and pull any of the data that
the [post-receive][postreceive] hook provides.

And that's it. Instant feedback about what's getting pushed to your GitHub
repository. I'll have a demo and the source up soon.

[postreceive]: http://help.github.com/post-receive-hooks/
[sinatra]: http://www.sinatrarb.com/
[emw]: http://github.com/igrigorik/em-websocket
[previous]: http://jxs.me/2010/08/20/websockets-using-ruby-eventmachine/
