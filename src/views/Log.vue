<template>
  <div>
    <v-container>
      <div v-if="status" class="d-flex justify-center status-progress">
        <v-progress-linear indeterminate color="pink"></v-progress-linear>
      </div>
      <div v-else>
        <v-card>
          <v-card-title>
            操作日志
            <v-spacer></v-spacer>
            <v-text-field
              v-model="search"
              append-icon="mdi-magnify"
              label="Search"
              single-line
              hide-details
            ></v-text-field>
          </v-card-title>
          <v-data-table :headers="headers" :items="logList" :search="search"></v-data-table>
        </v-card>
      </div>
    </v-container>
  </div>
</template>

<script>
import moment from "moment";
export default {
  name: "Log",
  data() {
    return {
      search: "",
      logList: [],
      status: true,
      headers: [
        { text: "日期", value: "date" },
        { text: "类型", value: "type" },
        { text: "操作", value: "operation" },
        { text: "详情", value: "detail" }
      ]
    };
  },
  methods: {
    getLogList() {
      this.$http
        .get(`/log`, {})
        .then(res => {
          var resArray = res.data.data.data;
          var logArray = [];
          for (let i = 0; i < resArray.length; i++) {
            let temp = JSON.parse(resArray[i]);
            temp.date = moment(temp.date).format("YYYY-MM-DD HH:mm:ss")
            logArray.push(temp);
          }
          this.logList = logArray;
          console.log(this.logList);
          this.status = false;
        })
        .catch(error => {
          console.log(error);
        });
    }
  },
  created() {
    this.getLogList();
  }
};
</script>
