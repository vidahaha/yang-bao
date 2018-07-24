<template>
	<div class="livestock-sale">
		<el-table :data="tableData" :border="true">
			<el-table-column
				label="耳牌号"
				width="150"
				prop="tag"
			>
			</el-table-column>
			<el-table-column
				label="死亡原因"
				width="200"
				prop="reason"
			>
			</el-table-column>
			<el-table-column
				label="处理方法"
				width="150"
				prop="way"
			>
			</el-table-column>
			<el-table-column
				label="时间"
				width="150"
				prop="time"
			>
			</el-table-column>
		</el-table>
	</div>
</template>
<script>
export default {
	data() {
		return {
			tableData:[
				{
					tag: 'G165418',
					reason: 'daa',
					way: '大萨',
					time: '2018-07-09'
				},
				{
					tag: 'G165418',
					reason: 'daa',
					way: '大萨',
					time: '2018-07-09'
				},
				{
					tag: 'G165418',
					reason: 'daa',
					way: '大萨',
					time: '2018-07-09'
				},
				{
					tag: 'G165418',
					reason: 'daa',
					way: '大萨',
					time: '2018-07-09'
				},
			]
		}
	}
}
</script>
<style lang="stylus">
.livestock-sale
	padding 10px 0 0 30px
	.el-table
		display table-caption
</style>

