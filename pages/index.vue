<template>
  <div ref="mainBox" class="mainBox">
    <nuxt-link
      v-if="$device.isDesktop"
      class="sideLink redLink animation shadow"
      tag="div"
      to="catalog"
    >
      <div ref="left" class="back-box back-box-left" />
      <div class="frontbox" style="background: #e5765791">Каталог</div>
      <div ref="leftEyelet" class="eyelet eyelet-left animation">
        <v-icon large>mdi-chevron-right</v-icon>
      </div>
    </nuxt-link>

    <div class="centerBox balance">
      <h1 class="mainTitle">COFFEE MAKE</h1>
      <h2 class="subTitle">БОЛЬШЕ ЧЕМ КОФЕ!</h2>
    </div>

    <nuxt-link
      v-if="$device.isDesktop"
      class="sideLink blueLink animation shadow"
      tag="div"
      to="business"
    >
      <div ref="rightEyelet" class="eyelet eyelet-right animation">
        <v-icon large>mdi-chevron-left</v-icon>
      </div>
      <div ref="right" class="back-box back-box-right" />
      <div class="frontbox" style="background: #4494b6a1">Для Бизнеса</div>
    </nuxt-link>

    <div v-if="$device.isMobile" id="businessBox" class="container">
      <div class="head-row shadow">
        <ContactBox
          title="Бесплатный тест"
          text="Привезем 3 любых пробника и предоставим автоматическую кофемашину в тест на 7 дней"
          background-img="red_men.png"
        />
        <ContactBox
          style="margin-top: 1rem"
          title="МЫ ГОТОВЫ ПОМОЧЬ"
          title-color="red"
          text="Оставьте заявку и наши менеджеры помогут подобрать для вас кофе исходя из ваших предпочтений по вкусу и способу приготовления."
          background-img="coffee_machine3.jpg"
        />
      </div>
      <img
        v-for="i in 8"
        :key="i"
        class="infoImg shadow"
        :class="[i % 2 == 0 ? 'right-col' : 'left-col']"
        :src="'\\business\\' + i + '.png'"
      />
    </div>
  </div>
</template>

<script>
export default {
  layout: 'adaptivLayout',
  data() {
    return {
      isCenterActive: true,
    }
  },
  computed: {},
  mounted() {
    window.addEventListener('resize', this.resizeHandler)
    if (this.$device.isDesktop) this.resizeHandler()
    this.$refs.mainBox.style.minHeight = this.$store.state.windowHeight + 'px'
    this.$nextTick(() => {
      this.$nuxt.$loading.start()
      setTimeout(() => this.$nuxt.$loading.finish(), 200)
    })
  },
  destroyed() {
    window.removeEventListener('resize', this.resizeHandler)
  },
  methods: {
    resizeHandler() {
      this.$refs.leftEyelet.style.left =
        this.$refs.left.clientWidth -
        this.$refs.leftEyelet.clientWidth / 1.6 +
        'px'
      this.$refs.rightEyelet.style.right =
        this.$refs.right.clientWidth -
        this.$refs.rightEyelet.clientWidth / 1.6 +
        'px'
    },
  },
}
</script>

<style scoped lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Caveat:wght@700&display=swap');
.mainBox {
  position: relative;
  display: flex;
}
.shadow {
  filter: drop-shadow(0 0 0.2rem gray);
}
.animation {
  transition: all 0.6s;
}
.centerBox {
  position: relative;
  left: 25%;
  padding: 3rem 0 3rem 0;
  text-align: center;
  width: 50%;
  background: url(static\Coffee_logotype_styl.png) 47% 10%;
  // z-index: -1;
}
.mainTitle {
  margin: 0 0 2rem 0;
  color: white;
  text-shadow: 0 5px 8px gray;
}
.subTitle {
  color: $main-color;
  font-family: 'Caveat', cursive;
  font-weight: bold;
  // text-decoration: underline;
  transform: rotate(-4deg) translateX(0rem) translateY(-1rem);
}
#tryButton {
  display: inline-block;
  cursor: pointer;
  padding: 1rem;
  color: white;
  background: $main-color;
  border: 2px white solid;
  border-radius: 20px;
  z-index: 20;
  transition: all 0.2s;
}
#tryButton:hover {
  transform: scale(1.05);
}

.sideLink {
  position: absolute;
  height: 100%;
  width: 25%;
  cursor: pointer;
  z-index: 2;
}
.sideLink:hover {
  width: 27%;
}
.blueLink {
  right: 0;
}
.redLink {
  left: 0;
}
.sideLink:hover .eyelet-left {
  // transform: translateX(-50%);
}
.sideLink:hover .eyelet-right {
  // transform: translateX(50%);
}
.eyelet {
  display: flex;
  justify-content: center;
  position: absolute;
  width: 100px;
  height: 100px;
  top: 50%;
  margin-top: -50px;
  border-radius: 100%;
  z-index: -2;
}
.eyelet-left {
  padding: 0 0 0 2.5rem;
  background: $main-light-color;
  background: #db9b8e;
}
.eyelet-right {
  padding: 0 2.5rem 0 0;
  background: $side-dark-color;
  background: #6eacc6;
}
.back-box {
  position: absolute;
  height: 100%;
  width: 100%;
  // filter: blur(0px);
  background-position: 75% 50%;
  background-repeat: no-repeat;
  background-size: cover;
}
.back-box-left {
  background-image: url(static\left-background.jpg);
  // border-right: 5px $main-light-color solid;
}
.back-box-right {
  background-image: url(static\right-background.jpg);
  // border-left: 5px $side-dark-color solid;
}
.frontbox {
  position: absolute;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  width: 100%;
  color: white;
  font-weight: bold;
  font-size: 3rem;
  text-align: center;
  // text-shadow: 0 0 2rem black;
  text-shadow: 0 0 5px gray;
  text-decoration: underline;
}
@media screen and (max-width: $mobile) {
  .mainBox {
    flex-wrap: wrap;
  }
  .centerBox {
    left: 0;
    right: 0;
    padding: 3rem;
    width: 100%;
    height: 100%;
    background-size: cover;
  }
  .mainTitle {
    font-size: 3rem;
  }
  .subTitle {
    font-size: 2rem;
  }

  #businessBox {
    border-radius: 20px;
    padding: 1rem 2% 5rem 2%;
  }
  .container {
    display: grid;
    grid-template-columns: repeat(1, 1fr);
    grid-template-rows: repeat(auto-fill, 1fr);
    gap: 10px 0;
    justify-items: center;
    align-items: center;
  }

  .left-col {
    grid-column: 1;
  }
  .right-col {
    grid-column: 2;
  }
  .head-row {
    grid-row: 1;
    grid-column: span 2;
  }

  .shadow {
    filter: drop-shadow(0 0.05rem 0.2rem gray);
  }
  .infoImg {
    width: 100%;
    border-radius: 20px;
    background: gray;
    transition: all 0.3s;
  }
}
</style>
