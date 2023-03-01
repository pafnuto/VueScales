<script setup>
import { ref, shallowRef, computed, watch, nextTick } from 'vue';
import { useStore } from 'vuex';
import Chart from 'chart.js/auto'

const weights = ref([])
const weightChartRef = ref(null)
const weightChart = shallowRef(null)
const weightInput = ref(0)
const currentWeight = computed(() => {
	return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 }
})

const addWeight = () => {
	weights.value.push({
		weight: weightInput.value,
		date: new Date().getTime()
	})
}

watch(weights, (newWeights) => {
	const ws = [...newWeights]
	if (weightChart.value) {
		weightChart.value.data.labels = ws
			.sort((a, b) => a.date - b.date)
			.map(weight => new Date(weight.date).toLocaleDateString() )
			.slice(-7)
		weightChart.value.data.datasets[0].data = ws
			.sort((a, b) => a.date - b.date)
			.map(weight => weight.weight)
			.slice(-7)
		weightChart.value.update()

		return
	}
	nextTick(() => {
		weightChart.value = new Chart(weightChartRef.value.getContext('2d'), {
			type: 'line',
			data: {
				labels: ws
					.sort((a, b) => a.date - b.date)
					.map(weight => new Date(weight.date).toLocaleDateString()),
				datasets: [
					{
						label: 'Вес',
						data: ws
							.sort((a, b) => a.date - b.date)
							.map(weight => weight.weight),
						backgroundColor: 'gba(255, 105, 180, 0.2)',
						borderColor: 'rgba(255, 105, 180, 1)',
						borderWidth: 1,
						fill: true
					}]}	
		})
	})
}, { deep: true })

</script>

<template>
<main>
  <h1>График веса</h1>

<div class="currWeight">
  <span>{{ currentWeight.weight }}</span>
  <small> Вес(кг)</small>
</div>

<form @submit.prevent="addWeight">
  <input 
    type="number"
    step="0.1"
    v-model="weightInput" />

  <input	
    type="submit"
    value="Добавить вес" />
</form>

<div v-if="weights && weights.length > 0">
<h2> За последнюю неделю</h2>

<div class="weightBody">
  <canvas ref="weightChartRef"></canvas>
</div>

<div class="weightHist">
  <h2>История взвешеваний</h2>
  <ol>
    <li v-for="weight in weights">
      <span>{{ weight.weight }} кг </span>
      <span>
        {{ new Date(weight.date).toLocaleDateString() }}
      </span>
    </li>
  </ol>
</div>
</div>

</main>
</template>

<style scoped>

</style>
