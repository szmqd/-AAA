<template>
	<div class="Total">
		<!-- 上方输入框按钮 -->
		<div class="baseTop">
			<el-input v-model="listQuery.id" clearable class="filter-item" style="width: 200px;"
				placeholder="请输入品牌商ID" />
			<el-input v-model="listQuery.name" clearable class="filter-item" style="width: 200px;"
				placeholder="请输入品牌商名称" />
			</el-input>
			<el-button v-permission="['GET /admin/brand/list']" class="filter-item" type="primary" icon="el-icon-search"
				@click="handleFilter">查找</el-button>
			<el-button v-permission="['POST /admin/brand/create']" class="filter-item" type="primary"
				icon="el-icon-edit" @click="handleCreate">添加</el-button>
			<el-button :loading="downloadLoading" class="filter-item" type="primary" icon="el-icon-download"
				@click="handleDownload">导出</el-button>
		</div>
		<!-- 列表 -->
		<div class="Table">
			<el-table :data="list" border style="width: 100%">
				<el-table-column fixed prop="id" label="品牌商ID" width="162" align="center">
				</el-table-column>
				<el-table-column prop="name" label="品牌商名称" width="162" align="center">
				</el-table-column>
				<el-table-column prop="picUrl" label="品牌商图片" width="162" align="center">
					<template slot-scope="scope">
						<el-image :src="scope.row.picUrl" :lazy="true" :preview-src-list="scope.row.picUrl | filterImg"
							style="width: 40px; height: 40px; margin-right: 5px;" />
					</template>
				</el-table-column>
				<el-table-column prop="desc" label="介绍" width="805" align="center">
				</el-table-column>
				<el-table-column prop="floorPrice" label="底价" width="160" align="center">
				</el-table-column>
				<el-table-column label="操作" width="200" align="center">
					<template slot-scope="scope">
						<el-button type="primary" @click="handleUpdate(scope.row)">编辑</el-button>
						<el-button type="danger" @click="handleDelete(scope.row)">删除</el-button>
					</template>
				</el-table-column>
			</el-table>
		</div>
		<!-- 翻页 -->
		<pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit"
			@pagination="getList" />
		<!-- 添加或修改对话框 -->
		<el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
			<el-form ref="dataForm" :rules="rules" :model="dataForm" status-icon label-position="left"
				label-width="100px" style="width: 400px; margin-left:50px;">
				<el-form-item label="品牌商名称" prop="name">
					<el-input v-model="dataForm.name" />
				</el-form-item>
				<el-form-item label="介绍" prop="desc">
					<el-input v-model="dataForm.desc" auto-complete="off" />
				</el-form-item>
				<el-form-item label="品牌商图片" prop="picUrl">
					<el-upload :action="uploadPath" :show-file-list="false" :on-success="uploadpicUrl"
						class="picUrl-uploader" accept=".jpg,.jpeg,.png,.gif">
						<img v-if="dataForm.picUrl" :src="dataForm.picUrl" class="picUrl">
						<i v-else class="el-icon-plus picUrl-uploader-icon" />
					</el-upload>
				</el-form-item>
				<el-form-item label="底价" prop="floorPrice">
					<el-input v-model="dataForm.floorPrice" />
				</el-form-item>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="dialogFormVisible = false">取消</el-button>
				<el-button v-if="dialogStatus=='create'" type="primary" @click="createData">确定</el-button>
				<el-button v-else type="primary" @click="updateData">确定</el-button>
			</div>
		</el-dialog>
	</div>
</template>
<script>
	import {
		listBrand,
		createBrand,
		readBrand,
		updateBrand,
		deleteBrand
	} from '@/api/brand'
	import Pagination from '@/components/Pagination' // Secondary package based on el-pagination
	import {
		uploadPath
	} from '@/api/storage'
	export default {
		name: 'brand',
		components: {
			Pagination
		},
		data() {
			return {
				uploadPath,
				input: '',
				list: [],
				total: 0,
				listLoading: true,
				tab: 'all',
				listQuery: {
					page: 1,
					limit: 20,
					id: undefined,
					name: undefined,
					sort: 'add_time',
					order: 'desc'
				},
				dataForm: {
					name: undefined,
					desc: undefined,
					floorPrice: undefined,
					picUrl: undefined,
					roleIds: []
				},
				dialogFormVisible: false,
				dialogStatus: '',
				textMap: {
					update: '编辑',
					create: '创建'
				},
				rules: {
					name: [{
						required: true,
						message: '品牌商名称不能为空',
						trigger: 'blur'
					}],
				},
				downloadLoading: false
			}
		},
		methods: {
			getList() {
				this.listLoading = true
				listBrand(this.listQuery)
					.then(response => {
						console.log(response)
						this.total = response.data.data.total
						this.list = response.data.data.list
						this.listLoading = false
					})
					.catch(() => {
						this.list = []
						this.total = 0
						this.listLoading = false
					})
			},
			handleFilter() {
				this.listQuery.page = 1
				this.getList()
			},
			uploadpicUrl: function(response) {
				this.dataForm.picUrl = response.data.url.replace('http://localhost:8080', this.$store.state.user.url)
			},
			handleUpdate(row) {
				if(row.picUrl==undefined){
					row.picUrl=' '
				}
				console.log(this.$store.state.user.url, 'aaaa', row.picUrl)
				row.picUrl = row.picUrl.replace("http://localhost:8080", this.$store.state.user.url)
				this.dataForm = Object.assign({}, row)
				this.dialogStatus = 'update'
				this.dialogFormVisible = true
				this.$nextTick(() => {
					this.$refs['dataForm'].clearValidate()
				})
			},
			updateData() {
				this.$refs['dataForm'].validate(valid => {
					if (valid) {
						updateBrand(this.dataForm)
							.then(() => {
								for (const v of this.list) {
									if (v.id === this.dataForm.id) {
										const index = this.list.indexOf(v)
										this.list.splice(index, 1, this.dataForm)
										break
									}
								}
								this.dialogFormVisible = false
								this.$notify.success({
									title: '成功',
									message: '更新成功'
								})
							})
							.catch(response => {
								this.$notify.error({
									title: '失败',
									message: response.data.errmsg
								})
							})
					}
				})
			},
			handleDelete(row) {
				deleteBrand(row)
					.then(response => {
						this.$notify.success({
							title: '成功',
							message: '删除成功'
						})
						this.getList()
					})
					.catch(response => {
						this.$notify.error({
							title: '失败',
							message: response.data.errmsg
						})
					})
			},
			resetForm() {
				this.dataForm = {
					name: undefined,
					desc: undefined,
					floorPrice: undefined,
					picUrl: undefined,
					roleIds: []
				}
			},
			handleCreate() {
				this.resetForm()
				this.dialogStatus = 'create'
				this.dialogFormVisible = true
				this.$nextTick(() => {
					this.$refs['dataForm'].clearValidate()
				})
			},
			createData() {
				this.$refs['dataForm'].validate(valid => {
					if (valid) {
						createBrand(this.dataForm)
							.then(response => {
								this.list.unshift(response.data.data)
								this.dialogFormVisible = false
								this.$notify.success({
									title: '成功',
									message: '添加成功'
								})
							})
							.catch(response => {
								this.$notify.error({
									title: '失败',
									message: response.data.errmsg
								})
							})
					}
				})
			},
			handleDownload() {
				this.downloadLoading = true
				import('@/vendor/Export2Excel').then(excel => {
					const tHeader = ['品牌商ID', '品牌商名称', '品牌商图片', '介绍', '底价']
					const filterVal = ['id', 'name', 'picUrl', 'desc', 'floorPrice']
					excel.export_json_to_excel2(
						tHeader,
						this.list,
						filterVal,
						'商品制造商信息'
					)
					this.downloadLoading = false
				})
			}
		},
		mounted() {
			this.getList()
		},
		filters: {
			filterImg(item) {
				if (typeof item === "string") {
					return item.split("|");
				} else {
					return item
				}
			}
		}
	}
</script>
<style lang="scss" scoped>
	.Total {
		padding: 20px !important;
	}

	.Table {
		margin-top: 20px;
		text-align: center;
	}

	.picUrl-uploader .el-icon-plus {
		border: 1px dashed #d9d9d9;
		border-radius: 6px;
		cursor: pointer;
		position: relative;
		overflow: hidden;
	}

	.picUrl-uploader .el-icon-plus:hover {
		border-color: #20a0ff;
	}

	.picUrl-uploader-icon {
		font-size: 28px;
		color: #8c939d;
		width: 120px;
		height: 120px;
		line-height: 120px;
		text-align: center;
	}
	.picUrl {
		width: 145px;
		height: 145px;
		display: block;
	}
</style>
