<template>
  <div class="min-h-screen bg-gray-950 text-white p-8">
    <!-- Header -->
    <div class="mb-8">
      <h1 class="text-4xl font-bold text-green-400">🚀 HackPilot</h1>
      <p class="text-gray-400 mt-1">Your AI-powered hackathon copilot</p>
    </div>

    <!-- Stats Row -->
    <div class="grid grid-cols-3 gap-4 mb-8">
      <div class="bg-gray-900 rounded-xl p-4 border border-gray-800">
        <p class="text-gray-400 text-sm">Tracked Projects</p>
        <p class="text-3xl font-bold text-white mt-1">{{ stats.tracked_projects }}</p>
      </div>
      <div class="bg-gray-900 rounded-xl p-4 border border-gray-800">
        <p class="text-gray-400 text-sm">Upcoming Deadlines</p>
        <p class="text-3xl font-bold text-yellow-400 mt-1">{{ stats.upcoming_deadlines }}</p>
      </div>
      <div class="bg-gray-900 rounded-xl p-4 border border-gray-800">
        <p class="text-gray-400 text-sm">Matched Hackathons</p>
        <p class="text-3xl font-bold text-green-400 mt-1">{{ stats.matched_hackathons }}</p>
      </div>
    </div>

    <!-- Deadlines -->
    <div class="mb-8">
      <h2 class="text-xl font-semibold mb-4 text-gray-200">⏰ Deadlines</h2>
      <div v-if="deadlines.length === 0" class="bg-gray-900 rounded-xl p-4 border border-gray-800 text-gray-400">
        No deadlines tracked yet. Add one below!
      </div>
      <div class="space-y-3">
        <div v-for="d in deadlines" :key="d.name"
          :class="borderColor(d.status)"
          class="bg-gray-900 rounded-xl p-4 border flex justify-between items-center">
          <div>
            <p class="font-semibold text-white">{{ d.name }}</p>
            <p class="text-sm text-gray-400">{{ d.url }}</p>
          </div>
          <span :class="badgeColor(d.status)" class="px-3 py-1 rounded-full text-sm">
            {{ d.label }}
          </span>
        </div>
      </div>
    </div>

    <!-- Add Deadline Form -->
    <div class="mb-8 bg-gray-900 rounded-xl p-6 border border-gray-800">
      <h2 class="text-xl font-semibold mb-4 text-gray-200">➕ Track New Deadline</h2>
      <div class="grid grid-cols-2 gap-4">
        <input v-model="form.name" placeholder="Hackathon name"
          class="bg-gray-800 rounded-lg p-3 text-white border border-gray-700 focus:outline-none focus:border-green-500" />
        <input v-model="form.url" placeholder="URL"
          class="bg-gray-800 rounded-lg p-3 text-white border border-gray-700 focus:outline-none focus:border-green-500" />
        <input v-model="form.deadline" type="datetime-local"
          class="bg-gray-800 rounded-lg p-3 text-white border border-gray-700 focus:outline-none focus:border-green-500" />
        <button @click="addDeadline"
          class="bg-green-600 hover:bg-green-500 rounded-lg p-3 text-white font-semibold transition">
          Add Deadline
        </button>
      </div>
    </div>

    <!-- Matched Hackathons -->
    <div>
      <h2 class="text-xl font-semibold mb-4 text-gray-200">🔍 Matched Hackathons</h2>
      <div class="space-y-3">
        <div v-for="h in hackathons" :key="h.name"
          class="bg-gray-900 rounded-xl p-4 border border-gray-800 flex justify-between items-center">
          <div>
            <p class="font-semibold text-white">{{ h.name }}</p>
            <p class="text-sm text-gray-400">{{ h.themes.join(', ') }}</p>
          </div>
          <div class="text-right">
            <a :href="h.url" target="_blank"
              class="bg-green-900 text-green-300 px-3 py-1 rounded-full text-sm hover:bg-green-800">
              View →
            </a>
            <p class="text-xs text-gray-500 mt-1">{{ h.deadline }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const API = 'http://localhost:8002'

const stats = ref({ tracked_projects: 0, upcoming_deadlines: 0, matched_hackathons: 0 })
const deadlines = ref([])
const hackathons = ref([])
const form = ref({ name: '', url: '', deadline: '' })

async function loadData() {
  const [s, d, h] = await Promise.all([
    fetch(`${API}/api/stats`).then(r => r.json()),
    fetch(`${API}/api/deadlines`).then(r => r.json()),
    fetch(`${API}/api/hackathons`).then(r => r.json()),
  ])
  stats.value = s
  deadlines.value = d
  hackathons.value = h
}

async function addDeadline() {
  if (!form.value.name || !form.value.deadline) return
  await fetch(`${API}/api/deadlines/add`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(form.value),
  })
  form.value = { name: '', url: '', deadline: '' }
  await loadData()
}

function borderColor(status) {
  return {
    missed: 'border-gray-600',
    critical: 'border-red-800',
    urgent: 'border-orange-800',
    warning: 'border-yellow-800',
    ok: 'border-green-800',
  }[status] || 'border-gray-800'
}

function badgeColor(status) {
  return {
    missed: 'bg-gray-800 text-gray-400',
    critical: 'bg-red-900 text-red-300',
    urgent: 'bg-orange-900 text-orange-300',
    warning: 'bg-yellow-900 text-yellow-300',
    ok: 'bg-green-900 text-green-300',
  }[status] || 'bg-gray-800 text-gray-400'
}

onMounted(loadData)
</script>
