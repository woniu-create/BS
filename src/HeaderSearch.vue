<template>
  <div class="header_container">
    <div class="header_img">
      <img src="../../common/img/logo-round.png" alt="" />
    </div>
    <div class="header_search">
      <div>
        <input type="text" placeholder="搜索 商品名称" v-model="keywords" />
        <el-button
          type="danger"
          icon="el-icon-search"
          @click="postSearch"
        ></el-button>
      </div>
      <p>
        <a>自营日满减</a>
        <a>西二女装</a>
        <a>笔记本电脑</a>
        <a>电子书</a>
        <a>限时秒杀</a>
      </p>
    </div>
    <div class="shopping_bag">
      <a @click.prevent="goShopCar">
        <i class="el-icon-shopping-cart-2">> &nbsp;&nbsp;我的购物车 </i>
      </a>
    </div>
  </div>
</template>
<script>
  import { MessageBox } from 'element-ui'
  import {mapState} from 'vuex'
  import {mapActions} from 'vuex'
export default {
	data(){
		return {
			keywords: ''
		}
	},
	computed: {
		...mapState(['userInfo'])
	},
	methods:{
		async postSearch(){
          if(this.keywords){
			  let keywords = this.keywords;
			  this.$store.dispatch('reqSearch',{keywords})
			  this.$router.replace('/searchdetail')
		  }
		},
		goShopCar(){
		  if(this.userInfo.id){
			 this.$router.replace('/shopcar');
		  }else{
			  MessageBox({
				 type: 'info',
				 message: "请先登录!",
				 showClose:true
			  })
		  }
		}
	}
};
</script>
<style scoped>
.header_container {
  position: relative;
  width: 100%;
  height: 140px;
  margin: 0;
  padding: 0;
  border: 1px solid red;
}
.header_container>.header_img{
	position: absolute;
	left: 100px;
	margin-top: 10px;
	width: 140px;
	border: 1px solid darkgrey;
}
.header_img>img{
	width: 100%;
	height: 100%;
	border-radius: 50%;
}
.header_img:hover{
	transform-style: preserve-3d;
	transform: rotateY(360deg);
	transition: all 1500ms;
}
.header_search{
	position: absolute;
	left: 50%;
	margin-left: -290px;
	margin-top: 50px;
}
.header_search div{
	display: flex;
	align-items: center;
}
.header_search input{
	padding-left:10px ;
	width: 490px;
	height: 35px;
	font-size: 14px;
	line-height: 35px;
	border: 2px solid #FF0236;
	border-right: none;
	outline: none;
}
.header_search button.el-button.el-button--danger{
	padding: 0;
	width: 60px;
	height: 35px;
    font-size: 18px;
	border-radius: 0;
	outline: none;
	display: flex;
	justify-content: center;
	align-items: center;
}
.header_search>p>a{
	display: inline-block;
	padding: 5px 10px;
	font-size: 12px;
	color: #999;
	cursor: pointer;
}
.header_search>p>a:hover{
	color: red;
}
.header_container>.shopping_bag{
	position: absolute;
	margin-top: 60px;
	right: 120px;
	width: 190px;
	height: 35px;
	text-align: center;
	line-height: 35px;
	font-size: 12px;
	background: #f8f9fa;
	color: red;
	cursor: pointer;
}
.shopping_bag>img{
	display: inline-block;
	margin-left: 5px;
	margin-top: 5px;
	width: 17px;
	height: 17px;
}
.shopping_bag>a{
	color: red;
}
</style>
