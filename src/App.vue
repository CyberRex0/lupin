<template>
  <div v-if="animationMode == 0" class="UfvEjlW">
    <h1>Lupin</h1>
    <textarea id="dIBkFqwDMshRpZZKyP" v-model="text"></textarea>
    <br>

    <span>送り間隔 {{ oneCharAnimationInterval }}ms</span>
    <input type="range" :min="config.animIntvMin" :max="config.animIntvMax" v-model="oneCharAnimationInterval" step="10" />
    <button @click="startAnimation" v-bind:disabled="goButtonDisabled">Go</button>
    
    <br>
    ※音が鳴ります
    <br>

    <small>
      &copy; CyberRex<br>
      効果音およびジングルの著作権はそれぞれの著作者に帰属します。<br>
      <span v-if="config.repoUrl"><a :href="config.repoUrl" style="text-decoration: underline">GitHub</a></span>
    </small>

  </div>

  <div v-if="animationMode == 1" class="cUDntZmUJxMbspr qUtGMlUShDqyNszWbGel">
    <span v-bind:style="{ opacity: oneCharOpacity }">{{ oneChar }}</span>
  </div>

  <div v-if="animationMode == 2" class="cUDntZmUJxMbspr qUtGMlUShDqyNszWbGel">
    <span v-html="originalText" class="gsjCJttlIDuHuntI"></span>
    <span class="ukKbEbhmItNYzGJ">
      <button @click="returnHome" :style="{ opacity: returnHomeBtnVisible=='visible' ? 1 : 0 }">もう一度遊ぶ</button>
      &nbsp;
      <button ref="copyButton" @click="copyClipboard" :style="{ opacity: returnHomeBtnVisible=='visible' ? 1 : 0 }">{{linkCopyText}}</button>
    </span>
  </div>
</template>

<style scoped>
.UfvEjlW {
  text-align: center;
  margin: 0 auto;
}

.cUDntZmUJxMbspr {
  text-align: center;
  margin: 0 auto;
  color: white;
  font-size: 38pt;
}

.ukKbEbhmItNYzGJ {
  font-size: 10pt;
  display: block;
  margin-top: 32px;
  margin-left: auto;
  margin-right: auto;
  margin-bottom: 16px;
}

#dIBkFqwDMshRpZZKyP {
  width: 20rem;
  height: 4rem;
}

.gsjCJttlIDuHuntI {
  line-height: 1.25;
}
</style>

<script setup lang="ts">
import { onMounted, Ref, ref, watch } from 'vue';
import { AudioAssetList } from './interfaces';
import config from './config';

const returnHomeBtnVisible = ref('hidden');
const goButtonDisabled = ref(false);
const linkCopyText = ref('リンク共有');

const text = ref('Hello World');
const originalText = ref('');
const oneChar = ref('');
const animationMode = ref(0);
const oneCharOpacity = ref(1.0);
let oneCharAnimationInterval = ref(config.animIntvDefault);
let oneCharBlinkInterval = oneCharAnimationInterval.value / 2;

const audioAssetNameList = [
  'tw_1', 'tw_2', 'tw_3', 'tw_4', 'tw_5', 'tw_6', 'tw_7', 'tw_8', 'tw_9',
  'jingle'
]

const audioRes: AudioAssetList = {};

onMounted(async () => {
  await loadAudioAssets();
  audioRes['jingle'].onplaying = () => {
    returnHomeBtnVisible.value = 'hidden';
  }
  audioRes['jingle'].onended = () => {
    setTimeout(() => {
      returnHomeBtnVisible.value = 'visible';
    }, 1000);
  }

  try {
    const url = new URL(location.href);
    if (url.searchParams.get('text')) {
      text.value = decodeURIComponent(url.searchParams.get('text') || '');
    }
    if (url.searchParams.get('animIntv')) {
      try {
        let intv = Number(url.searchParams.get('animIntv') || config.animIntvDefault.toString());
        if ((intv < config.animIntvMin || intv > config.animIntvMax)) intv = config.animIntvDefault;
        oneCharAnimationInterval.value = intv;
      } catch (e) {

      }
    }
  } catch (e) {
    
  }
});

watch(text, (newText) => {
  // remove spaces
  newText = newText.replace(/\s/g, '');
  if (newText.length > 0) {
    goButtonDisabled.value = false;
  } else {
    goButtonDisabled.value = true;
  }
});

watch(oneCharAnimationInterval, (newInterval) => {
  oneCharBlinkInterval = newInterval / 2;
});

function randomInt(min: number, max: number) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

async function loadAudioAssets() {
  for (const sound of audioAssetNameList) {
    if (!audioRes[sound]) audioRes[sound] = new Audio(`./sounds/${sound}.wav`);
    // preload
    audioRes[sound].load();
    audioRes[sound].currentTime = 0;
    audioRes[sound].volume = 0.4;
  }
}

async function playAudio(assetName: string) {
  if (!audioRes[assetName]) return;
  if (audioRes[assetName].readyState === 0) return;
  audioRes[assetName].pause();
  audioRes[assetName].currentTime = 0;
  await audioRes[assetName].play();
}


function startAnimation() {
  animationMode.value = 1;
  returnHomeBtnVisible.value = 'hidden';
  originalText.value = text.value.replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/\n/g, '<br>');
  let i = 0;
  let aniText = text.value.replace(/\s/g, '');
  const textArray = aniText.split('');
  const textArrayLength = textArray.length;
  const timer = setInterval(async () => {

    // 打鍵音
    await playAudio('tw_' + randomInt(1, 9));
    
    // 1文字送る
    const chr = textArray[i];
    oneChar.value = chr;
    i++;
    oneCharOpacity.value = 1;
    
    // 点滅
    setTimeout(() => {
      oneCharOpacity.value = 0;
    }, oneCharBlinkInterval);

    // すべて送ったら全体表示へ
    if (i >= textArrayLength) {

      clearInterval(timer);

      setTimeout(async () => {

        animationMode.value = 2;
        await playAudio('jingle');

      }, 600);

    }

  }, oneCharAnimationInterval.value);

}

function returnHome() {
  animationMode.value = 0;
  linkCopyText.value = 'リンク共有';
}

async function copyClipboard() {
  const url = new URL(location.href);
  await navigator.clipboard.writeText(`${url.origin}${url.pathname}?text=${encodeURIComponent(text.value)}&animIntv=${oneCharAnimationInterval.value}`);
  linkCopyText.value = 'コピーしました';
}
</script>
