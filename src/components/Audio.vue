<template>
  <div class="player" :style="{'background-color': bgColor}">
    <!--在style里加上这一条会让进度条与鼠标不同步-->
    <!--, 'transform':'scale('+(parseInt(width/190))+')'-->
    <div class="player-panel">
      <div class="player-panel-details">
        <div class="player-panel-title text-ellipsis" v-cloak :title="currentAudio.songname">{{currentAudio.songname}}
        </div>
        <div class="player-panel-time">
          <span class="player-time-current">{{current|time}}</span>
          <span class="player-time-total"> of {{duration|time}}</span>
        </div>
      </div>
      <div class="player-panel-volume" @click="changeVolume($event)" ref="volume-control">
        <i class="player-volume-control" :volume-level="(6-index)*2" v-for="index in 5" :key="index"
          :class="{'backBlack': 6-volume/2 <= index}"></i>
        <i class="icon-headphones" @click="noVolume()"></i>
        <i class="no-volume" v-show="!volume"></i>
      </div>
    </div>
    <div class="player-controls">
      <div class="player-buttons">
        <div class="player-btn">
          <span class="player-prev" @click="prevClick()"></span>
        </div>
        <div class="player-btn">
          <span @click="toggleStatus()" :class="isPause?'player-play' : 'player-pause' "></span>
        </div>
        <div class="player-btn">
          <span class="player-next" @click="nextClick()"></span>
        </div>
      </div>
      <div style="clear: both"></div>
      <div class="player-range">
        <mtRange :thumbPosition="currentPercent" ref="mr">
        </mtRange>
      </div>
    </div>
    <audio :src="currentAudio.url" :autoplay="autoPlay" :loop="loop" id="player" ref="player" @ended="nextClick()"
      @timeupdate="timeChange()" @loadeddata="getDuration()"></audio>
  </div>
</template>



<script>
  import './style.css'
  import mtRange from './MtRange.vue'
  export default {
    props: {
      autoPlay: {
        type: Boolean,
        default: false
      },
      loop: {
        type: Boolean,
        default: false
      },
      width: {
        type: String,
        default: '300'
      },
      bgColor: {
        type: String,
        default: '#000000'
      },
      songs: {
        type: Array,
        default: [{
          "url": "http://fs.w.kugou.com/7840167216f9b80284d2bb2a9bd9554b/58ac0322/G076/M0A/0C/1D/7IYBAFgu5wmAOS2gAEuViOk9tuk748.mp3",
          "songname": "林俊杰-你的唯一"
        }]
      }

    },
    components: {
      mtRange
    },
    data() {
      return {
        RANGE_WIDTH: 177,
        thumbWidth: 4,
        current: 0,
        duration: 0,
        oldVolume: 0,
        volume: 2,
        audioIndex: 0,
        current: 0,
        duration: 0,
        isPause: !this.autoPlay,
        thumbPosition: 0,
      }
    },
    computed: {
      currentAudio() {
        return this.songs[this.audioIndex];
      },
      // 进度条与音乐时间双向绑定
      currentPercent() {
        this.thumbPosition = parseInt(this.current / this.duration * (this.RANGE_WIDTH - this.thumbWidth));
        if (this.thumbPosition > this.RANGE_WIDTH - this.thumbWidth) {
          this.thumbPosition = this.RANGE_WIDTH - this.thumbWidth;
        }
        //console.log(this.thumbPosition)
        return this.thumbPosition;
      },

    },
    // 初始音量
    mounted() {
      if (this.volume) {
        this.$refs.player.volume = this.volume / 10;
      } else {
        this.$refs.player.muted = true;
      }

      var mr = this.$refs.mr;
      var content = mr.content;
      var thumb = mr.thumb;
      //var progress = mr.progress;
      var thumbleft = 0;
      this.thumbWidth = parseInt(thumb.clientWidth / 2);
      //console.log(this.thumbWidth)
      this.RANGE_WIDTH = content.clientWidth;

      var _this = this;
      thumb.onmousedown = function (event) {
        var event = event;
        var leftVal = event.clientX - this.offsetLeft;
        var clientX1 = event.clientX;

        var that = this;
        document.onmousemove = function (event) {
          var event = event;

          thumbleft = event.clientX - leftVal;
          //thumbwidth = parseInt(that.style.width / 2)
          //console.log(that.style.width);
          //console.log(thumbleft);
          if (thumbleft < 0) {
            thumbleft = 0;
          } else if (thumbleft > content.offsetWidth - _this.thumbWidth) {

            thumbleft = content.offsetWidth - _this.thumbWidth;
          }
          _this.$refs.player.currentTime = _this.duration * thumbleft / (_this.RANGE_WIDTH - _this.thumbWidth);

        }
        //防止选择内容--当拖动鼠标过快时候，弹起鼠标，bar也会移动，修复bug
        window.getSelection ? window.getSelection().removeAllRanges() : document.selection.empty();
        document.onmouseup = function () {
          document.onmousemove = document.onmouseup = null;
        }
        return false;
      }
    },
    methods: {
      //音量调节
      changeVolume(event) {
        if (event.target.classList.contains('player-volume-control')) {
          this.volume = parseInt(event.target.getAttribute('volume-level'));
          this.$refs.player.volume = this.volume / 10;

        }
      },
      // 静音
      noVolume() {
        if (this.volume) {
          this.oldVolume = this.volume;
          this.volume = 0;
        } else if (this.oldVolume) {
          this.volume = this.oldVolume;
        } else {
          this.volume = 2;
        }
        // 点击时静音
        this.$refs.player.muted = !this.$refs.player.muted;
      },
      // 上一首
      prevClick() {
        if (this.audioIndex == 0) {
          this.audioIndex = this.songs.length - 1;
        } else {
          this.audioIndex--;
        }
        //点击上一首时由暂停变为播放
        if (this.isPause) {
          this.toggleStatus();
        }
        this.thumbPosition = 0;
      },
      //下一首
      nextClick() {
        if (this.audioIndex == this.songs.length - 1) {
          this.audioIndex = 0;
        } else {
          this.audioIndex++;
        }
        //点击下一首时由暂停变为播放
        if (this.isPause) {
          this.toggleStatus();
        }
        this.thumbPosition = 0;
      },
      //播放暂停与继续
      toggleStatus() {
        var player = this.$refs.player;
        this.isPause ? player.play() : player.pause()
        this.isPause = !this.isPause;
        this.current = this.$refs.player.currentTime;
        //console.log(this.current);
      },
      // 歌曲目前播放时间
      timeChange() {
        this.current = this.$refs.player.currentTime;
      },
      // 歌曲总时间
      getDuration() {
        this.duration = this.$refs.player.duration;
      },
      changeRange(event) {
        var offset = event.offsetX;
        //console.log(offset);
        if (this.max && offset < this.RANGE_WIDTH) {
          var currentClick = Math.floor(offset * this.max /
            this.RANGE_WIDTH);
          this.min = currentClick;
          if (offset > 177) {
            this.thumbPosition = 177;
          } else {
            this.thumbPosition = offset;
          }
        }
      },

    },
    // 自定义歌曲时间插件
    filters: {
      time(value) {
        //分钟
        var minute = Math.floor(value / 60);
        if (minute < 10) {
          minute = '0' + minute;
        }
        //秒钟
        var second = Math.floor(value % 60);
        if (second < 10) {
          second = '0' + second;
        }
        return minute + ":" + second;
      }
    }
  }

</script>
