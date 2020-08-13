<template>
  <div>
    <Table border :columns="columns" :data="order" size="large" no-data-text="你还有订单，快点去购物吧"></Table>
    <div class="page-size">
      <Page :total="100" show-sizer></Page>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import qs from 'qs'
export default {
  name: 'MyOrder',
  data () {
    return {
      order: [],
      columns: [
        {
          title: '订单号',
          key: 'orderid',
          width: 180,
          align: 'center'
        },
        {
          title: '商品内容',
          key: 'product',
          // width: 400,
          render: (h, params) => {
            let products = params.row.products;
            let contents = [];
            products.forEach(product=>{
              contents.push(h('div',product.name+"   "+product.price+"元*"+product.amount))
            })
            return h('div', contents);
          },
          // align: 'center'
        },
        {
          title: '价格',
          width: 80,
          key: 'price',
          align: 'center',
          render: (h, params) => {
            return h('div', params.row.price.toFixed(2))
          }
        },
        {
          title: '购买时间',
          width: 120,
          key: 'createtime',
          align: 'center'
        }
      ]
    };
  },
  created(){
    axios({
      method:'post',
      url:'http://127.0.0.1:9001/order/list',
      data:qs.stringify({}),
      withCredentials:true
    }).then(response=>{
      this.order = response.data.data.list;
    });
  }
};
</script>

<style scoped>
.page-size {
  margin: 15px 0px;
  display: flex;
  justify-content: flex-end;
  align-items: center;
}
</style>
