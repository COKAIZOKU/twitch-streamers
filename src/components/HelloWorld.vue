<script setup lang="ts">
    import {computed, onMounted, ref} from "vue";

    const USERNAMES = [
        "ESL_SC2",
        "OgamingSC2",
        "cretetion",
        "freecodecamp",
        "storbeck",
        "habathcx",
        "RobotCaleb",
        "noobs2ninjas"
    ]as const;
    const API_BASE = "https://twitch-proxy.freecodecamp.rocks/twitch-api";
    const ESL_SC2_AVATAR = "/esl_sc2.jpeg";
    const STORBECK_AVATAR = "/storbeck.png";
    const BACKGROUND_LINES = [
        "Streamers News Art Music Games Chatting IRL Live Stories Food Crafting Chess Sports",
        "Chess Sports Games Chatting IRL Live Streamers News Art Music Food Stories Crafting",
        "Music Streamers Chatting News Games Art Food IRL Live Crafting Stories Sports Chess",
        "News Art Streamers Games Chatting Music Live IRL Stories Food Sports Chess Crafting",
        "Games Chatting News Streamers Music Art Live Stories IRL Chess Food Sports Crafting",
        "Art Music Games Streamers News Chatting IRL Food Live Sports Stories Crafting Chess",
        "Chatting IRL Live Streamers Games News Music Art Stories Chess Food Crafting Sports",
        "Food Stories Crafting Chess Sports Streamers News Art Music Games Chatting IRL Live",
        "Live IRL Chatting Games Music News Art Streamers Sports Chess Crafting Food Stories",
        "Stories Food Streamers Art News Music Games Live IRL Chatting Crafting Chess Sports",
        "Streamers News Art Music Games Chatting IRL Live Stories Food Crafting Chess Sports",
        "Chess Sports Games Chatting IRL Live Streamers News Art Music Food Stories Crafting",
        "Music Streamers Chatting News Games Art Food IRL Live Crafting Stories Sports Chess",
        "News Art Streamers Games Chatting Music Live IRL Stories Food Sports Chess Crafting",
        "Games Chatting News Streamers Music Art Live Stories IRL Chess Food Sports Crafting",
        "Art Music Games Streamers News Chatting IRL Food Live Sports Stories Crafting Chess",
        "Chatting IRL Live Streamers Games News Music Art Stories Chess Food Crafting Sports",
        "Food Stories Crafting Chess Sports Streamers News Art Music Games Chatting IRL Live",
        "Live IRL Chatting Games Music News Art Streamers Sports Chess Crafting Food Stories",
        "Stories Food Streamers Art News Music Games Live IRL Chatting Crafting Chess Sports",     ];
    const HIGHLIGHT_LINE_INDEX = 1;
    const HIGHLIGHT_WORD = "Streamers";

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
        data?: Array < {
            game_name?: string;
            title?: string;
        } >;
    }

    interface StreamerState {
        username : string;
        name : string;
        avatar : string;
        isLive : boolean;
        game : string;
        title : string;
    }

    const streamers = ref < StreamerState[] > ([]);
    const selectedFilter = ref < Filter > ("all");
    const errorMessage = ref < string > ("");

    const filteredStreamers = computed < StreamerState[] > (() => {
        if (selectedFilter.value === "online") {
            return streamers
                .value
                .filter((streamer) => streamer.isLive);
        }
        if (selectedFilter.value === "offline") {
            return streamers
                .value
                .filter((streamer) => !streamer.isLive);
        }
        return streamers.value;
    });

    const truncateChars = (value : string, max = 70) : string => value.length > max
        ? `${value.slice(0, max)}...`
        : value;

    const subtitle = (streamer : StreamerState) : string => {
        if (!streamer.isLive) {
            return "Offline";
        }
        return truncateChars(`${streamer.game || "Unknown"} : ${streamer.title || "Live now"}`,);
    };

    const getAvatar = (username : string, logo?: string | null) : string => {
        const normalizedUsername = username.toLowerCase();

        if (normalizedUsername === "esl_sc2") {
            return ESL_SC2_AVATAR;
        }

        if (normalizedUsername === "storbeck") {
            return STORBECK_AVATAR;
        }

        return logo || "";
    };

    const preloadImage = async(src : string) : Promise < void > => new Promise((resolve) => {
        const image = new Image();
        image.onload = () => resolve();
        image.onerror = () => resolve();
        image.src = src;
    });

    const fetchStreamer = async(username : string) : Promise < StreamerState > => {
        const [channelRes,
            streamRes] = await Promise.all([
            fetch(`${API_BASE}/channels/${username}`),
            fetch(`${API_BASE}/streams/${username}`)
        ]);

        const channel = (await channelRes.json())as ChannelResponse;
        const stream = (await streamRes.json())as StreamResponse;

        if (channel.error) {
            return {
                username,
                name: username,
                avatar: getAvatar(username),
                isLive: false,
                game: "",
                title: channel.message || "Streamer not found"
            };
        }

        return {
            username,
            name: channel.display_name || channel.name || username,
            avatar: getAvatar(username, channel.logo),
            isLive: Boolean(stream.stream || stream.data
                ?.[0]),
            game: stream.stream
                ?.game || stream.data
                    ?.[0]
                        ?.game_name || channel.game || "",
            title: stream.stream
                ?.channel
                    ?.status || stream.data
                        ?.[0]
                            ?.title || channel.status || ""
        };
    };

    const loadStreamers = async() : Promise < void > => {
        try {
            const loadedStreamers = await Promise.all(USERNAMES.map((username) => fetchStreamer(username)),);
            await Promise.all(loadedStreamers.map((streamer) => preloadImage(streamer.avatar)),);
            streamers.value = loadedStreamers;
        } catch (error : unknown) {
            errorMessage.value = error instanceof Error
                ? error.message
                : "Failed to load streamers";
        }
    };

    onMounted(() => {
        void loadStreamers();
    });

    const splitHighlightedLine = (line: string): {before: string; after: string} => {
        const highlightStart = line.indexOf(HIGHLIGHT_WORD);

        if (highlightStart === -1) {
            return {before: line, after: ""};
        }

        const highlightEnd = highlightStart + HIGHLIGHT_WORD.length;
        return {
            before: line.slice(0, highlightStart),
            after: line.slice(highlightEnd)
        };
    };
</script>

<template>
    <div
        class="relative isolate flex min-h-screen w-screen overflow-hidden bg-purple-twitch">
        <div
            class="pointer-events-none absolute inset-0 z-0 select-none overflow-hidden">
            <div
                class="absolute ml-[-26rem] sm:ml-[-111rem] left-[50%] top-[-2%] flex h-full w-[1880px] -translate-x-[50%] flex-col justify-between">
                <p
                    v-for="(line, index) in BACKGROUND_LINES"
                    :key="index"
                    class="whitespace-nowrap text-center leading-none text-[4rem] sm:text-[8rem] font-semibold text-[#8040DA]">
                    <template v-if="index === HIGHLIGHT_LINE_INDEX">
                        {{ splitHighlightedLine(line).before }}<span class="text-white">{{ HIGHLIGHT_WORD }}</span>{{ splitHighlightedLine(line).after }}
                    </template>
                    <template v-else>
                        {{ line }}
                    </template>
                </p>
            </div>
        </div>
        <div class="relative z-20 bg-white mx-auto mt-40 sm:mt-60 h-180 w-160 py-10 px-5 sm:p-12">
            <div class="flex flex-col justify-between h-full">
                <div class="flex gap-6 justify-center text-gray-500 semibold mb-5">
                    <button
                        :class="[
              'nav-link cursor-pointer bg-transparent border-none p-0',
              { 'nav-link-active': selectedFilter === 'all' },
            ]"
                        type="button"
                        @click="selectedFilter = 'all'">
                        All
                    </button>
                    <button
                        :class="[
              'nav-link cursor-pointer bg-transparent border-none p-0',
              { 'nav-link-active': selectedFilter === 'offline' },
            ]"
                        type="button"
                        @click="selectedFilter = 'offline'">
                        Offline
                    </button>
                    <button
                        :class="[
              'nav-link cursor-pointer bg-transparent border-none p-0',
              { 'nav-link-active': selectedFilter === 'online' },
            ]"
                        type="button"
                        @click="selectedFilter = 'online'">
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
                            class="flex h-fit gap-6">
                            <div class="relative w-12 h-12 shrink-0">
                                <img
                                    :src="streamer.avatar"
                                    class="w-full h-full rounded-full object-cover shrink-0"/>
                                <div
                                    v-if="streamer.isLive"
                                    class="absolute -bottom-1 left-1/2 -translate-x-1/2 z-5 bg-red-500 h-4.5 w-9.5 rounded-sm flex items-center justify-center">
                                    <span class="text-xs font-semibold text-white tracking-wide select-none">LIVE</span >
                                </div>
                            </div>
                            <div class="flex flex-col justify-center">
                                <a
                                    :href="`https://twitch.tv/${streamer.username}`"
                                    target="_blank"
                                    rel="noopener noreferrer"
                                    class="text-md text-black hover:underline">
                                    {{ streamer.name }}
                                </a>
                                <p class="text-sm text-gray-400">{{ subtitle(streamer) }}</p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="flex justify-center">
                    <a href="https://twitch.tv/" target="_blank" rel="noopener noreferrer">
                        <svg class="logo w-4.5" version="1.1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 2400 2800">
                            <g>
                                <polygon fill="#FFFFFF" points="2200,1300 1800,1700 1400,1700 1050,2050 1050,1700 600,1700 600,200 2200,200"/>
                                <g>
                                    <path fill="#9146FF" d="M500,0L0,500v1800h600v500l500-500h400l900-900V0H500z M2200,1300l-400,400h-400l-350,350v-350H600V200h1600V1300z"/>
                                    <rect x="1700" y="550" class="eye" fill="#9146FF" width="200" height="600"/>
                                    <rect x="1150" y="550" class="eye" fill="#9146FF" width="200" height="600"/>
                                </g>
                            </g>
                        </svg>
                    </a>
                </div>
            </div>
        </div>
    </div>
</template>
