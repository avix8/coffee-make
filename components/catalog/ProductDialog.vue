<template>
  <div v-if="value" class="dialogCover" title="Свернуть">
    <div v-click-outside="hide" class="productDialogBox">
      <div class="previewBox">
        <hooper
          v-if="item.imgs.length > 1"
          :style="{
            height: [hooperHeight],
            width: [hooperWidth],
          }"
          :infinite-scroll="true"
          :mouse-drag="false"
          :wheel-control="false"
          :auto-play="true"
          :play-speed="10000"
          :transition="1000"
          :items-to-show="1"
          :center-mode="true"
        >
          <slide v-for="(img, index) in item.imgs" :key="index">
            <div class="container">
              <v-img :src="imageIdToURL(img)" contain class="itemImg" />
            </div>
          </slide>
          <hooper-navigation
            v-if="!$device.isMobile"
            slot="hooper-addons"
          ></hooper-navigation>
          <hooper-pagination
            v-if="!$device.isMobile"
            slot="hooper-addons"
          ></hooper-pagination>
          <hooper-pagination
            v-else
            slot="hooper-addons"
            mode="fraction"
          ></hooper-pagination>
        </hooper>
        <div v-else class="container">
          <v-img :src="imageIdToURL(item.imgs[0])" contain class="itemImg" />
        </div>
      </div>

      <div class="infoBox">
        <h2 class="title">
          {{ item.title }}
          <v-icon title="Поделиться" class="share">mdi-share-variant</v-icon>
        </h2>

        <div class="priceBox">
          <transition name="priceFadeDown">
            <div v-if="quantity.value > 1 && price" class="calc">
              {{ price }} руб &times; {{ quantity.value }} шт =
            </div>
          </transition>

          <div ref="price" class="price">
            <h3>{{ cost }} <span v-show="price">руб</span></h3>
          </div>

          <div v-show="price" class="property" style="margin: 0">
            <!-- Количество -->
            <InputOptions
              v-show="!inCart"
              :value="1"
              :max="9999"
              :min="1"
              @inputOption="inputOption($event)"
            ></InputOptions>
          </div>

          <CartButton
            v-show="price"
            :item="item"
            :sku="mySKU"
            :quantity="quantity.value"
          />
        </div>

        <div class="specificationsBox">
          <div v-for="(val, key) in variants" :key="key" class="option">
            {{ key }}
            <div class="line" />
            <ChoiceOptions
              style="position: absolute; right: 0"
              :options="Array.from(val)"
              @changeOption="changeOption($event, key)"
            />
          </div>

          <!-- <div class="solid-line" /> -->
          <div class="block-line" />
          <div v-for="(attr, i) in item.attributes" :key="i" class="property">
            {{ attr.title }}
            <div class="dashed-line" />
            {{ attr.value }}
          </div>
          <div
            v-for="chr in item.characteristics"
            :key="chr.title + chr.value"
            class="property"
          >
            {{ chr.title }}
            <div class="dashed-line" />
            {{ chr.value }}
          </div>
        </div>
        <!-- <div class="solid-line" /> -->
        <div class="block-line" />
        <div v-if="item.descr" class="descriptionBox">
          <h5>Описание:</h5>
          <h6>{{ item.descr }}</h6>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  Hooper,
  Slide,
  Navigation as HooperNavigation,
  Pagination as HooperPagination,
} from 'hooper'

export default {
  components: {
    Hooper,
    Slide,
    HooperNavigation,
    HooperPagination,
  },
  layout: 'header&footer',
  props: {
    item: { type: Object, required: true },
    value: { type: Boolean, required: true },
  },
  data() {
    return {
      hooperHeight: 0,
      hooperWidth: 0,

      price: 0,
      quantity: {
        value: 1,
        minValue: 1,
        maxValue: 9999,
      },

      variants: {},
      mySKU: '',
      now: {},
    }
  },
  computed: {
    cost() {
      if (!this.price) return 'Нет в наличии'
      return this.price * this.quantity.value
    },
    inCart() {
      for (const itemData of this.$store.state.cart.items)
        if (itemData.sku === this.mySKU) return true
      return false
    },
  },
  watch: {
    quantity(newValue) {
      this.$refs.price.style.transition = 'all 2s'
      if (newValue.value !== 1)
        this.$refs.price.style.transform = 'translateY(-50px)'
      else this.$refs.price.style.transform = 'translateY(-150px)'
    },
  },
  mounted() {
    if (this.$device.isMobile)
      this.hooperHeight =
        (document.clientWidth / parseFloat(16) - 2) * 1.15 + 'rem'
    else this.hooperHeight = '100%'
    if (this.$device.isMobile)
      this.hooperWidth = document.clientWidth / parseFloat(16) - 2 + 'rem'
    else this.hooperWidth = '100%'
  },
  created() {
    require('~/assets/hooperSlug.css')
    // this.choiceProperty = this.item.choiceProperty.variants[0]
    this.createVariants()
    this.createNow()
    this.getPrice()
  },
  methods: {
    createVariants() {
      this.variants = {}
      for (const variant of this.item.variants) {
        if (variant.inStock)
          for (const attr of variant.attributes)
            if (this.variants[attr.title])
              this.variants[attr.title].add(attr.value)
            else this.variants[attr.title] = new Set([attr.value])
      }
    },
    createNow() {
      this.mySKU = this.item.variants[0].SKU
      for (const attr of this.item.variants[0].attributes)
        this.now[attr.title] = attr.value
    },
    getPrice() {
      for (const variant of this.item.variants) {
        let flag = true
        for (const attr of variant.attributes)
          if (this.now[attr.title] !== attr.value) flag = false

        if (flag) {
          this.mySKU = variant.SKU
          this.price = variant.price
          return
        }
      }
      this.price = null
      this.mySKU = null
      return 0
    },
    changeOption(val, title) {
      this.$set(this.now, title, val)
      console.log(this.now)
      this.getPrice()
    },
    inputOption(option) {
      this.quantity.value = option
    },
    hide() {
      this.$emit('input', false)
    },
    imageIdToURL(id) {
      return `${this.$axios.defaults.baseURL}/storage/image/${id}`
    },
  },
}
</script>

<style scoped lang="scss">
.dialogCover {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  align-items: center;
  justify-content: center;

  height: 100%;

  background: rgba(0, 0, 0, 0.5);
  z-index: 100;
}
.productDialogBox {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;

  padding: 2rem 5rem 2rem 3rem;

  height: 90%;
  width: 80%;

  cursor: default;

  background: white;
  border-radius: 20px;
  box-shadow: 0 0 0.5rem black;
}
.previewBox {
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;

  grid-column: 1;

  // background: blue;
}
.container {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;

  // margin: 1rem;
  // padding: 5%;

  height: 100%;
  width: 100%;

  border-radius: 20px;
  // box-shadow: inset 0 0 0.8rem black;
  // background: white;
  // background: red;
}
.itemImg {
  height: 100%;
  width: 100%;
  border-radius: 20px;
}

.infoBox {
  position: relative;
  grid-column: 2;

  // background: slateblue;
}
.title {
  display: flex;
  align-items: center;
  justify-content: left;
  margin: 5% 0 1rem 0;
  width: 100%;
  font-weight: bold;
  // background: coral;
}
.share {
  position: absolute;
  right: -3rem;
  top: -1rem;
  margin: 0.5rem;
  padding: 1rem;
  cursor: pointer;
  border-radius: 50px;
  background: whitesmoke;
}
.share:hover {
  box-shadow: 0 0 0.2rem black;
}
.priceBox {
  position: relative;
  display: grid;
  grid-template-columns: 3fr 1fr 3fr;
  align-items: center;
  gap: 1rem;

  margin: 2rem 0 2rem 1rem;

  height: 4rem;

  // background: palegreen;
}
.calc {
  position: absolute;
  top: -0.5rem;
  // height: 35%;
  // width: 100%;
  // background: brown;
}
.priceFadeDown-enter-active,
.priceFadeDown-leave-active {
  transition: all 1s;
}
.priceFadeDown-enter {
  transform: translateY(-1rem);
  opacity: 0;
}
.priceFadeDown-leave-to {
  transform: translateY(-1rem);
  opacity: 0;
}
.price {
  // position: absolute;
  // top: 35%;
  // align-self: flex-end;

  height: 65%;
  width: 100%;

  // background: cyan;
  transition: all 1s;
}

.specificationsBox {
  grid-column: 1 / span 2;

  width: 100%;

  // background: yellow;
}
.property,
.option {
  display: grid;
  grid-template-columns: max-content auto max-content;

  margin-bottom: 0.5rem;

  width: 100%;
  font-size: 1.1rem;
  // background: darkorchid;
  // background: whitesmoke;
}
.option {
  align-items: center;
  height: 2rem;
  // background: whitesmoke;
  background: linear-gradient(90deg, white 0%, whitesmoke 40%);
}
.line {
  width: 100%;
}
.block-line {
  height: 2.5rem;
  width: 100%;
  // background: lightgreen;
}
.dashed-line {
  width: 100%;
  border-bottom: 2px dotted lightgray;
}
.solid-line {
  margin: 2rem 1% 1.5rem 1%;
  width: 98%;
  border-bottom: 2px solid lightgray;
}
.descriptionBox {
  grid-column: 1 / span 2;
  margin-bottom: 1rem;
  // padding: 0 2%;
  width: 100%;
  text-align: justify;
  // background: green;
}
.descriptionBox h5 {
  margin: 0 0 0.5rem 0;
  font-weight: bold;
}

.recentlyViewedBox {
  grid-column: 1 / span 2;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;

  padding: 4rem 0 1rem 0;

  width: 100%;
  // background: violet;
}
.recentlyViewedBox h2 {
  margin: 0 0 1rem 0;
  width: 100%;
  font-weight: bold;
}

@media screen and (max-width: $mobile) {
  .productDialogBox {
    display: flex;
    flex-direction: column;

    padding: 0rem 1rem 5.5rem 1rem;
    // background: saddlebrown;
  }
  .previewBox {
    height: auto;
    width: 100%;

    border-radius: 20px;
    box-shadow: 0 0 1rem lightgray;

    background: whitesmoke;
    // background: blue;
  }
  .container {
    background: whitesmoke;
    // background: red;
  }
  .circle {
    width: 15rem;
    height: 15rem;
  }
  .itemImg {
    transform: scale(1.3);
  }

  .title {
    margin: 1rem 0;

    text-align: center;
    font-size: 150%;
    font-weight: 900;
    // text-shadow: 0 0 0.1rem black;
    // background: coral;
  }

  .priceBox {
    // background: palegreen;
  }
  .calc {
    // background: brown;
  }
  .price {
    color: #2dbb97;
    // background: cyan;
  }

  .specificationsBox {
    // background: yellow;
  }
  .property {
    display: grid;
    align-items: center;
    grid-template-columns: 2fr 1fr;
    // grid-template-rows: 1fr 1fr;

    margin: 0;
    padding: 0.5rem 0 0.5rem 0;

    width: 100%;

    border-bottom: 1px solid lightgray;
    // background: darkorchid;
  }
  .property h4 {
    font-size: 1.2rem;
    // font-weight: bold;
  }
  .property h5 {
    font-size: 1.1rem;
  }

  .cartBtn {
    position: fixed;
    bottom: 3rem;
    left: 0;
    right: 0;

    display: flex;
    justify-content: center;

    padding: 0.5rem 1rem 0.5rem 1rem;

    width: 100%;

    background: white;
    // box-shadow: 0 60px 100px 10px #3a3736;
    box-shadow: 0 0 1rem lightgray;
    z-index: 98;
  }
  .cartBtn button {
    height: 2.1rem;
    width: 100%;

    background-color: #00a199;

    border-radius: 10px;
  }
  .cartBtn button h6 {
    font-size: 0.9rem;
  }
  .cartBtn button:hover {
    background-color: #00a199;
    box-shadow: none;
    transition: 0;
  }
  .cartBtn button:active {
    box-shadow: inset 3px 2px 0.2rem rgb(2, 87, 82);
  }

  .descriptionBox {
    grid-column: 1 / span 2;
    padding: 1rem 0;
    width: 100%;
    text-align: justify;
    // background: green;
  }
  .descriptionBox h3 {
    margin: 0 0 0.5rem 0;
    font-weight: bold;
  }
  .descriptionBox h5 {
    font-size: 0.7rem;
  }

  .recentlyViewedBox {
    // display: flex;
    // flex-wrap: wrap;
    // justify-content: center;

    display: grid;
    grid-template-columns: 1fr 1fr;
    align-items: center;
    justify-items: center;

    // display: flex;
    // flex-wrap: nowrap;
    // justify-content: center;

    margin: 1rem 0;
    padding: 1rem 0;

    width: 100%;

    border-top: 1px solid lightgray;
    // background: violet;
  }
  .recentlyViewedBox h2 {
    grid-column: 1 / span 2;
    // position: absolute;
    // margin: -2rem 0 1rem 10%;
    font-size: 150%;
  }
}
</style>
