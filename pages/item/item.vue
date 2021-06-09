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
			    <uni-list-item v-for="(item,index) in listItem" :key="index" :title="item.name"  @click="gggg(item)" :rightText="item.nodetime" link></uni-list-item>
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
						}
					}).then(res => {

						const list=res.result.data
						for(var i=0;i<list.length;i++){
							var date = new Date(list[i].nodetime);
							var year = date.getFullYear();
							var mon  = date.getMonth()+1;
							var day  = date.getDate();
							var hours = date.getHours();
							var minu = date.getMinutes();
							var sec = date.getSeconds();
							var trMon = mon<10 ? '0'+mon : mon
							var trDay = day<10 ? '0'+day : day
							list[i].nodetime=year+'-'+trMon+'-'+trDay
						}
						this.listItem=list
					})
				},
		methods: {
		gggg(gg){
			 uni.navigateTo({
								url:"/pages/item/list?id="+gg.id
							})
			}
		}
	}
</script>

<style>

</style>
