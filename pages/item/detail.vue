<template>
	<view>
		<view class="uni-header">
			<view class="uni-group hide-on-phone">
				<!-- <view class="uni-title">任务分配</view> -->
				<view class="uni-sub-title"></view>
			</view>
			<view class="uni-group">
				<button class="uni-button" type="warn" size="mini" @click="delTable"
					:disabled="!selectedIndexs.length">批量删除</button>
				<!-- #ifdef H5 -->
				<download-excel class="hide-on-phone" :fields="expExcel.json_fields" :data="expData"
					:type="expExcel.type" :name="expExcel.filename">
					<button class="uni-button" type="primary" size="mini">导出 Excel</button>
				</download-excel>
				<!-- #endif -->
			</view>
		</view>
		<view class="uni-container">
			<unicloud-db ref="udb" @load="onqueryload" collection="procedure" :options="options"
				:where="where" field="procedure_id,procedure_name,comment"
				page-data="replace" :orderby="orderby" :getcount="true" :page-size="options.pageSize"
				:page-current="options.pageCurrent" v-slot:default="{data,pagination,loading,error}">
				<uni-table :loading="loading" :emptyText="error.message || '没有更多数据'" border stripe type="selection"
					@selection-change="selectionChange">
					<uni-tr>
						<uni-th align="center">类别</uni-th>
						<uni-th align="center">任务名称</uni-th>
						<uni-th align="center">人员</uni-th>
						<uni-th align="center">数量</uni-th>
						<uni-th align="center">单位</uni-th>
						<uni-th align="center">难度</uni-th>
						<uni-th align="center">状态</uni-th>
						<uni-th width="204" align="center">操作</uni-th>
					</uni-tr>
					<uni-tr v-for="(item,index) in data" :key="index">
						<uni-td align="center">{{item.procedure_name}}</uni-td>
						<uni-th v-for="(iteme,index) in item.comment" :key="index" width="170" align="center">{{iteme}}</uni-th>
						<uni-td align="center">
							<view v-if="item.role_id === 'admin'">-</view>
							<view v-else class="uni-group">
								<button class="uni-button" type="primary" size="mini" @click="navigateTo('./add')">分配</button>
								<navigator url="./edit" open-type="navigate">
									<button v-show="item.comment[1].length>0" class="uni-button" size="mini" type="primary">修改</button>
								</navigator>
								<!-- <button @click="navigateTo('./edit?id='+item._id, false)" class="uni-button" size="mini" type="primary">修改</button> -->
								<button @click="confirmDelete(item.role_id)" v-show="item.comment[1].length>0" class="uni-button" size="mini"
									type="warn">删除</button>
							</view>
						</uni-td>
					</uni-tr>
				</uni-table>
				<view class="uni-pagination-box">
					<picker class="select-picker" mode="selector" :value="pageSizeIndex" :range="pageSizeOption"
						@change="changeSize">
						<button type="default" size="mini" :plain="true">
							<text>{{pageSizeOption[pageSizeIndex]}} 条/页</text>
							<uni-icons class="select-picker-icon" type="arrowdown" size="12" color="#999"></uni-icons>
						</button>
					</picker>
					<uni-pagination show-icon :page-size="pagination.size" v-model="pagination.current"
						:total="pagination.count" @change="onPageChanged" />
				</view>
			</unicloud-db>
		</view>
		<!-- #ifndef H5 -->
		<fix-window />
		<!-- #endif -->
	</view>
</template>

<script>
	const db = uniCloud.database()
	// 表查询配置
	const dbOrderBy = 'create_date desc' // 排序字段
	const dbSearchFields = ['role_id', 'role_name'] // 支持模糊搜索的字段列表
	// 分页配置
	const pageSize = 20
	const pageCurrent = 1

	export default {
		data() {
			return {
				selecValue: '科技馆',
				query: '',
				where: '',
				orderby: dbOrderBy,
				options: {
					pageSize,
					pageCurrent
				},
				mainid:"",
				selectedIndexs: [], //批量选中的项
				pageSizeIndex: 0,
				pageSizeOption: [10, 20, 50, 100, 500],
				expData: [],
				expExcel: {
					filename: "角色.xls",
					type: "xls",
					json_fields: {
						"角色Id": "role_id",
						"角色名称": "role_name",
						"备注": "comment",
						"创建时间": "create_date"
					}
				}
			}
		},
		watch: {
			pageSizeIndex: {
				immediate: true,
				handler(val, old) {
					this.options.pageSize = this.pageSizeOption[val]
					this.options.pageCurrent = 1
				}
			}
		},
		onLoad(e) {
			const id = e.id
			this.mainid=id
			const department=e.department
			this.where="procedure_department=='"+department+"'"
		},
		methods: {
		
			selectOne(options) {
				this.selecValue = options.label
			},
			useOutClickSide() {
				this.$refs.easySelect.hideOptions && this.$refs.easySelect.hideOptions()
			},

			onqueryload(data, ended) {
				for (var i = 0; i < data.length; i++) {
					let item = data[i]
						const dbs = uniCloud.database()
						dbs.collection('item_task,uni-id-users')
						  .where('task_item=="'+this.mainid+'" && task_pro=="'+item.procedure_id+'"')
						  .field('task_pro,unit,number,task_name,difficulty,state,users_id{username}')
						  .get()
						  .then(res => {
							   var ztitle=["","","","","",""]
							  if(res.result.data.length>0){
								  const items=res.result.data[0];
								  ztitle=[items.task_name,items.users_id[0].username,items.number,items.unit,items.difficulty,items.state]
							  }
								item.comment=ztitle
						}).catch(err => {
						  console.error(err)
						})



					item.create_date = this.$formatDate(item.create_date)
				}
				this.expData = data //仅导出当前页
			},
			changeSize(e) {
				this.pageSizeIndex = e.detail.value
			},
			getWhere() {
				const query = this.query.trim()
				if (!query) {
					return ''
				}
				const queryRe = new RegExp(query, 'i')
				return dbSearchFields.map(name => queryRe + '.test(' + name + ')').join(' || ')
			},
			search() {
				const newWhere = this.getWhere()
				const isSameWhere = newWhere === this.where
				this.where = newWhere
				if (isSameWhere) { // 相同条件时，手动强制刷新
					this.loadData()
				}
			},
			loadData(clear = true) {
				this.$refs.udb.loadData({
					clear
				})
			},
			onPageChanged(e) {
				this.options.pageCurrent = e.current
			},
			navigateTo(url, clear) { // clear 表示刷新列表时是否清除当前页码，true 表示刷新并回到列表第 1 页，默认为 true
				uni.navigateTo({
					url,
					events: {
						refreshData: () => {
							this.loadData(clear)
						}
					}
				})
			},
			// 多选处理
			selectedItems() {
				var dataList = this.$refs.udb.dataList
				return this.selectedIndexs.map(i => dataList[i].role_id)
			},
			//批量删除
			// delTable() {
			// 	uni.showModal({
			// 		title: '提示',
			// 		content: '确认删除多条记录？',
			// 		success: (res) => {
			// 			res.confirm && this.delete(this.selectedItems())
			// 		}
			// 	})
			// },
			// 多选
			selectionChange(e) {
				this.selectedIndexs = e.detail.index
			},
			confirmDelete(id) {
				uni.showModal({
					title: '提示',
					content: '确认删除该记录？',
					success: (res) => {
						res.confirm && this.delete(id)
					}
				})
			},
			async delete(id) {
				uni.showLoading({
					mask: true
				})
				await this.$request('system/role/remove', {
						id
					})
					.then(res => {
						uni.showToast({
							title: '删除成功'
						})
					}).catch(err => {
						uni.showModal({
							content: err.message || '请求服务失败',
							showCancel: false
						})
					}).finally(err => {
						uni.hideLoading()
					})
				this.loadData(false)
			},
			praseRoleArr(permission) {
				return permission ? permission.map(pItem => pItem.permission_name).join('、') : '-'
			}
		}
	}
</script>
<style>
	/* #ifndef H5 */
	page {
		padding-top: 85px;
	}

	/* #endif */
	.uni-button2 {
		background-color: #409EFF;
		border-color: #409EFF;
		border-width: 0;
		color: #fff;
		border-radius: 3px;
		padding: 2px 8px;
	}
	.uni-header {
		padding-top: 10px;
	}
</style>
