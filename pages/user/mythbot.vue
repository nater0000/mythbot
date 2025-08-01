<template>
  <div class="grid h-full w-full gap-4">
    <div>
      <MythbotMythbot />
    </div>
    <div class="w-full gap-4 border dark:border-gray-800 border-gray-200 rounded-md p-4">
      <NuxtPage />
    </div>
  </div>
</template>

<script setup>
definePageMeta({
  layout: "user",
});
</script>
<script>
export default {
  data() {
    return {
      ui: {
        open: {
          sideDraw: false,
        },
      },
    };
  },
};
</script>
