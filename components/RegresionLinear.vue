<template>
    <div>
      <table>
        <tbody>
          <tr>
            <td>Repeticiones</td>
            <td><input type="number" v-model="repeticiones" /></td>
          </tr>
          <tr>
            <td>Seleccionar fecha</td>
            <td><input type="date" v-model="fechaSeleccionada" /></td>
          </tr>
          <tr>
            <td>Valor de x (días desde el inicio)</td>
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
      <canvas ref="myChart" width="900" height="400"></canvas>
    </div>
  </template>
  
  <script setup>
  
  // Importamos TensorFlow.js y Chart.js
  import * as tf from '@tensorflow/tfjs';
  import Chart from 'chart.js/auto';
  import { ref, onMounted, watch } from 'vue';
  
  // Variables reactivas para el estado
  const repeticiones = ref(100);
  const fechaSeleccionada = ref('');
  const nuevoValX = ref(10); // días desde el inicio, se calcula automáticamente al seleccionar fecha
  const valYResultado = ref(0);
  const epocaActual = ref(0);
  
  // Datos iniciales
  const valX = [1, 2, 3, 4, 5, 6]; // días desde el inicio
  const valY = [140, 120, 100, 98, 95, 90]; // precios
  const datosGrafica = deArrayAMatriz(valX, valY);
  const datosPrediccion = [];
  
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
        datasets: [
          {
            label: "Datos Reales",
            data: datosGrafica,
            borderColor: "red",
            showLine: true,
          },
          {
            label: "Predicciones",
            data: datosPrediccion,
            borderColor: "blue",
            showLine: true,
          }
        ]
      },
      options: {
        responsive: false,
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
      epocaActual.value = i + 1;
    }
  
    // Actualizar predicciones en la gráfica
    datosPrediccion.length = 0; // Limpiar las predicciones anteriores
  
    for (let x = 1; x <= nuevoValX.value; x++) {
      const prediccionY = model.predict(tf.tensor2d([x], [1, 1])).dataSync()[0];
      datosPrediccion.push({ x, y: prediccionY });
      if (x === nuevoValX.value) {
        valYResultado.value = prediccionY; // Actualizar el valor de Y resultante
      }
    }
  
    grafica.data.datasets[1].data = datosPrediccion;
    grafica.update();
  }
  
  // Actualizar nuevoValX cuando se selecciona una fecha
  watch(fechaSeleccionada, (nuevaFecha) => {
    const fechaInicio = new Date("2024-01-01"); // Ajustar según la fecha de inicio real
    const fechaSeleccionadaDate = new Date(nuevaFecha);
    const diferenciaEnDias = Math.floor((fechaSeleccionadaDate - fechaInicio) / (1000 * 60 * 60 * 24));
    nuevoValX.value = diferenciaEnDias;
  });
  
  </script>
  
  <style scoped>
  /* Aquí puedes agregar estilos si lo necesitas */
  </style>
  