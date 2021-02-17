<template>
  <!-- 商品列表 -->
  <v-container>
    <div v-if="status" class="d-flex justify-center">
      <v-progress-linear indeterminate color="pink"></v-progress-linear>
    </div>
    <div v-else>
      <v-row>
        <v-col cols="9">
          <div>
            <p class="title">
              <span>商品列表</span>
            </p>
          </div>
        </v-col>
        <v-col cols="3">
          <v-btn class="ma-2" tile outlined color="success" @click="jumpToAdd">
            <v-icon left>mdi-plus</v-icon>添加商品
          </v-btn>
           <v-btn class="ma-2" tile outlined color="warning" @click="jumpToStatistics">
            <v-icon left>mdi-podium-gold</v-icon>库存统计
          </v-btn>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="4" v-for="(item, index) in productList" :key="index">
          <v-card class="mx-auto" outlined min-height="180">
            <v-list-item three-line>
              <v-list-item-content>
                <div class="overline mb-4">￥{{item.price}}</div>
                <v-list-item-title class="headline mb-1">{{item.name}}</v-list-item-title>
                <v-list-item-subtitle>{{item.detail}}</v-list-item-subtitle>
              </v-list-item-content>
              <v-list-item-avatar tile size="80" color="grey">
                <v-img :src="item.miniImg"></v-img>
              </v-list-item-avatar>
            </v-list-item>
            <v-card-actions>
              <v-btn color="primary" text @click="viewMore(item)">详情</v-btn>
              <v-btn color="blue" text @click="editProduct(item)">编辑</v-btn>
              <v-btn color="red" text @click="deleteProduct(item)">删除</v-btn>
              <v-chip class="ma-2" color="pink" small text-color="white" v-if="item.storage < 10">
                <v-icon left dense>mdi-garage-alert</v-icon>
                仅剩 {{item.storage}} 件
              </v-chip>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
      <v-row justify="center">
        <v-dialog v-model="dialog" max-width="500">
          <v-card>
            <v-img
              :src="dialogItem.bigImg"
              :lazy-src="dialogItem.miniImg"
              aspect-ratio="1"
              class="grey lighten-2"
              max-width="500"
              height="160px"
            >
              <template v-slot:placeholder>
                <v-row class="fill-height ma-0" align="center" justify="center">
                  <v-progress-circular indeterminate color="grey lighten-5"></v-progress-circular>
                </v-row>
              </template>
            </v-img>
            <v-card-title>{{dialogItem.name}}</v-card-title>

            <v-card-text>
              <v-row align="center" class="mx-0">
                <v-chip class="ma-1 subtitle-1" color="success" outlined>
                  <v-icon left>mdi-numeric-1-circle-outline</v-icon>
                  {{firstCategory}}
                </v-chip>
                <v-chip class="ma-2 subtitle-1" color="primary" outlined>
                  <v-icon left>mdi-numeric-2-circle-outline</v-icon>
                  {{secondCategory}}
                </v-chip>
              </v-row>
              <v-row align="center" class="mx-0">
                <v-col class="col-6">
                  <div class="subtitle-1">
                    <v-icon left>mdi-cart-outline</v-icon>
                    库存: {{dialogItem.storage}}
                  </div>
                </v-col>
                <v-col class="col-6">
                  <div class="subtitle-1">
                    <v-icon left>mdi-currency-jpy</v-icon>
                    价格: {{dialogItem.price}}
                  </div>
                </v-col>
              </v-row>
              <v-divider class="mx-4"></v-divider>
              <div class="my-4 ma-1 subtitle-1">{{dialogItem.detail}}</div>
            </v-card-text>
            <v-card-actions>
              <v-btn color="green darken-1" text @click="dialog = false">关 闭</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-row>
      <div class="text-center" @click="getProductList">
        <v-pagination v-model="page" :length="pagelength"></v-pagination>
      </div>
    </div>
  </v-container>
</template>


<script>
export default {
  name: "Home",

  components: {},

  data: () => ({
    //
    keyword:"empty",//空字符串
    page: 1,
    pagelength: 1,
    status: true,
    productList: [],
    // dialogVisible: false,
    dialogItem: {},
    dialog: false,
    categoryList: [],
    firstCategory: "",
    secondCategory: ""
  }),

  methods: {
    getToken() {
      this.$http
        .get("/getAccessToken", {})
        .then(res => {})
        .catch(error => {
          console.log(error);
        });
    },
    getProductList() {
      if(this.$route.params.keyword!= null){
        this.keyword = this.$route.params.keyword;
      }
      this.$http
        .get(`/getProductList/${this.page}/${this.keyword}`, {})
        .then(res => {
          var resArray = res.data.data.data;
          var productArray = [];
          for (let i = 0; i < resArray.length; i++) {
            let temp = JSON.parse(resArray[i]);
            productArray.push(temp);
          }
          this.productList = productArray;
          this.status = false;
        })
        .catch(error => {
          console.log(error);
        });
    },
    getProductListLength() {
      if(this.$route.params.keyword!= null){
        this.keyword = this.$route.params.keyword;
      }
      this.$http
        .get(`/getProductListLength/${this.keyword}`, {})
        .then(res => {
          let pageCount = res.data.data.count;
          this.pagelength = Math.ceil(pageCount / 9);
        })
        .catch(error => {
          console.log(error);
        });
    },
    jumpToAdd() {
      this.$router.push({
        name: "EditProduct"
      });
    },
    jumpToStatistics(){
    this.$router.push({
        name: "Statistics"
      });
    },
    editProduct(item) {
      this.$router.push({
        name: "EditProduct",
        params: {
          type: 1,
          product: item
        }
      });
    },
    deleteProduct(item) {
      var productId = item.id;
      this.$confirm("此操作将永久删除" + item.name + ", 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http
            .get(`/deleteProductById/${productId}`, {})
            .then(res => {
              console.log("productId", productId);
              this.$message({
                type: "success",
                message: "删除成功!"
              });
              let log = {
                date: new Date(),
                type: "商品",
                operation: "删除",
                detail: `商品名字:${item.name};商品id:${productId}`
              };
              this.addToLog(log);
              this.getProductList();
              this.getProductListLength();
            })
            .catch(error => {
              console.log(error);
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    viewMore(item) {
      // this.dialogVisible = true;
      this.dialogItem = item;
      this.dialog = true;
      this.setCategoryValue();
    },
    addToLog(postBody) {
      this.$http
        .post(`/log`, postBody)
        .then(res => {})
        .catch(error => {
          console.log(error);
        });
    },
    getCategoryList() {
      this.$http
        .get(`/getCatergoryList`, {})
        .then(res => {
          var resArray = res.data.data.data;
          var categoryArray = [];
          for (let i = 0; i < resArray.length; i++) {
            let temp = JSON.parse(resArray[i]);
            categoryArray.push(temp);
          }
          this.categoryList = categoryArray;
        })
        .catch(error => {
          console.log(error);
        });
    },
    setCategoryValue() {
      let firstIndex = this.categoryList.findIndex(
        e => e.value == this.dialogItem.category
      );
      let secondIndex = this.categoryList[firstIndex].children.findIndex(
        e => e.value == this.dialogItem.secondCategory
      );
      this.firstCategory = this.categoryList[firstIndex].label;
      this.secondCategory = this.categoryList[firstIndex].children[
        secondIndex
      ].label;
    }
  },

  created() {
    this.$http
      .get("/getAccessToken", {})
      .then(res => {
        this.getProductList();
        this.getProductListLength();
        this.getCategoryList();
      })
      .catch(error => {
        console.log(error);
      });
    // this.getToken();
  }
};
</script>

<style>
body {
  /*解决element确认框偏移*/
  padding-right: 0 !important;
}
</style>