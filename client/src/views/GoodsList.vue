<template>
  <div>
      <NavHead/>
    <BredCrumb>商品列表</BredCrumb>

      <div class="accessory-result-page accessory-page">
        <div class="container">
            <div class="filter-nav">
                <span class="sortby">Sort by:</span>
                <a href="javascript:void(0)" class="default cur">Default</a>
                <a href="javascript:void(0)" class="price" @click="sortGoods">价格 <svg class="icon icon-arrow-short"><use xlink:href="#icon-arrow-short"></use></svg></a>
                <a href="javascript:void(0)" class="filterby stopPop">Filter by</a>
            </div>
            <div class="accessory-result">
                <!-- filter -->
                <div class="filter stopPop" id="filter">
                    <dl class="filter-price">
                        <dt>Price:</dt>
                        <dd><a href="javascript:void(0)" :class="{'cur':priceChecked == 'all'}"  @click="setPriceFilter('all')">All</a></dd>
                        <dd>
                            <a href="javascript:void(0)" :class="{'cur':priceChecked == index}" :key="index" v-for="(item,index) in priceFilter" @click="setPriceFilter(index)">{{item.startPrice}} - {{item.endPrice}}</a>
                        </dd>
                        
                    </dl>
                </div>
    
                <!-- search result accessories list -->
                <div class="accessory-list-wrap">
                    <div class="accessory-list col-4">
                        <ul>
                            <li v-for="(item,index) in GoodsList" :key="index">
                                <div class="pic">
                                    <a href="#"><img  v-lazy="'/static/img/' + item.productImage" alt=""></a>
                                </div>
                                <div class="main">
                                    <div class="name">{{item.productName}}</div>
                                    <div class="price">{{item.salePrice}}</div>
                                    <div class="btn-area">
                                        <a href="javascript:;" class="btn btn--m" @click="addCart(item.productId)">加入购物车</a>
                                    </div>
                                </div>
                            </li>
                            <div v-infinite-scroll="loadMore" infinite-scroll-disabled="busy" infinite-scroll-distance="10">
                            ...
                            </div>
                        </ul>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <modal :mdShow="mdShowCart">
       <p slot="message">
            加入购物车成功
       </p> 
       <div  slot="btnGroup">
             <a href="javascript:;" class="btn  btn--m" @click="mdShowCart = false">继续购物</a>
            <router-link class="btn  btn--m" to="/cart">查看购物车列表</router-link>
        </div>
       
    </modal>
  </div>
</template>
<script>
import NavHead from '@/components/NavHead'
import BredCrumb from '@/components/BredCrumb'
  import axios from 'axios'
  import Modal from '@/components/Modal'
  var count = 0;
  export default {
    components:{
        Modal,
        NavHead,
        BredCrumb
    },
    data(){
      return{
        mdShowCart:false,
        GoodsList:'',
        sortFlag:true,
        priceChecked:'all',
        data: [],
        busy: true,
        page:1,
        pagesize:8,
        priceFilter:[
            {
                startPrice:'0',
                endPrice:'100'
            },
            {
                startPrice:'100',
                endPrice:'500'
            },
            {
                startPrice:'500',
                endPrice:'1000'
            },
            {
                startPrice:'1000',
                endPrice:'2000'
            },
        ]
      }
    },
    methods:{
        getGoodsList(flag){
            let sort = this.sortFlag ? 1 : -1;
            let params = {
                'sort':sort,
                'priceLevel':this.priceChecked,
                'page':this.page,
                'pagesize':this.pagesize
                };
            axios.get('/goods/list',{params})
                .then(res=>{
                    if(flag){ //flag 为true的时候代表，第二次加载数据
                        // 把数据拼接起来
                        this.GoodsList = this.GoodsList.concat(res.data.result);

                        if(res.data.result.length == 0){
                            // 把监听事件关闭
                            this.busy = true;
                        }else{
                            // 把监听事件打开
                            this.busy = false;
                        }
                    }else{
                        this.GoodsList = res.data.result
                        // 把监听事件打开
                        this.busy = false;
                    }
                })
        },
        sortGoods(){
            this.sortFlag = !this.sortFlag;
            this.getGoodsList();
        },
        setPriceFilter(index){
            this.page = 1;
            this.priceChecked = index;
            this.getGoodsList();
        },
        loadMore: function() {
            this.busy = true;
            console.log("触发到底部了");
            setTimeout(()=>{
                this.page ++;
                this.getGoodsList(true);
            },1000)
            // setTimeout(() => {
            //     for (var i = 0, j = 10; i < j; i++) {
            //         this.data.push({ name: count++ });
            //     }
            //     this.busy = false;
            // }, 1000);
        },
        addCart(productId){
            axios.post('/goods/addCart',{productId:productId})
            .then(res=>{
                // alert(res.data.result);
                this.mdShowCart = true;
            })
        }
    },
    created(){
        this.getGoodsList();
    }
  }
</script>