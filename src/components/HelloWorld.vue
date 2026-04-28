<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import logo from '/glitch_twitch.svg'

const USERNAME = 'cretetion'
const API_BASE = 'https://twitch-proxy.freecodecamp.rocks/twitch-api'

interface UserResponse {
  display_name?: string
  name?: string
  logo?: string | null
  error?: string
  message?: string
}

interface StreamResponse {
  stream?: {
    game?: string
    channel?: {
      status?: string
    }
  } | null
  data?: Array<{
    game_name?: string
    title?: string
  }>
}

interface StreamerState {
  name: string
  avatar: string
  isLive: boolean
  game: string
  title: string
}

const streamer = ref<StreamerState>({
  name: 'cretetion',
  avatar: '/avatar.png',
  isLive: false,
  game: '',
  title: '',
})

const loading = ref<boolean>(true)
const errorMessage = ref<string>('')

const subtitle = computed<string>(() => {
  if (loading.value) {
    return 'Loading...'
  }

  if (errorMessage.value) {
    return errorMessage.value
  }

  if (!streamer.value.isLive) {
    return 'Offline'
  }

  return `${streamer.value.game || 'Unknown'}: ${streamer.value.title || 'Live now'}`
})

const loadStreamer = async (): Promise<void> => {
  try {
    const [userRes, streamRes] = await Promise.all([
      fetch(`${API_BASE}/users/${USERNAME}`),
      fetch(`${API_BASE}/streams/${USERNAME}`),
    ])

    const user = (await userRes.json()) as UserResponse
    const stream = (await streamRes.json()) as StreamResponse

    if (user.error) {
      throw new Error(user.message || 'Streamer not found')
    }

    streamer.value.name = user.display_name || user.name || 'cretetion'
    streamer.value.avatar = user.logo || '/avatar.png'

    const liveGame = stream.stream?.game || stream.data?.[0]?.game_name || ''
    const liveTitle = stream.stream?.channel?.status || stream.data?.[0]?.title || ''
    streamer.value.isLive = Boolean(stream.stream || stream.data?.[0])
    streamer.value.game = liveGame
    streamer.value.title = liveTitle
  } catch (error: unknown) {
    errorMessage.value = error instanceof Error ? error.message : 'Failed to load streamer'
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  void loadStreamer()
})
</script>


<template>
  <div class="flex bg-purple-twitch min-h-screen w-screen">
    <div class="bg-white mx-auto mt-60 h-160 w-150 p-12">
      <div class="flex flex-col justify-between h-full">
        <div class="flex gap-6 justify-center text-gray-500 semibold mb-10">
          <a class="nav-link" href="#">All</a>
          <a class="nav-link" href="#">Offline</a>
          <a class="nav-link" href="#">Online</a>
        </div>
        <div class="flex h-full justify-start">
          <div class="flex h-fit gap-6">
            <div class="relative">
              <img :src="streamer.avatar" class="w-15 h-15 rounded-full object-cover" />
              <div v-if="streamer.isLive" class="absolute -bottom-1 left-1/2 -translate-x-1/2 z-5 bg-red-500 h-5 w-10 rounded-sm flex items-center justify-center">
                <span class="text-xs font-semibold text-white tracking-wide select-none">LIVE</span>
              </div>
            </div>
            <div class="flex flex-col justify-center">
              <p class="text-md">{{ streamer.name }}</p>
              <p class="text-sm text-gray-400">{{ subtitle }}</p>
            </div>
          </div>
        </div>
        <div class="flex justify-center">
          <a :href="`https://twitch.tv/${USERNAME}`" target="_blank" rel="noopener noreferrer">
            <img :src="logo" alt="Icon" class="w-5" />
          </a>
        </div>
      </div>
    </div>
  </div>
</template>
