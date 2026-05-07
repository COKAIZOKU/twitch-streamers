<script setup lang="ts">
    import {computed, ref} from "vue";
    import api from "./api.vue";
    import logo from "./logo.vue";

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
        "Stories Food Streamers Art News Music Games Live IRL Chatting Crafting Chess Sports"
    ];
    const HIGHLIGHT_LINE_INDEX = 1;
    const HIGHLIGHT_WORD = "Streamers";
    
    type Filter = "all" | "offline" | "online";

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

    /* title of the stream */
    const subtitle = (streamer : StreamerState) : string => {
        if (!streamer.isLive) {
            return "Offline";
        }
        return `${streamer.game || "Unknown"} : ${streamer.title || "Live now"}`;
    };

    const handleStreamersLoaded = (loadedStreamers : StreamerState[]) : void => {
        streamers.value = loadedStreamers;
    };

    const handleApiError = (message : string) : void => {
        errorMessage.value = message;
    };

    /* "Streamers" white title */
    const splitHighlightedLine = (line : string) : {
        before: string;
        after: string
    } => {
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
        <api @loaded="handleStreamersLoaded" @error="handleApiError" />
        <div
            class="pointer-events-none absolute inset-0 z-0 select-none overflow-hidden">
            <div
                class="absolute ml-[-26rem] sm:ml-[-111rem] left-[50%] top-[-2%] flex h-full w-[1880px] -translate-x-[50%] flex-col justify-between">
                <p
                    v-for="(line, index) in BACKGROUND_LINES"
                    :key="index"
                    class="whitespace-nowrap text-center leading-none text-[4rem] sm:text-[8rem] font-semibold text-[#8040DA]">
                    <template v-if="index === HIGHLIGHT_LINE_INDEX">
                        {{ splitHighlightedLine(line).before }}
                        <span class="text-white">{{ HIGHLIGHT_WORD }}</span>{{ splitHighlightedLine(line).after }}
                    </template>
                    <template v-else>
                        {{ line }}
                    </template>
                </p>
            </div>
        </div>
        <div
            class="relative z-20 bg-white mx-auto mt-40 sm:mt-60 h-180 w-160 py-10 px-5 sm:p-12">
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
                                    class="absolute ping -bottom-1 left-1/2 -translate-x-1/2 z-5 bg-red-500 h-4.5 w-9.5 rounded-sm flex items-center justify-center">
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
                                <p class="text-sm text-gray-400 truncate max-w-[240px] sm:max-w-[450px]">{{ subtitle(streamer) }}</p>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="flex justify-center">
                    <a href="https://twitch.tv/" target="_blank" rel="noopener noreferrer">
                        <logo />
                    </a>
                </div>
            </div>
        </div>
    </div>
</template>