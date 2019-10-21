<template>
  <div>
    <svg :height="svg.height" :width="svg.width" fill="#e6e6e6" id="over">
      <template v-for="ban in plotDiputados">
        <g
          :id="ban.bancada.toLowerCase().split(' ').join('-')"
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
            :edoLegalPrincipal="dip.estadoLegalPrincipal"
            :edoLegalSuplente="dip.estadoLegalSuplente"
            :bancada="dip.bancada"
            ref="dip"
          />
          <rect
            class="txtBackground"
            :y="400"
            width="295"
            height="180"
            :fill="ban.banColor"
            rx="8"
            ry="8"
            :id="ban.bancada.toLowerCase().split(' ').join('-')"
          />
          <text
            class="count-container"
            x="12"
            :y="440"
            fill="#ffffff"
            :id="ban.bancada.toLowerCase().split(' ').join('-')"
          >{{ban.bancada}}</text>
          <text
            class="cambiar"
            x="12"
            :y="470"
            fill="#ffffff"
            :id="ban.bancada.toLowerCase().split(' ').join('-')"
          >Nº de Diputados:</text>

          <text
            class="count"
            x="12"
            :y="540"
            fill="#ffffff"
            :id="ban.bancada.toLowerCase().split(' ').join('-')"
          >{{ban.count}}</text>
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