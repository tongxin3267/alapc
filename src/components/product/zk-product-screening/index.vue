<template>
  <div class="zk-product-screening" v-loading.fullscreen.lock="loading">
    <div class="pc-x-product-listpage">
      <div class="pc-x-product-class">
        <ul>
          <li class="pc-x-product-class_li" v-for="(item,index) in classModel" :key="index">
            <dl class="pc-xproductclass_dl">
              <dt class="pc-xproductclass_dt"> {{item.name}}</dt>
              <dd class="pc-xproductclass_dd" v-for="(t,i) in item.childClass" :key="i">
                <span class="content_grid" @click="xProductClass(t.name,t.id,index,i)" :class="{'active':hu[index]==i&&nums==index}">
                  {{t.name}}
                </span>
              </dd>
            </dl>
          </li>
        </ul>
      </div>
      <div class="pc-listpage_theprice">
        <dl class="pc-listpage_dl">
          <dt class="pc-listpage_dt">价格:</dt>
          <dd class="pc-listpage_dd">
            <ul class="pc-listpage_ul">
              <li>
                <div @click="restrictPrice(0,120,1)" :class="{'listdigital':gitalIndex==1}">0-120</div>
              </li>
              <li>
                <div @click="restrictPrice(120,240,2)" :class="{'listdigital':gitalIndex==2}">120-240</div>
              </li>
              <li>
                <div @click="restrictPrice(240,360,3)" :class="{'listdigital':gitalIndex==3}">240-360</div>
              </li>
              <li>
                <div @click="restrictPrice(360,480,4)" :class="{'listdigital':gitalIndex==4}">360-480</div>
              </li>
              <li>
                <div @click="restrictPrice(480,600,5)" :class="{'listdigital':gitalIndex==5}">480-600</div>
              </li>
              <li class="li_input-box">
                <input class="input-box" v-model="productList.MinPrice" type="text" />
                <i class="input-box_i"></i>
                <input class="input-box" v-model="productList.MaxPrice" type="text" />
                <div class="input-box_determine" @click="init">确定</div>
              </li>
            </ul>
          </dd>
        </dl>
      </div>
      <div class="choose">
        <div to="" @click="sortNum(2)" :class="{'chooseclick':conterIndex==2}">
          <span class="choose_span">最新</span>
        </div>
        <div @click="sortNum(5)" :class="{'chooseclick':conterIndex==5}">
          <span class="choose_span">最热</span>
          <i class="icon iconfont zk-unfoldfill choose_i"></i>
        </div>
        <div @click="sortNum(6)" :class="{'chooseclick':conterIndex==6}">
          <span class="choose_span">关注</span>
          <i class="icon iconfont zk-unfoldfill choose_i"></i>
        </div>
        <div @click="sortNum(1)" :class="{'chooseclick':conterIndex==1}">
          <span class="choose_span">价格</span>
          <i class="icon iconfont zk-unfoldfill choose_i"></i>
        </div>
        <div @click="sortNum(3)" :class="{'chooseclick':conterIndex==3}">
          <span class="choose_span">人气</span>
          <i class="icon iconfont zk-unfoldfill choose_i"></i>
        </div>
        <div @click="sortNum(4)" :class="{'chooseclick':conterIndex==4}">
          <span class="choose_span">销量</span>
          <i class="icon iconfont zk-unfoldfill choose_i"></i>
        </div>
      </div>
      <div class="cont" v-show="listtype"></div>
      <div class="list-of-goods">
        <div class="productclass-item_right">
          <div class="notData_box" v-if=" viewModel&&viewModel.productItems.length===0">
            <div class="message">
              <i class="glyph-icon flaticon-exclamation-1"></i>暂无数据
            </div>
          </div>
          <ul v-if="async">
            <li class="item_right-li" v-for="(item,index) in viewModel.productItems" :key="index">
              <router-link :to="'/product/show?id='+item.id">
                <div class="item_right-box">
                  <img class="item_right-img" :src="item.thumbnailUrl" />
                </div>
                <div class="item_right-conter">
                  <div class="conter_money" v-if="$user.isLogin()">
                    <em class="conter_money-span1" v-if="viewModel.isFrontShowPrice">￥{{item.displayPrice}}</em>
                    <span class="conter_money-span2" v-if="viewModel.isFrontShowPrice">已售0件</span>
                    <em class="conter_money-span1" v-if="!viewModel.isFrontShowPrice">{{viewModel.priceAlterText}}</em>
                  </div>
                  <div class="conter_money" v-else>
                    <em class="conter_money-span1">登录查看价格</em>
                  </div>
                  <div class="conter_weight">
                    <router-link to="">{{item.name}}</router-link>
                  </div>
                  <!-- <div class="goods-collect">
                  <div class="goods-collect_1">
                    <p class="fonts-icon_p">
                      <router-link   to="">
                        <i class="icon iconfont zk-helpuser"></i>
                        对比
                      </router-link>
                      <router-link   to="">
                        <i class="icon iconfont zk-favorite"></i>
                        收藏
                      </router-link>
                      <router-link   to="">
                        <i class="icon iconfont zk-news"></i>
                        0
                      </router-link>
                    </p>
                    <p class="online-supermarket">
                      <router-link   to="">
                        <span class="online-supermarket_span1">自营</span>
                        <span>融合城商贸网上超市</span>
                      </router-link>
                    </p>
                  </div>
                  <div class="goods-collect_2">
                    <router-link   class="item_right-conter_a" to="">
                      <x-icon :pdisplay="pdisplay" src="zk-cart" size="16"></x-icon>
                    </router-link>
                  </div>
                </div> -->
                </div>
              </router-link>
            </li>
          </ul>
        </div>
      </div>
      <div class="screening_block">
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page.sync="currentPage3" layout="prev, pager, next, jumper" :page-count="viewModel.totalSize">
        </el-pagination>
      </div>
    </div>
  </div>
</template>

<script>

  import { PRODUCT_LIST_GET, PRODUCT_CLASS_GET } from '@/service/core/apiUrl'
  export default {

    data () {
      return {
        async: false,
        widgetModel: '',
        current: 0,
        test: 'lorem',
        viewModel: '',
        listtype: false,
        boxhigh: '',
        currentPage3: 1,
        classModel: '',
        pageIndex: 1, // 从第一页开始加载
        pageSize: 20,
        productList: {
          SortOrder: '', // 商品排序方式
          Keyword: '', // 搜索关键字
          MinPrice: '', // 最低价格
          MaxPrice: '', // 最高价格
          ClassIds: '', // 商品分类Id
          TagIds: '', // 商品标签ID
          ProductIds: '', // 商品Id
          BrandId: '', // 商品品牌Id
          PriceStyleId: '', //  商品模式
          OrderType: '', // 排序方式
          pageIndex: 1,
          pageSize: 20
        },
        hu: [],
        nums: '',
        gitalIndex: '',
        conterIndex: '',
        loading: true,
        isLogin: false
      }
    },
    props: {
      widget: {}
    },
    created () {
      for (var i in this.$route.query) {
        for (var q in this.productList) {
          if (i === q) {
            this.productList[q] = this.$route.query[i]
          }
        }
      }
    },
    mounted () {
      this.init()
    },
    methods: {
      async  init () {
        if (this.$user.isLogin()) {
          this.isLogin = true
          this.productList.loginUserId = this.$user.loginUser().id
        }
        // this.widgetModel = await this.$api.themeWidget(this.widget)
        // 获取商品class名称
        var response = await this.$api.httpGet(PRODUCT_CLASS_GET)
        this.classModel = response.result
        // 获取全部商品
        if (!this.$api.isEmpty(this.productList.Keyword)) {
          this.$bus.$emit('zkHeadSearch', this.productList.Keyword)
        }
        var listResponse = await this.$api.httpGet(PRODUCT_LIST_GET, this.productList)
        this.viewModel = listResponse.result
        this.async = true
        setTimeout(() => {
          this.loading = false
        }, 500)
      },
      async restrictPrice (minPrice, maxPrice, umber) {
        this.gitalIndex = umber
        this.productList.MaxPrice = maxPrice
        this.productList.MinPrice = minPrice
        var response = await this.$api.httpGet(PRODUCT_LIST_GET, this.productList)
        this.viewModel = response.result
      },
      async sortNum (id) {
        this.conterIndex = id
        this.productList.SortOrder = id
        var response = await this.$api.httpGet(PRODUCT_LIST_GET, this.productList)
        this.viewModel = response.result
      },
      async xProductClass (name, id, index, i) {
        this.productList.Keyword = ''
        this.productList.ClassIds = String(id)
        var response = await this.$api.httpGet(PRODUCT_LIST_GET, this.productList)
        this.viewModel = response.result
        this.$set(this.hu, index, i)
        this.nums = index
      },
      handleSizeChange (val) {
      },
      async handleCurrentChange (val) {
        this.productList.pageIndex = val
        var response = await this.$api.httpGet(PRODUCT_LIST_GET, this.productList)
        if (response.status === 1) {
          this.viewModel = response.result
        }
      },
      listpages () {
        let offtop = this.$refs.jyt.offsetTop
        let scrolltop = window.pageYOffset || document.documentElement.scrollTop || document.body.scrollTop
        if (scrolltop >= offtop) {
          this.listtype = true
        } else {
          this.listtype = false
        }
      }
    }
  }
</script>

<style scoped lang="scss">
  @import "./var.scss";
</style>
