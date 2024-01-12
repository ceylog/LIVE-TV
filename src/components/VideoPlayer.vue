<template>
    <div>
        <video ref="videoPlayer" class="video-js vjs-default-skin"></video>
    </div>
</template>

<script setup>
import videojs from 'video.js';
import 'video.js/dist/video-js.min.css'

import { onMounted, onUnmounted, ref, watch } from 'vue'
const props = defineProps({
    options: {
        type: Object,
        default: {
            autoplay: true,
            muted: true,
            controls: true,
            loop: false,
            sources: [{ src: 'https://live-play.cctvnews.cctv.com/cctv/merge_cctv13.m3u8', type: 'application/x-mpegURL' }],
            language: 'zh-CN',
            fluid: true,
            liveui:true,
        }
    }
});
const videoPlayer = ref(null);

let player = null;
onMounted(() => {
    player = videojs(videoPlayer.value, props.options, () => { console.log("ready..") });
});
onUnmounted(() => {
    if (player) {
        player.dispose();
    }
});
watch(props.options, async (nenv) => {
    player.muted(nenv.mouted);
    player.src(nenv.sources[0].src);
    player.play()
});
</script>