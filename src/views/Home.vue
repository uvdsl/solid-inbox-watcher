<template>
<!-- <div class="full-screen"> -->
    <Suspense>
  <template #default>
   <Messenger />
  </template>
  <template #fallback>
    <div>Loading...</div>
  </template>
</Suspense>
<!-- </div> -->
</template>

<script lang="ts">
import { defineComponent } from "vue";
import Messenger from "@/components/Messenger.vue";

export default defineComponent({
  name: "Home",
  components: {
    Messenger,
  },
});
</script>

<style lang="scss">
</style>
