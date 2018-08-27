<template>
    <div class="livestock-index">
		<p class="info-p">新建栏栋</p>
		<div class="input-group">
			<el-input size="small" type="text" class="input" v-model="column">
				<template slot="prepend">栋号:</template>
			</el-input>
			<el-input size="small" type="text" class="input" v-model="columnNum">
				<template slot="prepend">本栋栏数:</template>
			</el-input>
			<el-button type="small" @click="addColumn()">新建</el-button>
			<el-button type="small">编辑</el-button>
			<p style="margin-left: 450px">
				(编辑时栋号不变，栏数只能增加不能减少)
			</p>
		</div>
		<p class="info-p">本厂羊只总数：{{allNum}}</p>
        <el-table :data="tableData" :span-method="objectSpanMethod" :border="true" height="450" ref="table">
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
				<template slot-scope="scope">
					<input v-model="scope.row.lnum" style="text-align: center;width: 60%;margin: 0 auto;display: block;"/>
				</template>
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

import { getGeneaRec, postGeneaRec, updateGeneaRec } from '@/util/getdata'

export default {
    components: {

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
		this.dnum = sum;
		this.allNum = sum.reduce( (a, b) => {
			return a + b;
		})     
		count.forEach((ele, index) => {
			this.merge.len.set(index, ele) 
		})               
	},

	watch: {
		tableData: {
			handler(o, n) {
				console.log(n)
				let count = []
				let sum = []		
				n.forEach(ele => {
					if (ele.d) {
						if(sum[ele.d] === undefined) {
							sum[ele.d] = 0
							count[ele.d] = 0
						}		
						sum[ele.d] += parseInt(ele.lnum)	
						count[ele.d] ++ 	
					} 
				});
				this.dnum = sum;
				this.allNum = sum.reduce( (a, b) => {
					return a + b;
				})  
				count.forEach((ele, index) => {
					this.merge.len.set(index, ele) 
				}) 
			},
			deep: true		
		},
	},
	
    data () {
        return {
            getGeneaRec,
            postGeneaRec,
			updateGeneaRec,
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
			tradeMarkEartag: '',
			immuneEartag: '', //批量分配
			checkList:[],
			crowdTag:['G123456','G123415','G1232465','G1233456','G142345','G1523465'],
            items: [
                {label: '性别', model: 'sex', type: 'radio'},
                {label: '商标耳牌', model: 'tradeMarkEartag', trade: true, block: 1},
                {label: '羊场地理位置', model: 'breedLocation', type: 'address'},
                {label: '出生基地', model: 'breedingSheepBase'},
                {label: '出生时间', model: 'birthTime', type: 'time', mr: 1},
                {label: '初生体重(kg)', model: 'birthWeight'},
                {label: '颜色', model: 'color'},
                {label: '品种名', model: 'typeName', mr: 1},
                {label: '父号', model: 'eartagOfFather'},
                {label: '父父号', model: 'eartagOfFathersFather'},
                {label: '母父号', model: 'eartagOfMothersFather', mr: 1},
                {label: '母号', model: 'eartagOfMother'},
                {label: '父母号', model: 'eartagOfFathersMother'},
                {label: '母母号', model: 'eartagOfMothersMother', mr: 1},
            ],
			// 用于检查字段值是否填写，所以均初始化为null
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
            models: {
                tradeMarkEartag: null,
				breedingSheepBase: null,
            }
		}
		
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

	methods: {
		objectSpanMethod({ row, column, rowIndex, columnIndex }) {
			let d = row.d;
			let len = this.merge.len.get(parseInt(d));
			let flag = 0 
			for (let index of this.merge.len.keys()) {
				if ( index <= d - 1 ) {
					flag += this.merge.len.get(index)
				}
			}
			if (columnIndex === 3) {
				if (rowIndex === flag) {
					console.log( len )
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

		addColumn() {
			let c = this.column
			if ( c <= 0 ) return;
			let n = this.columnNum
			let len = this.merge.len.get(parseInt(c)) || 0;
			if ( len && n <= len ) return;
			let flag = 0
			for ( let f = 1; f <= c; f++ ) {
				flag += this.merge.len.get(parseInt(f)) || 0;
			}
			for ( let i = n; i > len; i-- ) {
				this.tableData.splice(flag, 0, {d:c, l:i, lnum: 1})
			}
		}
	}
}
</script>

<style lang="stylus">
.livestock-index
	.input-group
		margin 10px 0 30px 10px
	.el-input
		width 270px	!important 
		vertical-align middle
		margin-left 35px
	.el-button
		margin-left 10px
	.el-table	
		margin-left 30px
		width 481px
		display inline-block
	.info-p
		margin-left 30px
</style>
