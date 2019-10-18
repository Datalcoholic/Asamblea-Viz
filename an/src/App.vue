
<template>
  <div id="app">
    <div class="container1">
      <div class="chart-container">
        <h1>Composicion de la Asamble Nacional</h1>
        <chart v-bind="{svg, plotDiputados}" class="chart" />
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
import _ from "lodash";

Vue.use(VueScrollmagic);

const colors = ["#fb3640", "#772271", "#579633", "#2e86ab"];

export default {
  name: "app",
  components: { chart },
  data() {
    return {
      stepsText: steps,
      svg: { height: 700, width: 1100, left: 30, right: 30, top: 30 },
      diputadosXBancadas: [],
      bancadas: ["oposicion", "oposicion minoritaria", "GPP", "Disidentes GGP"],
      bancadasCS: [
        "GPP",
        "oposicion minoritaria",
        "Disidentes GPP",
        "oposicion"
      ]
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
      .then(data => group(data, d => d.bancada2))
      .then(data => Array.from(data, ([key, value]) => ({ key, value })))
      .then(d => (this.diputadosXBancadas = d));

    this.test("#step_2");
  },
  computed: {
    plotDiputados() {
      const bancadasToLower = this.bancadas.map(d => d.toLowerCase());

      const distrGroups = d3
        .scalePoint()
        .domain(bancadasToLower)
        .range([0, this.svg.width])
        .padding(0.5);

      console.log("test dist", distrGroups("ggp"));
      const test = [];
      this.diputadosXBancadas.forEach(ban =>
        test.push({
          bancada: ban.key,
          diputados: this.crossArrays(ban.value, 10),
          groupX: distrGroups(ban.key.toLowerCase()),
          banColor: this.bancadaColorScale(ban.key.toLowerCase()),
          count: 0
        })
      );
      return test;
    }
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
          return "GPP";
          break;
      }
    },
    bancadaColorScale(val) {
      // const col = bancadaColorScale
      const bancadaLowerCase = this.bancadasCS.map(d => d.toLowerCase());

      const col = d3
        .scaleOrdinal()
        .domain(bancadaLowerCase)
        .range(colors);
      return col(val);
    },
    crossArrays(array, chunk) {
      const chunks = _.chunk(array, chunk);

      const flatenArray = [];
      // prettier-ignore
      const verticalGap = 30
      const horizontalGap = 30;

      chunks.forEach((arr, index) => {
        arr.forEach((obj, i) => {
          flatenArray.push({
            diputado: obj.diputado,
            condicion: obj.condicion,
            bancada: obj.bancada,
            partido: obj.partido,
            edo: obj.edo,
            circuscripcion: obj.circuscripcion,
            suplente: obj.suplente,
            estadoLegalPrincipal: obj.estadoLegalPrincipal,
            estadoLegalSuplente: obj.estadoLegalSuplente,
            x: i * horizontalGap,
            y: index * verticalGap,
            fill: this.bancadaColorScale(obj.bancada2.toLowerCase())
          });
        });
      });

      return flatenArray;
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
  background: rgb(245, 245, 245);
}

.count-container {
  font-size: 38px;
  font-weight: 600;
}
.count {
  font-size: 60px;
  font-weight: 600;
}
</style>
