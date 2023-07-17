<template>
	<view class="u-p-25">

		<u-form :model="form" ref="uForm">
			<u-form-item label="商品名" label-width="110" prop="name">
				<u-input v-model="form.name" />
			</u-form-item>
			<u-form-item label="简介" prop="memo">
				<u-input type="textarea" v-model="form.memo" />
			</u-form-item>
			<u-form-item label="类别" >
				<u-input :disabled="true" @click="showSel=true" v-model="selName" type="select" />
			</u-form-item>
			<u-form-item label="价格" prop="price">
				<u-input v-model="form.price" type="text" />
			</u-form-item>
			<u-form-item label="库存" prop="num">
				<u-input v-model="form.num" type="text" />
			</u-form-item>

			<u-form-item :border-bottom="false" label-width="180" label-position="top" label="简介图" prop="pic1">
				</u-form-item>
				<view class="u-flex u-direction-row u-p-l-25 u-border-bottom u-p-b-25">
			          <u-image border-radius="15" @tap="ChooseImage('pic1')" class="u-image-shadow" height="180rpx"  
				width="180rpx" :src="form.pic1" mode="aspectFill">
				           <view slot="error" style="font-size: 24rpx;" class="u-text-center">
					             <u-icon size="30" name="plus"></u-icon><br />选择图片
					            </view>
				           <view slot="loading" style="font-size: 24rpx;" class="u-text-center">
					             <u-icon size="30" name="reload"></u-icon><br />加载中
					            </view>
				          </u-image>      
			     </view>
			
			<u-form-item :border-bottom="false" label-width="180" label-position="top" label="详情图1" prop="pic2">
				</u-form-item>
				<view class="u-flex u-direction-row u-p-l-25 u-border-bottom u-p-b-25">
			          <u-image border-radius="15" @tap="ChooseImage('pic2')" class="u-image-shadow" height="180rpx"  
				width="180rpx" :src="form.pic2" mode="aspectFill">
				           <view slot="error" style="font-size: 24rpx;" class="u-text-center">
					             <u-icon size="30" name="plus"></u-icon><br />选择图片
					            </view>
				           <view slot="loading" style="font-size: 24rpx;" class="u-text-center">
					             <u-icon size="30" name="reload"></u-icon><br />加载中
					            </view>
				          </u-image>      
			     </view>
			
			<u-form-item :border-bottom="false" label-width="180" label-position="top" label="详情图2" prop="pic3">
				</u-form-item>
				<view class="u-flex u-direction-row u-p-l-25 u-border-bottom u-p-b-25">
			          <u-image border-radius="15" @tap="ChooseImage('pic3')" class="u-image-shadow" height="180rpx"  
				width="180rpx" :src="form.pic3" mode="aspectFill">
				           <view slot="error" style="font-size: 24rpx;" class="u-text-center">
					             <u-icon size="30" name="plus"></u-icon><br />选择图片
					            </view>
				           <view slot="loading" style="font-size: 24rpx;" class="u-text-center">
					             <u-icon size="30" name="reload"></u-icon><br />加载中
					            </view>
				          </u-image>      
			     </view>
			

			<u-button type="primary" class="u-m-t-50" @click="submit"    :ripple="true">发布
				 </u-button>


			<!-- <u-form-item label="水果">
				<u-checkbox-group>
					<u-checkbox v-model="item.checked" v-for="(item, index) in checkboxList" :key="index" :name="item.name">
						{{ item.name }}
					</u-checkbox>
				</u-checkbox-group>
			</u-form-item>
			<u-form-item label="味道">
				<u-radio-group v-model="radio">
					<u-radio v-for="(item, index) in radioList" :key="index" :name="item.name" :disabled="item.disabled">
						{{ item.name }}
					</u-radio>
				</u-radio-group>
			</u-form-item>
			<u-form-item label="开关"><u-switch slot="right" v-model="switchVal"></u-switch></u-form-item> -->
		</u-form>
		<u-select v-model="showSel" :list="typeList" @confirm="confirm"></u-select>
	</view>
</template>

<script>
	import ygyUploadImg from '../../components/ygy-upload-img/ygy-upload-img.vue'
	import appRequest from "@/common/appRequestUrl.js";
	export default {
		components: {
			ygyUploadImg
		},
		data() {
			return {
				form: {
					name: "",
					num: "",
					memo: "",
					price: "",
					stat: 0,
					pic1: "",
					pic2: "",
					pic3: "",
					typeFK: "",
					type:2
				},user:"",
				selName: "",
				showSel: false,
				typeList: [],
				info: {
					iconCloseStyle: { //关闭图标样式
						fontSize: '60rpx',
						color: "#f40"
					}
				},rules: {
				name: [
					{
								required: true,
								message: '请填写商品名',
								trigger: ['change','blur']
					}
				],num: [{
								required: true,
								message: '请填写库存',
								trigger: ['change','blur']
					},
					{
								type: "number",
								message: '库存格式异常',
								trigger: ['change','blur']
					}
				],memo: [
					{
								required: true,
								message: '请填写详情',
								trigger: ['change','blur']
					}
				],price: [{
								required: true,
								message: '请填写价格',
								trigger: ['change','blur']
					},
					{
								type: "number",
								message: '价格格式异常',
								trigger: ['change','blur']
					}
				]
				}
			}
		},
		onLoad:function() {
			this.getGoodsType();
		},
		onShow() {
			this.user = appRequest.getUserInfo();
			if(!this.user){
				uni.switchTab({
					url:"../index/index"
				})
			}
		},
		methods: {
			ChooseImage(name) {
				let _this = this;
				uni.chooseImage({
					count: 1,
					sizeType: ["compressed"],
					success: function(res) {
						let src = res.tempFiles[0].path;
						uni.compressImage({
							src: src,
							width: 450,
							quality: 70,
							success: function(path) {
								let filePath = path.tempFilePath;
								wx.getFileSystemManager().readFile({
									filePath: filePath,
									encoding: 'base64',
									fail: function(errMsg) {
										console.log(errMsg);
									},
									success: function(res) {
										_this.form[name]=
											"data:image/jpeg;base64," + res.data;
									}
								});
							}
						});
					},
					fail: function(errMsg) {
						console.log(errMsg);
					}
				});
			},
			submit(){
				let _this = this;
				this.$refs.uForm.validate(valid => {
					if (valid) {
						_this.form.type = 2;
						appRequest.request({
							method: "POST",
							url: appRequest.saveGoods,
							data:{
								data:JSON.stringify(_this.form)
							},
							success: function(res) {
								if (res.data.code == 200) {
									uni.showModal({
										title:"成功",
										content:"食材信息提交成功",
										showCancel:false,
										success:function(res){
											if(res.confirm){
												uni.switchTab({
													url:"../user/user"
												})
											}
										}
									})({
										
									})
								} else {
									_this.$api.msg('保存失败');
								}
							},
							fail: function(res) {
								_this.$api.msg("请求异常");
							}
						})
						
						
					} else {
						_this.$api.msg('验证异常');
					}
				});
			},
			confirm(e) {
				this.selName = e[0].label;
				this.form.typeFK = e[0].value;
			},
			getGoodsType() {

				let _this = this;
				appRequest.request({
					method: "POST",
					url: appRequest.getGoodsType,
					success: function(res) {
						if (res.data.code == 200) {
							_this.typeList = [];
							res.data.data.map(function(obj, index, arr) {
								_this.typeList.push({
									label: obj.name,
									value: obj.id
								})
							});
						} else {
							_this.$api.msg('类型获取失败');
						}
						
						
						
					},
					fail: function(res) {
						_this.$api.msg("请求异常");
					}
				})
			},
			getImgList(arr, name) {
				this.form[name] = arr[0];
			}
		},onReady() {
		this.$refs.uForm.setRules(this.rules);
	}
	}
</script>

<style lang="scss">

</style>
