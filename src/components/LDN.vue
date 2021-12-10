<template>
  <Card>
    <template #title>
      {{ uri }}
    </template>
    <template #content>
      {{ ldn }}
    </template>
  </Card>
</template>

<script lang="ts">
import { useSolidSession } from "@/composables/useSolidSession";
import { getResource, parseToN3 } from "@/lib/solidRequests";
import { defineComponent, ref } from "vue";
export default defineComponent({
  name: "Messenger",
  components: {},
  props: {
    uri: { default: "" },
  },
  async setup(props) {
    const { authFetch } = useSolidSession();
    const ldn = await getResource(props.uri, authFetch.value).then((resp) =>
      resp.text()
    );
    //   .then((txt) => {
    //     return parseToN3(txt, props.uri);
    //   });
    
    return { ldn };
  },
});
</script>

<style>

</style>