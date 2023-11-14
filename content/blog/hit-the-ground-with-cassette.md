+++
contenttype = "article"
date = "2019-04-24T10:08:17-04:00"
draft = true
image = "/img/async_debugging.png"
slug = "async-debugging"
title = "How to Hit the Ground Running Building Media Players with Cassette and React"

+++

TODO TODO: add stuff about screenshots and mobile system integration in beginning when we talk about what Cassette brings out of the box!!!!!!!!!!!!!!!!!!!!!!

[Cassette](https://github.com/benwiley4000/cassette) (just released in beta!) is a React component library that helps build media player applications for web browsers. It's meant to combine ease-of-onramp and a solid default end-user experience with a powerful degree of flexibility. You get a totally-functional video/audio player component from the start, but you're able to implement your own controls and eventually maybe even decide to throw all the default UI away and create your own (or use them in combination). Cassette is flexible enough to help you build apps like Netflix, Soundcloud, YouTube, Vimeo, Spotify, Facebook video... all with the same API!

It's kind of hard to imagine what Cassette can do without just seeing it in action, so here we go!

## Cassette Quickstart

### I: Hello World

If you're using HTML with script tags, here's your minimum Hello World with Cassette:

```html
<!DOCTYPE html>

<!-- include @cassette/player's stylesheet -->
<link rel="stylesheet" href="https://unpkg.com/@cassette/player/dist/css/cassette-player.css">

<!-- app container -->
<div style="max-width:700px" id="app"></div>

<!-- dependencies -->
<script src="https://unpkg.com/react@16.8.6/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@16.8.6/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/prop-types@15.7.2/prop-types.js"></script>
<script src="https://unpkg.com/resize-observer-polyfill@1.5.1/dist/ResizeObserver.js"></script>

<!-- other @cassette packages which are dependencies -->
<script src="https://unpkg.com/@cassette/core@2.0.0-beta.1/dist/es5/cassette-core.js"></script>
<script src="https://unpkg.com/@cassette/components@2.0.0-beta.1/dist/es5/cassette-components.js"></script>

<!-- @cassette/player's javascript -->
<script src="https://unpkg.com/@cassette/player@2.0.0-beta.1/dist/es5/cassette-player.js"></script>

<script>
  // your code!

  var MediaPlayer = cassettePlayer.MediaPlayer;

  var playlist = [
    {
      url:
        'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4',
      title: 'Big Buck Bunny'
    },
    {
      url:
        'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4',
      title: 'Elephants Dream'
    }
  ];

  ReactDOM.render(
    React.createElement(
      MediaPlayer,
      {
        playlist: playlist,
        showVideo: true
      }
    ),
    document.getElementById('app')
  );
</script>
```

<div class="screenshot_wrapper">
  <img src="img/hello_cassette.png">
  <div class="notice"></div>
</div>

### II: Hello World with npm

If you're using npm or yarn and a bundler like Parcel or Webpack, you can do the same thing with this JavaScript (just `npm install @cassette/player` first):

```jsx static
import { MediaPlayer } from '@cassette/player';

import '@cassette/player/dist/css/cassette-player.css';

const playlist = [
  {
    url:
      'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4',
    title: 'Big Buck Bunny'
  },
  {
    url:
      'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4',
    title: 'Elephants Dream'
  }
];

ReactDOM.render(
  <MediaPlayer playlist={playlist} showVideo />,
  document.getElementById('app')
);
```

### III: Controls configuration

Let's say you want to change the controls a bit. You don't need the back skip button, but you _would_ like to add a mute control. That's easy:

```jsx static
import { MediaPlayer } from '@cassette/player';

import '@cassette/player/dist/css/cassette-player.css';

const playlist = /* unchanged */;

ReactDOM.render(
  <MediaPlayer
    playlist={playlist}
    showVideo
    controls={[
      'spacer',
      'playpause',
      'forwardskip',
      'mute',
      'spacer',
      'progress'
    ]}
  />,
  document.getElementById('app')
);
```

<div class="screenshot_wrapper">
  <img src="img/custom_controls.png">
  <div class="notice"></div>
</div>

### IV: Bring your own controls

You also want to throw in a control to adjust the playback rate. But that's not included with Cassette! The good news is, it's pretty simple to implement a control like that yourself:

```jsx static
import { MediaPlayer } from '@cassette/player';

import '@cassette/player/dist/css/cassette-player.css';

const playlist = /* unchanged */;

function PlaybackRateControl({ playbackRate, onSetPlaybackRate }) {
  return (
    <div style={{ color: 'white', display: 'flex', alignItems: 'center' }}>
      Speed: x
      <input
        type="number"
        min={0.5}
        max={3}
        step={0.25}
        value={playbackRate}
        onChange={e => onSetPlaybackRate(Number(e.target.value))}
        style={{ width: 50 }}
      />
    </div>
  );
}

ReactDOM.render(
  <MediaPlayer
    playlist={playlist}
    showVideo
    controls={[
      'spacer',
      'playpause',
      'forwardskip',
      'mute',
      playerContext => (
        <PlaybackRateControl
          playbackRate={playerContext.playbackRate}
          onSetPlaybackRate={playerContext.onSetPlaybackRate}
        />
      ),
      'spacer',
      'progress'
    ]}
  />,
  document.getElementById('app')
);
```

<div class="screenshot_wrapper">
  <img src="img/playbackrate_control.png">
  <div class="notice"></div>
</div>

### V: Blowing off the lid

Now you want something crazy.. you want to display a menu to select and play a track from the playlist, and _outside_ of your `MediaPlayer`. How can we do this? The answer is, by dividing `MediaPlayer` up into its component parts - `MediaPlayerControls` and `PlayerContextProvider`.

Note that some `MediaPlayer` props belong to `PlayerContextProvider`, and some props belong to `MediaPlayerControls`:

```jsx static
import { MediaPlayerControls } from '@cassette/player';
import { PlayerContextProvider } from '@cassette/core';

import '@cassette/player/dist/css/cassette-player.css';

const playlist = /* unchanged */;

function PlaybackRateControl() {/* unchanged */}

ReactDOM.render(
  <PlayerContextProvider playlist={playlist}>
    <MediaPlayerControls
      showVideo
      controls={/* unchanged */}
    />
  </PlayerContextProvider>,
  document.getElementById('app')
);
```

At this point, your app should still look the same.

Lastly, you'll create your playlist menu component using a higher-order component called `playerContextFilter`. You can render it anywhere as a descendant of your `PlayerContextProvider`:

```jsx static
import { MediaPlayerControls } from '@cassette/player';
import { PlayerContextProvider, playerContextFilter } from '@cassette/core';

import '@cassette/player/dist/css/cassette-player.css';

const playlist = /* unchanged */;

function PlaybackRateControl() {/* unchanged */}

function PlaylistMenu({ playlist, activeTrackIndex, onSelectTrackIndex }) {
  return (
    <ol>
      {playlist.map((track, i) => {
        const playing = activeTrackIndex === i;
        return (
          <li key={track.title}>
            {playing && <strong>{track.title} (playing)</strong>}
            {!playing &&
              <button
                onClick={() => onSelectTrackIndex(i)}
              >
                {track.title}
              </button>}
          </li>
        );
      })}
    </ol>
  );
}

PlaylistMenu = playerContextFilter(PlaylistMenu, [
  'playlist',
  'activeTrackIndex',
  'onSelectTrackIndex'
]);

ReactDOM.render(
  <PlayerContextProvider {/* props unchanged */}>
    <MediaPlayerControls {/* props unchanged */} />
    <div>
      <h3>Select a track:</h3>
      <PlaylistMenu />
    </div>
  </PlayerContextProvider>,
  document.getElementById('app')
);
```

<div class="screenshot_wrapper">
  <img src="img/playlist_menu.png">
  <div class="notice"></div>
</div>

Hopefully that brief overview helps you hit the ground running with Cassette!

## What else can I build with Cassette?

Obviously the quick tutorial above didn't produce the most glamorous UI, but Cassette is already being used in real life apps and libraries, such as...

### [rexmort.us](https://rexmort.us)
<a href="https://rexmort.us">
  <img width="300" src="img/rexmortus.png">
</a>

A site showcasing creative content, including several podcast series which can be listened to while navigating the rest of the site.

#### How is Cassette used?
The `MediaPlayerControls` UI from the `@cassette/player` package can be seen at the bottom of the page, featuring the included play/pause, mute toggle, and media progress controls (with some custom CSS styles applied). `PlayerContextProvider` wraps the whole page, and is used on each of the podcast pages where we can select a podcast to play, and see an indication of which podcast is playing currently.

### [OwlTail](https://owltail.com)
<a href="https://owltail.com">
  <img width="300" src="img/owltail.png">
</a>

A web app where users can explore curated popular podcasts and schedule queues of podcasts to listen to in the browser.

#### How is Cassette used?
Instead of using Cassette's default UI, OwlTail's player UI is all custom-built. It relies on a page-level `PlayerContextProvider` to provide media data and functionality. The control UI at the bottom of the screen features some controls which don't even exist in the default Cassette UI, like a playback rate control, and buttons for skipping back and forward by 30 second intervals; even though Cassette doesn't provide this UI, its callbacks make it simple to implement this sort of behavior. The progress bar, although featuring custom UI, relies on the `MediaProgressBar` helper from the `@cassette/components` package, which is designed to work well with both mouse and touch devices.

The UI in the rest of the app is synced with the player via the `PlayerContextProvider` wrapper, so that the currently-playing podcast will always display as such when encountered in the queue or a podcast listing.

### [benwiley.org](https://benwiley.org)
<a href="https://benwiley.org">
  <img width="300" src="img/benwiley.org.png">
</a>

A personal site featuring a portfolio of work, including some video game soundtrack pieces, which can be sampled while browsing the rest of the site's content.

#### How is Cassette used?
This site uses `@cassette/player`'s `PlayPauseButton` and `ForwardSkipButton` components, while wrapping the `MediaProgressBar` helper from `@cassette/components` for a custom progress navigation UI. Because the `PlayerContextProvider` wraps the whole page, we can integrate playback controls inline on the music portfolio page, dynamically displaying a media progress control below the description for the currently-selected track.

## In conclusion

Cassette offers a solid media player experience up front and some powerful capabilities for building a custom media player UI. To learn more, you can [jump into the docs](https://benwiley4000.github.io/cassette/styleguide/)!
