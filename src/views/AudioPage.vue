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

export default {
  name: 'AudioPage',
  data() {
    return {
      analyser: null,
      barWidth: null,
      barHeight: null,
      bufferLength: null,
      canvasHeight: null,
      canvasWidth: null,
      context: null,
      frequencyDataList: null,
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

      this.player = videoJs(document.getElementById('videoPlayer'), this.options);

      this.player.on(['loadeddata'], () => {
        // 동영상 데이터 호출하고 미디어 스트림 분석 및 오디오 데이터화
        const audioContext = new AudioContext();
        const stream = this.player.el_.children[0].captureStream();
        const audioSrc = audioContext.createMediaStreamSource(stream);

        this.analyser = audioContext.createAnalyser();
        audioSrc.connect(this.analyser);
        this.analyser.fftSize = 256;
        this.bufferLength = this.analyser.frequencyBinCount;
        // 주파수 데이터 초기화
        this.frequencyDataList = new Uint8Array(this.bufferLength);

        // 캔버스 초기화 및 시각화 준비
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

      // video.js의 hls 재생 이슈로 플레이어 제거 및 생성
      this.disposePlayer();

      const el = document.createElement('video');

      el.setAttribute('id', 'videoPlayer');
      el.setAttribute('class', 'video-js vjs-16-9');
      el.setAttribute('crossOrigin', 'anonymous');

      this.$refs.videoContainer.appendChild(el);

      this.initializePlayer();
    },
    renderFrame() {
      // renderFrame 애니메이션 업데이트 호출
      // 플레이어 제거 시나리오 적용 시 cancelAnimationFrame 호출 필요
      // https://developer.mozilla.org/ko/docs/Web/API/Window/cancelAnimationFrame
      requestAnimationFrame(this.renderFrame);

      let x = 0;

      // 주파수 데이터 복사
      this.analyser.getByteFrequencyData(this.frequencyDataList);

      this.context.fillStyle = '#000000';
      this.context.fillRect(0, 0, this.canvasWidth, this.canvasHeight);

      for (let i = 0; i < this.bufferLength; i++) {
        this.barHeight = this.frequencyDataList[i];

        const r = this.barHeight + (25 * (i / this.bufferLength));
        const g = 250 * (i / this.bufferLength);
        const b = 50;

        this.context.fillStyle = `rgb(${r},${g},${b})`;
        this.context.fillRect(x, this.canvasHeight - this.barHeight, this.barWidth, this.barHeight);

        x += this.barWidth + 1;
      }
    },
  },
};
</script>
<style scoped>
#container {
  display: flex; flex-direction: column; background-color: #1d1d1d; height: 100vh;
}
#videoPart {
  width: 70%; border-right: 1px solid #6b6b6b;
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
</style>
