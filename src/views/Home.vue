<template>
  <div class="home">
    <div class="title">
      <img
        src="../image/hacker.png"
        :style="{ transform: `translateZ(${titleZ}px)` }"
      />
    </div>
    <div class="person">
      <img
        @click="toAccounts"
        src="../image/spaceman.png"
        :style="{
          width: personWidth + 'px',
          transform: `translateX(-${personLeft}px)`,
        }"
      />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      top: 0,
      titleZ: 0,
      personWidth: 200,
      personLeft: 0,
      personBottom: -101,
    };
  },
  mounted() {
    window.addEventListener("scroll", this.bindHandleScroll);
  },
  methods: {
    bindHandleScroll(event) {
      this.top = event.srcElement.documentElement.scrollTop;
      this.titleZ = this.top * 10;
      this.personWidth = this.top * 12 + 200;
      this.personLeft = this.top * 7;
    },
    toAccounts() {
      this.$router.push({
        path: '/tally'
      })
    },
  },
};
</script>

<style lang="scss" scoped>
.home {
  width: 100%;
  height: 100vh;
  overflow: hidden;
  background-image: url("../image/background.jpg");
  background-size: cover;
  .title {
    width: 100%;
    margin: 0 auto;
    text-align: center;
    perspective: 800px;
    img {
      width: 90%;
      height: 90%;
    }
  }
  .person {
    position: absolute;
    left: 40%;
    bottom: -100px;
    img {
      width: 200px;
    }
  }
}
</style>
