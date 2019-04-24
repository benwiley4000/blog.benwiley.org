+++
contenttype = "article"
date = "2019-02-21T23:18:09-04:00"
draft = false
image = "/img/async_debugging.png"
slug = "async-debugging"
title = "Async/Await in JavaScript: Not as Dumb as I Thought"

+++

Among my colleagues who also write JavaScript for a living, for a long time I've been one of the sole detractors of the `async` and `await` keywords introduced with the ECMAScript 2015 standard.

For those time-traveling from the year 2014, an `async` function in JavaScript uses a new syntax to run a series of dependent asynchronous tasks.

If you had a music library database from which you wanted to fetch the first artist, then their most recent album, then the first song on that album, using JavaScript Promises circa 2015 you might write:

{{<highlight js>}}
function getTrackOne() {
  return fetchArtist(0).then(artist => {
    return fetchAlbumForArtist(artist, 0).then(album => {
      return fetchSongFromAlbum(album, 0);
    });
  });
}
getTrackOne().then(track =>{
  console.log(track);
});
{{</highlight>}}

Using an `async` function you can make this a bit flatter:

{{<highlight js>}}
async function getTrackOne() {
  const artist = await fetchArtist(0);
  const album = await fetchAlbumForArtist(artist, 0);
  const track = await fetchSongFromAlbum(album, 0);
  return track;
}
getTrackOne().then(track =>{
  console.log(track);
});
{{</highlight>}}

Many people find the second version much clearer to read - especially those coming to JavaScript from languages like Java, C# and Python where passing one function into another function isn't a daily occurrence (much less passing a function that will be run *sometime later*).

Even though I think callback functions are fine, and that "callback hell" is a mostly made-up idea invented to motivate needless refactorings, I can buy the readability argument. In the `async` function above, it takes very little effort to understand the order of things.

What I haven't been excited over is the hype that has led to folks compiling `async` functions into older JavaScript using Babel and an expensive ES2015 generator compatibility layer which has made them less performant than regular Promises. Despite `async` functions being, in essence, syntactic sugar for `Promise` chaining, I sometimes hear other developers discussing `async` functions as a kind of revolutionary technology enabling unseen possibilities with JavaScript. Which really, they aren't. *Or so I thought*, anyway.

I'm definitely super late to the party. But yesterday I discovered something that `async` functions allow that you really *can't* do with callback functions, and that is using a browser debugger to step through an `async` function, one line at a time, the same way you would a synchronous function:

<video
  style="max-width: 100%"
  src="/video/async_debugging.mp4"
  muted
  autoplay
  loop
>

Finding what went wrong in an asynchronous routine can be hard in JavaScript. Being able to place a single `debugger;` statement at the top of an `async` function and then step through each task until you find the problem in your code makes debugging a breeze. With callback functions it's difficult to trace cause-and-effect between async task definition and execution, and you also have to deal with problems like missing values from outer scopes because the debugger decided to forget them. Despite all my prior apprehensions I now want to write `async` functions wherever I can.

Of course you don't get this with whatever Babel outputs when you write an `async` function, so if you're not able to target browsers that support `async` and `await` natively, I'd still say don't bother.
