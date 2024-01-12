<template>
  <v-app>
    <v-app-bar>
      <v-app-bar-nav-icon @click="drawer=!drawer"></v-app-bar-nav-icon>
      <v-app-bar-title>LIVE TV</v-app-bar-title>
      <v-btn icon='mdi-white-balance-auto' @click="toggleTheme"></v-btn>
    </v-app-bar>
    <v-navigation-drawer v-model="drawer">
    <v-list>
            <v-list-item v-for="b in sl" :key="b.name" @click="slist(b.url)" :title="b.name" :subtitle="b.url" />
            <v-divider></v-divider> 
            <v-divider></v-divider> 
            <v-divider></v-divider> 
            <template v-for="(v, k) in cl.map" :key="k">
              <v-list-item :title="v[0]"/>
              <v-divider/>
              <v-list-item v-for="c in v[1]" :key="c.name" :title="c.title" @click="switchch(c.url)"></v-list-item>
              </template>
          </v-list>
    </v-navigation-drawer>
    <v-main>
      <video-player :options="options"></video-player>
    </v-main>
  </v-app>
</template>

<script setup>
  //
import VideoPlayer from './VideoPlayer.vue';
import { ref,reactive } from 'vue'
import { useTheme } from 'vuetify';
const drawer = ref(false)

const sl = [{ url: "https://mirror.ghproxy.com/https://raw.githubusercontent.com/Ftindy/IPTV-URL/main/IPTV.m3u", name: 'IPTV' },
{
  url: "https://mirror.ghproxy.com/https://raw.githubusercontent.com/YanG-1989/m3u/main/Adult.m3u"
  , name: 'Adult'
},
{ url: "https://mirror.ghproxy.com/https://raw.githubusercontent.com/YanG-1989/m3u/main/Gather.m3u", name: "Gather" }
];
const cl = reactive({
  name: '3',
  map: null
});


const options = reactive({
  autoplay: true,
  muted: true,
  controls: true,
  loop: false,
  sources: [{ src: 'https://live-play.cctvnews.cctv.com/cctv/merge_cctv13.m3u8', type: 'application/x-mpegURL' }],
  language: 'zh-CN',
  fluid: true,
  liveui: true,
});

function slist(url) {
  fetch(url).then(resp => {
    return resp.text()
  }).then(txt => {
    return parseTxt(txt)
  }).then(categres => { cl.map = categres });
}


function parseTxt(txt) {
  const lines = txt.split('\n');
  let categres = new Map();
  for (let i = 1; i < lines.length; i = i + 2) {
    let line = lines[i];
    if (line.startsWith("#EXTINF:-1")) {
      const cs = line.split(" ");
      let ch = {};
      let group;
      for (let j = 1; j < cs.length; j++) {
        const p1 = cs[j];
        if (p1.startsWith("tvg-name=")) {
          ch.name = p1.split("=")[1].replace(/^["']|["']$/g, '');
        } else if (p1.startsWith("group-title=")) {
          const gp = p1.split("=")[1].split(",");
          group = gp[0].replace(/^["']|["']$/g, '');
          ch.title = gp[1].replace('\r', '');
        } else if (p1.startsWith("tvg-logo=")) {
          ch.logo = p1.split('=')[1].replace(/^["']|["']$/g, '');
        }
      }
      ch.url = lines[i + 1].replace('\r', '');
      let ls = categres.get(group)
      if (ls && ls.length > 0) {
        ls.push(ch);
      } else {
        categres.set(group, [ch]);
      }
    }
  }
  return categres;
}
const theme = useTheme();
function toggleTheme() {
  theme.global.name.value = theme.global.current.value.dark ? 'light' : 'dark'
}
function switchch(url) {
  console.log(url)
  options.muted = false;
  options.sources = [{ src: url, type: 'application/x-mpegURL' }]
}
</script>
