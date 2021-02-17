<template>
  <div>
    <v-container>
      <div v-if="status" class="d-flex justify-center status-progress">
        <v-progress-linear indeterminate color="pink"></v-progress-linear>
      </div>
      <div v-else>
        <v-card>
          <v-card-title>
            出货统计
            <v-spacer></v-spacer>
            <v-text-field
              v-model="shipmentSearch"
              append-icon="mdi-magnify"
              label="Search"
              :sort-by="['price','shipment']"
              :sort-desc="[false, true]"
              multi-sort
              single-line
              hide-details
            ></v-text-field>
          </v-card-title>
          <v-data-table :headers="shipmentHeaders" :items="shipmentProductList" :search="shipmentSearch"></v-data-table>
        </v-card>
        <v-card>
          <v-card-title>
            商品库存
            <v-spacer></v-spacer>
            <v-text-field
              v-model="productSearch"
              append-icon="mdi-magnify"
              label="Search"
              :sort-by="['storage']"
              :sort-desc="[false, true]"
              multi-sort
              single-line
              hide-details
            ></v-text-field>
          </v-card-title>
          <v-data-table :headers="productHeaders" :items="productList" :search="productSearch"></v-data-table>
        </v-card>
      </div>
    </v-container>
  </div>
</template>

<script>
import moment from "moment";
export default {
  name: "Statistics",
  data() {
    return {
      shipmentSearch: "",
      productSearch:"",
      productList: [],
      shipmentProductList: [],
      status: true,
      shipmentHeaders: [
        { text: "ID", value: "id" },
        { text: "名字", value: "name" },
        { text: "价格", value: "price" },
        { text: "出货", value: "shipment" }
      ],
      productHeaders: [
        { text: "ID", value: "id" },
        { text: "名字", value: "name" },
        { text: "价格", value: "price" },
        { text: "一级分类", value: "category" },
        { text: "二级分类", value: "secCategory" },
        { text: "库存", value: "storage" }
      ],
      categoryList: []
    };
  },
  methods: {
    getCloseOrderList() {
      this.$http
        .get(`/order/close`, {})
        .then(res => {
          var resArray = res.data.data.data;
          var productArray = [];
          for (let i = 0; i < resArray.length; i++) {
            let temp = JSON.parse(resArray[i]).payList;
            for (let j = 0; j < temp.length; j++) {
              let tempProduct = temp[j];
              let index = productArray.findIndex(e => e.id == tempProduct.id);
              if (index < 0) {
                let newProduct = {
                  id: tempProduct.id,
                  name: tempProduct.productInfo.name,
                  price: tempProduct.productInfo.price,
                  shipment: tempProduct.number
                };
                productArray.push(newProduct);
              } else {
                productArray[index].shipment += tempProduct.number;
              }
            }
          }
          this.shipmentProductList = productArray;
        })
        .catch(error => {
          console.log(error);
        });
    },
    getProductList() {
      this.$http
        .get(`/getAllProductList`, {})
        .then(res => {
          var resArray = res.data.data.data;
          var productArray = [];
          for (let i = 0; i < resArray.length; i++) {
            let temp = JSON.parse(resArray[i]);
            let categoryLabel = this.setCategoryValue(
              temp.category,
              temp.secondCategory
            );
            let newProduct = {
              id: temp.id,
              name: temp.name,
              price: temp.price,
              category: categoryLabel[0],
              secCategory: categoryLabel[1],
              storage: temp.storage
            };
            productArray.push(newProduct);
          }
          this.productList = productArray;
          this.status = false;
        })
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
          this.getProductList()
        })
        .catch(error => {
          console.log(error);
        });
    },
    setCategoryValue(category, secondCategory) {
      let firstIndex = this.categoryList.findIndex(e => e.value == category);
      let secondIndex = this.categoryList[firstIndex].children.findIndex(
        e => e.value == secondCategory
      );
      return [
        this.categoryList[firstIndex].label,
        this.categoryList[firstIndex].children[secondIndex].label
      ];
    }
  },
  created() {
    this.getCloseOrderList();
    this.getCategoryList();
  }
};
</script>
