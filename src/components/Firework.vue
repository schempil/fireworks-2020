<template>
  <div class="fireworks" :class="this.gradient">
    <h1>New Year's Eve 2020</h1>
    <span class="time">
      {{ pad(currentDate.getHours(), 2) }}:{{ pad(currentDate.getMinutes(), 2) }}:{{ pad(currentDate.getSeconds(), 2) }}
    </span>
    <span class="timezone">{{ Intl.DateTimeFormat().resolvedOptions().timeZone }}</span>
    <svg id="fireworks-svg" @click="clickSvg"/>
    <span class="fire-button" v-if="rockets.length > 0" @click="startExplosions">FIRE!</span>
    <span class="fire-button" v-if="rockets.length < 1" @click="loadRockets">RELOAD!</span>
  </div>
</template>

<script lang="js">
import { Component, Vue } from 'vue-property-decorator';
import { SVG } from "@svgdotjs/svg.js";
import colors from "@/constants/colors";
import settings from "@/constants/settings";

@Component
export default class Firework extends Vue {

  currentDate = new Date()
  rockets = []
  svg = null
  gradient = this.getSkyGradient()
  explosionStarted = false

  mounted() {

    this.svg = SVG("#fireworks-svg")
    this.updateTime()
    this.loadRockets()
  }

  loadRockets() {

    for (let i = 0; i < settings.rocketCount; i++) {
      this.rockets.push(this.createRandomRocket())
    }
  }

  updateTime() {

    this.currentDate = new Date()

    if (this.currentDate.getFullYear() >= settings.newYear && !this.explosionStarted) {
      this.explosionStarted = true
      this.startExplosions()
    }

    this.gradient = this.getSkyGradient()

    setTimeout(this.updateTime, 1000)
  }

  getSkyGradient() {

    const hours = new Date().getHours()

    if (hours > 21 || hours < 6) {
      return `sky-gradient-${0}`
    }

    return `sky-gradient-${hours}`
  }

  clickSvg(e) {
    const rocket = this.createRocket(e.clientX, this.svg.node.clientHeight - settings.stickHeight)
    this.doRocketExplosion(rocket, settings.baseDuration, 200)
  }

  startExplosions() {
    this.rockets.forEach((rocket, i) => {
      const delay = settings.baseDuration / 20 * i * (Math.floor(Math.random() * 2) + 1)
      this.doRocketExplosion(rocket, settings.baseDuration, delay)
    })

    this.rockets = []
  }

  doRocketExplosion(rocket, duration, delay) {

    const targetY = Math.floor(Math.random() * 200) + 1

    rocket.animate({
      duration: duration,
      when: 'now',
      swing: true,
      delay: delay,
    }).move(rocket.x(), 100 + targetY)

    setTimeout(() => {
      this.startExplosion(this.svg, rocket)
      rocket.remove()
    }, duration + delay)
  }

  createRocket(x, y) {

    const rocket = this.svg.group()

    const stick = {
      width: 1,
      height: 40,
      color: '#ffa54f'
    }

    const bomb = {
      width: 5,
      height: 10,
      color: '#880606'
    }

    const hatFactor = 3

    stick.x = x
    stick.y = y

    rocket.rect(stick.width, settings.stickHeight)
        .attr({ fill: stick.color })
        .move(stick.x, stick.y)

    bomb.x = stick.x - (bomb.width / 2)
    bomb.y = this.svg.node.clientHeight - settings.stickHeight

    rocket.rect(bomb.width, bomb.height)
        .attr({ fill: bomb.color })
        .move(bomb.x, bomb.y)

    rocket.polygon(`${bomb.x},${bomb.y} ${bomb.x + hatFactor},${bomb.y - hatFactor} ${bomb.x + (2 * hatFactor)},${bomb.y}`)
        .attr({ fill: colors[(Math.floor(Math.random() * colors.length) + 1) - 1] })
        .move(stick.x - hatFactor, this.svg.node.clientHeight - settings.stickHeight - hatFactor)

    return rocket
  }

  createRandomRocket() {

    const x = Math.floor(Math.random() * this.svg.node.clientWidth) + 1
    const y = this.svg.node.clientHeight - settings.stickHeight

    return this.createRocket(x, y)
  }

  startExplosion(svg, rocket) {

    const explosion = svg.group()
    const circleCount = settings.circleCount
    const color = colors[(Math.floor(Math.random() * colors.length) + 1) - 1]
    const x = rocket.x()
    const y = rocket.y()

    for (let i = 1; i < circleCount; i++) {
      const circle = explosion.circle().radius(1)
        .attr({fill: color})
        .move(x, y)

      const newX = x + 250 * Math.cos(360 / circleCount * i)
      const newY = y + 250 * Math.sin(360 / circleCount * i)

      const diffX = Math.floor(i % circleCount / 10 * (Math.cos(360 / (i % circleCount / 10))) * circleCount)
      const diffY = Math.floor(i % circleCount / 10 * (Math.sin(360 / (i % circleCount / 10))) * circleCount)

      circle.animate(settings.baseDuration, 0, "now").move(newX + diffX, newY + diffY)
            .animate(settings.baseDuration / 4, 0, "now").attr({r: 10, opacity: 0})

    }

    setTimeout(() => {
      explosion.remove()
    }, settings.baseDuration / 4)
  }

  pad(num, size) {

    num = num.toString();
    while (num.length < size) num = "0" + num;

    return num;
  }

}
</script>

<style scoped lang="scss">

  h1 {
    color: #ddd;
    font-size: 4rem;
  }

  .fireworks {
    position: absolute;
    top: 0;
    display: flex;
    flex-direction: column;
    height: 100%;
    width: 100vw;
    background: #222;
  }

  .sky-gradient-0, .sky-gradient-24 { background: #00000c; }
  .sky-gradient-1 { background: linear-gradient(to bottom, #020111 85%,#191621 100%); }
  .sky-gradient-2 { background: linear-gradient(to bottom, #020111 60%,#20202c 100%); }
  .sky-gradient-3 { background: linear-gradient(to bottom, #020111 10%,#3a3a52 100%); }
  .sky-gradient-4 { background: linear-gradient(to bottom, #20202c 0%,#515175 100%); }
  .sky-gradient-5 { background: linear-gradient(to bottom, #40405c 0%,#6f71aa 80%,#8a76ab 100%); }
  .sky-gradient-6 { background: linear-gradient(to bottom, #4a4969 0%,#7072ab 50%,#cd82a0 100%); }
  .sky-gradient-7 { background: linear-gradient(to bottom, #757abf 0%,#8583be 60%,#eab0d1 100%); }
  .sky-gradient-8 { background: linear-gradient(to bottom, #82addb 0%,#ebb2b1 100%); }
  .sky-gradient-9 { background: linear-gradient(to bottom, #94c5f8 1%,#a6e6ff 70%,#b1b5ea 100%); }
  .sky-gradient-10 { background: linear-gradient(to bottom, #b7eaff 0%,#94dfff 100%); }
  .sky-gradient-11 { background: linear-gradient(to bottom, #9be2fe 0%,#67d1fb 100%); }
  .sky-gradient-12 { background: linear-gradient(to bottom, #90dffe 0%,#38a3d1 100%); }
  .sky-gradient-13 { background: linear-gradient(to bottom, #57c1eb 0%,#246fa8 100%); }
  .sky-gradient-14 { background: linear-gradient(to bottom, #2d91c2 0%,#1e528e 100%); }
  .sky-gradient-15 { background: linear-gradient(to bottom, #2473ab 0%,#1e528e 70%,#5b7983 100%); }
  .sky-gradient-16 { background: linear-gradient(to bottom, #1e528e 0%,#265889 50%,#9da671 100%); }
  .sky-gradient-17 { background: linear-gradient(to bottom, #1e528e 0%,#728a7c 50%,#e9ce5d 100%); }
  .sky-gradient-18 { background: linear-gradient(to bottom, #154277 0%,#576e71 30%,#e1c45e 70%,#b26339 100%); }
  .sky-gradient-19 { background: linear-gradient(to bottom, #163C52 0%,#4F4F47 30%,#C5752D 60%,#B7490F 80%, #2F1107 100%); }
  .sky-gradient-20 { background: linear-gradient(to bottom, #071B26 0%,#071B26 30%,#8A3B12 80%,#240E03 100%); }
  .sky-gradient-21 { background: linear-gradient(to bottom, #010A10 30%,#59230B 80%,#2F1107 100%); }
  .sky-gradient-22 { background: linear-gradient(to bottom, #090401 50%,#4B1D06 100%); }
  .sky-gradient-23 { background: linear-gradient(to bottom, #00000c 80%,#150800 100%); }

  svg {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
  }

  .time {
    color: #fff;
    font-weight: 100;
    font-size: 2rem;
  }

  .timezone {
    color: #aaa;
    font-weight: 100;
    font-size: 1rem;
  }

  .fire-button {
    position: fixed;
    bottom: 0;
    right: 0;
    border: 1px solid #fff;
    padding: 20px;
    border-radius: 43px;
    margin: 20px;
    color: #fff;
    font-weight: 700;
    min-width: 75px;
    background: rgba(255, 255, 255, 0.1);
    transition: all 250ms ease;
  }

  .fire-button:hover {
    background: rgba(255, 255, 255, 0.3);
    cursor: pointer;
  }

</style>
