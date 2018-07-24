<template>
	<div class="livestock-list">
		<div class="header">
			<el-input v-model="id">
				<template slot="prepend">出售给厂编号</template>
			</el-input>
			<el-button type="primary">确定</el-button>
		</div>
		<el-table :data="tableData" :border="true">
			<el-table-column
				label="出售"
				width="120"
			>
				<template slot-scope="scope">
					<el-checkbox v-model="checked"></el-checkbox>
				</template>
			</el-table-column>
			<el-table-column
				label="商标耳牌号"
				width="120"
				prop="tradeMarkEartag"
			>
			</el-table-column>
			<el-table-column
				label="免疫耳牌号"
				width="120"
				prop="immuneEartag"
			>
			</el-table-column>
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
				label="操作"
				width="200"
			>
				<template slot-scope="scope">
					<el-button
						size="small"
						@click="handleMove(scope.row)">移动
					</el-button>
					<el-button
						size="small"
						type="danger"
						@click="handleDead(scope.row)">死亡
					</el-button>
				</template>
			</el-table-column>
		</el-table>
		<el-dialog :visible.sync="dialogMoveVisible" class="move">
			<el-form :model="form">
				<span>商标耳牌号: {{this.currentRow.tradeMarkEartag}}</span>
				<span>从栋号: {{this.currentRow.d}}</span>
				<span>栏号: {{this.currentRow.l}}</span>
				<span>移至</span><br/><br/>
				<span>栋号：</span>
				<el-input v-model="form.d" size="small"></el-input>
				<span>栏号：</span>
				<el-input v-model="form.l" size="small"></el-input>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="dialogMoveVisible  = false" size="small">取 消</el-button>
				<el-button type="primary" @click="dialogMoveVisible  = false" size="small">添 加</el-button>
			</div>
		</el-dialog>
		<el-dialog :visible.sync="dialogDeadVisible" class="dead">
			<el-form :model="form">
				<span>商标耳牌号: {{this.currentRow.tradeMarkEartag}}</span><br/>
				<span>死亡原因：</span>
				<el-input v-model="form.d" size="small"></el-input><br/>
				<span>处理方法：</span>
				<el-input v-model="form.l" size="small"></el-input>
			</el-form>
			<div slot="footer" class="dialog-footer">
				<el-button @click="dialogDeadVisible  = false" size="small">取 消</el-button>
				<el-button type="primary" @click="dialogDeadVisible  = false" size="small">确 定</el-button>
			</div>
		</el-dialog>
	</div>
</template>

<script>
export default {
	data() {
		return {
			tableData: [
				{
					tradeMarkEartag: 'G2166',
					immuneEartag: 'G65',
					d: 1,
					l: 1
				},
				{
					tradeMarkEartag: 'G2266',
					immuneEartag: 'G65',
					d: 1,
					l: 1
				},
				{
					tradeMarkEartag: 'G2366',
					immuneEartag: 'G65',
					d: 1,
					l: 1
				},
				{
					tradeMarkEartag: 'G2466',
					immuneEartag: 'G65',
					d: 1,
					l: 1
				},
			],
			checked: 1,
			id: 1,
			dialogMoveVisible: false,
			dialogDeadVisible: false,
			form: {
				name: '',
				region: '',
				date1: '',
				date2: '',
				delivery: false,
				type: [],
				resource: '',
				desc: ''
			},
			currentRow: {},
		}
	},

	methods: {
		handleMove( row ) {
			this.currentRow = row
			this.dialogMoveVisible = true
		},
		handleDead( row ) {
			this.currentRow = row
			this.dialogDeadVisible = true
		}
	}
}
</script>

<style lang="stylus">
.livestock-list
	.header
	    display flex
		justify-content space-evenly
		width 50%
		margin 10px 0px 30px 0px
		.el-input
			margin-left 50px
			width 250px
	.el-table
		display table-caption
		margin-left 51px
	.el-dialog
		width 400px		
	.move 
		.el-input
			width 100px
	.dead
		.el-input
			width 277px		
			padding: 5px 0 5px 0
</style>


