<template>
	<view class="inware-content">
		<view class="erCodeInfo">
			<view class="left">
				<img src="@/static/images/5_03.png" alt="">
			</view>
			<view class="right">
				<view class="id">
					<text>二维码ID:{{boxid}}</text>
				</view>
				<view class="again" @click.prevent="scanAgain">
					<text>重新扫码</text>
				</view>
			</view>
		</view>
		<view class="border">
			<view class="A"></view>
			<view class="B"></view>
		</view>
		<view class="gods">
			<view class="godsImg">
				<img :src="godsInfo.img" alt="">
			</view>
			<view class="godsInfo">
				<view class="first" v-if='godsInfo.name !== null'>
					<text>{{godsInfo.name}}</text>
				</view>
				<view class="second">
					<text>入库数量:{{list.items_info.length}}</text>
				</view>
				<view class="last" @click.prevent="useScan">
					<text>新增货码</text>
				</view>
			</view>
		</view>
		<view class="allbinding">
			<text>累计绑定货码:{{list.items_info.length}}</text>
		</view>
		<view class="save" @click="sendList">
			<text>保</text>
			<text>存</text>
		</view>
	</view>
</template>

<script>
	// #ifdef H5
	import jweixin from "jweixin-module"
	// #endif
	export default{
		onLoad(options) {
			uni.showLoading({
			    title: '加载中'
			});
			// let type = option.type
			// if(type === "item"){
			// 	uni.showToast({
			// 		title:'扫码错误,请重新扫码',
			// 		icon:"none"
			// 	})
			// 	setTimeout(()=>{
			// 		uni.redirectTo({
			// 			url:"./scanQrCode"
			// 		})
			// 	},1100)
			// 	return 
			// }
			this.boxid = options.id / 1
			this.list.box_id = options.id / 1
			uni.hideLoading()
		},
		data(){
			return{
				boxid:null,
				erCode:null,
				list:{
					box_id:null,
					items_info:[],
				},
				godsInfo:{
					name:null,
					img:null,
					number:1
				},
				wx_info:{
					appId:null,
					nonceStr:null,
					rawString:null,
					signature:null,
					timestamp:null,
					url:null,
				},
			}
		},
		methods:{
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
			scanAgain(){
				this.list = []
				uni.navigateTo({
					url:"../scanQrcode/scanQrCode"
				})
			},
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
						localStorage.removeItem('token')
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
					jweixin.ready(()=> {
						jweixin.scanQRCode({
						needResult : 1,
						scanType : [ "qrCode","barCode"],
						success : res => {
								uni.showLoading({
									title: '加载中'
								});
								let web = res.resultStr
								let index = web.lastIndexOf("\?");
								web = web.substring(index + 1,web.length);
								let obj = {}
								obj.type = web.substring(web.indexOf("=") +1,web.indexOf("&"))
								obj.goods_id = web.substring(web.indexOf("_")+4 ,web.lastIndexOf("&")) / 1
								obj.id = web.substring(web.lastIndexOf("=") +1,web.length) / 1
								if(obj.type !== 'item'){
									uni.showToast({
										title:'扫码错误,请重新扫码',
										icon:"none"
									})
									return 
								}
								if(this.list.items_info.length === 0){
									this.list.items_info.push(obj)
								}else{
									for(let a in this.list.items_info){
										if(this.list.items_info[a].id === obj.id){
											uni.showToast({
												title:"此二维码已被扫描,请重新扫码",
												icon:"none"
											})
											return 
										}
									}
									this.list.items_info.push(obj)
								}
								this.$http.post('/api/warehouse/scanGoodsInfo',{
									goods_id:this.list.items_info[0].goods_id / 1
								}).then(response =>{
									uni.showToast({
										title:"扫描成功",
										icon:"none"
									})
									this.godsInfo.name = response.data.goods_name
									this.godsInfo.img = response.data.goods_image
									uni.hideLoading()
									setTimeout(()=>{
										this.getWXInfo()
									},1000)
								})
							},
						});
					});
				})
				// #endif
				// #ifdef MP-WEIXIN
				uni.hideLoading()
				wx.scanCode({
					success : res =>{
						uni.showLoading({
							title: '加载中'
						});
						let web = res.result
						let index = web.lastIndexOf("\?");
						web = web.substring(index + 1,web.length)
						let obj = {}
						obj.type = web.substring(web.indexOf("=") +1,web.indexOf("&"))
						obj.goods_id = web.substring(web.indexOf("_")+4 ,web.lastIndexOf("&")) / 1
						obj.id = web.substring(web.lastIndexOf("=") +1,web.length) / 1
						if(obj.type !== 'item'){
							uni.showToast({
								title:'扫码错误,请重新扫码',
								icon:"none"
							})
							return 
						}
						if(this.list.items_info.length === 0){
							this.list.items_info.push(obj)
						}else{
							for(let a in this.list.items_info){
								if(this.list.items_info[a].id === obj.id){
									uni.showToast({
										title:"此二维码已被扫描,请重新扫码",
										icon:"none"
									})
									return 
								}
							}
							this.list.items_info.push(obj)
						}
						this.$http.post('/api/warehouse/scanGoodsInfo',{
							goods_id:this.list.items_info[0].goods_id / 1
						}).then(response =>{
							uni.showToast({
								title:"扫描成功",
								icon:"none"
							})
							this.godsInfo.name = response.data.goods_name
							this.godsInfo.img = response.data.goods_image
							uni.hideLoading()
							setTimeout(()=>{
								this.getWXInfo()
							},1000)
						})
					}
				})
				// #endif
			},
			sendList(){
				this.$http.post('/api/warehouse/itemReceiving',this.list).then(response =>{
					if(response.code === 200){
						uni.showToast({
							title:"保存成功",
							icon:"none"
						})
						this.list.box_id = null
						this.list.items_info = []
						this.godsInfo.name = null
						this.godsInfo.img = null
					}else{
						setTimeout(()=>{
							uni.redirectTo({
								url:"./scanQrCode"
							})
						},1100)
					}
				})
			},
		}
	}
</script>

<style lang="scss" scoped>
	.inware-content{
		width:100%;
		height:100vh;
		background:#f3f4f6;
		.erCodeInfo{
			width:690upx;
			height:284upx;
			margin:0 auto;
			background:#FFFFFF;
			border-radius: 25upx 25upx 25upx 25upx;
			display: flex;
			.left{
				width:388upx;
				height:284upx;
				img{
					margin-top:40upx;
					margin-left: 76upx;
					width:234upx;
					height:198upx;
					border-radius: 25upx 25upx 25upx 25upx;
				}
			}
			.right{
				width:302upx;
				height:284upx;
				.id{
					margin-top:60upx;
					text{
						font-size:26upx;
						color:#343434;
					}
				}
				.again{
					margin-top: 40upx;
					text{
						font-size:22upx;
						color:#5685dd;
					}
				}
			}
		}
		.border{
			width:490upx;
			height:40upx;
			margin:0 auto;
			margin-top:40upx;
			display: flex;
			justify-content: space-between;
			.A{
				width:228upx;
				height:40upx;
				border-bottom: 1px solid #c5d9e4;
			}
			.B{
				width:228upx;
				height:40upx;
				border-bottom: 1px solid #c5d9e4;
			}
		}
		.gods{
			width:484upx;
			height:198upx;
			margin:0 auto;
			margin-top:56upx;
			display: flex;
			.godsImg{
				width:198upx;
				height:198upx;
				img{
					width:100%;
					height:100%;
				}
			}
			.godsInfo{
				width:286upx;
				height:198upx;
				.first{
					margin:0 auto;
					margin-top:30upx;
					margin-left:76upx;
					position: relative;
					width:100%;
					text{
						font-size:26upx;
						color:#2e2e30;
						position: absolute;
						top:0''
					}
				}
				.second{
					margin:0 auto;
					margin-top:77upx;
					margin-left:76upx;
					position: relative;
					width:100%;
					text{
						font-size:26upx;
						color:#c2c2c4;
						position: absolute;
						top:0''
					}
				}
				.last{
					margin:0 auto;
					margin-top:122upx;
					margin-left:76upx;
					position: relative;
					width:100%;
					text{
						font-size:26upx;
						color:#3c80e1;
						position: absolute;
						top:0;
					}
				}
			}
		}
		.allbinding{
			width:178upx;
			margin:0 auto;
			margin-top: 280upx;
			text{
				color:#c3c3c5;
				font-size:22upx;
			}
		}
		.save{
			width:550upx;
			height:82upx;
			background:#4881d8;
			border-radius: 10upx 10upx 10upx 10upx;
			margin:0 auto;
			display: flex;
			align-items: center;
			margin-top:40upx;
			text{
				font-size:30upx;
				color:#FFFFFF
			}
			text:first-child{
				margin-left:230upx;
			}
			text:last-child{
				margin-left:30upx;
			}
		}
	}
</style>
