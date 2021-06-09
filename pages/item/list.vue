<template>
	<view>
		<view class="uni-header">
			<view class="uni-group hide-on-phone">
				<!-- <view class="uni-title">项目清单</view> -->
				<view class="uni-sub-title"></view>
			</view>
			<view class="uni-group">
				<view class="contentSelect" @click="useOutClickSide">
					<easy-select ref="easySelect" style="margin-right: 20px;" size="mini" :value="selecValue" @selectOne="selectOne"></easy-select>
				</view>
				<input class="uni-search" type="text" v-model="query" @confirm="search" placeholder="请输入搜索内容" />
				<button class="uni-button" type="default" size="mini" @click="search">搜索</button>
				<!-- <button class="uni-button" type="primary" size="mini" @click="navigateTo('./add')">新增</button> -->
				<!-- <button class="uni-button" type="warn" size="mini" @click="delTable"
					:disabled="!selectedIndexs.length">批量删除</button> -->
				<!-- #ifdef H5 -->
				<download-excel class="hide-on-phone" :fields="expExcel.json_fields" :data="expData"
					:type="expExcel.type" :name="expExcel.filename">
					<button class="uni-button" type="primary" size="mini">导出 Excel</button>
				</download-excel>
				<!-- #endif -->
			</view>
		</view>
		<view class="uni-container">
			<unicloud-db ref="udb" @load="onqueryload" collection="item_main" :options="options"
				:where="where" field="id,name,state,explain,department,remarks,explain,nodetime,ctime"
				page-data="replace" :orderby="orderby" :getcount="true" :page-size="options.pageSize"
				:page-current="options.pageCurrent" v-slot:default="{data,pagination,loading,error}">
				<uni-table :loading="loading" :emptyText="error.message || '没有更多数据'" border stripe type="selection"
					@selection-change="selectionChange">
					<uni-tr>
						<uni-th align="center"></uni-th>
						<uni-th align="center"></uni-th>

						<uni-th v-for="(itemd,index) in department" :key="index"  width="170" align="center" colspan="4">{{itemd.procedure_name}}</uni-th>
						<uni-th align="center"></uni-th>
						<uni-th align="center"></uni-th>
						<uni-th align="center"></uni-th>
						<uni-th width="204" align="center">操作</uni-th>
					</uni-tr>
					<uni-tr>
						<uni-th align="center">项目编号</uni-th>
						<uni-th align="center">名称</uni-th>
						<uni-th v-for="(itemt,index) in listtitlename" :key="index" width="170" align="center">{{itemt}}</uni-th>
						<uni-th align="center">节点时间</uni-th>
						<uni-th align="center">是否验收</uni-th>
						<uni-th align="center">创建时间</uni-th>
						<uni-th width="204" align="center">操作</uni-th>
					</uni-tr>
					<uni-tr v-for="(item,index) in data" :key="index">
						<uni-td align="center">{{item.id}}</uni-td>
						<uni-td align="center">{{item.name}}</uni-td>
						<uni-th v-for="(iteme,index) in item.explain" :key="index" width="170" align="center">{{iteme}}</uni-th>
						<uni-td align="center">{{item.nodetime}}</uni-td>
						<uni-td align="center">{{item.state}}</uni-td>
						<uni-td align="center">{{item.ctime}}</uni-td>


						<uni-td align="center">
							<view v-if="item.role_id === 'admin'">-</view>
							<view v-else class="uni-group">
								<navigator @click="detail(item.id,item.department)" size="mini" type="primary" class="uni-button2">分配任务</navigator>
								<!-- <button @click="navigateTo('./edit?id='+item._id, false)" class="uni-button" size="mini" type="primary">分配任务</button> -->
								<!-- <button @click="confirmDelete(item.role_id)" class="uni-button" size="mini"
									type="warn">删除</button> -->
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
				department:[],
				listtitlename:[],
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
			this.where="parent=='"+id+"'"
			this.getDetail(id)
		},
		methods: {
			detail(itmeid,department){
				uni.navigateTo({
						url:"./detail?id="+itmeid+"&department="+department
						})
			},
			getDetail(id) {
				const dbs = uniCloud.database()
				dbs.collection('item_main')
				  .where('id=="001001"')
				  .field('name,department')
				  .get()
				  .then(res => {

					 this.wer(res.result.data[0].department)
				  }).catch(err => {
				    console.error(err)
				  })

			},
			wer(z){
				const dbs = uniCloud.database()
				dbs.collection('procedure')
				  .where('procedure_department == "'+z+'"')
				  .field('procedure_id,procedure_name')
				  .orderBy('procedure_id')
				  .get()
				  .then(res => {
				    this.department=res.result.data;
					const list=["名称","数量","难度","人员"]
					for(var i=0;i<res.result.data.length;i++){
						this.listtitlename=this.listtitlename.concat(list)
					}

				  }).catch(err => {
				    console.error(err)
				  })
			},
			selectOne(options) {
				this.selecValue = options.label
			},
			useOutClickSide() {
				this.$refs.easySelect.hideOptions && this.$refs.easySelect.hideOptions()
			},

			onqueryload(data, ended) {
				for (var i = 0; i < data.length; i++) {
					let item = data[i]
					//console.log(JSON.stringify(item.id))
					var ztitles=[]
						const dbs = uniCloud.database()
						dbs.collection('item_task,uni-id-users')
						  .where('task_item=="'+item.id+'"')
						  .field('task_pro,unit,number,task_name,difficulty,users_id{username}')
						  .get()
						  .then(res => {
							  ztitles=[]
							for(var i = 0; i <this.department.length; i++){
								const xx=0

								for(var j = 0; j <res.result.data.length; j++){
									var dp=JSON.stringify(this.department[i].procedure_id)
									var ut=JSON.stringify(res.result.data[j].task_pro)
								if(dp==ut){
									xx=1
									const item=res.result.data[j];
									const ztitle=[item.task_name,item.number+item.unit,item.difficulty,item.users_id[0].username]
									ztitles=ztitles.concat(ztitle)
								}
								}
								if(xx==0){
									const ztitle=["","","",""]
									ztitles=ztitles.concat(ztitle)
								}

							}
								item.explain=ztitles
						  }).catch(err => {
							console.error(err)
						  })
					 console.log("tasks:"+JSON.stringify(item.explain))
					//item.permission = item.permission.map(pItem => pItem.permission_name).join('、')
					item.nodetime = this.$formatDate(item.nodetime)
					item.ctime = this.$formatDate(item.ctime)
				}
				this.expData = data //仅导出当前页
			},
			gettask(id) {


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
