<template>
	<view class="container">
		<view class="carousel">
			<swiper indicator-dots circular=true duration="400">
				<swiper-item class="swiper-item" v-for="(item,index) in [{src:goodsInfo.pic1}]" :key="index">
					<view class="image-wrapper">
						<image
							:src="picUrl + item.src" 
							class="loaded" 
							mode="aspectFill"
						></image>
					</view>
				</swiper-item>
			</swiper>
		</view>
		
		<view class="introduce-section u-flex u-row-between">
			<view><text class="title">{{goodsInfo.title}}</text></view>
			
			<view @click="collect(goodsInfo.id)">
				<u-icon name="star-fill" color="#F0AD4E" size="40"></u-icon>
				<text class="u-tips-color u-m-l-5">收藏</text>
				
			</view>
			
			<view v-if="user.type == 3" @click="del(goodsInfo.id)">
				<u-icon name="close-circle" color="#DD524D" size="40"></u-icon>
				<text class="u-tips-color u-m-l-5 ">删除</text>
				
			</view>
			
		</view>

		
		<!-- 评价 -->
		<view class="eva-section">
			<view class="e-header u-p-t-10 u-p-b-10">
				<text class="tit" style="min-width: 65rpx;">功效</text>
				<view class="u-m-l-30">{{goodsInfo.benefit}}</view>
			</view> 
		</view>
		
		<view class="eva-section">
			<view class="e-header u-p-t-10 u-p-b-10">
				<text class="tit" style="min-width: 65rpx;">食材</text>
				<view class="u-m-l-30">{{goodsInfo.goods}}</view>
			</view> 
		</view>
		
		<view class="detail-desc">
			<view class="d-header">
				<text>烹饪教程</text>
			</view>
			<view class="" style="">
				<view class="u-flex u-row-center u-p-20" style="width: 100vw">{{goodsInfo.context}}</view>
				<image :src="picUrl+goodsInfo.pic2" style="width: 750rpx;" mode="widthFix"></image>
				<image :src="picUrl+goodsInfo.pic3" style="width: 750rpx;" mode="widthFix"></image>
			</view>
		</view>
		
	</view>
</template>

<script>
	import appRequest from "@/common/appRequestUrl.js";
	import share from '@/components/share';
	export default{
		components: {
			share
		},
		data() {
			return {
				commentList:[],
				picUrl:appRequest.getPicById,
				user:false,
				specClass: 'none',
				specSelected:[],
				goodsInfo:{},
				favorite: true
			};
		},onShow() {
			this.user = appRequest.getUserInfo();
			
		},
		onLoad(options){
			this.getGoods(options.id);
		},
		methods:{
			del(id){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.saveRecipes,
					data:{
						data:JSON.stringify({id:id,validFlag:0})
					},
					success: function(res) {
						if(res.data.code == 200){
							_this.$api.msg("删除成功");
							uni.showModal({
								title:"删除成功",
								content:"当前食谱已删除，将返回上一页面",
								showCancel:false,
								success: (res) => {
									if(res.confirm){
										uni.navigateBack({
											
										})
									}
								}
							})
						}else{
							_this.$api.msg(res.data.msg);
						}
				
					},
					fail: function(res) {
						_this.$api.msg("请求异常");
					}
				})
			},
			collect(id){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.saveCollect,
					data:{
						id:id,
						type:1
					},
					success: function(res) {
						if(res.data.code == 200){
							_this.$api.msg("收藏成功");
						}else{
							_this.$api.msg(res.data.msg);
						}
				
					},
					fail: function(res) {
						_this.$api.msg("请求异常");
					}
				})
			},
			getGoods(id){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.getRecipes,
					data:{
						data:JSON.stringify({id:id})
					},
					success: function(res) {
						if(res.data.code == 200){
							_this.goodsInfo = res.data.data[0];
						}else{
							_this.$api.msg(res.data.msg);
						}
				
					},
					fail: function(res) {
						_this.$api.msg("请求异常");
					}
				})
				
				
				
			},
			//规格弹窗开关
			toggleSpec() {
				if(this.specClass === 'show'){
					this.specClass = 'hide';
					setTimeout(() => {
						this.specClass = 'none';
					}, 250);
				}else if(this.specClass === 'none'){
					this.specClass = 'show';
				}
			},
			//选择规格
			selectSpec(index, pid){
				let list = this.specChildList;
				list.forEach(item=>{
					if(item.pid === pid){
						this.$set(item, 'selected', false);
					}
				})

				this.$set(list[index], 'selected', true);
				//存储已选择
				/**
				 * 修复选择规格存储错误
				 * 将这几行代码替换即可
				 * 选择的规格存放在specSelected中
				 */
				this.specSelected = []; 
				list.forEach(item=>{ 
					if(item.selected === true){ 
						this.specSelected.push(item); 
					} 
				})
				
			},
			//分享
			share(){
				this.$refs.share.toggleMask();	
			},
			//收藏
			toFavorite(){
				this.favorite = !this.favorite;
			},
			buy(item){
				let _this= this;
				if(this.user.type != 1){
					this.$api.msg("游客、管理员及商家不能参与购物，请使用消费者帐户购买");
					return;
				}
				uni.navigateTo({
					url: `/pages/order/createOrder?data=${JSON.stringify(
						[{id:0,gid:_this.goodsInfo.id,num:1}]
					)}`
				})
			},
			stopPrevent(){}
		},

	}
</script>

<style lang='scss'>
	page{
		background: $page-color-base;
		padding-bottom: 160upx;
	}
	.icon-you{
		font-size: $font-base + 2upx;
		color: #888;
	}
	.carousel {
		height: 722upx;
		position:relative;
		swiper{
			height: 100%;
		}
		.image-wrapper{
			width: 100%;
			height: 100%;
		}
		.swiper-item {
			display: flex;
			justify-content: center;
			align-content: center;
			height: 750upx;
			overflow: hidden;
			image {
				width: 100%;
				height: 100%;
			}
		}
		
	}
	
	/* 标题简介 */
	.introduce-section{
		background: #fff;
		padding: 20upx 30upx;
		
		.title{
			font-size: 32upx;
			color: $font-color-dark;
			height: 50upx;
			line-height: 50upx;
		}
		.price-box{
			display:flex;
			align-items:baseline;
			height: 64upx;
			padding: 10upx 0;
			font-size: 26upx;
			color:$uni-color-primary;
		}
		.price{
			font-size: $font-lg + 2upx;
		}
		.m-price{
			margin:0 12upx;
			color: $font-color-light;
			text-decoration: line-through;
		}
		.coupon-tip{
			align-items: center;
			padding: 4upx 10upx;
			background: $uni-color-primary;
			font-size: $font-sm;
			color: #fff;
			border-radius: 6upx;
			line-height: 1;
			transform: translateY(-4upx); 
		}
		.bot-row{
			display:flex;
			align-items:center;
			height: 50upx;
			font-size: $font-sm;
			color: $font-color-light;
			text{
				flex: 1;
			}
		}
	}
	/* 分享 */
	.share-section{
		display:flex;
		align-items:center;
		color: $font-color-base;
		background: linear-gradient(left, #fdf5f6, #fbebf6);
		padding: 12upx 30upx;
		.share-icon{
			display:flex;
			align-items:center;
			width: 70upx;
			height: 30upx;
			line-height: 1;
			border: 1px solid $uni-color-primary;
			border-radius: 4upx;
			position:relative;
			overflow: hidden;
			font-size: 22upx;
			color: $uni-color-primary;
			&:after{
				content: '';
				width: 50upx;
				height: 50upx;
				border-radius: 50%;
				left: -20upx;
				top: -12upx;
				position:absolute;
				background: $uni-color-primary;
			}
		}
		.icon-xingxing{
			position:relative;
			z-index: 1;
			font-size: 24upx;
			margin-left: 2upx;
			margin-right: 10upx;
			color: #fff;
			line-height: 1;
		}
		.tit{
			font-size: $font-base;
			margin-left:10upx;
		}
		.icon-bangzhu1{
			padding: 10upx;
			font-size: 30upx;
			line-height: 1;
		}
		.share-btn{
			flex: 1;
			text-align:right;
			font-size: $font-sm;
			color: $uni-color-primary;
		}
		.icon-you{
			font-size: $font-sm;
			margin-left: 4upx;
			color: $uni-color-primary;
		}
	}
	
	.c-list{
		font-size: $font-sm + 2upx;
		color: $font-color-base;
		background: #fff;
		.c-row{
			display:flex;
			align-items:center;
			padding: 20upx 30upx;
			position:relative;
		}
		.tit{
			width: 140upx;
		}
		.con{
			flex: 1;
			color: $font-color-dark;
			.selected-text{
				margin-right: 10upx;
			}
		}
		.bz-list{
			height: 40upx;
			font-size: $font-sm+2upx;
			color: $font-color-dark;
			text{
				display: inline-block;
				margin-right: 30upx;
			}
		}
		.con-list{
			flex: 1;
			display:flex;
			flex-direction: column;
			color: $font-color-dark;
			line-height: 40upx;
		}
		.red{
			color: $uni-color-primary;
		}
	}
	
	/* 评价 */
	.eva-section{
		display: flex;
		flex-direction: column;
		padding: 20upx 30upx;
		background: #fff;
		margin-top: 16upx;
		.e-header{
			display: flex;
			align-items: center;
			height: 70upx;
			font-size: $font-sm + 2upx;
			color: $font-color-light;
			.tit{
				font-size: $font-base + 2upx;
				color: $font-color-dark;
				margin-right: 4upx;
			}
			.tip{
				flex: 1;
				text-align: right;
			}
			.icon-you{
				margin-left: 10upx;
			}
		}
	}
	.eva-box{
		display: flex;
		padding: 20upx 0;
		.portrait{
			flex-shrink: 0;
			width: 80upx;
			height: 80upx;
			border-radius: 100px;
		}
		.right{
			flex: 1;
			display: flex;
			flex-direction: column;
			font-size: $font-base;
			color: $font-color-base;
			padding-left: 26upx;
			.con{
				font-size: $font-base;
				color: $font-color-dark;
				padding: 20upx 0;
			}
			.bot{
				display: flex;
				justify-content: space-between;
				font-size: $font-sm;
				color:$font-color-light;
			}
		}
	}
	/*  详情 */
	.detail-desc{
		background: #fff;
		margin-top: 16upx;
		.d-header{
			display: flex;
			justify-content: center;
			align-items: center;
			height: 80upx;
			font-size: $font-base + 2upx;
			color: $font-color-dark;
			position: relative;
				
			text{
				padding: 0 20upx;
				background: #fff;
				position: relative;
				z-index: 1;
			}
			&:after{
				position: absolute;
				left: 50%;
				top: 50%;
				transform: translateX(-50%);
				width: 300upx;
				height: 0;
				content: '';
				border-bottom: 1px solid #ccc; 
			}
		}
	}
	
	/* 规格选择弹窗 */
	.attr-content{
		padding: 10upx 30upx;
		.a-t{
			display: flex;
			image{
				width: 170upx;
				height: 170upx;
				flex-shrink: 0;
				margin-top: -40upx;
				border-radius: 8upx;;
			}
			.right{
				display: flex;
				flex-direction: column;
				padding-left: 24upx;
				font-size: $font-sm + 2upx;
				color: $font-color-base;
				line-height: 42upx;
				.price{
					font-size: $font-lg;
					color: $uni-color-primary;
					margin-bottom: 10upx;
				}
				.selected-text{
					margin-right: 10upx;
				}
			}
		}
		.attr-list{
			display: flex;
			flex-direction: column;
			font-size: $font-base + 2upx;
			color: $font-color-base;
			padding-top: 30upx;
			padding-left: 10upx;
		}
		.item-list{
			padding: 20upx 0 0;
			display: flex;
			flex-wrap: wrap;
			text{
				display: flex;
				align-items: center;
				justify-content: center;
				background: #eee;
				margin-right: 20upx;
				margin-bottom: 20upx;
				border-radius: 100upx;
				min-width: 60upx;
				height: 60upx;
				padding: 0 20upx;
				font-size: $font-base;
				color: $font-color-dark;
			}
			.selected{
				background: #fbebee;
				color: $uni-color-primary;
			}
		}
	}
	
	/*  弹出层 */
	.popup {
		position: fixed;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
		z-index: 99;
		
		&.show {
			display: block;
			.mask{
				animation: showPopup 0.2s linear both;
			}
			.layer {
				animation: showLayer 0.2s linear both;
			}
		}
		&.hide {
			.mask{
				animation: hidePopup 0.2s linear both;
			}
			.layer {
				animation: hideLayer 0.2s linear both;
			}
		}
		&.none {
			display: none;
		}
		.mask{
			position: fixed;
			top: 0;
			width: 100%;
			height: 100%;
			z-index: 1;
			background-color: rgba(0, 0, 0, 0.4);
		}
		.layer {
			position: fixed;
			z-index: 99;
			bottom: 0;
			width: 100%;
			min-height: 40vh;
			border-radius: 10upx 10upx 0 0;
			background-color: #fff;
			.btn{
				height: 66upx;
				line-height: 66upx;
				border-radius: 100upx;
				background: $uni-color-primary;
				font-size: $font-base + 2upx;
				color: #fff;
				margin: 30upx auto 20upx;
			}
		}
		@keyframes showPopup {
			0% {
				opacity: 0;
			}
			100% {
				opacity: 1;
			}
		}
		@keyframes hidePopup {
			0% {
				opacity: 1;
			}
			100% {
				opacity: 0;
			}
		}
		@keyframes showLayer {
			0% {
				transform: translateY(120%);
			}
			100% {
				transform: translateY(0%);
			}
		}
		@keyframes hideLayer {
			0% {
				transform: translateY(0);
			}
			100% {
				transform: translateY(120%);
			}
		}
	}
	
	/* 底部操作菜单 */
	.page-bottom{
		position:fixed;
		left: 75upx;
		bottom:30upx;
		z-index: 95;
		display: flex;
		justify-content: center;
		align-items: center;
		width: 600upx;
		height: 100upx;
		background: rgba(255,255,255,.9);
		box-shadow: 0 0 20upx 0 rgba(0,0,0,.5);
		border-radius: 16upx;
		
		.p-b-btn{
			display:flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
			font-size: $font-sm;
			color: $font-color-base;
			width: 96upx;
			height: 80upx;
			.yticon{
				font-size: 40upx;
				line-height: 48upx;
				color: $font-color-light;
			}
			&.active, &.active .yticon{
				color: $uni-color-primary;
			}
			.icon-fenxiang2{
				font-size: 42upx;
				transform: translateY(-2upx);
			}
			.icon-shoucang{
				font-size: 46upx;
			}
		}
		.action-btn-group{
			display: flex;
			height: 76upx;
			border-radius: 100px;
			overflow: hidden;
			box-shadow: 0 20upx 40upx -16upx #fa436a;
			box-shadow: 1px 2px 5px rgba(219, 63, 96, 0.4);
			background: linear-gradient(to right, #ffac30,#fa436a,#F56C6C);
			margin-left: 20upx;
			position:relative;
			&:after{
				content: '';
				position:absolute;
				top: 50%;
				right: 50%;
				transform: translateY(-50%);
				height: 28upx;
				width: 0;
				border-right: 1px solid rgba(255,255,255,.5);
			}
			.action-btn{
				display:flex;
				align-items: center;
				justify-content: center;
				width: 180upx;
				height: 100%;
				font-size: $font-base ;
				padding: 0;
				border-radius: 0;
				background: transparent;
			}
		}
	}
	
</style>
