<template>
<div>
    <customHeader></customHeader>
  <div class="order-container">
      <div class="order-title">
          <h2>借阅记录</h2>
          <ul>
              <li v-bind:class="{'action': isA}" @click="transform('a')"><i>全部</i></li>
              <li v-bind:class="{'action': isB}" @click="transform('b')"><i>未归还</i></li>
              <li v-bind:class="{'action': isC}" @click="transform('c')"><i>已归还</i></li>
              <li v-bind:class="{'action': isD}" @click="transform('d')"><i>审核中</i></li>
          </ul>
      </div>
      <div class="my-order">
          <div class="no-list" v-if="orders.length==0&&isA">
              暂无任何借阅记录
          </div>
          <div class="no-list" v-if="ordersB.length==0&&isB">
              暂无任何未归还书籍
          </div>
          <div class="no-list" v-if="ordersC.length==0&&isC">
              暂无任何已归还书籍
          </div>
          <div class="no-list" v-if="ordersD.length==0&&isD">
              暂无任何审核中书籍
          </div>
          <div class="my-order-list">
                <ul style="margin-left:0px;padding-left:0px">
                  <li v-for="order in ordersShow" :key="order.id">
                      <p class="my-order-info">
                          <span>{{order.createTime | formatDate}}</span>
                      </p>
                      <div class="my-order-item">
                          <dl class="item-del">
                              <dd class="clearfix">
                                  <router-link :to="{path:'/custom/book',query:{id:order.itemVOS[0].vo.id}}">
                                    <img class="l" v-bind:src="order.itemVOS[0].vo.imgUrl" width="160" height="140">
                                  </router-link>
                                  <div class="del-box l">
                                      <router-link :to="{path:'/custom/book',query:{id:order.itemVOS[0].vo.id}}">
                                        <p class="book-name">{{order.itemVOS[0].vo.bookName}}</p>
                                      </router-link>
                                      <p class="book-desc">{{order.itemVOS[0].vo.description}}</p>
                                  </div>
                              </dd>
                              <div class="book-action l">
                                  <a href="javascript:void(0);" class="return-now" v-if="order.status==status.BORROWED" @click="returnNow(order)">立即归还</a>
                                  <p v-if="order.status==status.RETURNED" class="order-close">已归还</p>
                                  <p v-if="order.status==status.OVERDUE" class="order-close">已超期</p>
                                  <p v-if="order.status==status.CHECK_BORROW" class="order-close">借阅审核中</p>
                                  <p v-if="order.status==status.CHECK_RETURN" class="order-close">归还审核中</p>
                              </div>
                          </dl>
                      </div>
                  </li>
              </ul>
          </div>
      </div>
  </div>
</div>
</template>

<script>
import { getOrders, updateOrder } from 'service/order-service';
import { formatDate } from 'util/date-util.js';
import { OrderStatus } from 'constants/order-status';
import { getUser } from 'service/user-service';
import customHeader from 'custom/header.vue';
export default {
    name: 'order-list',
    data() {
        return {
            user:{},
            page: 1,
            rows: 20,
            isA:true,
            isB:false,
            isC:false,
            isD:false,
            orders:[],
            ordersB:[],
            ordersC:[],
            ordersD:[],
            ordersShow:[],
            status:{}
        }
    },
    created() {
        this.initUser();
        this.getOrders();
        this.status=OrderStatus;
    },
    methods:{
        change(event){
            event.target.style.background="red"
        },
        transform(s){
            if(s=='a'){
                this.isA=true;
                this.isB=false;
                this.isC=false;
                this.isD=false;
                this.ordersShow=this.orders;
            }else if(s=='b'){
                this.isA=false;
                this.isB=true;
                this.isC=false;
                this.isD=false;
                this.ordersShow=this.ordersB;
            }else if(s=='c'){
                this.isA=false;
                this.isB=false;
                this.isC=true;
                this.isD=false;
                this.ordersShow=this.ordersC;
            }else if(s=='d'){
                this.isA=false;
                this.isB=false;
                this.isC=false;
                this.isD=true;
                this.ordersShow=this.ordersD;
            }
        },
        async getOrders(){
            let res= await getOrders(this.page,this.rows,this.user.id);
            if(res.status==200){
                this.orders=res.data.list;
                this.orders.forEach(o=>{
                    if(o.status==this.status.RETURNED){
                        this.ordersC.push(o);
                    }else if(o.status==this.status.BORROWED){
                        this.ordersB.push(o);
                    }else if(o.status==this.status.CHECK_BORROW||o.status==this.status.CHECK_RETURN){
                        this.ordersD.push(o);
                    }
                })
                this.ordersShow=this.orders;
            }
        },
        async returnNow(order){
            order.status=OrderStatus.CHECK_RETURN;
            let res= await updateOrder(order);
            if(res==200){
                this.$route.push('/custom/orders');
            }
        },
        initUser() {
            this.user=getUser();
        }
    },
    filters: {
        formatDate(time) {
            return formatDate(time);
        }
    },
    components:{
        customHeader
    }
}
</script>

<style>
.order-container{
    margin: 36px auto;
    width: 992px;
    min-height: 530px;
}
.order-title{
    margin-bottom: 24px;
    overflow: hidden;
}
.order-title h2{
    margin: 0px 24px 0px 0px;
    float: left;
    font-size: 16px;
    color: #07111B;
    line-height: 32px;
    font-weight: bold;
}
.order-title ul{
    padding-left: 0px;
}
.order-title ul li{
    float: left;
    width: 95px;
    line-height: 32px;
    text-align: center;
    font-size: 12px;
}
.order-title ul li i{
    padding-left: 5px;
    font-style: normal;
}
.order-title ul::before{
    float: left;
    margin-top: 2px;
    margin-right: 20px;
    content: "|";
    color: #D9DDE1;
}
.order-title ul li.action {
    background: #4D555D;
    border-radius: 16px;
}
.order-title ul li.action i{
    color: #fff;
}
.my-order{
    width: 100%;
    /* padding: 96px 0px 0px;
    height: 360px;
    text-align: center;
    background: #fff;
    border-radius: 8px;
    font-size: 16px;
    color: #93999F;
    line-height: 24px; */
}
.my-order-list li{
    padding: 32px;
    padding-top: 0;
    box-shadow: 0px 2px 8px 2px rgba(0, 0, 0, 0.1);
    margin-bottom: 24px;
    background: #FFFFFF;
    border-radius: 8px;
}
.my-order-list dd:first-child{
    border-top: none;
    margin-top: 0px;
    padding-top: 0px;
}
.my-order-list li dd a{
    display: block;
}
.my-order-info{
    padding: 20px 0 19px;
    font-weight: bold;
    color: #4d555d;
    border-bottom: 1px solid #b7bbbf;
    font-size: 12px;
    line-height: 12px;
    box-sizing: border-box;
    margin: 0;
}
.my-order-info span a{
    color: #93999f;
    font-weight: 500;
    margin-left: 24px;
}
.my-order-item{
    position: relative;
    margin-top: 25px;
}
.item-del{
    width: 520px;
    /* border-right: 1px solid #d9dde1; */
}
.l{
    float: left;
}
.del-box{
    margin-left: 16px;
    width: 340px;
}
.book-name{
    word-break: break-word;
    color: #07111b;
    font-size: 14px;
    margin-bottom: 8px;
    line-height: 22px;
}
.book-desc{
    color: #93999f;
    font-size: 12px;
    line-height: 12px;
    margin-bottom: 12px;
}
.book-action{
    position: absolute;
    top: 0;
    width: 200px;
    height: 100%;
    /* border-left: 1px solid #d9dde1; */
    right: 0;
    text-align: center;
}
.return-now{
    margin: 12px auto;
    display: block;
    width: 120px;
    height: 36px;
    color: #fff;
    background: rgba(20, 152, 240, 0.8);
    border-radius: 18px;
    line-height: 36px;
}
.order-close{
    color: #93999f;
    margin-top: 36px;
    line-height: 14px;
}
.no-list{
    width: 100%;
    padding: 96px 0px 0px;
    height: 360px;
    text-align: center;
    background: #fff;
    border-radius: 8px;
    font-size: 16px;
    color: #93999F;
    line-height: 24px;
}
</style>