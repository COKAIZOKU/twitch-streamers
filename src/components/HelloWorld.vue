<script setup lang="ts">
import { computed, onMounted, ref } from "vue";
import logo from "/glitch_twitch.svg";

const USERNAMES = [
  "ESL_SC2",
  "OgamingSC2",
  "cretetion",
  "freecodecamp",
  "storbeck",
  "habathcx",
  "RobotCaleb",
  "noobs2ninjas",
] as const;
const API_BASE = "https://twitch-proxy.freecodecamp.rocks/twitch-api";
const FALLBACK_AVATAR = "/avatar.png";

type Filter = "all" | "offline" | "online";

interface ChannelResponse {
  display_name?: string;
  name?: string;
  logo?: string | null;
  status?: string;
  game?: string;
  error?: string;
  message?: string;
}

interface StreamResponse {
  stream?: {
    game?: string;
    channel?: {
      status?: string;
    };
  } | null;
  data?: Array<{
    game_name?: string;
    title?: string;
  }>;
}

interface StreamerState {
  username: string;
  name: string;
  avatar: string;
  isLive: boolean;
  game: string;
  title: string;
}

const streamers = ref<StreamerState[]>([]);
const selectedFilter = ref<Filter>("all");
const loading = ref<boolean>(true);
const errorMessage = ref<string>("");

const filteredStreamers = computed<StreamerState[]>(() => {
  if (selectedFilter.value === "online") {
    return streamers.value.filter((streamer) => streamer.isLive);
  }
  if (selectedFilter.value === "offline") {
    return streamers.value.filter((streamer) => !streamer.isLive);
  }
  return streamers.value;
});

const truncateChars = (value: string, max = 60): string =>
  value.length > max ? `${value.slice(0, max)}...` : value;

const subtitle = (streamer: StreamerState): string => {
  if (!streamer.isLive) {
    return "Offline";
  }
  return truncateChars(
    `${streamer.game || "Unknown"} : ${streamer.title || "Live now"}`,
  );
};

const onAvatarError = (event: Event): void => {
  const image = event.currentTarget as HTMLImageElement | null;
  if (!image || image.src.endsWith(FALLBACK_AVATAR)) {
    return;
  }
  image.src = FALLBACK_AVATAR;
};

const fetchStreamer = async (username: string): Promise<StreamerState> => {
  const [channelRes, streamRes] = await Promise.all([
    fetch(`${API_BASE}/channels/${username}`),
    fetch(`${API_BASE}/streams/${username}`),
  ]);

  const channel = (await channelRes.json()) as ChannelResponse;
  const stream = (await streamRes.json()) as StreamResponse;

  if (channel.error) {
    return {
      username,
      name: username,
      avatar: "/avatar.png",
      isLive: false,
      game: "",
      title: channel.message || "Streamer not found",
    };
  }

  return {
    username,
    name: channel.display_name || channel.name || username,
    avatar: channel.logo || "/avatar.png",
    isLive: Boolean(stream.stream || stream.data?.[0]),
    game:
      stream.stream?.game || stream.data?.[0]?.game_name || channel.game || "",
    title:
      stream.stream?.channel?.status ||
      stream.data?.[0]?.title ||
      channel.status ||
      "",
  };
};

const loadStreamers = async (): Promise<void> => {
  try {
    streamers.value = await Promise.all(
      USERNAMES.map((username) => fetchStreamer(username)),
    );
  } catch (error: unknown) {
    errorMessage.value =
      error instanceof Error ? error.message : "Failed to load streamers";
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  void loadStreamers();
});
</script>

<template>
  <div class="flex bg-purple-twitch min-h-screen w-screen">
    <div class="bg-white mx-auto mt-60 h-180 w-150 p-12">
      <div class="flex flex-col justify-between h-full">
        <div class="flex gap-6 justify-center text-gray-500 semibold mb-5">
          <button
            :class="[
              'nav-link cursor-pointer bg-transparent border-none p-0',
              { 'nav-link-active': selectedFilter === 'all' },
            ]"
            type="button"
            @click="selectedFilter = 'all'"
          >
            All
          </button>
          <button
            :class="[
              'nav-link cursor-pointer bg-transparent border-none p-0',
              { 'nav-link-active': selectedFilter === 'offline' },
            ]"
            type="button"
            @click="selectedFilter = 'offline'"
          >
            Offline
          </button>
          <button
            :class="[
              'nav-link cursor-pointer bg-transparent border-none p-0',
              { 'nav-link-active': selectedFilter === 'online' },
            ]"
            type="button"
            @click="selectedFilter = 'online'"
          >
            Online
          </button>
        </div>
        <div class="flex h-full justify-start overflow-auto">
          <div class="flex h-fit w-full flex-col gap-5">
            <p v-if="loading" class="text-sm text-gray-400">Loading...</p>
            <p v-else-if="errorMessage" class="text-sm text-red-500">
              {{ errorMessage }}
            </p>
            <div
              v-for="streamer in filteredStreamers"
              :key="streamer.username"
              class="flex h-fit gap-6"
            >
              <div class="relative w-12 h-12 shrink-0">
                <img
                  :src="streamer.avatar"
                  class="w-full h-full rounded-full object-cover shrink-0"
                  @error="onAvatarError"
                />
                <div
                  v-if="streamer.isLive"
                  class="absolute -bottom-1 left-1/2 -translate-x-1/2 z-5 bg-red-500 h-4.5 w-9.5 rounded-sm flex items-center justify-center"
                >
                  <span
                    class="text-xs font-semibold text-white tracking-wide select-none"
                    >LIVE</span
                  >
                </div>
              </div>
              <div class="flex flex-col justify-center">
                <a
                  :href="`https://twitch.tv/${streamer.username}`"
                  target="_blank"
                  rel="noopener noreferrer"
                  class="text-md text-black hover:underline"
                >
                  {{ streamer.name }}
                </a>
                <p class="text-sm text-gray-400">{{ subtitle(streamer) }}</p>
              </div>
            </div>
          </div>
        </div>
        <div class="flex justify-center">
          <a
            href="https://twitch.tv/"
            target="_blank"
            rel="noopener noreferrer"
          >
            <img :src="logo" alt="Icon" class="w-4" />
          </a>
        </div>
      </div>
    </div>
  </div>
</template>
