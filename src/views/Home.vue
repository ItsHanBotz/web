<template>
  <div id="home" class="wrapper">
    <IntroScene />
    <GapBlock />

    <TitleSection scene="curriculum">
      <TitleFunction params="/^.*$/gi" subtitle="&lt;WorkShowcase&gt;"
        >myCV</TitleFunction
      >
    </TitleSection>

    <BizScene :isPlaying="isPlaying.Biz" />
    <GapBlock />

    <TitleSection scene="ArtPhiGamesTitle">
      <TitleFunction subtitle="background.bmp">
        <span
          class="line"
          v-for="text in ['Art', 'Philosophy', 'Games']"
          :key="text"
        >
          <span class="params">${</span>{{ text }}<span class="params">}</span>
        </span>
      </TitleFunction>
    </TitleSection>

    <SuperMarioScene />
    <GapBlock />

    <GhibliScene :isPlaying="isPlaying.Ghibli" />

    <WrapperScene />

    <ThanksScene :isPlayng="isPlaying.Potion" />
  </div>
</template>

<script>
import { TimelineMax, TweenLite, Power0, Power1, Power2, Power3 } from 'gsap'
import * as ScrollMagic from 'scrollmagic'
import {
  DOM,
  removeBodyClass,
  addBodyClass,
  isReverse,
  isForward,
} from '@/utils'
import AudioMarioStart from '../components/Characters/SuperMario/assets/smw_princess_help.ogg'
import IntroScene from '../components/Home/IntroScene.vue'
import BizScene from '../components/Home/BizScene.vue'
import SuperMarioScene from '../components/Home/SuperMarioScene.vue'
import GhibliScene from '../components/Home/GhibliScene.vue'
import WrapperScene from '../components/Home/WrapperScene.vue'
import ThanksScene from '../components/Home/ThanksScene.vue'
import GapBlock from '../components/GapBlock.vue'
import TitleSection from '../components/TitleSection.vue'
import TitleFunction from '../components/TitleFunction.vue'

export default {
  name: 'HomeView',
  components: {
    IntroScene,
    BizScene,
    SuperMarioScene,
    GhibliScene,
    WrapperScene,
    ThanksScene,
    GapBlock,
    TitleSection,
    TitleFunction,
  },
  data() {
    return {
      audioMarioStart: new Audio(AudioMarioStart),
      scrollMagicController: new ScrollMagic.Controller(),
      scrollMagicScene: {},
      timelines: {},
      tweeners: {},
      isPlaying: {
        Biz: false,
        Ghibli: false,
        Potion: false,
      },
    }
  },
  created() {
    window.addEventListener('beforeunload', () => window.scroll(0, 0))
  },
  mounted() {
    // setup
    this.playIntroScene()
    this.setupScenes()
    this.manageLoops()
    // scenes animation
    this.sceneMyCV()
    this.sceneBizTitle()
    this.sceneBizZen()
    this.sceneBizEverybody()
    this.sceneBizEnding()
    this.sceneOcean()
    this.sceneSunset()
    this.sceneArtPhiGames()
    this.sceneMario()
    this.sceneGhibli()
    this.sceneWrapper()
  },
  beforeUnmount() {
    // loop animations
    this.isPlaying = {
      Biz: false,
      Ghibli: false,
      Potion: false,
    }
    // to avoid style issues
    removeBodyClass('is-playing-mario', 'blue-background')
    // timelines
    Object.values(this.timelines).forEach((timeLine) => timeLine.kill())
    this.timelines = {}
    // tweeners
    Object.values(this.tweeners).forEach((tweener) => tweener.kill())
    this.tweeners = {}
    // scrollMagic
    this.scrollMagicController.destroy(true)
    this.scrollMagicController = null
    Object.values(this.scrollMagicScene).forEach((scene) => scene.destroy(true))
    this.scrollMagicScene = {}
  },
  methods: {
    setupScenes() {
      const scenesElements = {
        myCV: DOM.get('#curriculum.scene'),
        bizTitle: DOM.get('#bizTitle.scene'),
        biz1: DOM.get('#biz1.scene'),
        biz2: DOM.get('#biz2.scene'),
        biz3: DOM.get('#biz3.scene'),
        artPhiGamesTitle: DOM.get('#ArtPhiGamesTitle.scene'),
        mario: DOM.get('#Mario.scene'),
        ghibli: DOM.get('#Ghibli.scene'),
        wrapper: DOM.get('#wrapperTitle.scene'),
        thanks: DOM.get('#thanks.scene'),
      }

      Object.entries(scenesElements).forEach(([scene, element]) => {
        // tweeners animate timelines' progress, to add momentum
        this.tweeners[scene] = new TimelineMax()
        this.timelines[scene] = new TimelineMax({ paused: true })

        // ScrollMagic scenes
        this.scrollMagicScene[scene] = new ScrollMagic.Scene({
          triggerElement: element,
          offset: -this.$viewport.height / 2, // start half screen before
          duration: element.offsetHeight, // lasts for the element height
        })
          .setTween(this.tweeners[scene])
          .addTo(this.scrollMagicController)
          .reverse(true)
          .setClassToggle(element, 'active')

        // animate timeline progress from tweeners
        this.tweeners[scene]
          .to(element, 1, { autoAlpha: 1 }) // fake, just to have some progress
          .eventCallback('onUpdate', () => {
            TweenLite.to(this.timelines[scene], 0.5, {
              progress: this.tweeners[scene].progress(),
              ease: Power0.easeNone,
            })
          })
      })
    },
    playIntroScene() {
      const timeline = new TimelineMax()
      timeline
        .addLabel('enter', 1)
        .from(
          '#intro .title',
          2,
          {
            autoAlpha: 0,
            rotationX: 90,
            transformOrigin: '50% 50% -100px',
            ease: Power3.easeOut,
          },
          'enter'
        )
        .from(
          '#intro .std',
          2,
          {
            autoAlpha: 0,
            x: -32,
            ease: Power3.easeOut,
          },
          'enter+=1.5'
        )
    },
    manageLoops() {
      // play & stop loop animations based on each scene
      this.scrollMagicScene.myCV.on('enter', () => (this.isPlaying.Biz = false))
      this.scrollMagicScene.bizTitle.on(
        'enter',
        () => (this.isPlaying.Biz = true)
      )
      this.scrollMagicScene.biz1.on('enter', () => (this.isPlaying.Biz = true))
      this.scrollMagicScene.biz2.on('enter', () => (this.isPlaying.Biz = true))
      this.scrollMagicScene.biz3.on('enter', () => (this.isPlaying.Biz = true))
      this.scrollMagicScene.artPhiGamesTitle.on('enter', () => {
        removeBodyClass('is-playing-mario', 'blue-background')
      })
      this.scrollMagicScene.mario
        .on('enter', (e) => {
          if (isForward(e)) {
            this.isPlaying.EarlyDays = false
          }
          if (isReverse(e)) {
            addBodyClass('blue-background')
          }
        })
        .on('leave', (e) => {
          if (isReverse(e)) {
            this.isPlaying.Ghibli = false
          }
          removeBodyClass('blue-background')
        })
      this.scrollMagicScene.ghibli
        .on('enter', () => {
          this.isPlaying.Ghibli = true
          removeBodyClass('is-playing-mario')
          addBodyClass('blue-background')
        })
        .on('leave', () => removeBodyClass('blue-background'))
      this.scrollMagicScene.wrapper
        .on('enter', () => addBodyClass('blue-background'))
        .on('leave', () => removeBodyClass('blue-background'))
      this.scrollMagicScene.thanks
        .on('enter', () => addBodyClass('blue-background'))
        .on('leave', () => removeBodyClass('blue-background'))
    },
    sceneMyCV() {
      ...
    },
    sceneBizTitle() {
      ...
    },
    sceneBizZen() {
      ...
    },
    sceneBizEverybody() {
      ...
    },
    sceneBizEnding() {
      ...
    },
    sceneOcean() {
      ...
    },
    sceneSunset() {
      ...
    },
    sceneArtPhiGames() {
      ...
    },
    sceneMario() {
      ...
    },
    sceneGhibli() {
      ...
    },
    sceneWrapper() {
      ...
    },
  },
}
</script>
