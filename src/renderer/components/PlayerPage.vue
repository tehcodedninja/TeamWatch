<template>
  <div class="wrapper">
    <div class="video-content">
          <video-js id="my_video_1" class="video-js vjs-default-skin vjs-big-play-centered" controls preload="auto" autoplay>
          </video-js>
    </div>
    <md-snackbar md-position="center" :md-duration="duration" :md-active.sync="showSnackbar" md-persistent>
      <span>Stream is not available yet.</span>
    </md-snackbar>
  </div>
</template>

<script>

  var moment = require('moment-timezone');

  // import videojs from 'video.js'
  var videojs = require('video.js');

  export default {
    name: 'player',
    
    data: () => ({
      duration: 8000,
      player: null,
      quailty: "360",
      showSnackbar: false
    }),
    mounted: function() {
      this.player = videojs('my_video_1',);
      
      // Listen for async-reply message from main process
      this.$electron.ipcRenderer.on('stream-url', (event, arg) => {
          var start = moment.tz(arg[0].gameDate, "America/Danmarkshavn");
          var gamedate = start.clone().tz("America/New_York").format("YYYY-MM-DD");

          this.$http.get("http://freegamez.ga/getM3U8.php?league=nhl&date="+gamedate+"&id="+arg[1].mediaPlaybackId+"&cdn=akc").then((res) => {
            if(res.data == "Not available yet")
            {
              this.showSnackbar = true;
            }
            else {
              this.player.src({ type: "application/x-mpegURL", src: "http://localhost:3000/m3u8?url="+encodeURIComponent(res.data) });
            }
          });
          
      });
    }
  }
</script>

<style>
  @import "~video.js/dist/video-js.css";

  div.app, div.page-container, div.md-app {
    height: 100vh;
  }

  div.wrapper, div.video-content {
    height: 100%;
  }

  .md-app {
      max-height: 100vh;
      border: 1px solid rgba(#000, .12);
    }

  .electron-drag {
    -webkit-app-region: drag;
  }

  .system-icons {
    -webkit-app-region: no-drag;
  }
  

  #my_video_1 {
    width: 100%; 
    height: 100%; 
  }
</style>
