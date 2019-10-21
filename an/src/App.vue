
<template>
  <div id="app">
    <div class="container1">
      <div class="chart-container">
        <h1>Composicion de la Asamblea Nacional</h1>
        <chart v-bind="{svg, plotDiputados}" class="chart" />
      </div>
      <div class="steps-container">
        <div
          v-for="(step, i) in stepsText"
          :key="i"
          class="steps"
          :id="['step_'+[i+1]]"
          ref="steps"
        >
          <p v-html="step.step"></p>
        </div>
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
import { group, count } from "d3-array";
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
      ],
      newValue: [],
      oldValue: []
    };
  },

  mounted() {
    //Setup animacion del texto
    const textElements = this.$refs.steps; // Selecciona los pasos
    this.setTextScenes(textElements);

    this.enterStep("#step_1");
    this.animationStep2("#step_2");
    this.animationStep3("#step_3");

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
      .then(data =>
        Array.from(data, ([key, value]) => ({ key, value, count: 0 }))
      )
      .then(d => (this.diputadosXBancadas = d));
  },
  computed: {
    plotDiputados() {
      const bancadasToLower = this.bancadas.map(d => d.toLowerCase());

      const distrGroups = d3
        .scalePoint()
        .domain(bancadasToLower)
        .range([0, this.svg.width])
        .padding(0.2);

      const test = [];
      this.diputadosXBancadas.forEach(ban =>
        test.push({
          bancada: ban.key,
          diputados: this.crossArrays(ban.value, 10),
          groupX: distrGroups(ban.key.toLowerCase()),
          banColor: this.bancadaColorScale(ban.key.toLowerCase()),
          count: ban.count
        })
      );
      return test;
    }
  },
  watch: {
    plotDiputados(newValue, oldValue) {
      this.newValue = newValue;
      this.oldValue = oldValue;

      // console.log("new", newValue[0].count);
      // console.log("old", oldValue[0].count);
    }
  },
  methods: {
    //Animaciones
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
    //inicio de la animaciones de pasos
    enterStep(triEl) {
      const tl = new TimelineMax();
      const testScene = this.$scrollmagic
        .scene({
          triggerElement: triEl,
          triggerHook: 0.5
        })
        .setTween(tl)

        .on("enter", event => {
          //console.log(event.scrollDirection);

          // Change counter
          const newCountOpo = this.diputadosXBancadas[0].value.length;
          const newCountGpp = this.diputadosXBancadas[1].value.length;
          this.$set(this.diputadosXBancadas[0], "count", newCountOpo);
          this.$set(this.diputadosXBancadas[1], "count", newCountGpp);

          tl.staggerTo(".diputados", 1, {
            transformOrigin: "50% 50%",
            opacity: 1,
            stagger: { amount: 0.8, from: 0 }
          })
            .fromTo(
              [".txtBackground", ".count-container", ".count"],
              0.8,
              { transformOrigin: "50% 0%", scaleY: 0, opacity: 0 },
              { transformOrigin: "50% 0%", scaleY: 1, opacity: 1 },
              "-=0.9"
            )
            .from([this.diputadosXBancadas[0], this.diputadosXBancadas[1]], 2, {
              count: 0,
              roundProps: "count",
              ease: Power3.easeOut
            });
        });

      this.$scrollmagic.addScene(testScene);
    },

    animationStep2(triEl) {
      const tl = new TimelineMax();
      const sceneStep2 = this.$scrollmagic
        .scene({
          triggerElement: triEl,
          triggerHook: 0.5
        })
        .setTween(tl)
        .on("enter", () => {
          const foo = Array.from(this.$el.querySelectorAll(".diputados"));
          const dipDesincorporados = foo.filter(
            d => d.getAttribute("edoLegalPrincipal") === "desinrcorporado"
          );

          // Cambiar count
          const dD = this.plotDiputados[0].diputados;
          const dDnumber = dD.filter(
            d => d.estadoLegalPrincipal !== "desinrcorporado"
          ).length;

          // Change diputados opisicion count
          this.$set(this.diputadosXBancadas[0], "count", dDnumber);

          //Animacion
          tl.staggerTo(dipDesincorporados, 0.7, {
            transformOrigin: "50% 50%",
            scale: 2.5,
            stroke: "#fff",
            stagger: { amount: 0.5 }
          })
            .to(
              dipDesincorporados,
              0.7,
              {
                x: i => {
                  return (i * i + 1) * 10;
                }
              },
              "-=0.8"
            )
            .staggerTo(dipDesincorporados, 0.7, {
              fill: "#d1d1d1",
              stroke: "#949494",
              stagger: { amount: 0.5 }
            })
            .to(dipDesincorporados, 0.7, { scale: 1.05, x: 0 })
            .from(
              this.diputadosXBancadas[0],
              0.7,
              {
                count: this.oldValue[0].count,
                roundProps: "count",
                ease: Power3.easeOut
              }
              // "-=1"
            );
        });

      this.$scrollmagic.addScene(sceneStep2);
    },
    animationStep3(triEl) {
      //Set timeline
      const tl = new TimelineMax();

      //Set Scrollmagic
      const sceneStep3 = this.$scrollmagic
        .scene({
          triggerElement: triEl,
          triggerHook: 0.5
          //duration: "300px"
        })
        .setTween(tl)
        .on("enter", () => {
          //Get diputados clandestinidad,exilio, asilo y detenido MUD
          const bar = this.plotDiputados[0].diputados;
          const superBar = bar.filter(
            d =>
              d.estadoLegalPrincipal === "asilo" ||
              d.estadoLegalPrincipal === "exilio" ||
              d.estadoLegalPrincipal === "clandestinidad" ||
              d.estadoLegalPrincipal === "detenido"
          );
          const curulesVacios = superBar.filter(
            d =>
              d.estadoLegalSuplente === "asilo" ||
              d.estadoLegalSuplente === "exilio" ||
              d.estadoLegalSuplente === "clandestinidad" ||
              d.estadoLegalSuplente === "detenido"
          );

          // Change count
          this.$set(
            this.diputadosXBancadas[0],
            "count",
            this.diputadosXBancadas[0].count - curulesVacios.length
          );
          //console.log("dipMUD", curulesVacios);
          //Get elements to animate
          const foo = Array.from(this.$el.querySelectorAll(".diputados"));
          const dipCondicion = foo.filter(d => {
            const attr = d.getAttribute("edoLegalPrincipal");
            const attr2 = d.getAttribute("edoLegalSuplente");
            return (
              (attr === "asilo" ||
                attr === "exilio" ||
                attr === "clandestinidad" ||
                attr === "detenido") &&
              attr2 !== "exilio"
            );
          });
          const dipVacio = foo.filter(d => {
            const attr = d.getAttribute("edoLegalPrincipal");
            const attr2 = d.getAttribute("edoLegalSuplente");
            return (
              (attr === "asilo" ||
                attr === "exilio" ||
                attr === "clandestinidad" ||
                attr === "detenido") &&
              attr2 === "exilio"
            );
          });

          console.log("elements", dipVacio);

          // Add bancada
          this.diputadosXBancadas.splice(1, 0, {
            key: "oposicion minoritaria",
            value: []
          });
          //Animate
          tl.set("#over", { zindex: 10 });
          tl.to([dipCondicion], 0.7, {
            fill: "#41acda"
          })
            .to(dipVacio, 1, {
              scale: 2.5,
              fill: "#d1d1d1",
              stroke: "#949494",
              x: i => {
                return (i * i + 1) * 10;
              }
            })
            .to(dipVacio, 0.7, { scale: 1, x: 0 });

          //this.$set(this.diputadosXBancadas[1], "value", superBar);
        })
        .on("leave", () => {
          //Get elements to animate
          const foo = Array.from(this.$el.querySelectorAll(".diputados"));
          const dipconcertacion = foo.filter(
            d => d.getAttribute("bancada") === "concertacion"
          );

          tl.to(dipconcertacion, 1, {
            transformOrigin: "50% 50%",
            opacity: 1,
            stagger: { amount: 0.8, from: 0 }
          }).fromTo(
            [
              "#oposicion-minoritaria.txtBackground",
              "#oposicion-minoritaria.count-container",
              "#oposicion-minoritaria.count"
            ],
            0.8,
            { transformOrigin: "50% 0%", scaleY: 0, opacity: 0 },
            { transformOrigin: "50% 0%", scaleY: 1, opacity: 1 },

            "-=0.9"
          );
        });

      this.$scrollmagic.addScene(sceneStep3);
    },
    animationStep(triEl) {
      //Set timeline
      const tl = new TimelineMax();

      //Set Scrollmagic
      const sceneStep3 = this.$scrollmagic
        .scene({
          triggerElement: triEl,
          triggerHook: 0.5,
          duration: "300px"
        })
        .on("enter", () => {
          //Get diputados disidentes MUD
          const bar = this.plotDiputados[0].diputados;
          const superBar = bar.filter(d => d.bancada === "CONCERTACION");

          //Get elements to animate
          const foo = Array.from(this.$el.querySelectorAll(".diputados"));
          const dipconcertacion = foo.filter(
            d => d.getAttribute("bancada") === "concertacion"
          );

          // Add bancada
          this.diputadosXBancadas.splice(1, 0, {
            key: "oposicion minoritaria",
            value: []
          });
          //Animate

          //this.$set(this.diputadosXBancadas[1], "value", superBar);
        })
        .on("leave", () => {
          //Get elements to animate
          const foo = Array.from(this.$el.querySelectorAll(".diputados"));
          const dipconcertacion = foo.filter(
            d => d.getAttribute("bancada") === "concertacion"
          );

          tl.to(dipconcertacion, 1, {
            transformOrigin: "50% 50%",
            opacity: 1,
            stagger: { amount: 0.8, from: 0 }
          }).fromTo(
            [
              "#oposicion-minoritaria.txtBackground",
              "#oposicion-minoritaria.count-container",
              "#oposicion-minoritaria.count"
            ],
            0.8,
            { transformOrigin: "50% 0%", scaleY: 0, opacity: 0 },
            { transformOrigin: "50% 0%", scaleY: 1, opacity: 1 },

            "-=0.9"
          );
        });

      this.$scrollmagic.addScene(sceneStep3);
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
    //Tools
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
  height: 1000vh;
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
  opacity: 0;
  text-align: initial;
  font-size: 22px;
  background-color: #eeeeeec5;
  margin: 10em 0 10em 0;
  min-height: 150px;
  width: 50vw;
  border: 1px solid #5e5e5e;
  border-radius: 14px;
}
.steps p {
  margin-left: 20px;
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
.diputados {
  /* transform: scale(1); */
  opacity: 0;
}

.txtBackground,
.count-container,
.count {
  opacity: 0;
}
.oposicion {
  text-align: center;
  color: #fdfdfd;
  background-color: #2e86ab;
  border-radius: 5px;
}
.gpp {
  text-align: center;
  color: #fdfdfd;
  background-color: #fb3640;
  border-radius: 5px;
}
.suplentes {
  text-align: center;
  color: #fdfdfd;
  background-color: #41acda;
  border-radius: 5px;
}
.vacios {
  text-align: center;
  color: black;
  background-color: #d1d1d1;
  border: 1px solid #949494;
  border-radius: 5px;
}

#oposicion-minoritaria.txtBackground,
#oposicion-minoritaria.count-container,
#oposicion-minoritaria.count {
  opacity: 0;
}
</style>
