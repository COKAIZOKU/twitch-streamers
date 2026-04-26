<script setup lang="ts">
import logo from '/glitch_twitch.svg'

const description =
  'Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.'
</script>


<template>
  <div class="flex bg-purple-twitch min-h-screen w-screen">
    <div class="bg-white mx-auto mt-60 h-160 w-150 p-12">
      <div class="flex flex-col justify-between h-full">
        <div class="flex gap-5 justify-center text-gray-500 font mb-10">
          <a>All</a>
          <a>Offline</a>
          <a>Online</a>
        </div>
        <div class="flex h-full justify-start">
          <div class="flex h-fit gap-6">
            <img src="/avatar.png" class="w-15 h-15 rounded-full object-cover" />
            <div class="flex flex-col justify-center">
              <p class="text-md">username</p>
              <p class="text-sm text-gray-400">
                {{
                  description.length > 65
                    ? description.slice(0, 65) + '...'
                    : description
                }}
              </p>
            </div>
          </div>
        </div>
        <div class="flex justify-center">
          <img :src="logo" alt="Icon" class="w-5" />
        </div>
      </div>
    </div>
  </div>
</template>
