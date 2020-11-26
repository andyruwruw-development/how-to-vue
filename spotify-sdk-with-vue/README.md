# Spotify Web Playback SDK with Vue.js

Spotify offers a [web playback SDK](https://developer.spotify.com/documentation/web-playback-sdk/quick-start/) to allow you to turn your website into a device the user can cast their playback to.

The SDK is accessed via a script tag and requires initialization.

# Setting Up

Create the project:

```
vue create .
```
For this implementation, create a [Vue.js](https://vuex.vuejs.org/) project with vuex enabled.

Add the script to your public/index.html:
```
<script src="https://sdk.scdn.co/spotify-player.js"></script>
```
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title><%= htmlWebpackPlugin.options.title %></title>
    
    <script src="https://sdk.scdn.co/spotify-player.js"></script>
  </head>
...
```

This will throw an error without being initialized.

I like to organize my state into modules:
```
src
|-- store
    |-- index.js
    |-- modules
        |-- player.js
```

Within index.js import the module:

```
import Vue from 'vue';
import Vuex from 'vuex';

import player from './modules/player';

Vue.use(Vuex);

export default new Vuex.Store({
  modules: {
    player,
  },
});
```

Within player.js provide an initialization method:
```
const state = {
  deviceID: null,
  playback: null,
  playbackContext: null,
};

const mutations = {
  setDeviceID(state, deviceID) {
    state.deviceID = deviceID;
  },
  setPlayback(state, playback) {
    state.playback = playback;
  },
  setPlaybackContext(state, playback) {
    state.playbackContext = playback;
  },
};

const actions = {
  init: async function({ commit, rootGetters, dispatch }) {
    window.onSpotifyWebPlaybackSDKReady = () => {};

    async function waitForSpotifyWebPlaybackSDKToLoad() {
      return new Promise((resolve) => {
        if (window.Spotify) {
          resolve(window.Spotify);
        } else {
          window.onSpotifyWebPlaybackSDKReady = () => {
            resolve(window.Spotify);
          };
        }
      });
    }

    async function waitUntilUserHasSelectedPlayer(sdk) {
      return new Promise((resolve) => {
        let interval = setInterval(async () => {
          let state = await sdk.getCurrentState();
          if (state !== null) {
            resolve(state);
            clearInterval(interval);
          }
        });
      });
    }

    (async () => {
      const { Player } = await waitForSpotifyWebPlaybackSDKToLoad();
      const token = rootGetters['auth/getAccessToken'];

      // eslint-disable-next-line
      const player = new Player({
        name: 'Boid Boogie',
        getOAuthToken: (cb) => {
          cb(token);
        }
      });

      // Error handling
      player.addListener('initialization_error', ({ message }) => {
        console.error('initialization_error', message);
      });

      player.addListener('authentication_error', ({ message }) => {
        console.error('authentication_error', message);
        //dispatch('auth/login', null, { root: true });
      });

      player.addListener('account_error', ({ message }) => {
        console.error('account_error', message);
      });

      player.addListener('playback_error', ({ message }) => {
        console.error('playback_error', message);
      });

      // Playback status updates
      player.addListener('player_state_changed', (state) => {
        if (state) {
          dispatch('setPlaybackContext', state);
          dispatch('setPlayback');
          dispatch('track/updateTrack', state, { root: true });
        }
      });

      // Ready
      player.addListener('ready', ({ device_id }) => {
        commit('setDeviceID', device_id);

        api.spotify.player.transferUsersPlayback([device_id], true);
      });

      // Not Ready
      player.addListener('not_ready', ({ device_id }) => {
        console.log('Device ID has gone offline', device_id);
      });

      // Connect to the player!
      let connected = await player.connect();

      if (connected) {
        await waitUntilUserHasSelectedPlayer(player);
      }
    })();
  },

  async setPlayback({ commit }) {
    try {
      const response = await api.spotify.player.getCurrentPlayback();
      commit('setPlayback', response.data);
    } catch (e) {
      console.log(e);
    }
  },

  setPlaybackContext({ commit }, context) {
    commit('setPlaybackContext', context);
  },
};

const module = {
  namespaced: true,
  state,
  getters,
  mutations,
  actions
};

export default module;
```

Now within App.vue run the init() function:

```
<template>
  <div id="app">
  </div>
</template>

<script>
import { mapActions } from 'vuex';

export default {
  name: 'App',
  methods: {
    ...mapActions('player', [
      'init',
    ]),
  },
  created() {
    this.init();
  },
};
</script>
```