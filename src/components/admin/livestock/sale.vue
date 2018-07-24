<template>
	<div class="livestock-sale">
		<el-table :data="tableData" :border="true">
			<el-table-column
				label="商标耳牌号"
				width="150"
				prop="tag"
			>
			</el-table-column>
			<el-table-column
				label="销售目的养殖厂"
				width="200"
				prop="factory"
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
					factory: 'daa',
					time: '2018-07-09'
				},
				{
					tag: 'G161548',
					factory: 'aadaa',
					time: '2018-07-09'
				},
				{
					tag: 'G165148',
					factory: 'aaaa',
					time: '2018-07-09'
				},
				{
					tag: 'G116548',
					factory: 'aaaa',
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

