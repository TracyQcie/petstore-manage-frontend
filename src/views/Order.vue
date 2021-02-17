<template>
  <v-container>
    <div v-if="status" class="d-flex justify-center status-progress">
      <v-progress-linear indeterminate color="pink"></v-progress-linear>
    </div>
    <div v-else class="order">
      <v-row>
        <v-col cols="6">
          <div>
            <p class="title">
              <span>订单列表</span>
            </p>
          </div>
        </v-col>
        <v-col cols="6">
          <v-tabs color="pink">
            <v-tab @click="setOrderListStatus('paid')">未发货</v-tab>
            <v-tab @click="setOrderListStatus('complain')">申诉中</v-tab>
            <v-tab @click="setOrderListStatus('delivered')">已发货</v-tab>
            <v-tab @click="setOrderListStatus('close')">已完成</v-tab>
            <v-tab @click="setOrderListStatus('all')">所有</v-tab>
          </v-tabs>
        </v-col>
      </v-row>
      <v-expansion-panels v-model="panel" multiple popout>
        <v-expansion-panel v-for="(item, index) in orderList" :key="index">
          <v-expansion-panel-header disable-icon-rotate>
            <template v-slot:actions>
              <span class="text--secondary status-label">{{item.status.label}}</span>
              <v-icon
                v-if="item.status.value == 'paid'"
                color="orange darken-1"
              >mdi-truck-delivery-outline</v-icon>
              <v-icon v-if="item.status.value == 'delivered'" color="cyan">mdi-truck-check-outline</v-icon>
              <v-icon v-if="item.status.value == 'close'" color="teal">mdi-check</v-icon>
              <v-icon v-if="item.status.value == 'complain'" color="error">mdi-alert-circle</v-icon>
            </template>
            <v-row no-gutters>
              <v-col cols="4">{{item.date.$date | moment("YYYY-MM-DD HH:mm:ss")}}</v-col>
              <v-col cols="8" class="text--secondary">
                <v-fade-transition leave-absolute>
                  <v-row>
                    <v-col cols="3">
                      <span>折扣:{{item.discount*100}}%</span>
                    </v-col>
                    <v-col cols="3">
                      <span>总计:{{item.totalPrice}}</span>
                    </v-col>
                    <v-col cols="2">
                      <span>实收:{{item.paymentFee}}</span>
                    </v-col>
                  </v-row>
                </v-fade-transition>
              </v-col>
            </v-row>
          </v-expansion-panel-header>
          <v-expansion-panel-content>
            <v-row no-gutters>
              <v-col cols="8">
                <v-row>
                  <v-col cols="6" v-for="(item, index) in item.payList" :key="index">
                    <v-card outlined>
                      <v-list-item two-line>
                        <v-list-item-content>
                          <div class="overline mb-4">￥{{item.productInfo.price}}</div>
                          <v-list-item-title class="subtitle-1 mb-1">{{item.productInfo.name}}</v-list-item-title>
                          <v-list-item-subtitle>× {{item.number}}</v-list-item-subtitle>
                        </v-list-item-content>
                        <v-list-item-avatar tile size="70" color="grey">
                          <v-img :src="item.productInfo.miniImg"></v-img>
                        </v-list-item-avatar>
                      </v-list-item>
                    </v-card>
                  </v-col>
                </v-row>
              </v-col>
              <v-col cols="1">
                <v-divider vertical class="mx-4"></v-divider>
              </v-col>
              <v-col cols="3">
                <div>
                  <span class="title">{{item.address.receiver}}</span>
                  <span class="subtitle-1">{{item.address.phone}}</span>
                </div>
                <div
                  class="body-2"
                >{{item.address.region[0].name}} {{item.address.region[1].name}} {{item.address.region[2].name}}</div>
                <div class="body-2">{{item.address.address}}</div>
                <br />
                <a
                  href="javascript:void(0)"
                  v-clipboard:copy="`${item.address.receiver} ${item.address.phone} ${item.address.region[0].name}${item.address.region[1].name}${item.address.region[2].name}${item.address.address}`"
                  v-clipboard:success="onCopy"
                  v-clipboard:error="onError"
                >复制地址</a>
              </v-col>
            </v-row>
            <v-card-actions v-if="item.status.value!='close'">
              <v-spacer></v-spacer>
              <v-btn
                text
                v-if="item.status.value == 'paid'"
                color="primary"
                @click="deliverConfirm(item._id)"
              >确认发货</v-btn>
              <v-btn
                text
                v-if="item.status.value == 'complain'"
                color="error"
                @click="dealingComplain(item._id,item)"
              >处理申诉</v-btn>
            </v-card-actions>
          </v-expansion-panel-content>
        </v-expansion-panel>
      </v-expansion-panels>
      <el-dialog title="申诉处理" :visible.sync="dialogFormVisible">
        <div>用户id:{{complain.userId}}</div>
        <div>订单id:{{complain.orderId}}</div>
        <div>申诉日期:{{complain.date.$date | moment("YYYY-MM-DD HH:mm:ss")}}</div>
        <div>申诉原因:{{complain.issue}}</div>
        <v-textarea outlined name="input-7-4" label="管理员批复" v-model="response"></v-textarea>
        <div slot="footer" class="dialog-footer">
          <el-button @click="reject()">拒 绝</el-button>
          <el-button type="primary" @click="pass()">通 过</el-button>
        </div>
      </el-dialog>
      <div class="text-center" v-if="orderList.length == 0">
        <div class="img-not-found">
          <v-img
            src="http://134.175.241.8:3000/upload/c3dbbdc57f124b4f7779787d75cc2e4d1dfc.png"
            aspect-ratio="1"
            height="350"
            contain
          ></v-img>
        </div>
        <div class="text-center">
          <span class="subtitle-1">现在没有这种类型订单哦</span>
        </div>
      </div>
      <div class="text-center" @click="getOrderList" v-else>
        <v-pagination v-model="page" :length="pagelength"></v-pagination>
      </div>
    </div>
  </v-container>
</template>

<script>
export default {
  name: "Order",
  data() {
    return {
      orderListStatus: "paid",
      status: true,
      orderList: [],
      page: 1,
      pagelength: 1,
      panel: 0,
      response: "",
      dialogFormVisible: false,
      complain: {
        date: {
          $date: new Date()
        }
      },
      order: {}
    };
  },
  methods: {
    setOrderListStatus(status) {
      this.page = 1;
      this.orderListStatus = status;
      this.getOrderList();
      this.getOrderListLength();
    },
    getOrderList() {
      this.$http
        .get(`/order/${this.page}/${this.orderListStatus}`, {})
        .then(res => {
          var resArray = res.data.data.data;
          var orderArray = [];
          for (let i = 0; i < resArray.length; i++) {
            let temp = JSON.parse(resArray[i]);
            orderArray.push(temp);
          }
          this.orderList = orderArray;
          this.status = false;
        })
        .catch(error => {
          console.log(error);
        });
    },
    getOrderListLength() {
      this.$http
        .get(`/getOrderListLength/${this.orderListStatus}`, {})
        .then(res => {
          let pageCount = res.data.data.count;
          this.pagelength = Math.ceil(pageCount / 5);
        })
        .catch(error => {
          console.log(error);
        });
    },
    deliverConfirm(orderId) {
      // 确认发货，将订单状态改为delivered
      let newStatus = {
        value: "delivered",
        label: "已发货"
      };
      let putBody = {
        orderId: orderId,
        newStatus: newStatus
      };
      this.$confirm("确认为此订单发货?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.$http
            .put(`/changeOrderStatus`, putBody)
            .then(res => {
              this.$message({
                type: "success",
                message: "成功发货!"
              });
              let log = {
                date: new Date(),
                type: "订单",
                operation: "发货",
                detail: `订单id:${orderId}`
              };
              this.addToLog(log);
              this.getOrderList();
            })
            .catch(error => {
              console.log(error);
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消发货"
          });
        });
    },
    dealingComplain(orderId, order) {
      this.order = order;
      this.$http
        .get(`/complain/${orderId}`, {})
        .then(res => {
          this.complain = JSON.parse(res.data.data.data);
          this.dialogFormVisible = true;
        })
        .catch(error => {
          console.log(error);
        });
    },
    reject() {
      // 拒绝，将订单状态恢复到上一个状态，然后更新complain
      let putBody = {
        orderId: this.complain.orderId,
        newStatus: this.complain.lastStatus
      };

      this.$confirm("确认拒绝退款?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.complain.status = {
            value: "reject",
            label: "拒绝退款"
          };
          this.$http
            .put(`/changeOrderStatus`, putBody)
            .then(res => {
              this.$message({
                type: "success",
                message: "申诉已拒绝!"
              });
              let log = {
                date: new Date(),
                type: "订单",
                operation: "拒绝退款",
                detail: `订单id:${putBody.orderId};申诉id:${this.complain._id}`
              };
              this.addToLog(log);
              this.updateResponse();
              this.getOrderList();
              this.dialogFormVisible = false;
            })
            .catch(error => {
              console.log(error);
            });
        })
        .catch(() => {});
    },
    pass() {
      let resetStorage = false;
      // 若未发货，需要重置库存
      if (this.complain.lastStatus.label != "delivered") {
        resetStorage = true;
      }
      let putRefundBody = {
        resetStorage: resetStorage,
        userId: this.complain.userId,
        productIndex: this.order.productsIndex,
        payList: this.order.payList,
        paymentFee: this.order.paymentFee
      };
      let putStatusBody = {
        orderId: this.complain.orderId,
        newStatus: {
          value: "close",
          label: "已退款"
        }
      };
      this.$confirm("确认批准退款?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.complain.status = {
            value: "pass",
            label: "已退款"
          };
          // 修改订单状态
          this.$http
            .put(`/changeOrderStatus`, putStatusBody)
            .then(res => {
              // 退款
              this.$http
                .put(`/refundToUser`, putRefundBody)
                .then(res => {
                  this.$message({
                    type: "success",
                    message: "退款成功!"
                  });
                  let log = {
                    date: new Date(),
                    type: "订单",
                    operation: "批准退款",
                    detail: `订单id:${this.complain.orderId};申诉id:${this.complain._id}`
                  };
                  this.addToLog(log);
                  this.updateResponse();
                  this.getOrderList();
                  this.dialogFormVisible = false;
                })
                .catch(error => {
                  console.log(error);
                });
            })
            .catch(error => {
              console.log(error);
            });
        })
        .catch(() => {});
    },
    updateResponse() {
      // 更新complain状态
      this.complain.response = this.response;
      this.$http
        .put("/complain", this.complain)
        .then(res => {})
        .catch(error => {});
    },
    onCopy: function(e) {
      this.$message({
                    type: "success",
                    message: "复制成功!"
                  });
      console.log("你刚刚复制: " + e.text);
    },
    onError: function(e) {
       this.$message({
                    type: "error",
                    message: "复制失败!"
                  });
      console.log("无法复制文本！");
    },
    addToLog(postBody) {
      this.$http
        .post(`/log`, postBody)
        .then(res => {})
        .catch(error => {
          console.log(error);
        });
    }
  },

  created() {
    this.getOrderList();
    this.getOrderListLength();
  }
};
</script>
<style scoped>
.status-label {
  margin-right: 20px;
}
.status-progress {
  margin-top: 20rpx;
}
.img-not-found {
  border-radius: 25px;
}
</style>