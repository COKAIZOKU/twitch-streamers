<script setup lang="ts">
    import {onMounted} from "vue";

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

    const emit = defineEmits < {
        loaded: [streamers: StreamerState[]];
        error: [message: string];
    } > ();

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
            emit("loaded", loadedStreamers);
        } catch (error : unknown) {
            emit("error", error instanceof Error
                ? error.message
                : "Failed to load streamers");
        }
    };

    onMounted(() => {
        void loadStreamers();
    });
</script>

<template></template>
