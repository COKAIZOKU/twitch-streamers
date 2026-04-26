<script setup lang="ts">
import logo from '/glitch_twitch.svg'

const description =
  'Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.'
</script>


<template>
  <div class="flex bg-purple-twitch min-h-screen w-screen">
    <div class="bg-white mx-auto mt-60 h-160 w-150 p-12">
      <div class="flex flex-col justify-between h-full">
        <div class="flex gap-6 justify-center text-gray-500 font mb-10">
          <a class="nav-link" href="#">All</a>
          <a class="nav-link" href="#">Offline</a>
          <a class="nav-link" href="#">Online</a>
        </div>
        <div class="flex h-full justify-start">
          <div class="flex h-fit gap-6">
            <div class="relative">
              <img src="/avatar.png" class="w-15 h-15 rounded-full object-cover" />
              <div class="absolute -bottom-1 left-1/2 -translate-x-1/2 z-5 bg-red-500 h-5 w-10 rounded-sm flex items-center justify-center">
                <span class="text-xs font-semibold text-white tracking-wide">LIVE</span>
              </div>
            </div>
            <div class="flex flex-col justify-center">
              <p class="text-md">username</p>
              <p class="text-sm text-gray-400">
                {{
                  description.length > 60
                    ? description.slice(0, 60) + '...'
                    : description
                }}
              </p>
            </div>
          </div>
        </div>
        <div class="flex justify-center">
          <a href="https://twitch.tv/" target="_blank" rel="noopener noreferrer">
            <img :src="logo" alt="Icon" class="w-5" />
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.nav-link {
  position: relative;
  display: inline-block;
  padding-bottom: 2px;
}

.nav-link::after {
  content: '';
  position: absolute;
  left: 0;
  bottom: 0;
  width: 100%;
  height: 2px;
  background-color: currentColor;
  transform: scaleX(0);
  transform-origin: bottom right;
  transition: transform 0.25s ease-out;
}

.nav-link:hover::after,
.nav-link:focus-visible::after {
  transform: scaleX(1);
  transform-origin: bottom left;
}
</style>
