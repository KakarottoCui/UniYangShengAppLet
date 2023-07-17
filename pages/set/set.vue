<template>
	<view class="container">
		<view class="list-cell m-t b-b" @click="show=true" hover-class="cell-hover" :hover-stay-time="50">
			<text class="cell-tit">信息修改</text>
			<text class="cell-more yticon icon-you"></text>
		</view>
		<view class="list-cell m-t b-b" @click="clean()" hover-class="cell-hover" :hover-stay-time="50">
			<text class="cell-tit">清除缓存</text>
			<text class="cell-more yticon icon-you"></text>
		</view>
		<view class="list-cell b-b" @click="about()" hover-class="cell-hover" :hover-stay-time="50">
			<text class="cell-tit">关于应用</text>
			<text class="cell-more yticon icon-you"></text>
		</view>
		<view class="list-cell log-out-btn" @click="toLogout">
			<text class="cell-tit">退出登录</text>
		</view>
		
		<u-modal z-index="10" v-model="show" title="信息修改" @confirm="submit">
			<view class="slot-content">
				<view class="u-p-15">
					<view class="u-m-10">
						<u-input placeholder="输入昵称" v-model="user.nick" type="text" :border="true" />
					</view>
					<view class="u-m-10">
						<u-input placeholder="选择性别" @click="showSel=true" :disabled="true" type="select" v-model="selSex" :border="true" />
					</view>
					<view class="u-m-10">
						<u-input placeholder="输入年龄" v-model="user.age" type="number"  />
					</view>
				</view>
				
			</view>
		</u-modal>
		<u-select  v-model="showSel" :list="sex" @confirm="selSub"></u-select>
	</view>
</template>

<script>
	import appRequest from "@/common/appRequestUrl.js";
	import {  
	    mapMutations  
	} from 'vuex';
	export default {
		data() {
			return {
				user:"",
				show:false,
				showSel:false,
				sex:[{label:"男",value:1},{label:"女",value:0}],
				selObj:{},
				selSex:""
			};
		},onShow:function(){
			this.user = appRequest.getUserInfo();
			if(!this.user){
				uni.redirectTo({
					url:"../index/index"
				})
			}
			switch (this.user.sex){
				case 1:
				this.selSex = '男';
					break;
				case 0:
				this.selSex = '女';
					break;	
				default:
				this.selSex = '';
					break;
			}
			
		},
		methods:{
			submit(){
				
				if(!this.user.age||!this.user.nick||(this.user.sex!==0&&this.user.sex!==1)){
					this.$api.msg('请填写完整');
					return;
				}
				
				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.changeInfo,
					data:{
						data:JSON.stringify(_this.user)
					},
					success: function(res) {
						if (res.data.code == 200) {
							uni.showModal({
								title:"信息",
								content:"保存成功，将返回首页",
								showCancel:false,
								success:function(res){
									uni.clearStorage();
									uni.switchTab({
										url:"../index/index"
									})
								}
							})
							
						} else {
							_this.$api.msg('保存失败');
						}
					},
					fail: function(res) {
						_this.$api.msg("请求异常");
					}
				})
			},
			selSub(e){
				this.selObj = e[0];
				this.selSex = e[0].value == 0?'女':'男';
				this.user.sex = e[0].value;
			},
			about(){
				uni.showModal({
					title:"养生食谱",
					content:"养生食谱小程序，版本0.808",
					showCancel:false
				})
			},
			clean(){
				uni.clearStorage();
			},
			//...mapMutations(['logout']),
			navTo(url){
				this.$api.msg(`跳转到${url}`);
			},
			//退出登录
			toLogout(){
				uni.showModal({
				    content: '确定要退出登录么',
				    success: (e)=>{
				    	if(e.confirm){
							uni.clearStorage();
				    		uni.switchTab({
				    			url:"../index/index"
				    		})
				    	}
				    }
				});
			},
			//switch
			switchChange(e){
				let statusTip = e.detail.value ? '打开': '关闭';
				this.$api.msg(`${statusTip}消息推送`);
			},

		}
	}
</script>

<style lang='scss'>
	page{
		background: $page-color-base;
	}
	.list-cell{
		display:flex;
		align-items:baseline;
		padding: 20upx $page-row-spacing;
		line-height:60upx;
		position:relative;
		background: #fff;
		justify-content: center;
		&.log-out-btn{
			margin-top: 40upx;
			.cell-tit{
				color: $uni-color-primary;
				text-align: center;
				margin-right: 0;
			}
		}
		&.cell-hover{
			background:#fafafa;
		}
		&.b-b:after{
			left: 30upx;
		}
		&.m-t{
			margin-top: 16upx; 
		}
		.cell-more{
			align-self: baseline;
			font-size:$font-lg;
			color:$font-color-light;
			margin-left:10upx;
		}
		.cell-tit{
			flex: 1;
			font-size: $font-base + 2upx;
			color: $font-color-dark;
			margin-right:10upx;
		}
		.cell-tip{
			font-size: $font-base;
			color: $font-color-light;
		}
		switch{
			transform: translateX(16upx) scale(.84);
		}
	}
</style>
