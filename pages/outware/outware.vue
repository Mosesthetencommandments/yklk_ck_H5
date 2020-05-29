<template>
	<view class="outware-content">
		<view>
			
		</view>
		<view class="list">
			<view class="listErCode">
				<view class="left">
					<img src="../../static/images/9_03.png" alt="" @click.prevent="scan1">
				</view>
				<view class="right">
					<view class="consignee">
						<text v-if="listInfo.name !== null">收货人:{{listInfo.name}}</text>
						<text v-else>收货人:</text>
					</view>
					<view class="goodsname">
						<text v-if="listInfo.goodsname !== null">收货名称:{{listInfo.goodsname}}</text>
						<text v-else>收货名称:</text>
					</view>
					<view class="number">
						<text v-if="listInfo.number !== null">收货数量:{{listInfo.number}}</text>
						<text v-else>收货数量:</text>
					</view>
				</view>
			</view>
			<view class="border"></view>
			<view class="wareErCode">
				<view class="left">
					<img src="../../static/images/qie_0001_上传照片-(2).png" alt="" @click.prevent="scan2">
				</view>
				<view class="right">
					<view class="godsname">
						<text v-if="name !== null">{{name}}</text>
						<text v-esle></text>
					</view>
					<view class="bind">
						<text v-if="number !== null">数量:{{number}}</text>
						<text v-else>数量:</text>
					</view>
				</view>
			</view>	
		</view>
		<view class="commit" @click="commit">
			<text>提交确认</text>
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
				},
				listInfo:{
					name:null,
					goodsname:null,
					number:null,
				},
				allInfo:{
					order_id:null,
					goods:[]
				},
				goods:[],
				outInfo:{},
				choose:1,
				number:0,
				name:null,
				canStart:false,
			}
		},
		methods:{
			scan1(){
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
						this.choose = 1
						// #ifdef H5
						var jweixin = require('jweixin-module')
						// #endif
						this.getWXInfo()
					}
				})
			},
			scan2(){
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
						this.choose = 2
						// #ifdef H5
						var jweixin = require('jweixin-module')
						// #endif
						this.getWXInfo()
					}
				})
			},
			getWXInfo(){
				if(this.choose === 2){
					if(this.canStart !== true){
						uni.showToast({
							title:"请先扫描订单信息",
							icon:"none"
						})
						return 
					}
				}
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
								if(this.choose === 1){
									let str = res.resultStr
									let index = str.indexOf('\=')
									str = str.substring(index + 1,str.length) / 1
									this.allInfo.order_id = str
									uni.showLoading({
									    title: '加载中'
									});
									this.$http.post('/api/warehouse/scanOrderInfo',{
										order_id:this.allInfo.order_id
									}).then(response =>{
										if(response.code === 200){
											if(response.data.length === 0){
												uni.showToast({
													title:"扫码错误",
													icon:"none"
												})
												uni.hideLoading()
												return
											}
											let arr = []
											arr = response.data
											this.listInfo.name = arr.customer_shop.user_nickname
											this.listInfo.goodsname = arr.agent_goods.goods_name
											this.listInfo.number = arr.number
											this.canStart = true
											uni.hideLoading()
										}
									})
								}
								if(this.choose === 2){
									let web = res.resultStr
									let index = web.lastIndexOf("\?");
									web = web.substring(index + 1,web.length);
									let obj = {}
									obj.type = web.substring(web.indexOf("=") +1,web.indexOf("&"))
									obj.goods_id = web.substring(web.indexOf("_")+4 ,web.lastIndexOf("&")) / 1
									obj.id = web.substring(web.lastIndexOf("=") +1,web.length) / 1
									uni.showLoading({
									    title: '加载中'
									});
									this.$http.post('/api/warehouse/scanOutStock',obj).then(response =>{
										if(response.code === 200){
											this.outInfo = response.data
											if(this.outInfo.type === 'box'){
												if(this.goods.length === 0){
													this.goods.push(this.outInfo)
													this.allInfo.goods.push(obj)
													for(let i in this.goods){
														this.number = this.goods[i].item_number / 1
														this.name = this.goods[0].agent_goods.goods_name
													}
													uni.hideLoading()
													setTimeout(()=>{
														this.getWXInfo()
													},1000)
												}else{
													for(let a in this.goods){
														if(this.goods[a].type === 'box'){
															let arr = []
															arr.push(this.goods[a])
															for(let b in arr){
																if(arr[b].id === this.outInfo.id){
																	uni.showToast({
																		title:"此货码已被扫描",
																		icon:"none"
																	})
																	uni.hideLoading()
																	return
																}
																if(arr[b].agent_goods.goods_name !==  this.outInfo.agent_goods.goods_name){
																	uni.showToast({
																		title:"货物不同,请重新扫码",
																		icon:"none"
																	})
																	uni.hideLoading()
																	return
																}
															}
														}														
													}
													this.goods.push(this.outInfo)
													this.allInfo.goods.push(this.obj)
													this.number = 0
													for(let i in this.goods){
														this.number += this.goods[i].item_number / 1
														this.name = this.goods[0].agent_goods.goods_name
													}
													uni.hideLoading()
													setTimeout(()=>{
														this.getWXInfo()
													},1000)
												}	
											}
											if(this.outInfo.type === 'item'){
												this.$set(this.outInfo, "item_number", 1)
												if(this.goods.length === 0){
													this.goods.push(this.outInfo)
													this.allInfo.goods.push(obj)
													for(let i in this.goods){
														this.number = this.goods[i].item_number / 1
														this.name = this.goods[0].agent_goods.goods_name
													}
													uni.hideLoading()
													setTimeout(()=>{
														this.getWXInfo()
													},1000)
												}else{
													for(let a in this.goods){
														if(this.goods[a].type === 'item'){
															let arr = []
															arr.push(this.goods[a])
															for(let b in arr){
																if(arr[b].id === this.outInfo.id){
																	uni.showToast({
																		title:"此货码已被扫描",
																		icon:"none"
																	})
																	uni.hideLoading()
																	return
																}
																if(arr[b].agent_goods.goods_name !==  this.outInfo.agent_goods.goods_name){
																	uni.showToast({
																		title:"货物不同,请重新扫码",
																		icon:"none"
																	})
																	uni.hideLoading()
																	return
																}
															}
														}														
													}
													this.goods.push(this.outInfo)
													this.allInfo.goods.push(obj)
													this.number = 0
													for(let i in this.goods){
														this.number += this.goods[i].item_number / 1
														this.name = this.goods[0].agent_goods.goods_name
													}
													uni.hideLoading()
													setTimeout(()=>{
														this.getWXInfo()
													},1000)
												}	
											}
										}
									})
								}
							}
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
						if(this.choose === 1){
							let str = res.result
							let index = str.indexOf('\=')
							str = str.substring(index + 1,str.length) / 1
							this.allInfo.order_id = str
							this.$http.post('/api/warehouse/scanOrderInfo',{
								order_id:this.allInfo.order_id
							}).then(response =>{
								if(response.code === 200){
									if(response.data.length === 0){
										uni.showToast({
											title:"扫码错误",
											icon:"none"
										})
										uni.hideLoading()
										return
									}
									let arr = []
									arr = response.data
									this.listInfo.name = arr.customer_shop.user_nickname
									this.listInfo.goodsname = arr.agent_goods.goods_name
									this.listInfo.number = arr.number
									this.canStart = true
									uni.hideLoading()
								}
							})
						}
						if(this.choose === 2){
							let web = res.result
							let index = web.lastIndexOf("\?");
							web = web.substring(index + 1,web.length);
							let obj = {}
							obj.type = web.substring(web.indexOf("=") +1,web.indexOf("&"))
							obj.goods_id = web.substring(web.indexOf("_")+4 ,web.lastIndexOf("&")) / 1
							obj.id = web.substring(web.lastIndexOf("=") +1,web.length) / 1
							this.$http.post('/api/warehouse/scanOutStock',obj).then(response =>{
								if(response.code === 200){
									this.outInfo = response.data
									if(this.outInfo.type === 'box'){
										if(this.goods.length === 0){
											this.goods.push(this.outInfo)
											this.allInfo.goods.push(obj)
											for(let i in this.goods){
												this.number = this.goods[i].item_number / 1
												this.name = this.goods[0].agent_goods.goods_name
											}
											uni.hideLoading()
											setTimeout(()=>{
												this.getWXInfo()
											},1000)
										}else{
											for(let a in this.goods){
												if(this.goods[a].type === 'box'){
													let arr = []
													arr.push(this.goods[a])
													for(let b in arr){
														if(arr[b].id === this.outInfo.id){
															uni.showToast({
																title:"此货码已被扫描",
																icon:"none"
															})
															uni.hideLoading()
															return
														}
														if(arr[b].agent_goods.goods_name !==  this.outInfo.agent_goods.goods_name){
															uni.showToast({
																title:"货物不同,请重新扫码",
																icon:"none"
															})
															uni.hideLoading()
															return
														}
													}
												}														
											}
											this.goods.push(this.outInfo)
											this.allInfo.goods.push(obj)
											this.number = 0
											for(let i in this.goods){
												this.number += this.goods[i].item_number / 1
												this.name = this.goods[0].agent_goods.goods_name
											}
											uni.hideLoading()
											setTimeout(()=>{
												this.getWXInfo()
											},1000)
										}	
									}
									if(this.outInfo.type === 'item'){
										this.$set(this.outInfo, "item_number", 1)
										if(this.goods.length === 0){
											this.goods.push(this.outInfo)
											this.allInfo.goods.push(obj)
											for(let i in this.goods){
												this.number = this.goods[i].item_number / 1
												this.name = this.goods[0].agent_goods.goods_name
											}
											uni.hideLoading()
											setTimeout(()=>{
												this.getWXInfo()
											},1000)
										}else{
											for(let a in this.goods){
												if(this.goods[a].type === 'item'){
													let arr = []
													arr.push(this.goods[a])
													for(let b in arr){
														if(arr[b].id === this.outInfo.id){
															uni.showToast({
																title:"此货码已被扫描",
																icon:"none"
															})
															uni.hideLoading()
															return
														}
														if(arr[b].agent_goods.goods_name !==  this.outInfo.agent_goods.goods_name){
															uni.showToast({
																title:"货物不同,请重新扫码",
																icon:"none"
															})
															uni.hideLoading()
															return
														}
													}
												}														
											}
											this.goods.push(this.outInfo)
											this.allInfo.goods.push(obj)
											this.number = 0
											for(let i in this.goods){
												this.number += this.goods[i].item_number / 1
												this.name = this.goods[0].agent_goods.goods_name
											}
											uni.hideLoading()
											setTimeout(()=>{
												this.getWXInfo()
											},1000)
										}	
									}
								}
							})
						}
					}
				})
				// #endif
			},
			commit(){
				if(this.allInfo.order_id === null || this.allInfo.order_id === undefined || this.allInfo.order_id === '' ){
					uni.showToast({
						title:"请重新扫描订单二维码",
						icon:"none"
					})
					return 
				}
				if(this.listInfo.number !== this.number){
					uni.showToast({
						title:"货物数量不对等,请重新扫码",
						icon:"none"
					})
					this.goods = []
					this.allInfo.goods = []
					this.number = 0
					this.name = null
					return 
				}
				this.$http.post('/api/warehouse/confirmOutStock',this.allInfo).then(response =>{
					if(response.code === 200){
						uni.showToast({
							title:"提交成功",
							icon:"none"
						})
						this.goods = []
						this.allInfo.goods = []
						this.number = 0
						this.name = null
						for(let i in this.listInfo){
							this.listInfo[i] = null
						}
						this.canStart = false
					}else{
						this.goods = []
						this.allInfo.goods = []
						this.number = 0
						this.name = null
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
						})
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

<style lang="scss" scoped>
	.outware-content{
		width:100%;
		height:100vh;
		background:#f3f4f6;
		.list{
			margin:0 auto;
			width:690upx;
			height:678upx;
			background:#FFFFFF;
			border-radius: 20upx 20upx 20upx 20upx;
			position: relative;
			top:30upx;
			.listErCode{
				width:100%;
				height:152upx;
				position:absolute;
				top:92upx;
				display: flex;
				.left{
					width:206upx;
					height:152upx;
					margin-left:84upx;
					img{
						width:100%;
						height:100%;
					}
				}
				.right{
					width:400upx;
					height:152upx;
					.consignee{
						margin-top:-20upx;
						margin-left:86upx;
						text{
							font-size:26upx;
							color:#2f2f2f;
							font-weight: 600;
						}
					}
					.goodsname{
						margin-top: 10upx;
						margin-left:86upx;
						text{
							font-size: 22upx;
							color:#c0c0c0;
							font-weight: 600;
						}
					}
					.number{
						margin-top: 10upx;
						margin-left:86upx;
						text{
							font-size: 22upx;
							color:#497fd7;
							font-weight: 600;
						}
					}
				}
			}
			.border{
				width:540upx;
				height:78upx;
				border-bottom: 1px solid #c5d9e2;
				position:absolute;
				top:244upx;
				margin-left:66upx;
			}
			.wareErCode{
				width:100%;
				height:152upx;
				position:absolute;
				top:400upx;
				display: flex;
				.left{
					width:206upx;
					height:152upx;
					margin-left:84upx;
					img{
						width:100%;
						height:100%;
					}
				}
				.right{
					width:400upx;
					height:152upx;
					.godsname{
						margin-top: 10upx;
						margin-left:86upx;
						text{
							font-size:26upx;
							color:#2f2f2f;
							font-weight: 600;
						}
					}
					.bind{
						margin-top: 20upx;
						margin-left:86upx;
						text{
							font-size: 22upx;
							color:#497fd7;
							font-weight: 600;
						}
					}
				}
			}
		}
		.commit{
			width:550upx;
			height:86upx;
			margin:0 auto;
			background:#4881d8;
			border-radius: 10upx 10upx 10upx 10upx;
			margin-top:285upx;
			display: flex;
			justify-content: center;
			align-items: center;
			text{
				font-size:30upx;
				color:#FFFFFF;
			}
		}
	}
</style>
