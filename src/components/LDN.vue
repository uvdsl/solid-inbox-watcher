<template>
  <Card class="margined">
    <template #content>
      <div> <i>{{ uri }} </i> </div>
      <Divider />
      <span v-if="!error" style="white-space: pre-line">
        {{ ldn }}
      </span>
      <span v-else style="color: red">
        {{ error }}
      </span>
    </template>
    <template #footer>
      <Button
        icon="pi pi-times"
        label="Delete"
        @click="deleteResource(uri, authFetch)"
      />
    </template>
  </Card>
</template>

<script lang="ts">
import { useSolidSession } from "@/composables/useSolidSession";
import { toTTL } from "@/lib/n3Extensions";
import { getResource, parseToN3, deleteResource } from "@/lib/solidRequests";
import { defineComponent, ref, watch, watchEffect } from "vue";
export default defineComponent({
  name: "Messenger",
  components: {},
  props: {
    uri: { default: "" },
    updateFlag: { default: false },
  },
  setup(props) {
    const { authFetch } = useSolidSession();
    let ldn = ref("Message loading.");
    let error = ref();
    const update = () => {
      return (
        getResource(props.uri, authFetch.value)
          .then((resp) => resp.text())
          // .then((txt) => (ldn.value = txt))
          .then((txt) => {
            return parseToN3(txt, props.uri)
              .then((parsedN3) => {
                const newContent = toTTL(
                  parsedN3.store,
                  parsedN3.prefixes,
                  props.uri
                );
                if (ldn.value !== newContent) ldn.value = newContent;
              })
              .catch((err) => (error.value = err));
          })
      );
    };

    watch(() => props.updateFlag, update, { immediate: true });

    return { ldn, authFetch, deleteResource, error };
  },
});
</script>

<style scoped>
.margined {
  margin-top: 5px;
  margin-bottom: 5px;
}
</style>