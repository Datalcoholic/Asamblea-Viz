
<template>
  <div id="app">
    <div class="container1">
      <div class="chart-container">
        <h1>Composicion de la Asamble Nacional</h1>
        <chart v-bind="{svg}" class="chart" />
      </div>
      <div class="steps-container">
        <div
          v-for="(step, i) in stepsText"
          :key="i"
          class="steps"
          :id="['step_'+[i+1]]"
          ref="steps"
          v-html="step.step"
        ></div>
      </div>
    </div>
  </div>
</template>

<script>
const steps = require("./pasos");
import chart from "./components/chart";
import { TweenLite, TweenLineMax } from "gsap";
import VueScrollmagic from "vue-scrollmagic";
import Vue from "vue";
import * as d3 from "d3";
import { group } from "d3-array";

Vue.use(VueScrollmagic);

export default {
  name: "app",
  components: { chart },
  data() {
    return {
      stepsText: steps,
      svg: { height: 700, width: 1100, left: 30, right: 30, top: 30 },
      diputadosXBancadas: []
    };
  },

  mounted() {
    //Setup animacion del texto
    const textElements = this.$refs.steps; // Selecciona los pasos
    this.setTextScenes(textElements);

    // Load the data
    d3.csv("/Data/consolidado.csv", data => ({
      diputado: data.name,
      condicion: data.condicion,
      bancada: data.bancada,
      bancada2: this.swichtBancadas(data.bancada),
      partido: data.partido,
      edo: data.estado,
      circuscripcion: data.circuscripcion,
      suplente: data.suplente,
      estadoLegalPrincipal: data.estado_legal_principal,
      estadoLegalSuplente: data.estado_legal_suplente
    }))
      .then(data => d3.group(data, d => d.bancada2))
      .then(data => Array.from(data))
      .then(d => (this.diputadosXBancadas = d));

    this.test("#step_2");
  },
  methods: {
    // Crea escena por cada paso
    setTextScenes(Array) {
      for (const obj of Array) {
        const textAppear = TweenLite.to(`#${obj.id}`, 0.5, { opacity: 1 });
        const sceneText = this.$scrollmagic
          .scene({
            triggerElement: `#${obj.id}`,
            triggerHook: 0.5
          })
          .setTween(textAppear)
          .addIndicators({ name: `${obj.id}` });
        this.$scrollmagic.addScene(sceneText);
      }
    },
    test(triEl) {
      const testScene = this.$scrollmagic
        .scene({
          triggerElement: triEl,
          triggerHook: 0.5
        })
        .on("enter", () => {
          console.log(this.diputadosXBancadas);
        });
      this.$scrollmagic.addScene(testScene);
    },
    swichtBancadas(ban) {
      switch (ban) {
        case "MUD":
          return "OPOSICION";
          break;

        case "CONCERTACION":
          return "OPOSICION";
          break;

        case "16-J":
          return "OPOSICION";
          break;

        case "PSUV":
          return "GGP";
          break;
      }
    }
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Barlow+Condensed:400,600,900&display=swap");
#app {
  font-family: "Barlow Condensed", Helvetica, Arial, sans-serif;
}

.container1 {
  height: 250vh;
}

.steps-container {
  position: relative;
  margin: 0 0 40em 0;
  display: flex;
  align-items: center;
  /* justify-content: center; */
  flex-direction: column;
}
.steps {
  background-color: "#ffffff";
  opacity: 0;
  margin: 10em 0 10em 0;
  display: flex;
  align-items: center;
  min-height: 150px;
  width: 50vw;
  border: 1px solid blue;
  border-radius: 14px;
}

.chart-container {
  position: sticky;
  top: 0;
}
svg {
  background: yellowgreen;
}
</style>
