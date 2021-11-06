<template>
  <div>
    <b-container class="bv-example-row">
      <h2 class="text-primary">Rates</h2>

      <b-button-group>
        <b-button @click="getdata()" size="sm" variant="success">
          Reflash
        </b-button>
        <b-button @click="autoReflash()" size="sm" variant="primary">
          <b-icon icon="arrow-counterclockwise" :animation="anim"></b-icon>
          Auto {{reflashTime}}
        </b-button>
      </b-button-group>

      <div v-if="focusTable.length">
        <h4 v-if="focusTable">Focus On</h4>
        <b-table fixed striped hover dark :fields="fields" :items="focusTable">
          <template #cell(actions)="row">
            <b-button size="sm" variant="danger" @click="cancelfocus(row.index)">
              <b-icon icon="trash-fill"></b-icon>
            </b-button>
          </template>
        </b-table>
      </div>
      <b-row>
        <b-col lg="6" class="my-1">
          <b-form-group label="Filter" label-for="filter-input" label-cols-sm="3" label-align-sm="right" label-size="sm" class="mb-0">
            <b-input-group size="sm">
              <b-form-input id="filter-input" v-model="filter" type="search" placeholder="Type to Search"></b-form-input>

              <b-input-group-append>
                <b-button :disabled="!filter" @click="filter = ''">Clear</b-button>
              </b-input-group-append>
            </b-input-group>
          </b-form-group>
        </b-col>

        <b-col sm="5" md="6" class="my-1">
          <b-form-group label="Per page" label-for="per-page-select" label-cols-sm="6" label-cols-md="4" label-cols-lg="3" label-align-sm="right" label-size="sm" class="mb-0">
            <b-form-select class="form-control" id="per-page-select" v-model="perPage" :options="options" size="sm"></b-form-select>
          </b-form-group>
        </b-col>
      </b-row>

      <b-pagination v-model="currentPage" :total-rows="rows" :per-page="perPage" aria-controls="my-table" align="right"></b-pagination>

      <b-table striped hover :items="data" :fields="fields" :filter="filter" id="my-table" :per-page="perPage" :current-page="currentPage">
        <template #cell(actions)="row">
          <b-button size="sm" variant="success" @click="addfocus(row.item.name)">
            <b-icon icon="plus"></b-icon>
          </b-button>
        </template>
      </b-table>
    </b-container>
  </div>
</template>

<script>
export default {
  data() {
    return {
      fields: [
        { key: "name", sortable: true },
        { key: "type", sortable: false },
        { key: "unit", sortable: false },
        { key: "value", sortable: true },
        { key: "actions" },
      ],
      anim: "stop",
      focusOn: ["Bitcoin", "Ether", "Yearn.finance"],
      data: [],
      keys: [],
      autoSwitch: false,
      reflashTime: null,
      timer: null,
      countdownTimer: null,
      perPage: 10,
      perRelash: 30,
      currentPage: 1,
      options: [5, 10, 20],
      filter: null,
      focusTable: [],
    };
  },
  mounted() {
    this.getdata();
  },
  computed: {
    rows() {
      return this.data.length;
    },
  },
  methods: {
    getdata: function () {
      console.log(new Date());
      let vm = this;
      let loader = this.$loading.show();
      this.$http
        .get("https://api.coingecko.com/api/v3/exchange_rates")
        .then((a) => {
          let tempData = [];
          //this.$swal("成功", "api讀取完成", "success");
          for (const value of Object.values(a.data.rates)) {
            tempData.push(value);
          }
          vm.data = tempData;
          loader.hide();
        })
        .catch(() => {
          loader.hide();
          this.$swal("錯誤", "api異常，請稍後再試", "error");
        });
    },
    getfocusTable() {
      let vm = this;
      let tempData = [];

      let set = new Set(this.focusOn);
      for (const value of Object.values(vm.data)) {
        if (set.has(value.name)) {
          tempData.push(value);
        }
      }
      vm.focusTable = tempData;
    },
    addfocus(string) {
      let set = new Set(this.focusOn);

      if (!set.has(string)) {
        this.focusOn.push(string);
      } else {
        this.$swal("已關注", "", "info");
      }
    },
    cancelfocus(index) {
      this.focusOn.splice(index, 1);
    },
    autoReflash() {
      let temp = !this.autoSwitch;
      this.autoSwitch = temp;

      if (temp) {
        this.timer = setInterval(this.getdata, this.perRelash * 1000);
        this.countdownTimer = setInterval(this.countdown, 1000);

        this.anim = "spin-reverse";
        this.reflashTime = this.perRelash;
      } else {
        clearInterval(this.timer);
        clearInterval(this.countdownTimer);

        this.anim = "stop";
        this.reflashTime = null;
      }
    },
    countdown() {
      this.reflashTime--;
    },
  },
  watch: {
    focusOn: function () {
      this.getfocusTable();
    },
    data: function () {
      this.getfocusTable();
    },
    reflashTime: function(){
      if(this.reflashTime == 0){
        this.reflashTime = this.perRelash
      }
    }
  },
};
</script>