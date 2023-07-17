<template>
	<view class="u-p-25">
		
		<view class="container">
			<!-- 空白页 -->
			<view v-if="!user" class="empty">
				<view v-else class="empty-tips" v-if="!user">
					未登录
					<view class="navigator" @click="navToLogin">去授权登录></view>
				</view>
			</view>
		</view>	
		<view v-if="user">
			<u-tabs :list="list" :is-scroll="false" :current="current" @change="change"></u-tabs>
			<view v-if="current == 0">
				<view class="u-flex u-row-between u-p-15 u-p-l-50 u-p-r-50">
					<view class="title content">种类</view>
					<view class="title content">重量(g)</view>
					<view class="title content">总能量(大卡)</view>
				</view>	
				<view v-for="(item,index) in foodsList" :key="index" class="u-flex u-row-between u-p-15 u-p-l-50 u-p-r-50 u-tips-color">
					<view class="content">{{item.name}}</view>
					<view class="content">{{item.num}}</view>
					<view class="content">{{item.energy}}</view>
				</view>
				
				<view class="u-tips-color u-p-t-30 u-p-l-60">总计{{total}}大卡，{{ total>2000 ?"超标":"正常" }}</view>
				
				<u-button class="u-m-t-30 u-m-l-60 u-m-r-60" type="primary" size="default" @click="show=true">添加</u-button>
			</view>
			
			<view v-if="current == 1">
				<view class="u-flex u-row-between u-p-15 u-p-l-50 u-p-r-50">
					<view class="title content">日期</view>
					<view class="title content">总能量</view>
					<view class="title content">情况</view>
				</view>	
				<view v-for="(item,index) in foodsList" :key="index" class="u-flex u-row-between u-p-15 u-p-l-50 u-p-r-50 u-tips-color">
					<view class="content">{{item.fdate}}</view>
					<view class="content">{{item.energy}}</view>
					<view class="content" :style=" { color: item.energy > 2000 ?'red':'green'  } ">{{item.energy > 2000 ? "超标":"正常"}}</view>
				</view>
				
				<view class="u-tips-color u-p-t-30 u-p-l-60">总计{{total}}大卡</view>
				
			</view>
		</view>
		
		
		<u-select label-name="name" v-model="showSel" :list="energy" @confirm="selSub"></u-select>
		
		<u-modal z-index="10" v-model="show" title="添加食物" @confirm="submit">
			<view class="slot-content">
				<view class="u-p-15">
					<view class="u-m-10">
						<u-input placeholder="选择食物" @click="showSel=true" :disabled="true" type="select" v-model="selFood.name" :border="true" />
					</view>
					<view class="u-m-10">
						<u-input placeholder="输入重量(g)" v-model="num" type="number" :border="true" />
					</view>
					<view v-if="selFood.name" class="u-m-10">
						能量：每克{{selFood.value}}大卡
					</view>
				</view>
				
			</view>
		</u-modal>
		
	</view>
</template>

<script>
	import appRequest from "@/common/appRequestUrl.js";
	export default {
		data() {
			return {
				showSel:false,
				selFood:{},
				num:"",
				show:false,
				list: [{
					name: '每日打卡'
				}, {
					name: '历史记录'
				}],
				foodsList:[],
				current: 0,
				energy: [{
						name: '米饭',
						value: 1.16
					},
					{
						name: '玉米',
						value: 1.12
					},
					{
						name: '面包',
						value: 2.54
					},
					{
						name: '馒头',
						value: 2.23
					},
					{
						name: '面条',
						value: 1.07
					},
					{
						name: '地瓜',
						value: 1.33
					},
					{
						name: '鸡蛋',
						value: 1.39
					},
					{
						name: '鸡肉',
						value: 1.18
					},
					{
						name: '猪肉',
						value: 1.43
					},
					{
						name: '虾',
						value: 0.93
					},
					{
						name: '鱼肉',
						value: 0.79
					},
					{
						name: '牛肉',
						value: 1.25
					},
					{
						name: '羊肉',
						value: 2.06
					},
					{
						name: '鸭肉',
						value: 2.82
					},
					{
						name: '牛奶',
						value: 0.66
					},
					{
						name: '苹果',
						value: 0.53
					},
					{
						name: '香蕉',
						value: 0.93
					},
					{
						name: '橘子',
						value: 0.44
					},
					{
						name: '青菜',
						value: 0.16
					},
					{
						name: '黄瓜',
						value: 0.16
					},
					{
						name: '西兰花',
						value: 0.27
					},
					{
						name: '土豆',
						value: 0.81
					},
					{
						name: '葡萄',
						value: 0.42
					},
					{
						name: '蘑菇',
						value: 0.26
					},
					{
						name: '萝卜',
						value: 0.16
					}
				],
				form: {

					typeFK: "",
					
				},
				total:0,
				user: "",
			}
		},
		onLoad: function() {
			//this.getGoodsType();
		},
		onShow() {
			this.user = appRequest.getUserInfo();	
			if (this.user) {
				this.getData();
			}
		},
		methods: {
			navToLogin(){
				uni.switchTab({
					url:"../index/index"
				})
			},
			selSub(res){
				this.selFood = res[0];
				this.selFood['name'] = this.selFood.label;
			},
			change(index) {
				this.current = index;
				this.getData();
			},
			dateFormat(fmt, date) {
			    let ret;
			    const opt = {
			        "Y+": date.getFullYear().toString(),        // 年
			        "m+": (date.getMonth() + 1).toString(),     // 月
			        "d+": date.getDate().toString(),            // 日
			        "H+": date.getHours().toString(),           // 时
			        "M+": date.getMinutes().toString(),         // 分
			        "S+": date.getSeconds().toString()          // 秒
			        // 有其他格式化字符需求可以继续添加，必须转化成字符串
			    };
			    for (let k in opt) {
			        ret = new RegExp("(" + k + ")").exec(fmt);
			        if (ret) {
			            fmt = fmt.replace(ret[1], (ret[1].length == 1) ? (opt[k]) : (opt[k].padStart(ret[1].length, "0")))
			        };
			    };
			    return fmt;
			},
			getData(type){
				let date = {};
				if(this.current == 0){
					date = {
						date:this.dateFormat("YYYY-mm-dd",new Date())
					}
				}
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.getFoods,
					data:date,
					success: function(res) {
						if (res.data.code == 200) {
							_this.foodsList = [];
							_this.total = 0;
							res.data.data.map(function(obj,index,arr){
								obj.fdate= obj.fdate.slice(0,10);
								_this.foodsList.push(obj);
								_this.total += obj.energy;
							});
							_this.total = _this.total.toFixed(2); 
							console.log(JSON.stringify(_this.foodsList));
						} else {
							_this.$api.msg('查询失败');
						}
					},
					fail: function(res) {
						_this.$api.msg("请求异常");
					}
				})
			},
			submit() {
				
				if(!this.selFood.name || !this.num){
					this.$api.msg('请填写完整');
				}
				
				let _this = this;
				let data = JSON.stringify({
							name:_this.selFood.name,
							energy:_this.selFood.value * _this.num,
							num:_this.num
						});
				this.selFood = {};
				this.num = "";
				appRequest.request({
					method: "POST",
					url: appRequest.saveFoods,
					data: {
						data: data
					},
					success: function(res) {
						if (res.data.code == 200) {
							_this.$api.msg('保存成功');
							_this.getData();
						} else {
							_this.$api.msg('保存失败');
						}
					},
					fail: function(res) {
						_this.$api.msg("请求异常");
					}
				})
				
			}
		},
		onReady() {
			//this.$refs.uForm.setRules(this.rules);
		}
	}
</script>

<style lang="scss">
	.title{
		color: #E64340;
		font-weight: bold;
	}
	.content{
		width: 220rpx;
	}
	.container{
		padding-bottom: 134upx;
		/* 空白页 */
		.empty{
			position:fixed;
			left: 0;
			top:0;
			width: 100%;
			height: 100vh;
			padding-bottom:100upx;
			display:flex;
			justify-content: center;
			flex-direction: column;
			align-items:center;
			background: #fff;
			image{
				width: 240upx;
				height: 160upx;
				margin-bottom:30upx;
			}
			.empty-tips{
				display:flex;
				font-size: $font-sm+2upx;
				color: $font-color-disabled;
				.navigator{
					color: $uni-color-primary;
					margin-left: 16upx;
				}
			}
		}
	}
	
</style>
