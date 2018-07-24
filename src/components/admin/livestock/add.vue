<template>
	<div class="livestock-add">
		<div class="selectCrowd">
			<el-button type="primary" size="small">导入耳牌号文件</el-button>
			<p>给羊只分配栏/栋</p>
			<div class="select-group">
				<el-input size="small" type="text" class="input" v-model="selectD">
					<template slot="prepend">栋号:</template>
				</el-input>
				<el-input size="small" type="text" class="input" v-model="selectL">
					<template slot="prepend">栏号:</template>
				</el-input>
			</div>
			<div class="single">
				<span>单个分配:</span>
				<el-input size="small" type="text" class="input" v-model="tradeMarkEartag">
					<template slot="prepend">商标耳牌号:</template>
				</el-input>
				<el-input size="small" type="text" class="input" v-model="immuneEartag" >
					<template slot="prepend">免疫耳牌号:</template>
				</el-input>
				<el-button type="small">添加</el-button>
			</div>
			<div class="multiple">
				<span>批量分配:</span>
				<el-popover placement="bottom" width="200" trigger="hover" popper-class="multiple-select">
					<el-checkbox-group v-model="checkList">
						<el-checkbox v-for="(t, index) in crowdTag" :label="t" :key="index">{{t}}</el-checkbox>
					</el-checkbox-group>
					<el-input size="small" slot="reference" style="width: 152px" v-model="selectTag" placeholder="请选择" ><template slot="prepend">商标耳牌号:</template></el-input>
				</el-popover>
				<el-button type="small">添加</el-button>
			</div>
		</div>
		<el-table :data="tableData" :span-method="objectSpanMethod" :border="true">
			<el-table-column
				label="栋号"
				width="120"
				prop="d"
			>
			</el-table-column>
			<el-table-column
				label="栏号"
				width="120"
				prop="l"
			>
			</el-table-column>
			<el-table-column
				label="本栏羊数"
				width="120"
				prop="lnum"
			>
			</el-table-column>
			<el-table-column
				label="本栋羊数"
				width="120"
			>
				<template slot-scope="scope">
					<p style="text-align: center;">{{dnum[scope.row.d]}}</p>
				</template>
			</el-table-column>
		</el-table>
	</div>
</template>
<script>
export default {
	data() {
		return {
			tradeMarkEartag: '',
			immuneEartag: '',
			checkList: [],
			crowdTag:['G123456','G123415','G1232465','G1233456','G142345','G1523465'],
			dnum: [],
			merge: {  // 合并单元格需要的数据
				len: new Map(),
				flag: 0,
			},
			column: 0,
			columnNum: 0,
			allNum: 100,
			selectD: 1,
			selectL: 1, //单个分配
			tableData: [{
				d: 1,
				l: 1,
				lnum: 1,
			},{
				d: 1,
				l: 2,
				lnum: 2,
			},{
				d: 1,
				l: 3,
				lnum: 3,
			},{
				d: 2,
				l: 1,
				lnum: 4,
			},{
				d: 2,
				l: 2,
				lnum: 2,
			},{
				d: 2,
				l: 3,
				lnum: 3,
			},{
				d: 2,
				l: 4,
				lnum: 5,
			},{
				d: 3,
				l: 1,
				lnum: 5,
			},{
				d: 3,
				l: 2,
				lnum: 5,
			}],
		}
	},

	methods: {
		objectSpanMethod({ row, column, rowIndex, columnIndex }) {
			let d = row.d;
			let len = this.merge.len.get(d)
			let flag = 0 
			for (let index of this.merge.len.keys()) {
				if ( index <= d - 1 ) {
					flag += this.merge.len.get(index)
				}
			}
			if (columnIndex === 3) {
				if (rowIndex === flag) {
					return {
						rowspan: len,
						colspan: 1
					};
				}		
				else {
					return {
						rowspan: 0,
						colspan: 0
					};
				}
			}
		},
	},

	watch: {
		tableData: {
			handler(o, n) {
				let count = []
				n.forEach(ele => {
					if (ele.d) {
						if(count[ele.d] === undefined) {
							count[ele.d] = 0
						}	
						count[ele.d] += parseInt(ele.lnum)	
					} 		
				});
				this.dnum = count
			},
			deep: true		
		},
	},

	computed: {
		selectTag() {
			let res = ''
			this.checkList.forEach( ele => {
				res += ele + ';'
			})
			return res 
		}
	},

	mounted() {
		let count = []
        let sum = []		
		this.tableData.forEach(ele => {
			if (ele.d) {
				if(sum[ele.d] === undefined) {
					sum[ele.d] = 0
					count[ele.d] = 0
				}		
				 sum[ele.d] += parseInt(ele.lnum)	
				 count[ele.d] ++ 	
			} 
		});
		this.dnum = sum     
		count.forEach((ele, index) => {
			this.merge.len.set(index, ele) 
		})               
	},
}
</script>
<style lang="stylus">
.livestock-add
	padding-top 30px
	.el-table
		width 481px
		display inline-block
	.selectCrowd
		width: calc(100% - 608px);
		display: inline-block;
		padding: 0px 20px;	
		vertical-align: top;
		.select-group
			.el-input
				width 130px !important
			.el-input-group__prepend
				width 50px	
		.single
			margin-top 30px
			display: flex;
			flex-direction: row;
			flex-wrap: wrap;
			align-items: center;
			.el-input
				width 200px !important
				margin-left 8px
				padding 10px 0 
			.el-input-group__prepend
				width 80px	
		.multiple 
			margin-top 30px
			.el-input
				width 400px !important
				margin-left 8px
			.el-input-group__prepend
				width 80px	
.multiple-select
	.el-checkbox-group
		display flex
		flex-wrap wrap
		justify-content space-around
	.el-checkbox
		margin-left 0px !important
		width 50% !important
</style>
