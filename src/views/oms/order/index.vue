<template> 
	<div class="app-container">
		<el-card class="filter-container" shadow="never">
			<div style="overflow: hidden;">
				<i class="el-icon-search"></i>
				<span>筛选搜索</span>
				<el-button style="float:right" type="primary" @click="handleSearchList()" size="small">
					查询搜索
				</el-button>
				<el-button style="float:right;margin-right: 15px" @click="handleResetSearch()" size="small">
					重置
				</el-button>
			</div>
			<div style="margin-top: 15px">
				<el-form :inline="true" :model="listQuery" size="small" label-width="140px">
					<el-form-item label="输入搜索：">
						<el-input v-model="listQuery.orderSn" class="input-width" placeholder="订单编号" clearable></el-input>
					</el-form-item>
					<el-form-item label="订单创建人：">
						<el-input v-model="listQuery.nickName" class="input-width" placeholder="创建人昵称" clearable></el-input>
					</el-form-item>
					<el-form-item label="收货人姓名：">
						<el-input v-model="listQuery.receiverName" class="input-width" placeholder="收货人姓名" clearable></el-input>
					</el-form-item>
					<el-form-item label="收货人手机号码：">
						<el-input v-model="listQuery.receiverPhone" class="input-width" placeholder="收货人手机号码" clearable></el-input>
					</el-form-item>
					<el-form-item label="创建时间：">
						<el-date-picker clearable @change="getCreateTime" v-model="createTime" value-format="yyyy-MM-dd" type="daterange"
						 range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期">
						</el-date-picker>
					</el-form-item>
					<el-form-item label="订单状态：">
						<el-select v-model="listQuery.status" class="input-width" placeholder="全部" clearable>
							<el-option v-for="item in statusOptions" :key="item.value" :label="item.label" :value="item.value">
							</el-option>
						</el-select>
					</el-form-item>
					</el-form-item>
				</el-form>
			</div>
		</el-card>
		<el-card class="operate-container" shadow="never">
			<i class="el-icon-tickets"></i>
			<span>数据列表</span>
		</el-card>
		<div class="table-container">
			<el-table ref="orderTable" :data="list" style="width: 100%;" @selection-change="handleSelectionChange" v-loading="listLoading"
			 border>
				<el-table-column type="selection" width="60" align="center"></el-table-column>
				<el-table-column label="编号" width="80" align="center">
					<template slot-scope="scope">{{scope.row.id}}</template>
				</el-table-column>
				<el-table-column label="订单编号" width="180" align="center">
					<template slot-scope="scope">{{scope.row.orderSn}}</template>
				</el-table-column>
				<el-table-column label="提交时间" width="160" align="center">
					<template slot-scope="scope">{{scope.row.createTime | formatCreateTime}}</template>
				</el-table-column>
				<el-table-column label="创建人ID" width="80" align="center">
					<template slot-scope="scope">{{scope.row.memberId}}</template>
				</el-table-column>
				<el-table-column label="创建人" align="center">
					<template slot-scope="scope">{{scope.row.memberUsername}}</template>
				</el-table-column>
				<el-table-column label="收货人" align="center">
					<template slot-scope="scope">{{scope.row.receiverName}}</template>
				</el-table-column>
				<el-table-column label="收货人电话" width="110" align="center">
					<template slot-scope="scope">{{scope.row.receiverPhone}}</template>
				</el-table-column>
				<el-table-column label="总金额" width="120" align="center">
					<template slot-scope="scope">￥{{scope.row.totalAmount}}</template>
				</el-table-column>
				<el-table-column label="应付金额" width="120" align="center">
					<template slot-scope="scope">￥{{scope.row.payAmount}}</template>
				</el-table-column>
				<el-table-column label="支付方式" width="100" align="center">
					<template slot-scope="scope">{{scope.row.payType | formatPayType}}</template>
				</el-table-column>
				<el-table-column label="订单状态" width="120" align="center">
					<template slot-scope="scope">{{scope.row.status | formatStatus}}</template>
				</el-table-column>
				<el-table-column label="操作" width="200" align="center">
					<template slot-scope="scope">
						<el-button size="mini" type="primary" @click="handleViewOrder(scope.$index, scope.row)">查看订单</el-button>
						<el-button size="mini" type="warning" @click="handleCloseOrder(scope.$index, scope.row)" v-show="scope.row.status === 0">关闭订单</el-button>
						<el-button size="mini" type="success" @click="handleDeliveryOrder(scope.$index, scope.row)" v-show="scope.row.status === 1">订单发货</el-button>
						<!-- <el-button size="mini" type="success" @click="handleViewLogistics(scope.$index, scope.row)" v-show="scope.row.status === 2||scope.row.status === 3">订单跟踪</el-button> -->
						<el-button size="mini" type="danger" @click="handleDeleteOrder(scope.$index, scope.row)" v-show="scope.row.status === 4">删除订单</el-button>
					</template>
				</el-table-column>
			</el-table>
		</div>
		<div class="batch-operate-container">
			<el-select size="small" v-model="operateType" placeholder="批量操作">
				<el-option v-for="item in operateOptions" :key="item.value" :label="item.label" :value="item.value">
				</el-option>
			</el-select>
			<el-button style="margin-left: 20px" class="search-button" @click="handleBatchOperate()" type="primary" size="small">
				确定
			</el-button>
		</div>
		<div class="pagination-container">
			<el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange" layout="total, sizes,prev, pager, next,jumper"
			 :current-page.sync="listQuery.pageIndex" :page-size="listQuery.pageSize" :page-sizes="[10, 20, 50, 100]" :total="total">
			</el-pagination>
		</div>
		<el-dialog title="关闭订单" :visible.sync="closeOrder.dialogVisible" width="30%">
			<span style="vertical-align: top">操作备注：</span>
			<el-input style="width: 80%" type="textarea" :rows="5" placeholder="请输入内容" v-model="closeOrder.content">
			</el-input>
			<span slot="footer" class="dialog-footer">
				<el-button @click="closeOrder.dialogVisible = false">取 消</el-button>
				<el-button type="primary" @click="handleCloseOrderConfirm">确 定</el-button>
			</span>
		</el-dialog>
		<logistics-dialog v-model="logisticsDialogVisible"></logistics-dialog>
	</div>
</template>
<script>
	import {
		pageOrder,
		closeOrder,
		deleteOrder
	} from '@/api/order'
	import {
		formatDate
	} from '@/utils/date';
	import LogisticsDialog from '@/views/oms/order/components/logisticsDialog';
	const defaultListQuery = {
		pageIndex: 1,
		pageSize: 10,
		orderSn: null,
		nickName: null,
		receiverName: null,
		receiverPhone: null,
		status: null,
		startTime: null,
		endTime: null
	};
	export default {
		name: "orderList",
		components: {
			LogisticsDialog
		},
		data() {
			return {
				listQuery: Object.assign({}, defaultListQuery),
				listLoading: true,
				list: null,
				total: null,
				operateType: null,
				multipleSelection: [],
				closeOrder: {
					dialogVisible: false,
					content: null,
					orderIds: []
				},
				statusOptions: [{
						label: '待付款',
						value: 0
					},
					{
						label: '待发货',
						value: 1
					},
					{
						label: '已发货',
						value: 2
					},
					{
						label: '已完成',
						value: 3
					},
					{
						label: '已关闭',
						value: 4
					}
				],
				operateOptions: [{
						label: "批量发货",
						value: 1
					},
					{
						label: "关闭订单",
						value: 2
					},
					{
						label: "删除订单",
						value: 3
					}
				],
				logisticsDialogVisible: false,
				createTime: ''
			}
		},
		created() {
			this.getList();
		},
		filters: {
			formatCreateTime(time) {
				let date = new Date(time);
				return formatDate(date, 'yyyy-MM-dd hh:mm:ss')
			},
			formatPayType(value) {
				if (value === 1) {
					return '会员卡支付';
				} else if (value === 2) {
					return '微信支付';
				}
			},
			formatStatus(value) {
				if (value === 1) {
					return '待发货';
				} else if (value === 2) {
					return '已发货';
				} else if (value === 3) {
					return '已完成';
				} else if (value === 4) {
					return '已关闭';
				} else if (value === 5) {
					return '无效订单';
				} else {
					return '待付款';
				}
			},
		},
		methods: {
			getCreateTime(val) {
				if (val) {
					this.listQuery.startTime = val[0]
					this.listQuery.endTime = val[1]
				} else {
					this.listQuery.startTime = ''
					this.listQuery.endTime = ''
				}
			},
			handleResetSearch() {
				this.listQuery = Object.assign({}, defaultListQuery);
			},
			handleSearchList() {
				this.listQuery.pageIndex = 1;
				this.getList();
			},
			handleSelectionChange(val) {
				this.multipleSelection = val;
			},
			handleViewOrder(index, row) {
				this.$router.push({
					path: '/oms/orderDetail',
					query: {
						id: row.id
					}
				})
			},
			handleCloseOrder(index, row) {
				this.closeOrder.dialogVisible = true;
				this.closeOrder.orderIds = [row.id];
			},
			handleDeliveryOrder(index, row) {
				let listItem = this.covertOrder(row);
				this.$router.push({
					path: '/oms/deliverOrderList',
					query: {
						list: [listItem]
					}
				})
			},
			handleViewLogistics(index, row) {
				this.logisticsDialogVisible = true;
			},
			handleDeleteOrder(index, row) {
				let ids = [];
				ids.push(row.id);
				this.deleteOrder(ids);
			},
			handleBatchOperate() {
				if (this.multipleSelection == null || this.multipleSelection.length < 1) {
					this.$message({
						message: '请选择要操作的订单',
						type: 'warning',
						duration: 1000
					});
					return;
				}
				if (this.operateType === 1) {
					//批量发货
					let list = [];
					for (let i = 0; i < this.multipleSelection.length; i++) {
						if (this.multipleSelection[i].status === 1) {
							list.push(this.covertOrder(this.multipleSelection[i]));
						}
					}
					if (list.length === 0) {
						this.$message({
							message: '选中订单中没有可以发货的订单',
							type: 'warning',
							duration: 1000
						});
						return;
					}
					this.$router.push({
						path: '/oms/deliverOrderList',
						query: {
							list: list
						}
					})
				} else if (this.operateType === 2) {
					//关闭订单
					this.closeOrder.orderIds = [];
					for (let i = 0; i < this.multipleSelection.length; i++) {
						this.closeOrder.orderIds.push(this.multipleSelection[i].id);
					}
					this.closeOrder.dialogVisible = true;
				} else if (this.operateType === 3) {
					//删除订单
					let ids = [];
					for (let i = 0; i < this.multipleSelection.length; i++) {
						ids.push(this.multipleSelection[i].id);
					}
					this.deleteOrder(ids);
				}
			},
			handleSizeChange(val) {
				this.listQuery.pageIndex = 1;
				this.listQuery.pageSize = val;
				this.getList();
			},
			handleCurrentChange(val) {
				this.listQuery.pageIndex = val;
				this.getList();
			},
			handleCloseOrderConfirm() {
				if (this.closeOrder.content == null || this.closeOrder.content === '') {
					this.$message({
						message: '操作备注不能为空',
						type: 'warning',
						duration: 1000
					});
					return;
				}
				let params = new URLSearchParams();
				params.append('ids', this.closeOrder.orderIds);
				params.append('note', this.closeOrder.content);
				closeOrder(params).then(response => {
					this.closeOrder.orderIds = [];
					this.closeOrder.dialogVisible = false;
					this.getList();
					this.$message({
						message: '修改成功',
						type: 'success',
						duration: 1000
					});
				});
			},
			getList() {
				this.listLoading = true;
				let data = {
					pageIndex: this.listQuery.pageIndex,
					pageSize: this.listQuery.pageSize
				}
				if(this.listQuery.orderSn){
					data.orderSn = this.listQuery.orderSn
				}
				if(this.listQuery.nickName){
					data.nickName = this.listQuery.nickName
				}
				if(this.listQuery.receiverName){
					data.receiverName = this.listQuery.receiverName
				}
				if(this.listQuery.receiverPhone){
					data.receiverPhone = this.listQuery.receiverPhone
				}
				if(this.listQuery.startTime && this.listQuery.endTime){
					data.startTime = this.listQuery.startTime
					data.endTime = this.listQuery.endTime
				}
				if(this.listQuery.status != null){
					data.status = this.listQuery.status
				}
				pageOrder(data).then(response => {
					this.listLoading = false;
					this.list = response.data.records;
					this.total = response.data.total;
				});
			},
			deleteOrder(ids) {
				this.$confirm('是否要进行该删除操作?', '提示', {
					confirmButtonText: '确定',
					cancelButtonText: '取消',
					type: 'warning'
				}).then(() => {
					let params = new URLSearchParams();
					params.append("ids", ids);
					deleteOrder(params).then(response => {
						this.$message({
							message: '删除成功！',
							type: 'success',
							duration: 1000
						});
						this.getList();
					});
				})
			},
			covertOrder(order) {
				let address =  order.receiverRegion + " " + order.receiverDetailAddress;
				let listItem = {
					orderId: order.id,
					orderSn: order.orderSn,
					receiverName: order.receiverName,
					receiverPhone: order.receiverPhone,
					receiverPostCode: order.receiverPostCode,
					receiverProvince: order.receiverProvince,
					receiverCity: order.receiverCity,
					receiverRegion: order.receiverRegion,
					receiverDetailAddress: order.receiverDetailAddress,
					address: address,
					deliveryCompany: null,
					deliverySn: null
				};
				return listItem;
			}
		}
	}
</script>
<style scoped>
	.input-width {
		width: 203px;
	}
</style>
