<template>
  <div v-bind:class="{ margined: true, highlight: isHighlighted }">
    <Card>
      <template #content>
        <div>
          <i class="text-primary">{{ uri }} </i>
        </div>
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
          class="p-button-text p-button-rounded p-button-raised"
          @click="deleteResource(uri, authFetch)"
        />
      </template>
    </Card>
  </div>
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

    const isHighlighted = ref(false);
    const flash = async () => {
      isHighlighted.value = true;
      await new Promise((resolve) => setTimeout(resolve, 3000));
      isHighlighted.value = false;
    };

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
                if (ldn.value !== newContent) {
                  // if (ldn.value !== "Message loading.") // flash only on update
                  flash();
                  ldn.value = newContent;
                }
              })
              .catch((err) => (error.value = err));
          })
      );
    };

    watch(() => props.updateFlag, update, { immediate: true });

    return { ldn, authFetch, deleteResource, error, isHighlighted };
  },
});
</script>

<style lang="scss" scoped>
.margined {
  margin-top: 25px;
  margin-bottom: 25px;
}

@-webkit-keyframes borderBlink {
  from,
  to {
    // border-color: transparent
    box-shadow: 0 0 0 0 var(--primary-color);
  }
  50% {
    // border-color: var(--primary-color);
    box-shadow: 0 0 10px 5px var(--primary-color);
  }
}
@keyframes borderBlink {
  from,
  to {
    box-shadow: 0 0 0 0 var(--primary-color);
  }
  50% {
    // border-color: var(--primary-color);
    box-shadow: 0 0 10px 5px var(--primary-color);
  }
}
.borderBlink {
  // border-color: var(--primary-color);
  box-shadow: 0 0 10px 5px var(--primary-color);
  /* add 'border-color: transparent' if you wish no border to show initially */
}
.highlight {
  -webkit-animation: borderBlink 1s step-end 3;
  animation: borderBlink 1s step-end 3;
}
</style>