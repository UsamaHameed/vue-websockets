<template>
  <div class="hello">
    <input v-model="newIsin" placeholder="add isin here" />
    <button @click="addIsin">Add</button>
    <ul id="isins">
      <li v-for="isin in isins" :key="isin">
        {{ isin }} <button @click="removeIsin(isin)">x</button>
      </li>
    </ul>
    <ul id="prices">
      <li v-for="(price, isin) in prices" :key="isin">{{ isin }} {{ price }}</li>
    </ul>
  </div>
</template>

<script lang="ts">
import Vue from 'vue';

const sendMessage = (webSocket: WebSocket, message: object) => {
  webSocket.send(JSON.stringify(message));
};
const subscribeToIsin = (webSocket: WebSocket, isin: string) => {
  sendMessage(webSocket, { subscribe: isin });
};
const unSubscribeToIsin = (webSocket: WebSocket, isin: string) => {
  sendMessage(webSocket, { unsubscribe: isin });
};

const socket = 'ws://159.89.15.214:8080/';

type Price = { [isin: string]: string };

interface ComponentData {
  isins: Array<string>;
  prices: { [isin: string]: string };
  newIsin: string;
}

const webSocket = new WebSocket(socket);

export default Vue.extend({
  name: 'HelloWorld',
  props: {},
  data(): ComponentData {
    return {
      isins: ['DE000BASF111', 'DE000BASF112'],
      prices: {},
      newIsin: '',
    };
  },
  methods: {
    addIsin() {
      if (this.newIsin) {
        if (!this.isins.find((isin) => isin === this.newIsin)) {
          this.isins.push(this.newIsin);
          this.newIsin = '';
        }
      }
    },
    removeIsin(isinToRemove: string) {
      this.isins = this.isins.filter((isin) => isin !== isinToRemove);
    },
  },
  created() {
    const { isins, prices } = this;

    webSocket.onopen = function onOpen() {
      isins.map((isin) => subscribeToIsin(webSocket, isin));
    };

    webSocket.onmessage = function onMessage(message) {
      const { data } = message;
      const parsedData = JSON.parse(data) as Price;
      Vue.set(prices, parsedData.isin, parsedData.price);
    };
  },
  watch: {
    isins: function callback(newIsins: string[], oldIsins: string[]) {
      const { prices } = this;
      if (newIsins.length > oldIsins.length) {
        const newIsin = newIsins.find((isin) => !oldIsins.includes(isin));
        if (newIsin) {
          subscribeToIsin(webSocket, newIsin);
        }
      } else if (newIsins.length === oldIsins.length) {
        // for some reason, sometimes this watcher is called with the updated oldValue
        // param and then there is no diff b/w old and new
        // TODO: fix this...
        subscribeToIsin(webSocket, newIsins[newIsins.length - 1]);
      } else {
        const isinToRemove = oldIsins.find((isin) => !newIsins.includes(isin));
        if (isinToRemove) {
          unSubscribeToIsin(webSocket, isinToRemove);
          // need to remove exsisting isin price from data.prices...
          Vue.delete(prices, isinToRemove);
        }
      }
    },
  },
});
</script>

<style scoped></style>
