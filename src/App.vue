<script setup>
import { computed, onMounted, ref } from 'vue'

const countries = ref([])
const search = ref('')
const region = ref('All')
const selectedCountry = ref(null)
const loading = ref(true)
const error = ref('')

const regions = ['All', 'Africa', 'Americas', 'Asia', 'Europe', 'Oceania']

const filteredCountries = computed(() => {
  const normalizedSearch = search.value.trim().toLowerCase()

  return countries.value
    .filter((country) => region.value === 'All' || country.region === region.value)
    .filter((country) => {
      if (!normalizedSearch) {
        return true
      }

      const values = [
        country.name.common,
        country.name.official,
        country.capital?.[0] ?? '',
      ]

      return values.some((value) => value.toLowerCase().includes(normalizedSearch))
    })
    .slice(0, 24)
})

const stats = computed(() => ({
  total: countries.value.length,
  filtered: filteredCountries.value.length,
  selectedRegion: region.value === 'All' ? 'Global' : region.value,
}))

const borderCountries = computed(() => {
  if (!selectedCountry.value?.borders?.length) {
    return []
  }

  return selectedCountry.value.borders
    .map((borderCode) => countries.value.find((country) => country.cca3 === borderCode))
    .filter(Boolean)
})

function formatPopulation(population) {
  return new Intl.NumberFormat('en-US').format(population)
}

async function loadCountries() {
  loading.value = true
  error.value = ''

  try {
    const response = await fetch(
      'https://restcountries.com/v3.1/all?fields=name,flags,population,region,subregion,capital,languages,borders,cca3,timezones',
    )

    if (!response.ok) {
      throw new Error('Unable to load country data.')
    }

    const data = await response.json()
    countries.value = data.sort((a, b) => a.name.common.localeCompare(b.name.common))
    selectedCountry.value = countries.value[0]
  } catch (requestError) {
    error.value = requestError.message || 'Unable to load country data.'
  } finally {
    loading.value = false
  }
}

onMounted(loadCountries)
</script>

<template>
  <main class="min-h-screen px-4 py-8 text-slate-100">
    <div class="mx-auto max-w-7xl space-y-6">
      <section class="rounded-[32px] border border-white/10 bg-white/6 p-6 shadow-2xl shadow-sky-950/30 backdrop-blur xl:p-8">
        <div class="flex flex-col gap-6 lg:flex-row lg:items-end lg:justify-between">
          <div class="max-w-3xl">
            <p class="mb-2 text-xs font-semibold uppercase tracking-[0.25em] text-sky-300">Vue 3 + public API</p>
            <h1 class="text-4xl font-black tracking-tight text-white md:text-5xl">Country Explorer</h1>
            <p class="mt-3 text-sm leading-6 text-slate-300 md:text-base">
              Browse countries, search by name or capital, and open a compact details panel powered by the REST Countries API.
            </p>
          </div>

          <div class="grid gap-3 sm:grid-cols-3">
            <div class="rounded-3xl border border-white/10 bg-slate-950/45 px-4 py-3">
              <p class="text-xs uppercase tracking-[0.2em] text-slate-400">Loaded</p>
              <p class="mt-2 text-2xl font-bold text-white">{{ loading ? '...' : stats.total }}</p>
            </div>
            <div class="rounded-3xl border border-white/10 bg-slate-950/45 px-4 py-3">
              <p class="text-xs uppercase tracking-[0.2em] text-slate-400">Showing</p>
              <p class="mt-2 text-2xl font-bold text-white">{{ loading ? '...' : stats.filtered }}</p>
            </div>
            <div class="rounded-3xl border border-white/10 bg-slate-950/45 px-4 py-3">
              <p class="text-xs uppercase tracking-[0.2em] text-slate-400">Region</p>
              <p class="mt-2 text-2xl font-bold text-white">{{ stats.selectedRegion }}</p>
            </div>
          </div>
        </div>
      </section>

      <section class="grid gap-6 xl:grid-cols-[1.2fr_0.8fr]">
        <div class="space-y-5">
          <div class="grid gap-4 md:grid-cols-[1fr_auto]">
            <label class="rounded-3xl border border-white/10 bg-slate-950/40 p-4">
              <span class="mb-2 block text-sm font-medium text-slate-300">Search country or capital</span>
              <input
                v-model="search"
                class="w-full rounded-2xl border border-white/10 bg-slate-900/80 px-4 py-3 text-white outline-none transition focus:border-sky-400"
                placeholder="Try Japan, Sofia, or Brazil"
              />
            </label>

            <label class="rounded-3xl border border-white/10 bg-slate-950/40 p-4">
              <span class="mb-2 block text-sm font-medium text-slate-300">Region</span>
              <select
                v-model="region"
                class="w-full rounded-2xl border border-white/10 bg-slate-900/80 px-4 py-3 text-white outline-none transition focus:border-sky-400"
              >
                <option v-for="option in regions" :key="option" :value="option">{{ option }}</option>
              </select>
            </label>
          </div>

          <div v-if="loading" class="rounded-3xl border border-white/10 bg-slate-950/40 p-8 text-center text-slate-300">
            Loading countries...
          </div>

          <div v-else-if="error" class="rounded-3xl border border-rose-300/20 bg-rose-400/10 p-6 text-rose-100">
            {{ error }}
          </div>

          <div v-else class="grid gap-4 md:grid-cols-2">
            <button
              v-for="country in filteredCountries"
              :key="country.cca3"
              type="button"
              class="rounded-3xl border p-4 text-left transition hover:-translate-y-0.5"
              :class="
                selectedCountry?.cca3 === country.cca3
                  ? 'border-sky-300/60 bg-sky-300/12 shadow-lg shadow-sky-900/20'
                  : 'border-white/10 bg-slate-950/40 hover:border-white/20 hover:bg-white/6'
              "
              @click="selectedCountry = country"
            >
              <div class="flex gap-4">
                <img :src="country.flags.svg" :alt="`${country.name.common} flag`" class="h-14 w-20 rounded-xl object-cover" />
                <div class="min-w-0">
                  <p class="truncate text-lg font-bold text-white">{{ country.name.common }}</p>
                  <p class="text-sm text-slate-400">{{ country.capital?.[0] || 'No capital listed' }}</p>
                  <div class="mt-3 flex flex-wrap gap-2 text-xs">
                    <span class="rounded-full bg-white/8 px-3 py-1 text-slate-200">{{ country.region }}</span>
                    <span class="rounded-full bg-white/8 px-3 py-1 text-slate-200">{{ formatPopulation(country.population) }}</span>
                  </div>
                </div>
              </div>
            </button>
          </div>
        </div>

        <aside class="rounded-[32px] border border-white/10 bg-slate-950/40 p-6 backdrop-blur">
          <template v-if="selectedCountry">
            <img
              :src="selectedCountry.flags.svg"
              :alt="`${selectedCountry.name.common} flag`"
              class="h-44 w-full rounded-3xl object-cover"
            />

            <div class="mt-5">
              <p class="text-xs font-semibold uppercase tracking-[0.25em] text-sky-300">Selected country</p>
              <h2 class="mt-2 text-3xl font-black text-white">{{ selectedCountry.name.common }}</h2>
              <p class="mt-2 text-sm leading-6 text-slate-300">{{ selectedCountry.name.official }}</p>
            </div>

            <dl class="mt-6 grid gap-4">
              <div class="rounded-2xl border border-white/10 bg-white/5 p-4">
                <dt class="text-xs uppercase tracking-[0.2em] text-slate-400">Capital</dt>
                <dd class="mt-2 text-sm leading-6 text-slate-100">{{ selectedCountry.capital?.[0] || 'No capital listed' }}</dd>
              </div>
              <div class="rounded-2xl border border-white/10 bg-white/5 p-4">
                <dt class="text-xs uppercase tracking-[0.2em] text-slate-400">Population</dt>
                <dd class="mt-2 text-sm leading-6 text-slate-100">{{ formatPopulation(selectedCountry.population) }}</dd>
              </div>
              <div class="rounded-2xl border border-white/10 bg-white/5 p-4">
                <dt class="text-xs uppercase tracking-[0.2em] text-slate-400">Subregion</dt>
                <dd class="mt-2 text-sm leading-6 text-slate-100">{{ selectedCountry.subregion || 'N/A' }}</dd>
              </div>
              <div class="rounded-2xl border border-white/10 bg-white/5 p-4">
                <dt class="text-xs uppercase tracking-[0.2em] text-slate-400">Timezone</dt>
                <dd class="mt-2 text-sm leading-6 text-slate-100">{{ selectedCountry.timezones?.[0] || 'N/A' }}</dd>
              </div>
              <div class="rounded-2xl border border-white/10 bg-white/5 p-4">
                <dt class="text-xs uppercase tracking-[0.2em] text-slate-400">Languages</dt>
                <dd class="mt-2 text-sm leading-6 text-slate-100">
                  {{ selectedCountry.languages ? Object.values(selectedCountry.languages).join(', ') : 'N/A' }}
                </dd>
              </div>
            </dl>

            <div class="mt-6">
              <p class="text-sm font-semibold uppercase tracking-[0.2em] text-slate-400">Border countries</p>
              <div class="mt-3 flex flex-wrap gap-2">
                <span
                  v-for="country in borderCountries"
                  :key="country.cca3"
                  class="rounded-full border border-white/10 bg-white/5 px-3 py-2 text-sm text-slate-200"
                >
                  {{ country.name.common }}
                </span>
                <span v-if="borderCountries.length === 0" class="text-sm text-slate-400">No land borders</span>
              </div>
            </div>
          </template>
        </aside>
      </section>
    </div>
  </main>
</template>
