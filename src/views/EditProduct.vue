<template>
  <div>
    <v-container>
      <p class="title">
        <span v-if="type === 0">上传商品</span>
        <span v-else>编辑商品</span>
      </p>
      <v-row>
        <v-col cols="12" md="8">
          <v-form ref="form" v-model="valid" lazy-validation>
            <div class="category-list">
              <el-cascader
                v-model="value"
                :options="categoryList"
                @change="handleChange"
                placeholder="请选择商品类别"
              ></el-cascader>
              <v-chip class="ma-2" color="success" outlined>
                <v-icon left>mdi-numeric-1-circle-outline</v-icon>
                {{firstCategory}}
              </v-chip>
              <v-chip class="ma-2" color="primary" outlined>
                <v-icon left>mdi-numeric-2-circle-outline</v-icon>
                {{secondCategory}}
              </v-chip>
            </div>
            <v-text-field v-model="product.name" label="商品名字" outlined></v-text-field>
            <v-text-field v-model.number="product.price" label="商品定价" outlined></v-text-field>
            <v-text-field v-model.number="product.storage" label="商品库存" outlined></v-text-field>
            <v-textarea v-model="product.detail" outlined name="input-7-4" label="商品详情" value></v-textarea>
            <v-btn
              v-if="type === 0"
              :disabled="!valid"
              color="success"
              class="mr-4"
              @click="upload()"
            >上 传</v-btn>
            <v-btn v-else :disabled="!valid" color="success" class="mr-4" @click="save()">保 存</v-btn>
          </v-form>
        </v-col>
        <v-col cols="6" md="4">
          <el-upload
            v-if="type === 0"
            class="avatar-uploader"
            action="http://134.175.241.8:3000/upload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img v-if="imageUrl" :src="imageUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
          <v-avatar v-else class="profile" color="grey" size="164" tile>
            <v-img :src="product.miniImg"></v-img>
          </v-avatar>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
export default {
  name: "Home",
  data() {
    return {
      type: 0,
      value: [],
      product: {
        id: "",
        category: "",
        secondCategory: "",
        name: "",
        price: 0,
        miniImg: "",
        bigImg: "",
        storage: 0,
        detail: ""
      },
      categoryList: [],
      valid: true,
      firstCategory: "",
      secondCategory: "",
      imageUrl: ""
    };
  },
  methods: {
    getDataFromParams() {
      if (
        this.$route.params.type != null &&
        this.$route.params.product != null
      ) {
        this.type = this.$route.params.type;
        this.product = this.$route.params.product;
        if (
          this.product.category == null &&
          this.product.secondCategory == null
        ) {
          this.$set(this.product, "category", "");
          this.$set(this.product, "secondCategory", "");
        }
      }
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
          this.setCategoryValue();
          console.log(this.categoryList);
        })
        .catch(error => {
          console.log(error);
        });
    },
    setCategoryValue() {
      if (this.type === 1) {
        let firstIndex = this.categoryList.findIndex(
          e => e.value == this.product.category
        );
        let secondIndex = this.categoryList[firstIndex].children.findIndex(
          e => e.value == this.product.secondCategory
        );
        this.firstCategory = this.categoryList[firstIndex].label;
        this.secondCategory = this.categoryList[firstIndex].children[
          secondIndex
        ].label;
      }
    },
    handleChange(value) {
      this.product.category = this.value[0];
      this.product.secondCategory = this.value[1];
      let firstIndex = this.categoryList.findIndex(e => e.value == value[0]);
      let secondIndex = this.categoryList[firstIndex].children.findIndex(
        e => e.value == value[1]
      );
      this.firstCategory = this.categoryList[firstIndex].label;
      this.secondCategory = this.categoryList[firstIndex].children[
        secondIndex
      ].label;
    },
    // 保存修改
    save() {
      this.$http
        .put("/updateProduct", this.product)
        .then(res => {
          this.$message({
            showClose: true,
            message: "保存成功",
            type: "success"
          });
          let log = {
            date: new Date(),
            type: "商品",
            operation: "编辑",
            detail: `商品名字:${this.product.name};商品id:${this.product.id}`
          };
          this.addToLog(log);
          this.$router.go(-1);
        })
        .catch(error => {
          this.$message.error("保存失败");
        });
    },

    upload() {
      this.product.id = new Date().valueOf().toString();
      this.$http
        .post("/addProduct", this.product)
        .then(res => {
          console.log(res);
          this.$message({
            showClose: true,
            message: "上传成功",
            type: "success"
          });
          let log = {
            date: new Date(),
            type: "商品",
            operation: "添加",
            detail: `商品名字:${this.product.name};商品id:${this.product.id}`
          };
          this.addToLog(log);
          this.$router.go(-1);
        })
        .catch(error => {
          this.$message.error("上传失败");
        });
    },
    handleAvatarSuccess(res, file) {
      this.imageUrl = URL.createObjectURL(file.raw);
      this.product.bigImg = res;
      this.product.miniImg = res;
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
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
    this.getDataFromParams();
    this.getCategoryList();
  }
};
</script>

<style>
.category-list {
  margin-bottom: 30px;
}
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px !important;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>