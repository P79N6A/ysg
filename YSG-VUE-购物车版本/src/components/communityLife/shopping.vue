<template>
  <div class="search" style="height: 100%">
    <div class="nav_mark"></div>
    <yd-navbar :title="language.community.shopping" fixed>

    <router-link to="/home" slot="left" v-if="pageFlag == 'home'">
        <span class="close"></span>
    </router-link>

    <router-link to="/s_home" slot="left" v-else-if="pageFlag == 's_home'">
        <span class="close"></span>
    </router-link>

    <router-link to="/communityLife" slot="left" v-else>
        <span class="close"></span>
    </router-link>


    </yd-navbar>
    <section class="promotiom_list top_nav" style="height: 100%">
      <scroller :on-infinite="infinite">
        <ul class="type-buy" style="padding-top: 0.5rem">
          <li v-for="data in dataList">
              <div class="col-4" @click="goDetail(data.id)">
                  <img  :src="data.pic" alt="">
              </div>
              <div class="col-6">
                  <h4>{{data.title}}</h4>
                  <p>{{data.introduct}}</p>
                  <ul class="s-price">
                    <li class="col-5">RMB {{data.price}}</li>
                    <li class="col-5"><button type="button" @click="addToCart(data)">{{language.community.buy}}</button></li>
                  </ul>
              </div>
          </li>
        </ul>
        <p class="no_data" v-show="noData">{{language.common.noMoreData}}</p>
      </scroller>
    </section>
    <!-- 购物车foot -->
        <div class="action-bar" > 
            <div class="action-btn buy-btn" @click="checkOut()">{{language.common.shoppingCart}}({{ selectedNum }})</div> 
            <div class="total">{{language.common.total}}：<span>¥<b>{{ totalPrice }}</b></span></div>
        </div>
  </div>
</template>

<style>
.action-bar {
    position: fixed;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 52px;
    font-size: 15px;
    background-color: #fff;
    border-top: .1px solid #ddd;
}

.buy-btn {
    background-color: #7A45E5;
    border-top: 1px solid #ddd;
}  
 
.action-btn {
    float: right;
    width: 120px;
    height: 100%;
    color: #fff;
    text-align: center;
    font-weight: 300;
    line-height: 52px;
    cursor: pointer;
}
.total {
    float: right;
    color: #363636;
    font-size: 14px;
    line-height: 50px;
    margin-right: 20px;
}

.total span {
    color: #000;
}

.total b {
    font-size: 17px;
    margin-left: 4px;
}
</style>
<script>
    import { mapGetters } from 'vuex'
    import { mapState } from 'vuex'
    export default {
        data() {
            return {
                dataList: [],
                nextPage: 1,
                noData: false,
                pageFlag:'',
                selectedNum:0,
                itemQuantity:1,
                totalPrice:0,
                cart:[]
            }
        },
        created:function () {
            this.pageFlag = this.$route.query.pageFlag
            if(localStorage.Quantity){
                this.cart = JSON.parse(localStorage.getItem('cart'));
                this.selectedNum = parseInt(localStorage.Quantity);
                this.totalPrice = parseInt(localStorage.totalPrice);
            }
        },
        methods: {
            getData:function () {
                let _this = this
                let params = {
                    hotelid: localStorage.HOTELID,
                    lang: localStorage.LANGUAGE,
                    limit:6,
                    page:_this.nextPage
                }
                this.$store.dispatch('getShoppingList', params).then(function (res) {
                    var arrList = res.data.data.list
                    _this.dataList = _this.dataList.concat(arrList)
                    _this.nextPage = res.data.data.nextPage
                })
            },
            infinite:function (done) {
                let _this = this
                done(true);
                if (this.nextPage != '-1') {
                    _this.getData()
                } else {
                    done(false);
                    _this.noData = true;
                }
            },
            goDetail:function (id) {
                let data = {};
                for(var key in this.dataList){
                    if (this.dataList[key].id == id){
                        data = this.dataList[key];
                        break;
                    }
                }
                this.saveCart();
                this.$router.push({path:'/buy',query:{info:data,quantity:this.selectedNum}});
            },
            addToCart:function(data){ 
                alert(1);
                let _this = this; 
                this.selectedNum ++;
                this.totalPrice += parseInt(data.price);
                var alreadyIndex = this.cart.findIndex(function(item,index){ 
                    return item.id === data.id
                    });  
                if(alreadyIndex==-1){  
                    var cartIndex = this.cart.length;
                    this.cart.push(data);
                    this.$set(this.cart[cartIndex],'quantity',1); 
                    this.saveCart();
                    return;
                }else{   
                    this.cart[alreadyIndex].quantity +=1;
                    this.saveCart();
                    return;
                }

            },
            checkOut:function(){ 
                this.$router.push({path:'/shoppingCart'});
            },
            saveCart:function(){
                localStorage.setItem('cart',JSON.stringify(this.cart));
                localStorage.setItem('Quantity',parseInt(this.selectedNum));
                localStorage.setItem('totalPrice',parseInt(this.totalPrice)); 
            }
        },
        mounted:function () {
            //一级页面falg
            isHomePage(0)
        },
        components: {
        },
    computed: {
    ...mapState({
            language: state => state.language.language
        })
    },

    };
</script>


