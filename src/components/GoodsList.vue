<template>
  <div>
    <i-input v-model="sreachData" size="large" class="sreach" placeholder="输入你想查找的商品">
        <Button slot="append" icon="ios-search" @click="sreach"></Button>
    </i-input>
    <div class="search-nav">
      <div class="search-nav-container">
        <ul>
          <li>全部商品分类</li>
          <li @click="menu()">首页</li>
          <li @click="menu('水果蔬菜')">水果蔬菜</li>
          <li @click="menu('蜜饯果干')">蜜饯果干</li>
          <li @click="menu('米面粮油')">米面粮油</li>
          <li @click="menu('茶叶冲饮')">茶叶冲饮</li>
          <li @click="menu('酒水饮料')">酒水饮料</li>
          <li @click="menu('滋补养生')">滋补养生</li>
        </ul>
      </div>
    </div>
    <div class="container">
      <!-- 商品展示容器 -->
      <div class="goods-box">
        <div class="goods-list-box">
          <div class="goods-list-tool">
            <ul>
              <li v-for="(item,index) in goodsTool" :key="index" @click="orderBy(item.en, index)"><span :class="{ 'goods-list-tool-active': isAction[index]}">{{item.title}} <Icon :type="icon[index]"></Icon></span></li>
            </ul>
          </div>
          <div class="goods-list">
            <div class="goods-show-info" v-for="(item, index) in orderGoodsList" :key="index">
              <div class="goods-show-img">
                <img :src="'http://127.0.0.1:9001/product/img/'+item.img" class="goods-item-img" @click="goodsDetail(item.productid)" />
              </div>
              <div class="goods-show-price">
                <span>
                  <Icon type="social-yen text-danger"></Icon>
                  <span class="seckill-price text-danger">{{item.price}}</span>
                </span>
              </div>
              <div class="goods-show-detail">
                <span>{{item.name}}</span>
              </div>
              <div class="goods-show-seller" v-for="(i,index) in item.tag" :key="index">
                <span class="sale">{{i}}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="goods-page">
        <Page :total="100" show-sizer></Page>
      </div>
    </div>
    <Footer></Footer>
    <Spin size="large" fix v-if="isLoading"></Spin>
  </div>
</template>

<script>
import Footer from '@/components/footer/Footer';
import store from '@/vuex/store';
import { mapState, mapActions, mapGetters, mapMutations } from 'vuex';
import axios from "axios"
import qs from 'qs'
export default {
  name: 'GoodsList',
  beforeRouteEnter (to, from, next) {
    window.scrollTo(0, 0);
    next();
  },
  data () {
    return {
      orderGoodsList: [],
      sreachData: '',
      isAction: [ true, false, false ],
      icon: [ 'arrow-up-a', 'arrow-down-a', 'arrow-down-a' ],
      goodsTool: [
        {title: '综合', en: 'sale'},
        {title: '评论数', en: 'remarks'},
        {title: '价格', en: 'price'}
      ]
    };
  },
  computed: {
    ...mapState(['asItems', 'isLoading']),
    // ...mapGetters(['orderGoodsList'])
  },
  methods: {
    ...mapActions(['loadGoodsList']),
    ...mapMutations(['SET_GOODS_ORDER_BY']),
    sreach () {
      if(this.sreachData!==''){
        axios({
          method:'post',
          url:'http://127.0.0.1:9001/product/search',
          data:qs.stringify({
            key: this.sreachData
          }),
          withCredentials:true
        }).then(response=>{
          let data=response.data.data.list;
          data.forEach(item=>{
            item.tag=item.tag.split('|');
          })
          this.orderGoodsList=data;
        });
      }
      this.$router.push({path: '/goodsList', query: { sreachData: this.sreachData }});
    },
    orderBy (data, index) {
      // console.log(data);
      this.icon = [ 'arrow-down-a', 'arrow-down-a', 'arrow-down-a' ];
      this.isAction = [ false, false, false ];
      this.isAction[index] = true;
      this.icon[index] = 'arrow-up-a';
      this.SET_GOODS_ORDER_BY(data);
    },
    menu(type){
      axios({
        method:'post',
        url:'http://127.0.0.1:9001/product/list',
        data:qs.stringify({
          type:type
        }),
        withCredentials:true
      }).then(response=>{
        let data=response.data.data.list;
        data.forEach(item=>{
          // console.log(item);
          item.tag=item.tag.split('|');
        })
        this.orderGoodsList=data;
      });
    },
    goodsDetail(id){
      this.$router.push({name:'GoodsDetail',query:{productid:id}});
    }
  },
  created () {
    this.loadGoodsList();
  },
  mounted () {
    this.sreachData = this.$route.query.sreachData;
    if(this.sreachData){
      axios({
        method:'post',
        url:'http://127.0.0.1:9001/product/search',
        data:qs.stringify({
          key: this.sreachData
        }),
        withCredentials:true
      }).then(response=>{
        let data=response.data.data.list;
        data.forEach(item=>{
          item.tag=item.tag.split('|');
        })
        this.orderGoodsList=data;
      });
    }else if(this.sreachData===undefined || this.sreachData===''){
      axios({
        method:'post',
        url:'http://127.0.0.1:9001/product/list',
        data:qs.stringify({}),
        withCredentials:true
      }).then(response=>{
        let data=response.data.data.list;
        data.forEach(item=>{
          // console.log(item);
          item.tag=item.tag.split('|');
        })
        this.orderGoodsList=data;
      });
    }
  },
  components: {
    Footer
  },
  store
};
</script>

<style scoped>
.container {
  margin: 15px auto;
  width: 93%;
  min-width: 1000px;
}
.sreach {
  width:500px;
  margin: 10px auto;
}
.text-danger {
  color: #A94442;
}
.seckill-price{
  margin-right: 5px;
  font-size: 25px;
  font-weight: bold;
}
.goods-box {
  /*margin:0 auto;*/
  display: flex;
}
/* ---------------侧边广告栏开始------------------- */
.as-box {
  width: 200px;
  border: 1px solid #ccc;
}
.item-as-title{
  width: 100%;
  height: 36px;
  color: #B1191A;
  line-height: 36px;
  font-size: 18px;
}
.item-as-title span:first-child{
  margin-left: 20px;
}
.item-as-title span:last-child{
  float: right;
  margin-right: 15px;
  font-size: 10px;
  color: #ccc;
}
.item-as{
  width: 160px;
  margin: 18px auto;
}
.item-as-img{
  width: 160px;
  height: 160px;
  margin: 0px auto;
}
.item-as-price span{
  font-size: 18px;
}
.item-as-intro{
  margin-top: 5px;
  font-size: 12px;
}
.item-as-selled{
  margin-top: 5px;
  font-size: 12px;
}
.item-as-selled span{
  color: #005AA0;
}
/* ---------------侧边广告栏结束------------------- */

/* ---------------商品栏开始------------------- */
.goods-list-box {
  margin:0 auto;
  width: calc(100% - 215px);
}
.goods-list-tool{
  width: 100%;
  height: 38px;
  border: 1px solid #ccc;
  background-color: #F1F1F1;
}
.goods-list-tool ul{
  padding-left: 15px;
  list-style: none;
}
.goods-list-tool li{
  cursor: pointer;
  float: left;
}
.goods-list-tool span{
  padding: 5px 8px;
  border: 1px solid #ccc;
  border-left: none;
  line-height: 36px;
  background-color: #fff;
}
.goods-list-tool span:hover{
  border: 1px solid #E4393C;
}
.goods-list-tool i:hover{
  color: #E4393C;
}
.goods-list-tool-active {
  color: #fff;
  border-left: 1px solid #ccc;
  background-color: #E4393C !important;
}

.goods-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}
.goods-show-info{
  width: 240px;
  padding: 10px;
  margin: 15px 0px;
  border: 1px solid #fff;
  cursor: pointer;
}
.goods-show-info:hover{
  border: 1px solid #ccc;
  box-shadow: 0px 0px 15px #ccc;
}
.goods-show-price{
  margin-top: 6px;
}
.goods-show-detail{
  font-size: 12px;
  margin: 6px 0px;
}
.goods-show-num{
  font-size: 12px;
  margin-bottom: 6px;
  color: #009688;
}
.goods-show-num span{
  color: #005AA0;
  font-weight: bold;
}
.goods-show-seller{
  font-size: 12px;
  color:#E4393C;
}
.goods-page {
  margin-top: 20px;
  display: flex;
  justify-content: flex-end;
}
.goods-item-img{
  width:220px;
  height:220px;
}
.search-nav{
  width: 100%;
  height: 64px;
  border-bottom: 2px solid #B1191A;
}
.search-nav-container{
  width: 80%;
  min-width: 1000px;
  height: 64px;
  margin: 0px auto;
}
.search-nav-container-90{
  width: 90%;
}
.search-nav-container ul{
  margin: 0px;
  padding-left: 0px;
  list-style: none;
}
.search-nav-container li{
  cursor: pointer;
  margin-left: 50px;
  line-height: 64px;
  color: #C81623;
  font-size: 18px;
  /*font-weight: bold;*/
  float: left;
}
.search-nav-container a{
  color: #C81623;
  text-decoration: none;
}
.search-nav-container li:first-child{
  padding: 0px 38px;
  background:#B1191A;
  margin: 0px;
  color: #fff;
}
.sale{
  float:left;
  margin-right:10px;
}
/* ---------------商品栏结束------------------- */
</style>
