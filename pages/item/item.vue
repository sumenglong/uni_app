<template>
	<view>
		<view class="uni-header">
			<view class="uni-group hide-on-phone">
				<view class="uni-title">项目列表</view>
				<view class="uni-sub-title"></view>
			</view>
			<view class="uni-group">
				<input class="uni-search" type="text" placeholder="请输入搜索内容" />
				<button class="uni-button" type="default" size="mini">搜索</button>
			</view>
		</view>
		<view class="uni-container">
			<uni-list>
			    <uni-list-item title="柳州科技馆柳州科技馆柳州科技馆" link to="/pages/item/list" rightText="2021年4月22日"></uni-list-item>
			    <uni-list-item title="遂宁展馆遂宁展馆遂宁展馆遂宁展馆" link to="/pages/item/list" rightText="2021年4月20日"></uni-list-item>
			    <uni-list-item title="咸阳市科技馆咸阳市科技馆咸阳市科技馆" link to="/pages/item/list" rightText="2021年4月19日"></uni-list-item>
			    <uni-list-item title="宝鸡市科技馆宝鸡市科技馆宝鸡市科技馆" link to="/pages/item/list" rightText="2021年4月18日"></uni-list-item>
			    <uni-list-item title="四川省科技馆四川省科技馆四川省科技馆四川省科技馆" link to="/pages/item/list" rightText="2021年4月17日"></uni-list-item>
			</uni-list>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				listItem:[]
			}
		},
		onLoad() {
					uni.getNetworkType()
					uniCloud.callFunction({
						name:"item",
						data:{
							name:"sml",
							type:"item_data"
						},
						success(res) {
							console.log(res)
							this.listItem=res
						},fail(err) {
						   console.log(err)
						}
					})
				},
		methods: {

		}
	}
</script>

<style>

</style>
