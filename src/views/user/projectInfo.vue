<template>
    <div class="height-100">
            <Table
               ref="table"
              :tableLabel="tableLabel"
              :getData='queryData'
              :filter='false'
              @beforeAdd = 'beforeAdd'
              @editItem = 'editItem'
              :disabledAdd="disabledAdd"
              :deleteRow = 'deleteItem'
              :exportData="exportFile" 
              @submit="addOrUpdate"
            >
                <el-form slot="dialog" :model="addData" inline ref="form">
                    <el-form-item label="所属范围">
                        <el-input readonly v-model="currentNode.FAreaName"></el-input>
                    </el-form-item>
                    <el-form-item label="项目编码" prop="FProjectCode" :rules="[{ required: true, message: '请输入'}]">
                        <el-input v-model="addData.FProjectCode"></el-input>
                    </el-form-item>
                    <el-form-item label="项目全称" prop="FProjectName" :rules="[{ required: true, message: '请输入'}]">
                        <el-input v-model="addData.FProjectName"></el-input>
                    </el-form-item>
                    <el-form-item label="项目简称"  prop="FSimpleName" :rules="[{ required: true, message: '请输入'}]">
                        <el-input  v-model="addData.FSimpleName"></el-input>
                    </el-form-item>
                    <el-form-item label="业态" prop="BuildTypeID" :rules="[{ required: true, message: '请选择'}]">
                        <el-select v-model="addData.BuildTypeID">
                            <el-option :value="31" label="商住"></el-option>
                            <el-option :value="32" label="商业"></el-option>
                            <el-option :value="33" label="住宅"></el-option>
                            <el-option :value="34" label="酒店会所"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="建筑面积" prop="BuildArea" :rules="[{ required: true, message: '请输入'}]">
                        <el-input class="unit" type="number" v-model="addData.BuildArea" autocomplete="off">
                           <span slot="suffix">m²</span>
                        </el-input>
                    </el-form-item>
                    <el-form-item label="排序" prop="FSortID" :rules="[{ required: true, message: '请输入'}]">
                        <el-input v-model="addData.FSortID" type="number"></el-input>
                    </el-form-item>
<!--                     <el-form-item label="地区" prop="FAreaCode">
                        <el-cascader
                            ref="project"
                            v-model="FAreaCode"
                            :options="provinces"
                            :props="{ checkStrictly: true,label:'name',value:'id' }"
                            collapse-tags
                            clearable>
                        </el-cascader>
                    </el-form-item> -->
                    <el-form-item label="详细地址" prop="Address" :rules="[{ required: true, message: '请输入'}]">
                        <el-input class="block" type="textarea" v-model="addData.Address"></el-input>
                    </el-form-item>
                </el-form>
            </Table>
    </div>
</template>
<script>
import Table from '../component/table.vue'
/* import provinceList from '@/mapJson/provinceList.json' */
const param = {
    PageIndex:1,
    PageSize:1000,
    SearchKey:''
}
export default {
    data(){
        return{
            show:false,
            filterText:'',
            tableLabel:[
                {
                    prop: 'FProjectCode',
                    label: '项目编码',
                },
                {
                    prop: 'FSimpleName',
                    label: '项目名称',
                },
                {
                    prop: 'FAreaName',
                    label: '所属范围',
                },
                {
                    prop: 'SystemParamName',
                    label: '业态',
                },
                {
                    prop: 'FSortID',
                    label: '排序',
                },
            ],
            treeData:[],
            treeProp:{
                children:'ListData',
                label:'FAreaName'
            },
            defaultAddData:{ //新增默认数据
            },
            provinces:[],
            addData:{
                ProjectID:0,         
                FORGNodeGUID:'',      
                FProjectName:'',
                FSimpleName:'',   
                FProjectCode:'',  
                FAreaName:'',
                FAreaCode:'',  
                FProjectlng:'',  
                FProjectlat:'',    
                FSortID:'',    
                Address:'',
                BuildArea:'',   
                BuildTypeID:'',     
                OtherSourceID:0,   
                SystemType:0,   
                OnlineDateTime:'',
            },
            FAreaCode:[],
            currentNode:{}, //当前选中节点
        }
    },
    components:{
        Table
    },
    computed:{
        disabledAdd(){
            let value = this.currentNode.FGUID?false:true
            return value
        }
    },
    watch:{
        filterText(val){
            this.$refs.tree.filter(val);
        },
    },
    created(){
        this.defaultAddData = JSON.parse(JSON.stringify(this.addData))
        this.provinces = provinceList
    },
    methods:{
        /**
         * 查询项目信息
         */
        queryData(that){
            let param = {
                FGUID: '',
                PageIndex:that.pageIndex,
                FAreaCode:this.currentNode.FAreaCode,
                FAreaLevel:this.currentNode.FAreaLevel,
                PageSize:10,
                SearchKey:''
            }
            return this.$post('QueryPageTORGProject',param,true)
        },
        /**
         * 点击新增
         */
        beforeAdd(){
            this.addData = Object.assign({},this.defaultAddData)
            this.FAreaCode = []
        },
        /**
         * 编辑
         */
        editItem(row){
            Object.keys(this.addData).forEach(key => {
                this.addData[key] = row[key]
            })
        },
        /**
         * 新增或修改项目
         */
        async addOrUpdate(){
            this.addData.FORGNodeGUID = this.currentNode.FGUID
            this.addData.FAreaCode = this.currentNode.FAreaCode
            await new Promise(resolve => {
                this.$refs.form.validate((valid) => {
                  if (valid) {
                      resolve()
                  } 
                });
            })
            let myGeo = new BMap.Geocoder()
            let address = this.addData.Address
            await new Promise((resolve) => {
                myGeo.getPoint(address,point => {
                    this.addData.FProjectlng = point.lng||this.addData.FProjectlng
                    this.addData.FProjectlat = point.lat||this.addData.FProjectlat
                    resolve()
                })
            })
            this.$post('AddOrUpdateTORGProject',{
                MTORGProject:this.addData
            })
            .then((result) => {
                this.$refs.table.queryData()
                this.$refs.table.show = false
            }).catch((err) => {
                
            });
        },
        /**
         * 删除项目
         */
        deleteItem(row){
            return this.$post('DeleteTORGProject',{ProjectID:row.ProjectID})
        },
        exportFile(){
            return this.$post('ExportTORGProject',{FGUID:this.currentNode.FGUID,SearchKey:''})
        }
    }
}
</script>
<style lang="scss">
</style>