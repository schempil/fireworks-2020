<template>
  <div class="fireworks">
    <h1>New Year's Eve 2020</h1>
    <svg id="fireworks-svg" @click="clickSvg"/>
  </div>
</template>

<script lang="js">
import { Component, Vue } from 'vue-property-decorator';
import { SVG } from "@svgdotjs/svg.js";
import colors from "@/constants/colors";
import settings from "@/constants/settings";

@Component
export default class Firework extends Vue {

  mounted() {

    const svg = SVG("#fireworks-svg")

    const rockets = []

    for (let i = 0; i < settings.rocketCount; i++) {
      const x = Math.floor(Math.random() * svg.node.clientWidth) + 1
      const y = svg.node.clientHeight - settings.stickHeight
      rockets.push(this.createRocket(svg, x, y))
    }

    this.startExplosions(svg, rockets)

  }

  clickSvg(e) {
    const svg = SVG("#fireworks-svg")
    const rocket = this.createRocket(svg, e.clientX, svg.node.clientHeight - settings.stickHeight)
    this.doRocketExplosion(svg, rocket, settings.baseDuration, 200)
  }

  startExplosions(svg, rockets) {
    rockets.forEach((rocket, i) => {
      const delay = settings.baseDuration / 10 * i * (Math.floor(Math.random() * 2) + 1)
      this.doRocketExplosion(svg, rocket, settings.baseDuration, delay)
    })
  }

  doRocketExplosion(svg, rocket, duration, delay) {

    const targetY = Math.floor(Math.random() * 200) + 1

    rocket.animate({
      duration: duration,
      when: 'now',
      swing: true,
      delay: delay,
    }).move(rocket.x(), 100 + targetY)

    setTimeout(() => {
      this.startExplosion(svg, rocket)
      rocket.remove()
    }, duration + delay)
  }

  createRocket(svg, x, y) {

    const rocket = svg.group()

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
    bomb.y = svg.node.clientHeight - settings.stickHeight

    rocket.rect(bomb.width, bomb.height)
        .attr({ fill: bomb.color })
        .move(bomb.x, bomb.y)

    rocket.polygon(`${bomb.x},${bomb.y} ${bomb.x + hatFactor},${bomb.y - hatFactor} ${bomb.x + (2 * hatFactor)},${bomb.y}`)
        .attr({ fill: colors[(Math.floor(Math.random() * colors.length) + 1) - 1] })
        .move(stick.x - hatFactor, svg.node.clientHeight - settings.stickHeight - hatFactor)

    return rocket
  }

  startExplosion(svg, rocket) {

    const explosion = svg.group()
    const circleCount = settings.circleCount
    const color = colors[(Math.floor(Math.random() * colors.length) + 1) - 1]
    const x = rocket.x()
    const y = rocket.y()

    for (let i = 0; i < circleCount; i++) {
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

}
</script>

<style scoped lang="scss">

  h1 {
    color: #ddd;
    font-size: 5rem;
  }

  .fireworks {
    position: absolute;
    top: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    width: 100vw;
    background: #222;
    background: linear-gradient(to bottom, #020111 60%,#20202c 100%);
  }

  svg {
    position: absolute;
    top: 0;
    left: 0;
    height: 100vh;
    width: 100vw;
  }

</style>
