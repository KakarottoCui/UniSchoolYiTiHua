<template>
	<view class="u-p-20">
		<u-field
					v-model="empInfo"
					label="学生"
					placeholder="请选择学生"
					:disabled="true"
				>
			<u-button size="mini" slot="right" type="success" @click="showStu=true;">选择学生</u-button>
		</u-field>
		
		<view v-if="setEmp">
			
			<view  class="item-view u-p-15 u-m-15" v-for="(item,index) in scoreList">
				<u-field
					v-model="item.name"
					label="科目"
					placeholder="请填写科目"
				>
				</u-field>
				<u-field
					v-model="item.score"
					label="分数"
					placeholder="请填写分数"
				>
				</u-field>
				<view class="u-flex u-row-around u-p-20">
					<u-button size="mini" type="success" @click="editScore(item)">修改</u-button>
					<u-button size="mini" type="error" @click="item.validFlag = 0;editScore(item);">删除</u-button>
				</view>
				
			</view>		
			<view class="u-p-20">
				<u-button @click="showModel=true;" type="primary" style="width: 100%;">新增</u-button>
			</view>
			
		</view>		
				
		<u-select @confirm="selStu" value-name="userNo" label-name="selName" v-model="showStu" :list="userList"></u-select>		
	
		<u-popup v-model="showModel" mode="center" border-radius="14" width="600">
			<view class="c-model-view u-p-25">
				<u-field v-model="addScore.name" label="科目" placeholder="请填写科目"></u-field>
				<u-field v-model="addScore.score" label="分数" type="number" placeholder="请填写分数"></u-field>
				<u-button type="primary" class="u-m-t-50" @click="submitCheck"  :ripple="true">新增
				</u-button>
			</view>
		</u-popup>
	</view>	  
</template>

<script>
	import appRequest from "@/common/appRequestUrl.js"
	export default {
		data() {
			return {
				showModel:false,
				empInfo:"",
				showStu:false,
				setEmp:"",
				userList:[],
				scoreList:[],
				addScore:{
					score:"",
					name:""
				}
			};
		},
		onLoad(options) {
			this.getAllStu();
		},
		methods: {
			editScore:function(item){
				let _this = this;
				appRequest.request({
					url: appRequest.addScore,
					data:{
						data:JSON.stringify(item)
					},
					success: function(res) {
						uni.showToast({
							title:res.msg,
							icon:"none"
						});
						_this.showModel = false;
						_this.getScore();
						
					},
					fail: function(res) {
							
					}
				})
			},
			submitCheck:function(){
				this.editScore({
							uid:this.setEmp,
							score:this.addScore.score,
							name:this.addScore.name
						})
			},
			selStu:function(e){
				this.setEmp = e[0].value;
				this.empInfo = e[0].label;
				this.getScore();
			},
			getScore:function(){
				let _this = this;
				appRequest.request({
					url: appRequest.getScoreByUid,
					data:{
						uid:this.setEmp
					},
					success: function(res) {
						_this.scoreList = res.data.data;
					},
					fail: function(res) {
							
					}
				})
			},
			getAllStu: function() {
				let _this = this;
				appRequest.request({
					url: appRequest.getUserList,
					success: function(res) {
						_this.userList = res.data.data;
						_this.userList.map(function(item,index){
							item['selName']="学号:"+item.userNo+",姓名:"+item.name+",专业:"+item.project;
						});
					},
					fail: function(res) {
			
					}
				})
			},
			delArticle: function(item) {
				let _this = this;
				appRequest.request({
					method: "GET",
					data: {
						id: item.id
					},
					url: appRequest.delArticle,
					success: function(res) {
						_this.getAtricle();
					},
					fail: function(res) {
			
					}
				})
			},
			addArticle: function() {
				let _this = this;
				uni.navigateTo({
					url: "../common/addArticle?type=2&level=2"
				})
			},
			getDeptInfo: function() {
				let _this = this;
				appRequest.request({
					method: "GET",
					url: appRequest.getDeptData,
					success: function(res) {
						_this.deptList = [];
						res.data.data.map(function(item, index, arr) {
							if(_this.article.level == item.level){
								_this.deptList.push({
									value: item.id,
									label: item.name
								})
							}
						});
					},
					fail: function(res) {

					}
				})
			},
			ChooseImage() {
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
										_this.article.picBase64 =
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
			selConfirm(e) {
				this.article.deptFk = e[0].value;
				this.selValue = e[0].label;
			},
			submit() {
				let _this = this;
				if (_this.article.title == "" || _this.article.context == "") {
					uni.showToast({
						title: "请填写完整标题和正文",
						icon: "none"
					});
					return;
				}

				_this.article.pic = _this.article.picBase64 ? 1 : 0;
				
				if(_this.article.type < 3 && !_this.article.deptFk && !(_this.article.type==0&&_this.article.info>0)){
					uni.showToast({
						title: "部门通知需要选择部门",
						icon: "none"
					});
					return;
				}
				
				uni.showModal({
					title: "发布内容",
					content: "是否发布当前内容？",
					success: function(res) {
						if (res.confirm) {
							_this.addAtricle()
						}
					}
				})

			},
			addAtricle: function() {
				let _this = this;
				appRequest.request({
					method: "POST",
					data: {
						data: JSON.stringify(_this.article)
					},
					url: appRequest.addArticle,
					success: function(res) {
						if (res.data.code == 200) {
							uni.showToast({
								title: "发布成功",
								icon: "success",
								success: function() {
									uni.navigateBack({});
								}
							});
						} else {
							uni.showToast({
								title: res.data.msg,
								icon: "none"
							});
						}
					},
					fail: function(res) {

						uni.showToast({
							title: "发布异常",
							icon: "error"
						});

					}
				})
			}
		}
	}
</script>

<style lang="scss">
	.item-view {
		//border: 1px solid lightgray;
		border-radius: 15px;
		box-shadow: 3px 3px 20px lightgray;
	}
</style>
