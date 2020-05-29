<template>
	<view class="scanQrCode-content">
		<view class="scanImg" @click="useScan">
			<img src="../../static/images/qie_0000_上传照片-(1).png" alt="">
		</view>
	</view>
</template>

<script>
	// #ifdef H5
	import jweixin from "jweixin-module"
	// #endif
	export default{
		onLoad() {
			this.checkToken()
			// let web = "http://warehouseapp.home258.com/#/pages/inware/inware?type=item&goods_id=2323123&id=222"
			// let index = web.lastIndexOf("\?");
			// web = web.substring(index + 1,web.length);
			// let obj = {}
			// obj.type = web.substring(web.indexOf("=") +1,web.indexOf("&"))
			// obj.goods_id = web.substring(web.indexOf("_")+4 ,web.lastIndexOf("&")) / 1
			// obj.id = web.substring(web.lastIndexOf("=") +1,web.length) / 1
			// console.log(obj)
		},
		data(){
			return{
				wx_info:{
					appId:null,
					nonceStr:null,
					rawString:null,
					signature:null,
					timestamp:null,
					url:null,
				}
			}
		},
		methods:{
			useScan(){
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
								url:'/pages/login/login'
							})
						},1100)
					}else{
						// #ifdef H5
						var jweixin = require('jweixin-module')
						// #endif
						this.getWXInfo()
					}
				})
			},
			getWXInfo(){
				uni.showLoading({
				    title: '加载中'
				});
				// #ifdef H5
				this.$http.post('/api/shop/getWxConf',{
					url:window.location.href.split('#')[0] 
				}).then(response => {
					this.wx_info.appId = response.data.appId
					this.wx_info.nonceStr =  response.data.nonceStr
					this.wx_info.rawString = response.data.rawString
					this.wx_info.signature = response.data.signature
					this.wx_info.timestamp = response.data.timestamp
					this.wx_info.url = response.data.url
					jweixin.config({
					  debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
					  appId: this.wx_info.appId, // 必填，公众号的唯一标识
					  timestamp:this.wx_info.timestamp , // 必填，生成签名的时间戳
					  nonceStr: this.wx_info.nonceStr, // 必填，生成签名的随机串
					  signature: this.wx_info.signature,// 必填，签名
					  jsApiList: ['scanQRCode'] // 必填，需要使用的JS接口列表
					});
					uni.hideLoading()
					jweixin.ready(function() {
						jweixin.scanQRCode({
							needResult : 1,
							scanType : [ "qrCode","barCode"],
							success : res => {
								let web = res.resultStr
								let index = web.lastIndexOf("\?");
								web = web.substring( index + 1,web.length);
								setTimeout(()=>{
									uni.redirectTo({
										url:"../inware/inware?" + web 
									})
								},500)
							}
						});
					});
				})
				// #endif
				// #ifdef MP-WEIXIN
				uni.hideLoading()
				wx.scanCode({
				  success : res =>{
				   let web = res.result
				   let index = web.lastIndexOf("\?");
				   web = web.substring( index + 1,web.length);
				   setTimeout(()=>{
				   	uni.redirectTo({
				   		url:"../inware/inware?" + web 
				   	})
				   },500)
				  }
				})
				// #endif
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

<style lang="scss" scoped>
	.scanQrCode-content{
		width:100%;
		height:100vh;
		.scanImg{
			width:550upx;
			height:410upx;
			margin:0 auto;
			margin-top:58upx;
			img{
				width:100%;
				height:100%;
			}
		}
	}
</style>
