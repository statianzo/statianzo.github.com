---
layout: post
title: Drag and Drop with jQuery
---

The past week, I've been testing out the drag and drop APIs provided by Firefox
and Chrome. It's relatively simple to add a desktop feel to your web apps just
by hooking in.

Handling the Drag
---

Think about what happens you drag and drop a file onto your browser window. Your
browser will take the file and attempt to open it. To perform a custom behavior
for a drag and drop on
a web page, you need to tell the browser, "No thanks. I'll take it from here."
To do that in Javascript terms there are two calls to make on the event object:
`stopPropegation()` and `preventDefault()`. The former will stop the event flow
to other handlers, and the latter stops the browser from performing the default
behavior, opening the file.

{% highlight javascript %}
function ignoreDrag(e) {
  e.originalEvent.stopPropagation();
  e.originalEvent.preventDefault();
}
{% endhighlight %}

In the case that you're using jQuery, calling the attribute `originalEvent` is necessary to
access `stopPropegation()` and `preventDefault()`. You'll want to bind the
`ignoreDrag` function the *dragenter* and *dragover* events of the target.

{% highlight javascript %}
$('#target')
.bind('dragenter', ignoreDrag)
.bind('dragover', ignoreDrag);
.bind('drop', drop);
{% endhighlight %}

You can use these events to style your target as well, to give visual
confirmation that a file is able to be dropped on that element.

Using the Drop
---

Now that you've stopped the normal behavior for the drag, you need to do some
behavior when a file is dropped. The event argument of the *drop* event has an
attribute called `dataTransfer` which contains information about what was
dropped. For accessing data like text or urls, invoke `getData()` passing one of
the types given in `types`. For file access use the `files` attribute to
retreive an array of *File* objects.

{% highlight javascript %}
function drop(e) {
  ignoreDrag(e);
  var dt = e.originalEvent.dataTransfer;
  var files = dt.files;

  if(dt.files.length > 0){
    var file = dt.files[0];
    alert(file.name);
  }
}
{% endhighlight %}

<div id='alerter' style='background:#6666FF;'>Drop File Here</div>

*File* objects contain information about the name and size of the file. Also,
they can be passed to an *XmlHttpRequest* and sent as a blob of data. That's
useful for doing an upload of drag and dropped files. [Andrea Giammarchi][andrea]
wrote a small library called [sendfile][sendfile] for uploading an array of
*File* objects. It could be used as follows:

{% highlight javascript %}
function drop(e) {
  ignoreDrag(e);
  var dt = e.originalEvent.dataTransfer;
  var droppedFiles = dt.files;

  if(dt.files.length > 0){
    sendMultipleFiles({
      files: droppedFiles,
      url: 'http://mywebsite.com/upload',
      onload: function (){ alert('Done'); }
    });
  }
}
{% endhighlight %}

Client Side File Reading in Firefox
---

Firefox allows you to read from a *File* object through three different methods:
`getAsBinary()`, `getAsDataURL()`, and `getAsText(encoding)`. The first will
provide raw binary data. `getAsDataURL` will provide a base-64 formatted
string. Finally `getAsText` will return a string from the encoding provided.

Using `getAsDataURL` will allow you to populate the *src* attribute of an *img*
tag with a *data* string. 

{% highlight javascript %}
function drop(e) {
  ignoreDrag(e);
  var dt = e.originalEvent.dataTransfer;
  var files = dt.files;

  if(dt.files.length > 0){
    $(this).attr('src', files[0].getAsDataURL());
  }
}
{% endhighlight %}

If you're using Firefox (3.6+), drag an image over the image below. The image
will be replaced by the image you drop. No need to send data round trip!

<img id="droppedPic" style="width:500px; height:300px; border: 1px solid black" src="http://img651.imageshack.us/img651/6028/80099190727442.jpg" />

[andrea]: http://webreflection.blogspot.com 
[sendfile]: http://www.3site.eu/jstests/upload/sendFile.js

<script src="http://ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js"></script>
<script src="/js/dragndrop.js"></script>
<script language="javascript" type="text/javascript">
    function ignoreDrag(e) {
      e.originalEvent.stopPropagation();
      e.originalEvent.preventDefault();
    }
    function drop(e) {
      ignoreDrag(e);
      var dt = e.originalEvent.dataTransfer;
      var files = dt.files;

      if(dt.files.length > 0){
        var file = dt.files[0];
        alert(file.name);
      }
    }

    $('#alerter')
    .bind('dragenter', ignoreDrag)
    .bind('dragover', ignoreDrag)
    .bind('drop', drop);
</script>

