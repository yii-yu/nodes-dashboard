<template>
  <div class="flex flex-row">
    <input type="text" v-model.trim="modelName" placeholder="Chain name" />
    <input type="text" v-model.trim="modelHost" placeholder="Host address" />
    <button @click="addNodeChain">Add a new node</button>
  </div>
  <div class="error-validation">{{ error }}</div>
  <table>
    <tr>
      <th style="width: 30px">Ping</th>
      <th style="width: 50px">№</th>

      <th>
        <span>Chain name</span>
        <span></span>
      </th>
      <th>
        <span>Host address</span>
        <span></span>
      </th>
      <th>Number last block</th>
      <th>Time of receiving the last block</th>
      <th></th>
    </tr>

    <tr v-for="(item, key) in nodes" :key="key" :class="[item.status]">
      <td style="width: 30px">
        <img
          v-if="item.status"
          class="icon-status"
          :src="require(`../assets/images/status-${item.status}.svg`)"
          alt="light"
        />
      </td>
      <td style="width: 50px">{{ ++key }}</td>

      <td>{{ item.name }}</td>
      <td>{{ item.host }}</td>
      <td>
        <img
          v-if="item.status && item.status == 'loading'"
          class="icon-status"
          :src="require(`../assets/images/status-loading.svg`)"
          alt="loading"
        />

        <span v-else>{{ item.blockNumber }}</span>
      </td>
      <td :class="[item.timeStatus]">
        <span v-if="item.lastTimeSurvey">
          {{ new Date(item.lastTimeSurvey).toLocaleString() }}
        </span>
      </td>
      <td>
        <svg
          class="button-del"
          width="20"
          height="20"
          viewBox="0 0 24 24"
          stroke-width="2"
          stroke="currentColor"
          fill="none"
          stroke-linecap="round"
          stroke-linejoin="round"
          @click="deleteNodeChain(--key)"
        >
          <path stroke="none" d="M0 0h24v24H0z" />
          <line x1="4" y1="7" x2="20" y2="7" />
          <line x1="10" y1="11" x2="10" y2="17" />
          <line x1="14" y1="11" x2="14" y2="17" />
          <path d="M5 7l1 12a2 2 0 0 0 2 2h8a2 2 0 0 0 2 -2l1 -12" />
          <path d="M9 7v-3a1 1 0 0 1 1 -1h4a1 1 0 0 1 1 1v3" />
        </svg>
      </td>
    </tr>
  </table>
  <div class="update-time">
    <span>update time:</span>
    <input
      type="text"
      placeholder="update time"
      v-model.number="modelTimeUpdate"
    />
    <spn>seconds</spn>
  </div>
</template>

<script>
import Web3 from "web3";

const LOCAL_STORAGE_NODE_LIST = "nodes";

export default {
  name: "dashboard",
  data() {
    return {
      classTime: "active",
      intervalId: null,
      modelName: "",
      modelHost: "",
      modelTimeUpdate: 60,
      nodes: [
        { name: "BSC Node", host: "http://51.222.109.230:8545", web3: null },
        { name: "BSC Node 2", host: "http://51.222.109.222:8545", web3: null },
        { name: "Arb Node", host: "http://51.222.244.202:8545", web3: null },
        { name: "Arb Node 2", host: "http://51.222.244.210:8545", web3: null },
        {
          name: "Fantom Node 1",
          host: "http://51.222.245.41:8545",
          web3: null,
        },
        {
          name: "Fantom Node 2",
          host: "http://51.222.245.48:8545",
          web3: null,
        },
        {
          name: "Avalanche Node 1",
          host: "http://51.222.245.55:8545",
          web3: null,
        },
        {
          name: "Avalanche Node 2",
          host: "http://51.222.245.54:8545",
          web3: null,
        },
        {
          name: "Polygon Node 1",
          host: "http://51.222.244.109:8545",
          web3: null,
        },
        {
          name: "Polygon Node 2",
          host: "http://51.222.244.81:8545",
          web3: null,
        },
      ],
      web3: null,
      error: "",
    };
  },
  mounted() {
    // this.getData();

    this.getDataWeb3();
    this.init();
  },
  beforeUnmount() {
    clearInterval(this.intervalId);
  },
  computed: {},
  methods: {
    checkTime(time) {
      if (time && time + this.modelTimeUpdate > Date.now()) {
        return "active";
      }
      return "not-active";
    },
    getDataWeb3() {
      this.nodes.map((item) => {
        item.status = "loading";

        let provider = new Web3.providers.HttpProvider(item.host);
        let web3 = new Web3(provider);

        web3.eth
          .getBlockNumber()
          .then((res) => {
            if (item.blockNumber !== res) {
              item.lastTimeSurvey = Date.now();
            }
            item.blockNumber = res;
            item.timeStatus = this.checkTime(item.lastTimeSurvey);
            item.status = "success";
            // console.log("res", res);
          })
          .catch(() => {
            item.timeStatus = this.checkTime(item.lastTimeSurvey);
            item.status = "error";
          });
      });
      this.saveLocalStorage(LOCAL_STORAGE_NODE_LIST, this.nodes);
    },
    addNodeChain() {
      this.error = this.validationData();
      if (this.error) {
        return;
      }

      this.nodes.push({
        name: this.modelName,
        host: this.modelHost,
      });
      this.saveLocalStorage(LOCAL_STORAGE_NODE_LIST, this.nodes);
      this.init();
    },
    deleteNodeChain(key) {
      this.nodes.splice(key, 1);
      this.saveLocalStorage(LOCAL_STORAGE_NODE_LIST, this.nodes);
      this.init();
    },
    init() {
      if (this.intervalId) clearInterval(this.intervalId);

      let savedListNodes = this.getLocalStorage(LOCAL_STORAGE_NODE_LIST);

      if (savedListNodes.length) {
        this.nodes = savedListNodes;
      }
      this.getDataWeb3();
      this.intervalId = setInterval(() => {
        this.getDataWeb3();
      }, this.modelTimeUpdate * 1000);
    },
    validationData() {
      if (!this.modelName || !this.modelHost) {
        return "The chain name or host must not be empty";
      }

      if (this.nodes.some((item) => item.host == this.modelHost)) {
        return "A chain with such a host is already in the list";
      }

      return "";
    },
    getLocalStorage(key) {
      return JSON.parse(localStorage.getItem(key) ?? "[]");
    },

    saveLocalStorage(key, data) {
      localStorage.setItem(key, JSON.stringify(data));
    },
  },
  watch: {
    modelTimeUpdate() {
      this.init();
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
td,
th {
  width: 25%;
  text-align: left;
}
table {
  width: 100%;
  border: 2px solid #42b983;
  margin-top: 10px;
}

input,
table {
  border-radius: 3px;
  background-color: #fff;
}

input {
  width: 30%;
  border: 1px solid #42b983;
  padding: 10px;
  margin: 10px;
}

button {
  width: 200px;
  height: 40px;
  background-color: #42b983;
  color: #fff;
  margin: 10px;
  border: none;
  border-radius: 3px;
  font-size: 16px;
}

th {
  background-color: #42b983;
  color: rgba(255, 255, 255, 0.66);
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

td {
  background-color: #ebebeb;
  padding: 5px 10px;
}
.error td {
  background-color: #f5cdcd;
}

th {
  padding: 10px;
}

th.active {
  color: #fff;
}

th.active .arrow {
  opacity: 1;
}

.arrow {
  display: inline-block;
  vertical-align: middle;
  width: 0;
  height: 0;
  margin-left: 5px;
  opacity: 0.66;
}

.arrow.asc {
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-bottom: 4px solid #fff;
}

.arrow.dsc {
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-top: 4px solid #fff;
}
.icon-status {
  width: 18px;
}
.button-del,
button {
  cursor: pointer;
}
.button-del:hover,
button:hover {
  opacity: 0.7;
}
.flex {
  display: flex;
  justify-content: center;
  padding: 30px;
}
.flex-row {
  flex-direction: row;
}
.error-validation {
  position: absolute;
  top: 100px;
  color: #ff0000;
}
.active {
  color: green;
}
.not-active {
  color: red;
}
.update-time {
  text-align: left;
}
.update-time input {
  width: 50px;
}
</style>
