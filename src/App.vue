<script setup lang="ts">
import { computed, ref, watch } from 'vue';

const data = new Map()
  .set('ikea', new Map()
    .set(1, { description: 'Unterbau Schränke und Schubladen', price: 3000, link: 'https://www.ikea.com/de/de/cat/metod-einzelteile-23598/' })
  )
  .set('arbeitsplatte', new Map()
    .set(1, { description: 'Eiche Leimholz Keilgezinkt', price: 550, link: 'https://www.leipziger-kistenfabrik.de/produkte/plattenwerkstoffe/leimholz/' })
    .set(2, { description: 'Nussbaum Leimholz Keilgezinkt', price: 650, link: 'https://www.leipziger-kistenfabrik.de/produkte/plattenwerkstoffe/leimholz/' })
  )

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
    <h1>Kitchen4</h1>
  </header>
  <main class="container">
    <section>
      <table>
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
          <td>Pro Person (durchschnittlich)</td>
          <td>{{ totalPricePerMonthAndPerson.toFixed(2) }} €</td>
        </tr>
      </table>
    </section>
    <h2>Posten</h2>
    <section v-for="[category, items] in data">
      <form>
        <h3>{{ category }}</h3>
        <table class="striped">
          <thead>
            <th></th>
            <th class="desc">Beschreibung</th>
            <th class="price">Preis</th>
            <th>Link</th>
          </thead>
          <tbody>
            <tr v-for="[key, item] in items">
              <td>
                <input type="radio" :id="`${category}-${key}`" :name="category" :value="key" v-model.number="selectedOptions[category]">
              </td>
              <td class="desc">
                <label :for="`${category}-${key}`">{{ item.description }}</label>
              </td>
              <td class="price">{{ item.price }} €</td>
              <td><a :href="item.link" target="_blank">Link</a></td>
            </tr>
          </tbody>
        </table>
      </form>
    </section>
  </main>
</template>

<style scoped>
.desc {
  width: 70%;
}
.price {
  min-width: 6rem;
}
</style>