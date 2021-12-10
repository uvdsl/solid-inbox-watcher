<template>
  <div class="login-button" @click="login">
    <slot>
      <Button>Login</Button>
    </slot>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { useSolidSession } from "../../composables/useSolidSession";

export default defineComponent({
  name: "LoginButton",
  setup() {
    const { login } = useSolidSession();
    return { login };
  },
});
</script>


<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
</style>
