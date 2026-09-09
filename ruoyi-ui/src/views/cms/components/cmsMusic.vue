<template>
  <el-row :gutter="20">
    <el-col :sm="2" class="hidden-xs-only" style="opacity:0;">左侧占位</el-col>
    <el-col :xs="24" :sm="15">
      <el-card style="background-color: rgba(255,255,255,0.9)" class="left-item">
        <div slot="header">
          <span><i class="el-icon-service"></i> 音乐播放器</span>
        </div>

        <!-- 当前播放 -->
        <div class="music-playing">
          <div class="cover-wrapper">
            <div class="cover-img" :class="{ spinning: isPlaying }">
              <img :src="currentSong.cover" alt="封面">
            </div>
          </div>
          <div class="song-info">
            <div class="song-title">{{ currentSong.title }}</div>
            <div class="song-artist">{{ currentSong.artist }}</div>
          </div>

          <!-- 进度条 -->
          <div class="progress-bar">
            <span class="time">{{ formatTime(currentTime) }}</span>
            <el-slider class="progress-slider" v-model="progress" :show-tooltip="false" @change="changeProgress" :height="'4px'" />
            <span class="time">{{ formatTime(duration) }}</span>
          </div>

          <!-- 控制按钮 -->
          <div class="controls">
            <i class="ctrl-btn el-icon-caret-left" @click="prevSong"></i>
            <i class="ctrl-btn play-btn" :class="isPlaying ? 'el-icon-video-pause' : 'el-icon-video-play'" @click="togglePlay"></i>
            <i class="ctrl-btn el-icon-caret-right" @click="nextSong"></i>
          </div>

        </div>
      </el-card>
    </el-col>

    <el-col :xs="24" :sm="5">
      <!-- 播放列表 -->
      <el-card style="background-color: rgba(255,255,255,0.9)" class="right-item">
        <div slot="header">
          <b><i class="el-icon-menu"></i> 播放列表</b>
        </div>
        <div
          v-for="(song, index) in songList"
          :key="index"
          class="playlist-item"
          :class="{ active: currentIndex === index }"
          @click="playSong(index)"
        >
          <div class="item-index">
            <span v-if="currentIndex !== index">{{ index + 1 }}</span>
            <i v-else class="el-icon-video-play"></i>
          </div>
          <div class="item-info">
            <div class="item-title">{{ song.title }}</div>
            <div class="item-artist">{{ song.artist }}</div>
          </div>
        </div>
      </el-card>
    </el-col>
    <el-col :sm="2" class="hidden-xs-only" style="opacity:0;">右侧占位</el-col>

    <audio ref="audio" :src="currentSong.src" @timeupdate="onTimeUpdate" @loadedmetadata="onLoaded" @ended="nextSong" @play="isPlaying = true" @pause="isPlaying = false" />
  </el-row>
</template>

<script>
  export default {
    name: 'cmsMusic',
    data() {
      return {
        isPlaying: false,
        currentIndex: 0,
        currentTime: 0,
        duration: 0,
        volume: 60,
        progress: 0,
        songList: [
          {
            title: '春之声',
            artist: '未知艺术家',
            cover: require('@/assets/logo/logo.png'),
            src: ''
          },
          {
            title: '夏之恋',
            artist: '未知艺术家',
            cover: require('@/assets/logo/logo.png'),
            src: ''
          },
          {
            title: '秋之风',
            artist: '未知艺术家',
            cover: require('@/assets/logo/logo.png'),
            src: ''
          }
        ]
      }
    },
    computed: {
      currentSong() {
        return this.songList[this.currentIndex]
      }
    },
    watch: {
      volume(val) {
        if (this.$refs.audio) {
          this.$refs.audio.volume = val / 100
        }
      }
    },
    mounted() {
      if (this.$refs.audio) {
        this.$refs.audio.volume = this.volume / 100
      }
    },
    methods: {
      togglePlay() {
        const audio = this.$refs.audio
        if (this.isPlaying) {
          audio.pause()
        } else {
          if (audio.src) {
            audio.play()
          }
        }
      },
      playSong(index) {
        this.currentIndex = index
        this.$nextTick(() => {
          const audio = this.$refs.audio
          if (audio.src) {
            audio.play()
          }
        })
      },
      prevSong() {
        this.currentIndex = (this.currentIndex - 1 + this.songList.length) % this.songList.length
        this.$nextTick(() => {
          const audio = this.$refs.audio
          if (audio.src) {
            audio.play()
          }
        })
      },
      nextSong() {
        this.currentIndex = (this.currentIndex + 1) % this.songList.length
        this.$nextTick(() => {
          const audio = this.$refs.audio
          if (audio.src) {
            audio.play()
          }
        })
      },
      onTimeUpdate(e) {
        this.currentTime = e.target.currentTime
        if (this.duration > 0) {
          this.progress = (this.currentTime / this.duration) * 100
        }
      },
      onLoaded(e) {
        this.duration = e.target.duration
      },
      changeProgress(val) {
        const audio = this.$refs.audio
        audio.currentTime = (val / 100) * this.duration
        this.progress = val
      },
      changeVolume(val) {
        if (this.$refs.audio) {
          this.$refs.audio.volume = val / 100
        }
      },
      formatTime(seconds) {
        if (!seconds || isNaN(seconds) || !isFinite(seconds)) return '00:00'
        const min = Math.floor(seconds / 60)
        const sec = Math.floor(seconds % 60)
        return (min < 10 ? '0' + min : min) + ':' + (sec < 10 ? '0' + sec : sec)
      }
    }
  }
</script>

<style scoped>
  .music-playing {
    text-align: center;
    padding: 10px 0;
  }

  .cover-wrapper {
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
  }

  .cover-img {
    width: 150px;
    height: 150px;
    border-radius: 50%;
    overflow: hidden;
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
  }

  .cover-img.spinning {
    animation: spin 10s linear infinite;
  }

  .cover-img img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  @keyframes spin {
    from { transform: rotate(0deg); }
    to { transform: rotate(360deg); }
  }

  .song-info {
    margin-bottom: 20px;
  }

  .song-title {
    font-size: 20px;
    font-weight: 600;
    color: #303133;
    margin-bottom: 6px;
  }

  .song-artist {
    font-size: 14px;
    color: #909399;
  }

  .progress-bar {
    display: flex;
    align-items: center;
    margin-bottom: 16px;
    padding: 0 10px;
  }

  .progress-bar .time {
    font-size: 12px;
    color: #909399;
    flex-shrink: 0;
    width: 40px;
    text-align: center;
    line-height: 1;
  }

  .progress-bar .progress-slider {
    flex: 1;
    margin: 0 8px;
  }

  .progress-bar .progress-slider /deep/ .el-slider {
    margin: 0;
  }

  .progress-bar .progress-slider /deep/ .el-slider__runway {
    margin: 0;
  }

  .controls {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 20px;
  }

  .ctrl-btn {
    font-size: 28px;
    color: #606266;
    cursor: pointer;
    margin: 0 20px;
    transition: color 0.3s;
  }

  .ctrl-btn:hover {
    color: #409EFF;
  }

  .ctrl-btn.play-btn {
    font-size: 44px;
    color: #409EFF;
  }

  .volume-bar {
    display: flex;
    align-items: center;
    padding: 0 20px;
    color: #909399;
  }

  .playlist-item {
    display: flex;
    align-items: center;
    padding: 10px 12px;
    border-radius: 6px;
    cursor: pointer;
    transition: background 0.2s;
    margin-bottom: 4px;
  }

  .playlist-item:hover {
    background: #F5F7FA;
  }

  .playlist-item.active {
    background: #ECF5FF;
  }

  .playlist-item.active .item-title {
    color: #409EFF;
  }

  .item-index {
    width: 28px;
    height: 28px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    font-size: 14px;
    color: #909399;
  }

  .item-info {
    margin-left: 10px;
    overflow: hidden;
  }

  .item-title {
    font-size: 14px;
    color: #303133;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .item-artist {
    font-size: 12px;
    color: #909399;
    margin-top: 2px;
  }
</style>
