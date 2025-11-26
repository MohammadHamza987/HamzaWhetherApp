<template>
  <div id="app" class="min-h-screen bg-linear-to-br from-blue-400 via-blue-500 to-purple-600 dark:from-slate-900 dark:via-blue-900 dark:to-purple-900 p-4 md:p-8">
    <div class="max-w-2xl mx-auto">
      <!-- Header -->
      <div class="text-center mb-8">
        <h1 class="text-4xl md:text-5xl font-bold text-white mb-2">
          Weather App
        </h1>
        <p class="text-blue-100 dark:text-blue-200">
          Check the weather in any city worldwide
        </p>
      </div>

      <!-- Search Box -->
      <div class="relative">
        <Card class="backdrop-blur-lg bg-white/90 dark:bg-slate-800/90 border-0 shadow-2xl">
          <CardContent class="pt-6">
            <div class="flex gap-2">
              <div class="flex-1 relative">
                <Input
                  v-model="query"
                  autofocus
                  @keypress="fetchWhether"
                  @input="fetchCitySuggestions"
                  @focus="() => query.length >= 2 && suggestions.length > 0 && (showSuggestions = true)"
                  placeholder="Search for a city..."
                  class="h-12 text-lg"
                />
              </div>

              <Button 
                @click="searchCity" 
                :disabled="loading" 
                size="lg"
                class="px-6"
              >
                <span v-if="!loading">Search</span>
                <span v-else>Loading...</span>
              </Button>
            </div>
          </CardContent>
        </Card>

        <!-- City Suggestions Dropdown -->
        <Card 
          v-if="showSuggestions && suggestions.length > 0" 
          class="absolute top-full left-0 right-0 mt-2 z-50 bg-white dark:bg-slate-800 border-2 border-primary/20 shadow-2xl max-h-64 overflow-y-auto"
        >
          <CardContent class="p-0">
            <div
              v-for="city in suggestions"
              :key="city.lat + city.lon"
              @click="selectCity(city)"
              class="px-4 py-3 hover:bg-accent hover:text-accent-foreground cursor-pointer transition-colors border-b border-border last:border-b-0"
            >
              <div class="font-medium">{{ city.name }}</div>
              <div class="text-sm text-muted-foreground">
                {{ city.state ? `${city.state}, ` : '' }}{{ city.country }}
              </div>
            </div>
          </CardContent>
        </Card>
      </div>

      <!-- Loading Skeleton -->
      <div v-if="loading" class="mt-6 space-y-4">
        <Skeleton class="h-64 w-full rounded-xl" />
      </div>

      <!-- Weather Card -->
      <Card v-if="whether.main && !loading" class="mt-6 backdrop-blur-lg bg-white/95 dark:bg-slate-800/95 border-0 shadow-2xl overflow-hidden">
        <CardHeader class="pb-3">
          <div class="flex items-start justify-between">
            <div>
              <CardTitle class="text-3xl">
                {{ whether.name }}, {{ whether.sys.country }}
              </CardTitle>
              <p class="text-sm text-muted-foreground mt-1">
                {{ datebuilder() }}
              </p>
            </div>
            <Badge variant="secondary" class="text-base px-3 py-1">
              {{ whether.weather[0].main }}
            </Badge>
          </div>
        </CardHeader>

        <CardContent class="space-y-6">
          <!-- Main Temperature Display -->
          <div class="flex items-center justify-between">
            <div>
              <div class="text-7xl font-bold bg-linear-to-br from-blue-600 to-purple-600 bg-clip-text text-transparent">
                {{ Math.round(whether.main.temp) }}°C
              </div>
              <p class="text-lg text-muted-foreground mt-2 capitalize">
                {{ whether.weather[0].description }}
              </p>
            </div>
            <div class="text-6xl">
              {{ getWeatherIcon(whether.weather[0].main) }}
            </div>
          </div>

          <!-- Weather Details Grid -->
          <div class="grid grid-cols-2 gap-4 pt-4 border-t">
            <div class="space-y-1">
              <p class="text-sm text-muted-foreground">Feels Like</p>
              <p class="text-2xl font-semibold">{{ Math.round(whether.main.feels_like) }}°C</p>
            </div>
            <div class="space-y-1">
              <p class="text-sm text-muted-foreground">Humidity</p>
              <p class="text-2xl font-semibold">{{ whether.main.humidity }}%</p>
            </div>
            <div class="space-y-1">
              <p class="text-sm text-muted-foreground">Wind Speed</p>
              <p class="text-2xl font-semibold">{{ Math.round(whether.wind.speed) }} m/s</p>
            </div>
            <div class="space-y-1">
              <p class="text-sm text-muted-foreground">Pressure</p>
              <p class="text-2xl font-semibold">{{ whether.main.pressure }} hPa</p>
            </div>
          </div>

          <!-- Min/Max Temperature -->
          <div class="flex gap-4 pt-4 border-t">
            <div class="flex-1 text-center p-3 rounded-lg bg-blue-50 dark:bg-slate-700">
              <p class="text-sm text-muted-foreground mb-1">Min Temp</p>
              <p class="text-xl font-bold text-blue-600 dark:text-blue-400">
                {{ Math.round(whether.main.temp_min) }}°C
              </p>
            </div>
            <div class="flex-1 text-center p-3 rounded-lg bg-red-50 dark:bg-slate-700">
              <p class="text-sm text-muted-foreground mb-1">Max Temp</p>
              <p class="text-xl font-bold text-red-600 dark:text-red-400">
                {{ Math.round(whether.main.temp_max) }}°C
              </p>
            </div>
          </div>
        </CardContent>
      </Card>

      <!-- Error Alert -->
      <Alert v-if="whether.cod === '404'" variant="destructive" class="mt-6 text-white bg-red-600">
        <AlertTitle class="text-lg font-semibold">City Not Found</AlertTitle>
        <AlertDescription>
          We couldn't find weather data for "{{ query }}". Please check the spelling and try again.
        </AlertDescription>
      </Alert>

      <!-- Welcome Message -->
      <Card v-if="!whether.main && !whether.cod && !loading" class="mt-6 backdrop-blur-lg bg-white/90 dark:bg-slate-800/90 border-0 shadow-xl">
        <CardContent class="py-12 text-center">
          <div class="text-6xl mb-4">🌤️</div>
          <h2 class="text-2xl font-semibold mb-2">Welcome to Weather App</h2>
          <p class="text-muted-foreground">
            Enter a city name to get started
          </p>
        </CardContent>
      </Card>
    </div>
  </div>
</template>


<script setup>
import { ref, onMounted } from 'vue'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card'
import { Alert, AlertTitle, AlertDescription } from '@/components/ui/alert'
import { Skeleton } from '@/components/ui/skeleton'
import { Badge } from '@/components/ui/badge'

const api_key = '726d6983f19222589c387df030dd4372'
const url_base = 'https://api.openweathermap.org/data/2.5/'
const query = ref('')
const whether = ref({})
const loading = ref(false)
const suggestions = ref([])
const showSuggestions = ref(false)
let debounceTimer = null

// Load city from URL on mount
onMounted(() => {
  const urlParams = new URLSearchParams(window.location.search)
  const cityFromUrl = urlParams.get('city')
  if (cityFromUrl) {
    query.value = cityFromUrl
    searchCity()
  }
})

function fetchWhether(e) {
  if (e.key === 'Enter') {
    searchCity()
  }
}

async function searchCity() {
  if (!query.value.trim()) return
  loading.value = true
  whether.value = {}
  showSuggestions.value = false
  
  // Update URL with city parameter
  const url = new URL(window.location)
  url.searchParams.set('city', query.value)
  window.history.pushState({}, '', url)
  
  try {
    const res = await fetch(`${url_base}weather?q=${query.value}&units=metric&APPID=${api_key}`)
    const results = await res.json()
    whether.value = results
  } catch (error) {
    console.error('Error fetching weather:', error)
  } finally {
    loading.value = false
  }
}

function selectCity(city) {
  query.value = city.name
  showSuggestions.value = false
  searchCity()
}

function datebuilder() {
  const d = new Date()
  const months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"]
  const days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]
  const day = days[d.getDay()]
  const date = d.getDate()
  const month = months[d.getMonth()]
  const year = d.getFullYear()
  return `${day} ${date} ${month} ${year}`
}

function getWeatherIcon(weather) {
  const icons = {
    'Clear': '☀️',
    'Clouds': '☁️',
    'Rain': '🌧️',
    'Drizzle': '🌦️',
    'Thunderstorm': '⛈️',
    'Snow': '❄️',
    'Mist': '🌫️',
    'Smoke': '🌫️',
    'Haze': '🌫️',
    'Dust': '🌫️',
    'Fog': '🌫️',
    'Sand': '🌫️',
    'Ash': '🌫️',
    'Squall': '💨',
    'Tornado': '🌪️'
  }
  return icons[weather] || '🌤️'
}

async function fetchCitySuggestions() {
  // Clear previous timer
  if (debounceTimer) {
    clearTimeout(debounceTimer)
  }

  if (query.value.trim().length < 2) {
    suggestions.value = []
    showSuggestions.value = false
    return
  }

  // Debounce: Wait 500ms before making API call
  debounceTimer = setTimeout(async () => {
    try {
      const res = await fetch(
        `https://api.openweathermap.org/geo/1.0/direct?q=${query.value}&limit=5&appid=${api_key}`
      )
      const data = await res.json()
      suggestions.value = data
      showSuggestions.value = data.length > 0
    } catch (error) {
      console.error('Error fetching city suggestions:', error)
      suggestions.value = []
      showSuggestions.value = false
    }
  }, 500) // 500ms debounce delay
}

// Close suggestions when clicking outside
if (typeof window !== 'undefined') {
  document.addEventListener('click', (e) => {
    if (!e.target.closest('.relative')) {
      showSuggestions.value = false
    }
  })
}
</script>

<style scoped>
.card {
  transition: transform 0.3s ease;
}

.card:hover {
  transform: translateY(-2px);
}
</style>





