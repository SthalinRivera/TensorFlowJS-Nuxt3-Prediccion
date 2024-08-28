<template>
    <div>
      <table >
        <tbody>
          <tr>
            <td>Repeticiones</td>
            <td><input type="number" v-model="repeticiones" /></td>
          </tr>
          <tr>
            <td>Valor de x</td>
            <td><input type="number" v-model="nuevoValX" /></td>
          </tr>
          <tr>
            <td></td>
            <td><input type="button" value="Calcular" @click="learnLinear" /></td>
          </tr>
          <tr>
            <td>Valor de Y</td>
            <td><span>{{ valYResultado }}</span></td>
          </tr>
          <tr>
            <td>Época</td>
            <td><span>{{ epocaActual }}</span></td>
          </tr>
        </tbody>
      </table>
      <canvas ref="myChart" width="400" height="400"></canvas>
    </div>
  </template>
  
  <script setup >

  // Importamos TensorFlow.js y Chart.js
  import * as tf from '@tensorflow/tfjs';
  import Chart from 'chart.js/auto';
  import { ref, onMounted } from 'vue';
  
  // Variables reactivas para el estado
  const repeticiones = ref(100);
  const nuevoValX = ref(10);
  const valYResultado = ref(0);
  const epocaActual = ref(0);
  
  // Datos iniciales
  const valX = [1, 2, 3, 4, 5, 6]; //dias 
  const valY = [160, 150, 120, 100, 105, 90]; //precioss
  const datosGrafica = deArrayAMatriz(valX, valY);
  
  // Referencia para el canvas
  const myChart = ref(null);
  let grafica = null;
  
  // Función para convertir los arrays a formato de matriz
  function deArrayAMatriz(arx, ary) {
    const data = [];
    for (let i = 0; i < arx.length; i++) {
      data.push({ x: arx[i], y: ary[i] });
    }
    return data;
  }
  
  // Inicializar la gráfica cuando el componente esté montado
  onMounted(() => {
    grafica = new Chart(myChart.value, {
      type: 'scatter',
      data: {
        datasets: [{
          label: "Ventas",
          data: datosGrafica,
          borderColor: "red",
        }]
      },
      options: {
        responsive: false
      }
    });
  });
  
  // Función de aprendizaje de TensorFlow
  async function learnLinear() {
    // Definimos el modelo de regresión lineal
    const model = tf.sequential();
    model.add(tf.layers.dense({ units: 1, inputShape: [1] }));
    model.compile({ loss: 'meanSquaredError', optimizer: 'sgd' });
  
    // Creamos los tensores
    const xs = tf.tensor2d(valX, [6, 1]);
    const ys = tf.tensor2d(valY, [6, 1]);
  
    // Ciclo de entrenamiento
    for (let i = 0; i < repeticiones.value; i++) {
      await model.fit(xs, ys, { epochs: 1 });
      const prediccionY = model.predict(tf.tensor2d([nuevoValX.value], [1, 1])).dataSync()[0];
      valYResultado.value = prediccionY;
      epocaActual.value = i + 1;
  
      // Actualizar gráfica
      datosGrafica.push({ x: nuevoValX.value, y: prediccionY });
      grafica.data.datasets[0].data = datosGrafica;
      grafica.update();
    }
  }
  </script>

  <style scoped>
  /* Aquí puedes agregar estilos si lo necesitas */
  </style>
  