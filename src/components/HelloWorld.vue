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
const ESL_SC2_AVATAR = "/esl_sc2.jpeg";
const BACKGROUND_LINES = [
  "Streamers News Art Music Games Chatting IRL Live Stories Food",
  "Live Stories IRL Chatting Streamers Games Music Art News Food",
  "Food News Art Music Games Streamers Chatting IRL Live Stories",
  "Games Music Art Food News Live Stories IRL Chatting Streamers",
  "IRL Live Stories Streamers Chatting Food Games Music Art News",
  "News Food Art Music Games Chatting Streamers IRL Live Stories",
  "Chatting Streamers IRL Live Stories Games News Food Art Music",
  "Music Games Live Stories Art Food IRL Chatting Streamers News",
];

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

const getAvatar = (username: string, logo?: string | null): string => {
  const normalizedUsername = username.toLowerCase();

  if (normalizedUsername === "esl_sc2") {
    return ESL_SC2_AVATAR;
  }

  if (normalizedUsername === "storbeck") {
    return FALLBACK_AVATAR;
  }

  return logo || FALLBACK_AVATAR;
};

const preloadImage = async (src: string): Promise<void> =>
  new Promise((resolve) => {
    const image = new Image();
    image.onload = () => resolve();
    image.onerror = () => resolve();
    image.src = src;
  });

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
      avatar: getAvatar(username),
      isLive: false,
      game: "",
      title: channel.message || "Streamer not found",
    };
  }

  return {
    username,
    name: channel.display_name || channel.name || username,
    avatar: getAvatar(username, channel.logo),
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
    const loadedStreamers = await Promise.all(
      USERNAMES.map((username) => fetchStreamer(username)),
    );
    await Promise.all(
      loadedStreamers.map((streamer) => preloadImage(streamer.avatar)),
    );
    streamers.value = loadedStreamers;
  } catch (error: unknown) {
    errorMessage.value =
      error instanceof Error ? error.message : "Failed to load streamers";
  }
};

onMounted(() => {
  void loadStreamers();
});
</script>

<template>
  <div class="relative isolate flex min-h-screen w-screen overflow-hidden bg-purple-twitch">
    <div class="pointer-events-none absolute inset-0 z-0 select-none overflow-hidden">
      <div class="flex h-full flex-col justify-between py-2">
        <p
          v-for="(line, index) in BACKGROUND_LINES"
          :key="index"
          class="-ml-30 -mt-10 whitespace-nowrap leading-none text-[8rem] font-semibold text-black/12"
        >
          {{ line }}
        </p>
      </div>
    </div>
    <div class="relative z-20 bg-white mx-auto mt-60 h-180 w-150 p-12">
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
            <p v-if="errorMessage" class="text-sm text-red-500">
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
            <img :src="logo" alt="Icon" class="w-4.5" />
          </a>
        </div>
      </div>
    </div>
  </div>
</template>
