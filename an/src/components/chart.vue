<template>
  <div>
    <svg :height="svg.height" :width="svg.width" fill="#e6e6e6">
      <template v-for="ban in plotDiputados">
        <g
          :id="ban.bancadas"
          :key="ban.bancadas"
          :style="`transform:translate(${ban.groupX}px, 50px)`"
        >
          <rect
            class="diputados"
            :id="dip.diputado"
            v-for="(dip) in ban.diputados"
            :key="dip.diputado"
            :x="dip.x"
            :y="dip.y"
            :width="23"
            :height="23"
            :fill="dip.fill"
            rx="5"
            ry="5"
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