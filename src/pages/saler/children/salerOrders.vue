<template>
	<div class="page">
	  	<div class="page-content">
	  		<von-header theme="light">
			  <button class="button button-icon ion-ios-arrow-back" slot="left" @click="back"></button>
			  <span slot="title">卖家订单中心</span>
			</von-header>
			<div class="tabs">
		  			<tabs :tab-items="tabs" :tab-index="tabIndex" :on-tab-click="onTabClick" top="0" style="width: 100%;"></tabs>
				</div>
			<quick-loadmore :top-method="onRefresh" ref="vueLoad" 
                    :top-status-change="handleStatusChange" 
                    :bottom-method="onInfinite" 
                    :bottom-status-change="handleBottomStatusChange" 
                    :disable-top="disableTop" :disable-bottom="disableBottom" >
		  		<order-part v-for="(data,index) in 	allDatas" :data="data" class="mt30" :tabIndex="tabIndex" v-if="loadFinish && allDatas.length>0"></order-part>
		  		<!--<h5 v-if="allDatas < 1 && loadFinish == true">暂无订单</h5>-->
			</quick-loadmore>	
	  	</div>
	</div>
</template>

<script>
	import orderPart from '../components/orderPart.vue'
	export default{
		name:'salerOrders',
		components:{
	  		orderPart
		},
		data(){
			return{
				tabs:['全部','待付款','待发货','待收货'],
				tabIndex:0,
				allDatas:[],
				flag:'fresh',
		        disableTop:false,//是否禁止下拉，默认允许
		        disableBottom:false,
		        loadFinish:true,
		        isAll:false,
				page:1,
				type:0, //区别返回的数据 0全部 1待付款2待发货3待收货
			}
		},
		watch:{
			tabIndex(val){ //切换tab时设置this.page = 1
				this.allDatas = []
			 	switch (val){
			 		case 0:
			 			this.type = 0
			 			this.page = 1
			 			this.isAll = false
			 			this.flag == 'fresh'
			 			this.getOrders()
			 			break;
			 		case 1:
			 			this.type = 1
			 			this.page = 1
			 			this.isAll = false
			 			this.flag == 'fresh'
			 			this.getOrders()
			 			break;
			 		case 2:
			 			this.type = 2
			 			this.page = 1
			 			this.isAll = false
			 			this.flag == 'fresh'
			 			this.getOrders()
			 			break;
			 		case 3:
			 			this.type = 3
			 			this.page = 1
			 			this.isAll = false
			 			this.flag == 'fresh'
			 			this.getOrders()
			 			break;
			 		default:
			 			break;
			 	}
			},
			loadFinish(val){
				this.loadFinish = val
			}
		},
		methods:{
			back(){
				this.$router.go(-1)
			},
			onTabClick(index){
				console.log(index)
				this.tabIndex = index
			},
			handleStatusChange(status) {
		      // status监控下拉刷新状态--等待/下拉/到达阙值/刷新
		      const TOPSTATUS = {
		        wait: "wait",
		        pulling: "pulling",
		        limit: "limit",
		        loading: "loading"
		      }; 
		    },
		    handleBottomStatusChange(status) {
		      // status监控上拉加载状态--等待/加载/没有更多数据  
		      const BOTTOMSTATUS = {
		        wait: "wait",
		        loading: "loading",
		        nodata: "nodata"
		      };
		    },
	    	onRefresh(){ //下拉刷新  		
	    		this.disableBottom = true
    			this.flag = 'fresh'
	    		this.page = 1 //第一页
				this.getOrders()
	    	},
	    	onInfinite(){//上拉加载
	    		this.disableTop = true
				this.flag = 'infinite'
				this.page++ //下一页
				this.getOrders()
	    	},
			getOrders(){//获取订单 type为类型路由
				let that = this
	    		if(!that.loadFinish){
	    			return
	    		}
				that.$axios({
					method:'post',
					url:that.GLOBAL.develop_url+'api/business_order',
					data:{
						token:localStorage.getItem('token'),
						page:that.page,
						type:that.type						
					},
	    			transformRequest:[function(data){
	    				let ret = ''
	    				for(let it in data){
	    					ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&'
	    				}
	    				return ret
	    			}],
					xhrFields:{
		    			withCredentials:true
	    			},
		    		crossDomain:true,
		    		headers: { //解决跨域
						'Content-Type': 'application/x-www-form-urlencoded'
					}
				}).then(function(res){
					console.log(res.data)
					switch(res.data.code){
						case "200":
							if(res.data.data.length == 0){
								that.disableBottom = true
								that.$refs.vueLoad.onBottomLoaded(false)
								if(that.flag == 'fresh'){
									that.allDatas = []
									that.isAll = true
		    						that.$refs.vueLoad.onTopLoaded()
								}else{
									that.isAll = true
								}
							}else{
								that.disableBottom = false
								that.$refs.vueLoad.onBottomLoaded(true)
								if(that.flag == 'fresh'){
									that.allDatas = []
									res.data.data.forEach(function(item){
										item.c_time =  that.timeFormer(item.c_time)
//										item.photo = 'http://img.paipaiwang.com.cn/' + item.photo
//										item.thumb = 'http://img.paipaiwang.com.cn/' + item.thumb
										that.allDatas.push(item)
									})
		    						that.$refs.vueLoad.onTopLoaded()
								}else{
									res.data.data.forEach(function(item){
										item.c_time =  that.timeFormer(item.c_time)
//										item.photo = 'http://img.paipaiwang.com.cn/' + item.photo
//										item.thumb = 'http://img.paipaiwang.com.cn/' + item.thumb
										that.allDatas.push(item)
									})
								}
							}
							break;
						case "400":
							$toast.show('获取用户信息失败，请重新登录', 1000).then(() => {
								that.$router.push('/login')
							})
							break;
					}
					if(res.data.code == 404){
						$toast.show(res.data.code, 1000).then(() => {
								that.$router.push('/login')
							})
					}
	    			that.loadFinish = true
				}).catch(function(err){
					//$toast.show('网络错误', 2000).then(() => {})
				})
			}
		},
		created(){
			this.tabIndex = this.$route.params.index ||0
			if(this.tabIndex==0){
				this.getOrders()
			}
		}
	}
</script>

<style scoped lang="less">
.page-content {
  	padding-top: 93px;
  	.mt30{
  		margin-top: .3rem;
  	}
  	.tabs{
  		position: fixed;
  		top: 44px;
  		width: 100%;
  	}
}
</style>