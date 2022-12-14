<template>
  <div id="container">
    <div style="display: flex;">
      <div
        id="videoPart"
        ref="videoContainer"
      >
        <video
          id="videoPlayer"
          class="video-js vjs-16-9"
          crossOrigin="anonymous"
        />
        <div
          v-if="isShowVolumeList"
          id="volumeList"
        >
          <div
            v-for="(item, index) of volumeList"
            :key="`volumeList_${index}`"
            class="volume-bar"
            :style="{
              'height': `${item}%`,
              'background-image': `linear-gradient(0deg, hsl(280deg 100% 20%) 0%, hsl(350deg 100% 50%) 50%, hsl(${item * 0.5}deg 100% 50%) 100%)`}"
          />
        </div>
      </div>
      <div id="samplePart">
        <div class="sample-title">
          Sample List
        </div>
        <div id="sampleList">
          <div
            v-for="(item, index) of sampleList"
            :key="`sample_${index}`"
            class="sample-item"
            :class="{'focus': index === sampleIndex}"
            @click="onClickSample(index)"
          >
            {{ item.name }}
          </div>
        </div>
      </div>
    </div>
    <div id="canvasPart">
      <canvas ref="canvas" />
    </div>
  </div>
</template>

<script>
import videoJs from 'video.js';

const VOLUME_LIST = 10;

export default {
  name: 'AudioPage',
  data() {
    return {
      analyser: null,
      barWidth: null,
      barHeight: null,
      bufferLength: null,
      volumeList: Array(VOLUME_LIST).fill(0),
      canvasHeight: null,
      canvasWidth: null,
      context: null,
      fftSize: 256,
      frequencyDataList: null,
      isShowVolumeList: JSON.parse(localStorage.getItem('isShowVolumeList')) && true,
      options: {
        autoplay: true,
        controls: true,
        muted: false,
        sources: [],
      },
      player: null,
      sampleIndex: 0,
      sampleList: [
        {
          name: 'Parkour.m3u8',
          src: 'https://bitdash-a.akamaihd.net/content/MI201109210084_1/m3u8s/f08e80da-bf1d-4e3d-8899-f0f6155f6efa.m3u8',
          type: 'application/x-mpegURL',
        }, {
          name: 'Sintel.m3u8',
          src: 'https://bitdash-a.akamaihd.net/content/sintel/hls/playlist.m3u8',
          type: 'application/x-mpegURL',
        }, {
          name: 'Bipbop.m3u8',
          src: 'https://d2zihajmogu5jn.cloudfront.net/bipbop-advanced/bipbop_16x9_variant.m3u8',
          type: 'application/x-mpegURL',
        }, {
          name: 'Playhouse-VR.m3u8',
          src: 'https://bitmovin-a.akamaihd.net/content/playhouse-vr/m3u8s/105560.m3u8',
          type: 'application/x-mpegURL',
        }, {
          name: 'Tears of Steel.m3u8',
          src: 'http://content.jwplatform.com/manifests/vM7nH0Kl.m3u8',
          type: 'application/x-mpegURL',
        }, {
          name: 'Animal.m3u8',
          src: 'http://playertest.longtailvideo.com/adaptive/wowzaid3/playlist.m3u8',
          type: 'application/x-mpegURL',
        }, {
          name: 'Skate Phantom.m3u8',
          src: 'http://sample.vodobox.net/skate_phantom_flex_4k/skate_phantom_flex_4k.m3u8',
          type: 'application/x-mpegURL',
        }, {
          name: 'Apple.m3u8',
          src: 'http://qthttp.apple.com.edgesuite.net/1010qwoeiuryfg/sl.m3u8',
          type: 'application/x-mpegURL',
        }, {
          name: 'BigBuckBunny.mp4',
          src: 'http://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4',
          type: 'video/mp4',
        }],
    };
  },
  mounted() {
    this.initializePlayer();
  },
  methods: {
    disposePlayer() {
      this.player.dispose();
    },
    initializePlayer() {
      this.options.sources = [];
      this.options.sources.push(this.sampleList[this.sampleIndex]);

      this.player = videoJs(document.getElementById('videoPlayer'), this.options, () => {
        this.appendToggleButton(this.player);
      });

      this.player.on(['loadeddata'], () => {
        // ????????? ????????? ???????????? ????????? ????????? ?????? ??? ????????? ????????????
        const audioContext = new AudioContext();
        const stream = this.player.el_.children[0].captureStream();
        const audioSrc = audioContext.createMediaStreamSource(stream);

        this.analyser = audioContext.createAnalyser();
        audioSrc.connect(this.analyser);
        this.analyser.fftSize = this.fftSize;
        this.bufferLength = this.analyser.frequencyBinCount;
        // ????????? ????????? ?????????
        this.frequencyDataList = new Uint8Array(this.bufferLength);

        // ????????? ????????? ??? ????????? ??????
        const canvas = this.$refs.canvas;
        const rect = canvas.getBoundingClientRect();

        this.canvasWidth = canvas.width = rect.width;
        this.canvasHeight = canvas.height = rect.height;

        this.context = canvas.getContext('2d');
        this.barWidth = (canvas.width / this.bufferLength) * 2.5;

        this.renderFrame();
      });
    },
    onClickSample(index) {
      this.sampleIndex = index;

      // video.js??? hls ?????? ????????? ???????????? ?????? ??? ??????
      this.disposePlayer();

      const el = document.createElement('video');

      el.setAttribute('id', 'videoPlayer');
      el.setAttribute('class', 'video-js vjs-16-9');
      el.setAttribute('crossOrigin', 'anonymous');

      this.$refs.videoContainer.appendChild(el);

      this.initializePlayer();
    },
    renderFrame() {
      // renderFrame ??????????????? ???????????? ??????
      // ???????????? ?????? ???????????? ?????? ??? cancelAnimationFrame ?????? ??????
      // https://developer.mozilla.org/ko/docs/Web/API/Window/cancelAnimationFrame
      requestAnimationFrame(this.renderFrame);

      let x = 0;

      // ????????? ????????? ??????
      this.analyser.getByteFrequencyData(this.frequencyDataList);

      this.context.fillStyle = '#000000';
      this.context.fillRect(0, 0, this.canvasWidth, this.canvasHeight);

      for (let i = 0; i < this.bufferLength; i++) {
        this.barHeight = this.frequencyDataList[i];
        // ?????? ????????? ?????????
        if (i < VOLUME_LIST) this.volumeList[i] = Math.round((this.barHeight / this.fftSize) * 100);

        const r = this.barHeight + (25 * (i / this.bufferLength));
        const g = 250 * (i / this.bufferLength);
        const b = 50;

        this.context.fillStyle = `rgb(${r},${g},${b})`;
        this.context.fillRect(x, this.canvasHeight - this.barHeight, this.barWidth, this.barHeight);

        x += this.barWidth + 1;
      }
    },
    appendToggleButton() {
      const controlBarEl = this.player.controlBar.el();
      const buttonEl = document.createElement('button');
      const divEl = document.createElement('div');
      const iconEl = Array(3).fill(0).map(() => document.createElement('div'));
      buttonEl.classList.add('vjs-visualizer-toggle', 'vjs-control', 'vjs-button');
      divEl.classList.add('vjs-visualizer-toggle-icon');
      if (!JSON.parse(this.isShowVolumeList)) divEl.classList.add('off');

      iconEl.forEach((item) => {
        divEl.appendChild(item);
      });
      buttonEl.appendChild(divEl);
      buttonEl.onclick = () => {
        this.isShowVolumeList = !JSON.parse(this.isShowVolumeList);
        localStorage.setItem('isShowVolumeList', this.isShowVolumeList.toString());
        divEl.classList.toggle('off');
      };
      controlBarEl.appendChild(buttonEl);
    },
  },
};
</script>
<style scoped>
#container {
  display: flex; flex-direction: column; background-color: #1d1d1d; height: 100vh;
}
#videoPart {
  position: relative; width: 70%; border-right: 1px solid #6b6b6b;
}
#videoPlayer {
  width: 100%; height: 100%;
}
#samplePart {
  width: 30%; padding: 24px; user-select: none;
}
#sampleList {
  display: flex; flex-direction: column; align-items: flex-start; padding: 24px; gap: 24px;
}
#canvasPart {
  width: 100%; min-height: 0; flex-grow: 1; flex-shrink: 1; border-top: 1px solid #6b6b6b;
}
#canvasPart canvas {
  width: 100%; height: 100%;
}
.sample-title {
  width: 100%; padding-bottom: 24px; border-bottom: 1px solid #6b6b6b;
  font-weight: 700; font-size: 20px; color: white;
}
.sample-item {
  font-weight: 700; font-size: 20px; cursor: pointer; color: white;
}
.sample-item.focus {
  color: #00f700;
}
#volumeList {
  position: absolute;
  display: flex;
  align-items: flex-end;
  gap: 4px;
  height: 90%;
  right: 8px;
  bottom: 5%;
  background: rgba(255, 255, 255, 0.3);
  box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.37);
  backdrop-filter: blur( 9px );
  -webkit-backdrop-filter: blur( 9px );
  border-radius: 8px;
  padding: 8px;
  z-index: 1;
}
.volume-bar {
  width: 10px;
  border-radius: 8px;
  box-shadow: 5px 5px 5px 0 rgba(0, 0, 0, 0.3);
  filter: drop-shadow(5px 5px 5px rgba(0, 0, 0, 0.3));
}
</style>
