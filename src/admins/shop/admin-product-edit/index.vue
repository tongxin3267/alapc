<template>
  <x-border :title="addTitle&&addTitle.name" v-loading.fullscreen.lock="fullscreenLoading" color="focus">
    <div style="background:#ffffff;" class="list_detail" v-if="!isAdd&&viewModel">
      <div class="container">
        <div class="container-head">
          <el-breadcrumb separator="/" class="container-head_top">
            <el-breadcrumb-item>店铺：{{viewModel.store.name}}</el-breadcrumb-item>
            <el-breadcrumb-item>
              类目：{{viewModel.category.name}}
              <div class="categories" @click="isAdd = true">切换类目</div>
            </el-breadcrumb-item>
          </el-breadcrumb>
        </div>
        <div class="failText" v-if="operation()">操作信息:{{viewModel.productDetail.productDetailExtension.aidutMessage}}</div>
        <!-- <div class="failText" v-if="rouerId&&viewModel.productDetail.productDetailExtension.aidutMessage!==''">操作信息:{{viewModel.productDetail.productDetailExtension.aidutMessage}}</div> -->
        <basic ref="basic" :productView="viewModel"></basic>
        <sale :productView="viewModel"></sale>
        <imagepic :productView="viewModel" @changeImage="changeImage"></imagepic>
        <detail :productView="viewModel"></detail>
        <cat :widgetData="widgetData" :productView="viewModel" @changeClass="changeClass"></cat>
        <div class="goods_info buttom-save">
          <el-button class="goods_info-but right" v-if="judgeSave()" :loading="loadingFool" @click="save">保存</el-button>
          <el-button class="goods_info-but right" v-else-if="judgeExamine()" @click="goodsButton">审核</el-button>
          <el-button class="goods_button" v-if="$base.filter() === 4" @click="UnderCose">下架</el-button>
          <popup ref="dialog" :init="init" :viewModel="viewModel"></popup>
        </div>
        <el-dialog title="保存信息" :visible.sync="dialogVisible" width="30%" :before-close="handleClose">
          <el-form label-width="100px" class="el-form-List">
            <el-form-item label="保存信息">
              <el-input type="textarea" v-model="viewModel.productDetail.productDetailExtension.aidutMessage" placeholder="请输入内容"></el-input>
              <div class="tips">请输入保存信息</div>
            </el-form-item>
          </el-form>
          <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="clickSave">确 定</el-button>
          </span>
        </el-dialog>
      </div>
    </div>
    <add v-else @change="change" :filterType="$base.filter()"></add>
  </x-border>
</template>

<script>
  import add from './add'
  import service from './service'
  import basic from './widgets/basic.vue'
  import detail from './widgets/content.vue'
  import cat from './widgets/classtag.vue'
  import imagepic from './widgets/image.vue'
  import sale from './widgets/sale.vue'
  import popup from './widgets/dialog.vue'
  import judge from './widgets/js/judge'
  export default {
    props: {
      widgetData: {}
    },
    components: {
      basic,
      sale,
      detail,
      cat,
      imagepic,
      add,
      popup
    },
    data () {
      return {
        loading: true,
        product: {},
        isAdd: true, // 判断
        categoryId: '', // 类目id
        category: {},
        saleConfigs: [],
        displayConfigs: [],
        productSkus: [],
        detail: null,
        images: [],
        viewModel: '',
        fullscreenLoading: true,
        addTitle: '',
        rouerId: '',
        dialogVisible: false,
        loadingFool: false
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      async save () {
        this.loadingFool = true
        if (this.$base.filter() === 3 && this.$route.query.id !== '') {
          if (this.viewModel.product.storeId !== 0 && this.viewModel.product.storeId !== this.viewModel.store.id) {
            this.dialogVisible = true
            return
          }
        } else if (this.$base.filter() === 2) {
          this.viewModel.productDetail.productDetailExtension.aidutMessage = ''
        }
        await service.save(this)
        this.loadingFool = false
      },
      async clickSave () {
        if (this.viewModel.productDetail.productDetailExtension.aidutMessage === '') {
          return this.$notify({
            title: '警告',
            message: '保存信息不能为空',
            position: 'bottom-right'
          })
        }
        await service.save(this)
      },
      async init () {
        this.rouerId = this.$route.query.id
        if (JSON.parse(window.localStorage.getItem('productView'))) {
          this.viewModel = JSON.parse(window.localStorage.getItem('productView'))
          this.isAdd = false
        } else {
          this.viewModel = await service.getProductView(this)
          this.addTitle = await this.$api.adminPage(this.$route)
        }
        this.fullscreenLoading = false
      },
      // 修改分类 
      changeClass (classes) {
        this.viewModel.productDetail.productDetailExtension.storeClass = classes
      },
      // 修改图片
      changeImage (uploadImage) {
        this.viewModel.images = []
        this.images = uploadImage
        for (let item of uploadImage) {
          if (item) {
            this.viewModel.images.push(item)
          }
        }
      },
      change (id) {
        this.categoryId = id
        this.$api.localSet('categoryId', this.categoryId)
        this.init()
      },
      goodsButton () {
        this.$refs.dialog.dialogVisible = true
      },
      handleClose () {
        this.dialogVisible = false
      },
      async UnderCose () {
        var para = {
          id: this.viewModel.product.id
        }
        this.$confirm('是否下架此商品？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'success'
        }).then(async () => {
          await this.$api.httpPost('Api/Product/SoldOutProduct', para)
          this.init()
        }).catch(() => { })
      },
      judgeSave () {
        return judge.judgeSave(this)
      },
      judgeExamine () {
        return judge.judgeExamine(this)
      },
      operation () {
        return judge.operation(this)
      }
    }
  }
</script>
<style lang="scss">
  @import "./widgets/style/_index.scss";
</style>



