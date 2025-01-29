<script setup lang="ts">
import { computed, ref, watch } from 'vue';
import data from './data'

const selectedOptions = ref(Object.fromEntries([...data.keys()].map((key) => [key, 0])))

const years = ref(5)

const selectedParams = new URL(document.URL).searchParams
Object.keys(selectedOptions.value).forEach((key) => {
  const selectedId = selectedParams.get(key) || data.get(key).keys().next().value
  selectedOptions.value[key] = parseInt(selectedId)
})

const totalPrice = computed(() => (
  Object.entries(selectedOptions.value).reduce((sum, [key, id]) => {
    const price = data.get(key).get(id)?.price || 0
    return sum + price
  }, 0)
))
const totalPricePerMonth = computed(() => (totalPrice.value / (years.value * 12)))
const totalPricePerMonthAndPerson = computed(() => totalPricePerMonth.value / 9)

watch(selectedOptions, () => {
  const params = new URLSearchParams(selectedOptions.value)
  window.history.replaceState(null, "", `?${params.toString()}`)
}, {
  deep: true,
})
</script>

<template>
  <header class="container">
    <h1>Kitchen4 Rechner</h1>
  </header>
  <main class="container">
    <section>
      <table>
        <tbody>
          <tr>
            <td>Gesamtkosten</td>
            <td>{{ totalPrice.toFixed(2) }} €</td>
          </tr>
          <tr>
            <td>Umgelegt auf Jahre</td>
            <td><input type="number" v-model="years"></td>
          </tr>
          <tr>
            <td>Pro Monat</td>
            <td>{{ totalPricePerMonth.toFixed(2) }} €</td>
          </tr>
          <tr>
            <td>Pro Person/Monat (durchschnittlich)</td>
            <td>{{ totalPricePerMonthAndPerson.toFixed(2) }} €</td>
          </tr>
      </tbody>
      </table>
    </section>
    <h2>Posten</h2>
    <section v-for="[category, items] in data" class="items">
      <form>
        <h3 class="item-title">{{ category }}</h3>
        <table class="striped">
          <thead>
            <tr>
              <th></th>
              <th class="desc">Beschreibung</th>
              <th class="price">Preis</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="[key, item] in items">
              <td class="radio-field">
                <input type="radio" :id="`${category}-${key}`" :name="category" :value="key" v-model.number="selectedOptions[category]">
              </td>
              <td class="desc">
                <label :for="`${category}-${key}`">{{ item.description }}</label>
              </td>
              <td class="price">{{ item.price.toFixed(2) }} €</td>
              <td><a v-if="item.link" :href="item.link" target="_blank">Link</a></td>
            </tr>
          </tbody>
        </table>
      </form>
    </section>
  </main>
</template>

<style scoped>
.items .item-title {
  text-transform: capitalize;
}
.radio-field { width: 7%; }
.desc {
  width: 70%;
}
.price {
  min-width: 6rem;
  width: 15%;
}
</style>