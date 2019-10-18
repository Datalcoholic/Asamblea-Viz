<template>
  <div>
    <svg :height="svg.height" :width="svg.width" fill="#e6e6e6">
      <template v-for="ban in plotDiputados">
        <g
          :id="ban.bancada"
          :key="ban.bancada"
          :style="`transform:translate(${ban.groupX}px, 50px)`"
        >
          <rect
            v-for="(dip) in ban.diputados"
            class="diputados"
            :id="dip.diputado"
            :key="dip.diputado"
            :x="dip.x"
            :y="dip.y"
            :width="23"
            :height="23"
            :fill="dip.fill"
            rx="5"
            ry="5"
          />

          <rect :y="400" width="295" height="120" :fill="ban.banColor" rx="8" ry="8" />
          <text class="count-container" x="12" :y="440" fill="#ffffff">{{ban.bancada}}</text>
          <text class="count" x="12" :y="500" fill="#ffffff">{{ban.count}}</text>
          {{ban.bancada}}
          />
        </g>
      </template>
    </svg>
  </div>
</template>

<script>
export default {
  props: ["svg", "plotDiputados"]
};
</script>

<style>
</style>