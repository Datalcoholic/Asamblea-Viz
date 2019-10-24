
<template>
  <div id="app">
    <div class="container1">
      <div class="chart-container">
        <h1>COMPOSICIÓN DE LA ASAMBLEA NACIONAL</h1>
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
    <!-- <div class="presentacion">
      <div class="disclaimer">
        <img src="./assets/warning.svg" alt="warning" class="warning" />
        <p>Esta visualización esta realizada solo con fines demostrativos, la recoleccion de los datos de diputados todavia esta en progreso</p>
      </div>
    </div>-->
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
        "oposicion minoritaria" || "CONCERTACION",
        "Disidentes GPP",
        "oposicion"
      ],
      newValue: [],
      oldValue: [],
      oldDipElement: [],
      newDipElement: []
    };
  },

  mounted() {
    //Setup animacion del texto
    const textElements = this.$refs.steps; // Selecciona los pasos
    this.setTextScenes(textElements);

    this.enterStep("#step_1");
    this.animationStep2("#step_2");
    this.animationStep3("#step_3");
    this.animationStep4("#step_4");

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

    // elDipDisOpo(newValue, oldValue) {
    //   const elem = Array.from(this.$el.querySelectorAll(".diputados"));
    //   this.oldDipElement = oldValue;
    //   this.newDipElement = newValue;
    // }
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
              [".txtBackground", ".count-container", ".count", ".nDip"],
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
                count: this.oldValue[0].count + 1,
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
          //duration: "200px"
        })
        .setTween(tl)
        .on("enter", () => {
          //Get diputados clandestinidad,exilio, asilo y detenido MUD
          const bar = this.plotDiputados[0].diputados;
          const sinCurulesVacios = bar.filter(
            d =>
              (d.estadoLegalPrincipal !== "asilo" ||
                d.estadoLegalPrincipal !== "exilio" ||
                d.estadoLegalPrincipal !== "clandestinidad" ||
                d.estadoLegalPrincipal !== "detenido") &&
              d.estadoLegalSuplente !== "exilio"
          );

          // Change count
          this.$set(
            this.diputadosXBancadas[0],
            "count",
            sinCurulesVacios.length
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

          // Add bancada
          if (this.diputadosXBancadas.length === 2) {
            this.diputadosXBancadas.splice(1, 0, {
              key: "OPOSICION MINORITARIA",
              value: null,
              count: 0
            });
          }
          //Animate

          tl.set(".over", { zindex: 10 });
          tl.to([dipCondicion, dipVacio], 1, {
            scale: 2.5,
            stroke: "#ffffff",
            x: i => {
              const cons = [...dipCondicion, ...dipVacio];
              const foo = (i % 5) * 65;
              return cons[i].getAttribute("x") * -1 + foo;
            },
            y: i => {
              console.log("div", ~~(i / 5));
              const cons = [...dipCondicion, ...dipVacio];
              const foo = ~~(i / 5) * 62;
              return cons[i].getAttribute("y") * -1 + foo;
            }
          })
            .staggerTo(dipCondicion, 0.7, {
              fill: "#41acda",
              stagger: { amount: 0.5 }
            })
            .staggerTo(dipVacio, 0.7, {
              fill: "#d1d1d1",
              stroke: "#949494",
              stagger: { amount: 0.5 },
              zIndex: 10
            })
            .to([dipCondicion, dipVacio], 0.7, {
              scale: 1,
              x: 0,
              y: 0
            })
            .from(this.diputadosXBancadas[0], 0.7, {
              count: this.newValue[0].count,
              roundProps: "count",
              ease: Power3.easeOut
            });
        });

      this.$scrollmagic.addScene(sceneStep3);
    },
    animationStep4(triEl) {
      //Set timeline
      const tl = new TimelineMax();

      //Set Scrollmagic
      const sceneStep4 = this.$scrollmagic
        .scene({
          triggerElement: triEl,
          triggerHook: 0.5
        })
        .setTween(tl)
        .on("enter", () => {
          //Get diputados disidentes MUD

          //this.diputadosXBancadas[1].value.push(opoMinoritaria);

          //Get elements to animate
          const foo = Array.from(this.$el.querySelectorAll(".diputados"));

          const dipApCc = foo.filter(
            d => d.getAttribute("diputado") === "Julio Cesar Reyes"
          );
          const opoMin = foo.filter(
            d =>
              d.getAttribute("bancada") === "CONCERTACION" &&
              d.getAttribute("diputado") !== "Julio Cesar Reyes"
          );
          console.log("dip", dipApCc);
          console.log("nodip", opoMin);

          const test = this.newValue[1].diputados;

          //Animate
          tl.to(opoMin, 1, {
            transformOrigin: "50% 50%",
            opacity: 1,
            stagger: { amount: 0.8, from: 0 }
          })
            .fromTo(
              [
                "#oposicion-minoritaria.txtBackground",
                "#oposicion-minoritaria.count-container",
                "#oposicion-minoritaria.count",
                "#oposicion-minoritaria.nDip"
              ],
              0.8,
              { transformOrigin: "50% 0%", scaleY: 0, opacity: 0 },
              { transformOrigin: "50% 0%", scaleY: 1, opacity: 1 },

              "-=0.9"
            )
            .to([dipApCc, opoMin], 1, {
              scale: 2.5
            })
            .staggerTo(opoMin, 1, { fill: "#772271", stagger: { amount: 0.5 } })
            .to(
              opoMin,
              1,

              {
                x: i => {
                  let resetX = opoMin[i].getAttribute("x") * -1;
                  let newpos = resetX + i * 30;
                  return newpos + this.plotDiputados[1].groupX - 65;
                },
                y: i => {
                  //console.log("gsap", opoMin[i].getAttribute("y"));
                  return opoMin[i].getAttribute("y") * -1;
                }
              }
            )

            .to([opoMin, dipApCc], 1, { scale: 1 })
            .from(this.diputadosXBancadas[0], 0.7, {
              count: this.newValue[0].count,
              roundProps: "count",
              ease: Power3.easeOut
            });

          tl.from(this.diputadosXBancadas[1], 0.7, {
            count: 0,
            roundProps: "count",
            ease: Power3.easeOut
          });

          //Diputados disidentes
          const dip = this.plotDiputados[0].diputados;
          const opoMinoritaria = dip.filter(
            d =>
              d.bancada === "CONCERTACION" && d.diputado !== "Julio Cesar Reyes"
          );

          this.$set(this.diputadosXBancadas[1], "value", opoMinoritaria);

          const opoSinMinoritaria_paso1 = dip.filter(
            d => d.bancada !== "CONCERTACION"
          );

          const opoSinMinoritaria_paso2 = dip.filter(
            d => d.diputado === "Julio Cesar Reyes"
          );

          const opoSinMinoritaria = [
            ...opoSinMinoritaria_paso1,
            ...opoSinMinoritaria_paso2
          ];

          this.$set(this.diputadosXBancadas[0], "diputados", opoSinMinoritaria);
          this.$set(
            this.diputadosXBancadas[0],
            "count",
            opoSinMinoritaria.length - 5
          );
          this.$set(this.diputadosXBancadas[1], "count", opoMinoritaria.length);
          // Add bancada
          // if (this.diputadosXBancadas.length === 3) {
          //   this.diputadosXBancadas.splice(2, 0, {
          //     key: "DISIDENTES GPP",
          //     value: null,
          //     count: 0
          //   });
          // }
        });

      this.$scrollmagic.addScene(sceneStep4);
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
            //fill: this.bancadaColorScale(obj.bancada2.toLowerCase())
            fill: Object.keys(obj).includes("bancada2")
              ? this.bancadaColorScale(obj.bancada2.toLowerCase())
              : this.bancadaColorScale(
                  obj.bancada.toLowerCase() === "concertacion"
                    ? "oposicion minoritaria"
                    : obj.bancada.toLowerCase()
                )
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
  margin-left: 120px;
  position: sticky;
  top: 0;
}
svg {
  background: rgb(245, 245, 245);
}

.count-container {
  font-size: 32px;
  font-weight: 600;
}
.count {
  font-size: 60px;
  font-weight: 600;
}

.nDip {
  font-size: 18px;
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
#om {
  text-align: center;
  color: #fdfdfd;
  background-color: #772271;
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

.presentacion {
  position: relative;
}
.presentacion:after {
  content: " ";
  z-index: 10;
  display: block;
  position: absolute;
  height: 100vh;
  top: 0;
  left: 0;
  right: 0;
  background-color: #08080886;
}
.warning {
  height: 50px;
  width: 50px;
}

.disclaimer {
  z-index: 20;
  background: #ffffff;
  height: 120px;
  width: 500px;
  border: 1px solid black;
}
</style>
