<template>
	<view class="u-p-25">
		
		<view  @click="toRecipes(item.reFk)" v-for="(item,index) in collectList" :key="index" class="u-p-20 u-flex u-row-between u-border-bottom">
			<view>
				<u-icon name="star-fill" size="35" color="#F29100"></u-icon>
				<text class="u-m-l-15">{{item.title}}</text>
			</view>
			<view>
				<u-button @click.stop="del(item.id)" size="mini" type="primary">取消收藏</u-button>
			</view>
		</view>

	</view>
</template>

<script>
	import appRequest from "@/common/appRequestUrl.js";
	export default {
		data() {
			return {
				user: "",
				current:1,
				collectList:[]
			}
		},
		onLoad: function() {

		},
		onShow() {
			this.getData();
			this.user = appRequest.getUserInfo();
			// if (!this.user) {
			// 	uni.switchTab({
			// 		url: "../index/index"
			// 	})
			// }
		},
		methods: {
			getData(){
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.getCollect,
					success: function(res) {
						if (res.data.code == 200) {
							_this.collectList = res.data.data;
						} else {
							_this.$api.msg('查询失败');
						}
					},
					fail: function(res) {
						_this.$api.msg("请求异常");
					}
				})
			},
			toRecipes(id){
				uni.navigateTo({
					url:"../product/recidesInfo?id="+id
				})
			},
			del(id){

				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.saveCollect,
					data:{
						id:id,
						type:0
					},
					success: function(res) {
						if (res.data.code == 200) {
							_this.$api.msg('取消成功');
							_this.getData();
						} else {
							_this.$api.msg('取消失败');
						}
					},
					fail: function(res) {
						_this.$api.msg("请求异常");
					}
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
	
</style>
