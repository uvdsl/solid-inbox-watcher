<template>
  <div class="grid">
    <div class="col lg:col-6 lg:col-offset-3">
      <div class="p-inputgroup">
        <!-- list go here -->
        <InputText
          placeholder="The URI of the Inbox to watch."
          v-model="inboxURI"
          @keyup.enter="sub"
        />
        <Button @click="sub"> watch </Button>
      </div>
    </div>
  </div>
  <div class="grid">
    <div class="col lg:col-6 lg:col-offset-3">
      <LDN
        :uri="ldn"
        :updateFlag="updateFlag"
        v-for="ldn in ldn_list"
        :key="ldn"
      />
    </div>
  </div>
  <Toast position="bottom-right" />
</template>

<script lang="ts">
import { LDP } from "@/lib/namespaces";
import { defineComponent, ref } from "vue";
import { useSolidSession } from "@/composables/useSolidSession";
import { getResource, parseToN3 } from "@/lib/solidRequests";

import { useToast } from "primevue/usetoast";
import LDN from "@/components/LDN.vue";

export default defineComponent({
  name: "Messenger",
  components: { LDN },
  async setup() {
    const toast = useToast();
    const { authFetch } = useSolidSession();
    const ldn_list = ref(new Array<String>());
    const inboxURI = ref("");
    let socket: WebSocket;
    const updateFlag = ref(false);

    const get = async (uri: string) => {
      const inbox = await getResource(uri, authFetch.value)
        .then((resp) => resp.text())
        .then((txt) => {
          return parseToN3(txt, inboxURI.value);
        });
      return inbox.store
        .getObjects(inboxURI.value, LDP("contains"), null)
        .map((obj) => obj.value);
    };

    const sub = async () => {
      if (socket !== undefined) socket.close();
      const uri = new URL(inboxURI.value);
      uri.protocol = "wss";

      socket = new WebSocket(uri.href, ["solid-0.1"]);
      socket.onopen = function () {
        this.send(`sub ${inboxURI.value}`);
        get(inboxURI.value).then((list) => (ldn_list.value = list));
      };
      socket.onmessage = function (msg) {
        if (msg.data && msg.data.slice(0, 3) === "pub") {
          // resource updated, refetch resource
          console.log(msg);
          get(inboxURI.value).then((list) => {
            for (const e of ldn_list.value) {
              list.push(list.splice(list.indexOf(e.toString()), 1)[0]);
            }
            ldn_list.value = list;
            updateFlag.value = !updateFlag.value;
          });
          toast.add({
            severity: "success",
            summary: "Good news, everyone!",
            detail: "The inbox was updated.",
            life: 5000,
          });
        }
      };
    };

    return {
      sub,
      inboxURI,
      ldn_list,
      updateFlag,
    };
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.grid {
  margin: 5px;
}
::v-deep() {
  .p-speeddial {
    bottom: 0;
    right: calc(50% - 2rem);
    padding-bottom: 15px;
  }
  .sizing {
    height: calc(100vh - 240px);
    width: 100%;
    max-height: calc(100vh - 240px);
    max-width: 100%;
  }
}
.border {
  border: 1px solid var(--surface-d);
  border-radius: 3px;
}
.border:hover {
  border: 1px solid var(--primary-color);
}
</style>
