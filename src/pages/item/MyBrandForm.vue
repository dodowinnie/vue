<template>
  <v-form v-model="valid" ref="myBrandForm">
    <v-text-field v-model="brand.name" label="请输入品牌名称" required :rules="nameRules"/>
    <v-text-field v-model="brand.letter" label="请输入品牌首字母" required :rules="letterRules"/>
    <my-cascader
      url="/item/category/list"
      multiple required
      v-model="brand.categories"
      label="请选择商品分类"/>
    <v-layout row>
      <v-flex xs3>
        <span style="font-size: 16px; color: #444">品牌LOGO：</span>
      </v-flex>
      <v-flex>
        <v-upload
          v-model="brand.image"
          url="/up/upload/image"
          :multiple="false"
          :pic-width="250"
          :pic-height="90" ref="myUpload"
        />
      </v-flex>
    </v-layout>
    <v-layout class="my-4" row>
      <v-spacer/>
      <v-btn @click="submit" color="primary">提交</v-btn>
      <v-btn @click="clear">重置</v-btn>
    </v-layout>
  </v-form>
</template>

<script>
  export default {
    name: "my-brand-form",
    data() {
      return {
        valid: false, //  表单校验结果标记
        brand: {
          name: '', // 品牌名称
          letter: '', // 品牌首字母
          image: '',// 品牌logo
          categories: [], // 品牌所属的商品分类数组
        },

        nameRules: [
          v => !!v || "品牌名称不能为空",
          v => v.length > 1 || "品牌名称至少2位"
        ],
        letterRules: [
          v => !!v || "首字母不能为空",
          v => /^[A-Z]{1}$/.test(v) || "品牌字母只能是A~Z的大写字母"
        ]
      }
    },

    props: {
      // 接受父组件传的值
      fromParentBrand: {
        type: Object
      },
      formIsEdit: {
        type: Boolean,
        default: false
      }
    },

    methods: {
      submit() {
        // 表单校验
        if (this.$refs.myBrandForm.validate()) {
          // 定义一个请求参数对象，通过解构表达式来获取brand中的属性
          const {categories, letter, ...params} = this.brand;
          //数据库中只要保存分类的id即可，因此我们对categories的值进行处理,只保留id，并转为字符串
          params.cids = categories.map(c => c.id).join(",")
          // 将字母都处理为大写
          params.letter = letter.toUpperCase()
          // 提交
          this.$http({
            method: this.formIsEdit ? "put" : "post",
            url: "/item/brand/opt",
            data: this.$qs.stringify(params)
          }).then(() => {
            this.$message.success("保存成功");
            this.$emit("close-window");
          }).catch(() => {
            this.$message.error("保存失败");
          }).finally(() => {
            this.$emit("get-data-from-server")
          })
        }
      },
      clear() {
        this.$refs.myBrandForm.reset();
        this.categories = [];
      }
    },

    watch: {
      fromParentBrand: {
        handler(val) {
          // 删除图片
          this.$refs.myUpload.removeSingle()
          if (val) {
            this.brand = Object.deepCopy(val)
          } else {
            // 为空初始化brand
            console.log("修改品牌清空，初始化brand")
            this.brand = {
              name: '', // 品牌名称
              letter: '', // 品牌首字母
              image: '',// 品牌logo
              categories: [], // 品牌所属的商品分类数组
            }
          }
        }, deep: true
      }
    }


  }
</script>

<style scoped>

</style>
