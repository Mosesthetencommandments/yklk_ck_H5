<template>
	<view class="outwarelist-content">
		<view class="headInfo">
			<text>查看出库单</text>
		</view>
		<view class="listnumber">
			<text>今日订单数:{{list.length}}笔</text>
		</view>
		<scroll-view scroll-y="true" class="scroll-Y">
			<view class="listInfo" v-for="item in list" :key="item.id">
				<view class="left">
					<view class="peoplegodsInfo">
						<text>{{item.name}}提货{{item.number}}箱{{item.agent_goods.goods_name}}</text>
					</view>
					<view class="address" v-if="item.customer_address !== '' && item.customer_address !== null && item.customer_address !== undefined">
						<text class="first">收货地址: 
							<text class="second">{{item.customer_address.province_name}}{{item.customer_address.district_name}}{{item.customer_address.address}}
								<text class="third">{{item.customer_address.receive_name}}收 
									<text class="last">{{item.customer_address.phone}}</text>
								</text>
							</text>
						</text>
					</view>
				</view>
				<view class="right" :class="[item.status === 3 ? 'add' : item.status === 5 ? 'right1' : '']">
					<text>{{item.status === 4 ? '已导出' : item.status === 5 ? '已出库' : '未导出'}}</text>
				</view>
			</view>
		</scroll-view>
		<view class="exportlist" @click="exportList">
			<text>显示导出订单</text>
		</view>
		<view class="exportalllist" @click="noExportList">
			<text>显示未导出订单</text>
		</view>
		<view class="all"></view>	
		<view class="all1">
			<view class="exportalllist1" @click="allList">
				<text>显示今日全部订单</text>
			</view>
		</view>
		
	</view>
</template>

<script>
	export default{
		onLoad() {
			this.checkToken()
			this.allList()
		},
		data() {
			return {
				list:[],
			}
		},
		methods:{
			allList(){
				uni.showLoading({
				    title: '加载中'
				});
				this.$http.post('/api/warehouse/getOutStockOrderList').then(response => {
					if(response.code === 200){
						this.list = response.data
						uni.hideLoading()
					}
				})
			},
			exportList(){
				uni.showLoading({
				    title: '加载中'
				});
				this.$http.post('/api/warehouse/getOutStockOrderList',{
					status:4
				}).then(response => {
					if(response.code === 200){
						this.list = response.data
						uni.hideLoading()
					}
				})
			},
			noExportList(){
				uni.showLoading({
				    title: '加载中'
				});
				this.$http.post('/api/warehouse/getOutStockOrderList',{
					status:3
				}).then(response => {
					if(response.code === 200){
						this.list = response.data
						uni.hideLoading()
					}
				})
			},
			checkToken(){
				let token = uni.getStorageSync('token')
				this.$http.post('/api/warehouse/verifyToken',{
					token:token
				}).then(response =>{
					if(response.code !== 200){
						uni.showToast({
							title:'登录已过期,请重新登录',
							duration: 1000,
							icon:'none'
						}),
						localStorage.removeItem('token')
						setTimeout(()=>{
							uni.redirectTo({
								url:'/pages/login/login'
							})
						},1100)
					}
				})
			},
		}
	}
</script>

<style scoped lang="scss">
	.outwarelist-content{
		width:100%;
		height:100vh;
		background:#f0f0f0;
		.headInfo{
			width:100%;
			height:88upx;
			background:#53affc;
			display: flex;
			justify-content: center;
			align-items: center;
			text{
				font-size:34upx;
				color:#FFFFFF;
				font-weight: 600;
			}
		}
		.listnumber{
			width:100%;
			height:52upx;
			direction: rtl;
			text{
				font-size:28upx;
				color:#3f3f3f;
				font-weight: 600;
				margin-right:26upx;
			}
		}
		.scroll-Y{
			width:100%;
			height:720upx;
			margin:0 auto;
		}
		.listInfo{
			width:694upx;
			height:164upx;
			margin:0 auto;
			margin-top:18upx;
			background:#FFFFFF;
			border-radius: 10upx 10upx 10upx 10upx;
			display: flex;
			.left{
				width:566upx;
				height:164upx;
				.peoplegodsInfo{
					width:566upx;
					height:72upx;
					position:relative;
					top:10upx;
					text{
						margin-left:22upx;
						position: absolute;
						color:#474958;
						font-size:28upx;
					}
				}
				.address{
					margin-top:10upx;
					width:566upx;
					height:103upx;
					margin-left:22upx;
					line-height:1;
					text{
						font-size:26upx;
						color:#484a59;
					}
					.second{
						margin-left:20upx;
					}
					.third{
						line-height:0;
						margin-left:18upx;
					}
				}
			}
			.right{
				width:128upx;
				height:164upx;
				display:flex;
				justify-content: center;
				text{
					margin-top:22upx;
					color:#53adfb;
					font-size:26upx;
				}
				&.add{
					text{
						color:#df2928;
					}				
				}
				&.right1{
					text{
						color:#57e40f;
					}	
				}
			}
		}
		.exportlist{
			width:610upx;
			height:80upx;
			margin:0 auto;
			background:#53affc;
			border-radius: 10upx 10upx 10upx 10upx;
			margin-top:58upx;
			display:flex;
			justify-content: center;
			align-items: center;
			text{
				font-size:34upx;
				color:#FFFFFF;
			}
		}
		.exportalllist{
			width:610upx;
			height:80upx;
			margin:0 auto;
			background:#53affc;
			border-radius: 10upx 10upx 10upx 10upx;
			margin-top:24upx;
			display:flex;
			justify-content: center;
			align-items: center;
			text{
				font-size:34upx;
				color:#FFFFFF;
			}
		}
		.all{
			width:100%;
			height:24upx;
			background:#f0f0f0;
		}
		.all1{
			width:100%;
			height:80upx;
			background:#f0f0f0;
			.exportalllist1{
				width:610upx;
				height:80upx;
				margin:0 auto;
				background:#53affc;
				border-radius: 10upx 10upx 10upx 10upx;
				display:flex;
				justify-content: center;
				align-items: center;
				text{
					font-size:34upx;
					color:#FFFFFF;
				}
			}
		}
	}
</style>