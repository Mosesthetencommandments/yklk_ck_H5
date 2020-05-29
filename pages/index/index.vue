<template>
	<view class="index-content">
		<view class="topImg">
			<img src="@/static/images/1_01.png" alt="">
		</view>
		<view class="center">
			<view class="inware" @click="gotoscan">
				<text>扫码入库</text>
			</view>
			<view class="outware" @click="gotooutware">
				<text>扫码出库</text>
			</view>
			<view class="looklist" @click="gotowarelist">
				<text>查看出库单</text>
			</view>
			<view class="people">
				<text>操作人:{{user.nickname}}</text>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				user:{},
			}
		},
		onLoad() {
			// #ifdef H5
			this.checkToken()
			// #endif
		},
		onShow() {
			uni.showLoading({
				title:"加载中"
			})
			let token = uni.getStorageSync('token')
			this.$http.post("/api/warehouse/homePageInfo",{
				token:token
			}).then(response =>{
				if(response.code === 200){
					this.user = response.data.user
					uni.hideLoading()
				}
			})
		},
		methods: {
			gotoscan(){
				uni.navigateTo({
					url:"../scanQrcode/scanQrCode"
				})
			},
			gotowarelist(){
				uni.navigateTo({
					url:"../outwarelist/outwarelist"
				})
			},
			gotooutware(){
				uni.navigateTo({
					url:"../outware/outware"
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
						// #ifdef H5
						localStorage.removeItem('token')
						// #endif
						// #ifdef MP-WEIXIN
						wx.removeStorage({
							key:'token'
						})
						// #endif
						setTimeout(()=>{
							uni.redirectTo({
								url:'../pages/login/login'
							})
						},1100)
					}
				})
			},
		}
	}
</script>

<style scoped lang="scss">
	.index-content{
		width:100vw;
		height:100vh;
		background: #ebebeb;
		.topImg{
			width:100%;
			height:534upx;
			img{
				width:100%;
				height:100%;
			}
		}
		.center{
			width:100%;
			height:646upx;
			background: #ebebeb;
			.inware{
				margin:0 auto;
				width:525upx;
				height:86upx;
				background:#FFFFFF;
				margin-top:102upx;
				display: flex;
				justify-content: center;
				align-items: center;
				border-radius: 10upx 10upx 10upx 10upx;
				text{
					font-size:34upx;
					font-weight: 600;
					color:#3e3e3e;
				}
			}
			.outware{
				margin:0 auto;
				width:525upx;
				height:86upx;
				background:#FFFFFF;
				margin-top:62upx;
				display: flex;
				justify-content: center;
				align-items: center;
				border-radius: 10upx 10upx 10upx 10upx;
				text{
					font-size:34upx;
					font-weight: 600;
					color:#3e3e3e;
				}
			}
			.looklist{
				margin:0 auto;
				width:525upx;
				height:104upx;
				background:#3399fe;
				margin-top:62upx;
				display: flex;
				justify-content: center;
				align-items: center;
				border-radius: 10upx 10upx 10upx 10upx;
				text{
					font-size:44upx;
					font-weight: 600;
					color:#FFFFFF;
				}
			}
			.people{
				margin:0 auto;
				width:279upx;
				height:50upx;
				background:#FFFFFF;
				margin-top:78upx;
				display: flex;
				justify-content: center;
				align-items: center;
				border:1px solid #d7d6db;
				border-radius: 10upx 10upx 10upx 10upx;
				text{
					font-size:28upx;
					font-weight: 600;
					color:#3e3e3e;
				}
			}		
		}
	}
</style>
