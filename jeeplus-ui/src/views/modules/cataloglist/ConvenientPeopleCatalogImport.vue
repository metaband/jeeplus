<template>
  <div>
    <div class="public-search-box">
      <el-form :inline="true" class="demo-form-inline" :model="searchData" ref="searchForm">
        <el-form-item label="操作人员:" prop="userName">
          <el-input placeholder="请输入操作人员" v-model="searchData.userName"></el-input>
        </el-form-item>
        <el-form-item label="操作部门:" prop="branch">
          <el-input placeholder="请输入操作部门" v-model="searchData.branch"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="searchCatalogListData(searchData)">查询</el-button>
          <el-button @click="refreshCatalogList()">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="public-table-box">
      <div class="public-menu-box">
        <button class="public-btn" @click="templateDownload"><i class="fa fa-download"></i>下载模板</button>
      <!--  <button class="public-btn"><i class="fa fa-download"></i>导入数据</button>-->
        <el-upload
          style="float: right;margin-left: 10px;"
          :action="uploadAction"
          :on-progress="handleUploadProgress"
          :on-success="handUploadSucess"
          :show-file-list="false">
          <el-button class="public-btn"><i class="fa fa-download"></i>导入数据</el-button>
        </el-upload>
      </div>
      <el-table
        v-loading="loading"
        :data="tableData"
        style="width: 100%">
        <el-table-column
          type="index"
          label="序号"
          width="100">
        </el-table-column>
        <el-table-column
          prop="createDate"
          label="导入时间"
          min-width="16%">
        </el-table-column>
        <el-table-column
          prop="importCount"
          label="导入数量"
          min-width="12%">
        </el-table-column>
        <el-table-column
          prop="importLevel"
          label="操作层级"
          min-width="16%"
          :formatter="statusFormatter"
        >
        </el-table-column>
        <el-table-column
          prop="importOrgName"
          label="操作部门"
          min-width="16%">
        </el-table-column>
        <el-table-column
          prop="importUserName"
          label="操作人员"
          min-width="16%">
        </el-table-column>
        <el-table-column
          label="操作"
          min-width="16%">
          <template slot-scope="scope">
            <el-button @click="lookDetails(scope.row)" type="text" size="small">查看</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="pageNumber"
        :page-sizes="[20, 40, 60, 100]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next"
        :total="total">
      </el-pagination>
    </div>
    <ConvenientPeopleCatalogImportDetails ref="ConvenientPeopleCatalogImportDetails"></ConvenientPeopleCatalogImportDetails>
    <el-dialog
      placement="center"
      width="70%"
      trigger="click"
      :visible.sync="dialogVisible"
      :before-close="handleClose"
      title="导入数据"
    >
      <div style="margin: -30px 0 16px 0;">
        <el-button type="primary" size="mini" @click="saveImport">保存</el-button>
        <!--<el-button size="mini" type="danger" plain icon="el-icon-delete" :disabled="ifCanDelete" @click="deleleWaitImportData">删除</el-button>-->
        <el-button size="mini" @click="deleteWaitImport">取消</el-button>

      </div>
      <div style="text-align: right; margin: 0">
        <el-table
          ref="waitImportSystem"
          :key="tableKey"
          v-loading="wordTableLoading"
          :data="importCataList"
          size="mini"
          fit
          highlight-current-row
          style="width: 100%;"
          :header-cell-style="{background:'#ebeef5',color:'#909399'}"
        >
          <el-table-column prop="baseCode" label="基本编码" />
          <el-table-column prop="cataName" label="目录名称" />
          <el-table-column prop="cataType" label="事项类型">
            <template slot-scope="{row}">
              {{ cataTypeList[row.cataType] }}
            </template>
          </el-table-column>
          <el-table-column prop="cataLevel" label="目录层级" >
            <template slot-scope="{row}">
              {{ cataLevelList[row.cataLevel] }}
            </template>
          </el-table-column>
          <el-table-column prop="cataVersion" label="目录版本" />
          <el-table-column prop="importStatus" label="目录状态" />
          <el-table-column prop="remarks" label="备注" />
          <el-table-column prop="checkResult" label="查验结果" />
          <el-table-column prop="importReport" label="导入报告" />
          <el-table-column label="操作" >
            <template slot-scope="scope">
              <el-button @click="delImport(scope.row)" type="text" size="small" style="color:#FA5555 ">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          @size-change="handleImportSizeChange"
          @current-change="handleImportCurrentChange"
          :current-page="importPage"
          :page-sizes="[20, 40, 60, 100]"
          :page-size="importPageSize"
          layout="total, sizes, prev, pager, next"
          :total="importTotal">
        </el-pagination>
      </div>
      <!--        <el-button slot="reference" v-show="false" ref="btn" @click="import_systemList"></el-button>-->
    </el-dialog>

  </div>
</template>
<!--便民目录导入-->
<script>
  import {getCatalogListData} from "@/api/modules/catalog/catalogImport";
  import {getStateData} from "@/api/modules/catalog/dict";
  import ConvenientPeopleCatalogImportDetails from './ConvenientPeopleCatalogImportDetails'
  import {getImportCatalogListData,deleteImportCatalog,saveImportCatalog} from "@/api/modules/catalog/catalogList";
  const uploadUrl = process.env.VUE_APP_SERVER_URL + '/convience/catalog/import/import'
  export default {
    data () {
      return {
        searchData: {
          userName: '',
          branch: ''
        },
        total:0,
        pageNumber:1,
        pageSize:20,
        tableData: [],
        loading:true,
        handleLevelData:'',
        uploadAction: uploadUrl,
        dialogVisible: false,
        tableKey: 0,
        wordTableLoading: true,
        importCataList:[],
        importQuery: {},
        importPage:1,
        importPageSize:20,
        importTotal:0,
        cataTypeList:{},
        cataLevelList:{}
      }
    },
    components: {
      ConvenientPeopleCatalogImportDetails
    },
    methods: {
      // 下载文件
      templateDownload(){
        var a = document.createElement("a"); //创建一个<a></a>标签
        a.href = "/static/file/convenientPeopleCatalog.xlsx"; // 给a标签的href属性值加上地址，注意，这里是绝对路径，不用加 点.
        a.download = "便民目录.xlsx"; //设置下载文件文件名，这里加上.xlsx指定文件类型，pdf文件就指定.fpd即可
        a.style.display = "none"; // 障眼法藏起来a标签
        document.body.appendChild(a); // 将a标签追加到文档对象中
        a.click(); // 模拟点击了a标签，会触发a标签的href的读取，浏览器就会自动下载了
        a.remove(); // 一次性的，用完就删除a标签
      },
      handleSizeChange(val){
        this.pageSize=val
        this.getCatalogListData()
      },
      handleCurrentChange(val){
        this.pageNumber=val
        this.getCatalogListData()
      },
      handleImportSizeChange(val){
        this.importPageSize=val
        this.import_cataList()
      },
      handleImportCurrentChange(val){
        this.importPage=val
        this.import_cataList()
      },
      lookDetails(row){
        this.$refs.ConvenientPeopleCatalogImportDetails.init(row)
      },
      getCatalogListData(){
        this.tableData=[]
        this.loading=true
        getStateData({dictType:'item_level'}).then(({data})=>{
          this.handleLevelData=data.keyAndValue
          this.cataLevelList = data.keyAndValue
        }).then(()=>{
          getCatalogListData({pageNo: this.pageNumber, pageSize: this.pageSize}).then(({data}) => {
            if (data && data.success) {
              this.total=data.data.total
              this.tableData=data.data.records
            }
            this.loading = false
          })
        })

      },
      // 搜索
      searchCatalogListData(searchData){
        if (searchData.userName===''&&searchData.branch===''){
          this.$message('请输入要搜索的操作人员或部门')
        }else{
          this.tableData=[]
          this.loading=true
            getCatalogListData({pageNo: this.pageNumber, pageSize: this.pageSize,importOrgName:searchData.branch,importUserName:searchData.userName}).then(({data}) => {
              if (data && data.success) {
                this.total=data.data.total
                this.tableData=data.data.records
              }
              this.loading = false
          })
        }
      },
      //  刷新列表
      refreshCatalogList(){
        this.searchData.branch=''
        this.searchData.userName=''
        this.pageNumber=1
        this.pageSize=20
        this.getCatalogListData(this.searchData)
      },
      // 事项类型转换文字展示
      statusFormatter(row, column){
        const handleLevel = row.importLevel
        return this.handleLevelData[handleLevel]
      },
      // 文件正在导入
      handleUploadProgress(){
        this.dialogVisible = true
        this.wordTableLoading = true
        this.importCataList=[]
      },
      // 导入成功
      handUploadSucess(res, file) {
        if (res && res.success) {
          this.importQuery.importFileUuid = res.data
          this.import_cataList()
        } else {
          this.dialogVisible = false
          this.wordTableLoading = false
          this.$message('error', res.msg)
        }
      },
      import_cataList() {
        this.wordTableLoading = true
        this.importQuery.pageNo = 1
        this.importQuery.pageSize = 20
       getImportCatalogListData(this.importQuery).then(({data}) => {
         if (data && data.success) {
           this.importCataList = data.data.records
           this.importTotal = data.data.total
           if (this.importTotal === 0) {
             this.$message('error', '导入失败，请重新导入')
             this.visible = false
             this.dialogVisible = false
             this.getCatalogListData()
           }
           this.wordTableLoading = false
         }
       })
      },
      //关闭弹框
      handleClose(done) {
        this.$confirm('确认关闭吗？')
          .then(_ => {
            done()
            this.deleteWaitImport()
          })
          .catch(_ => {})
      },
      // 根据Type获取dict
      async getDictByType() {
        getStateData({dictType:'con_item_type'}).then(({data})=>{
          this.cataTypeList=data.keyAndValue
        })
      },
      //取消导入
      deleteWaitImport() {
        this.visible = false
        this.dialogVisible = false
        this.getCatalogListData(this.searchData)
        /* deleteWaitImport().then((response) => {
        })*/
      },
      delImport(row){
        this.$confirm('确认删除该条目录吗？')
          .then(_ => {
            this.wordTableLoading = true
            deleteImportCatalog(row).then(({data}) => {
              if (data && data.success) {
                this.import_cataList()
                this.wordTableLoading = false
                this.$message.success(data.msg)
              }
            })
          })
          .catch(_ => {
            this.$message.success("取消删除")
          })

      },
      saveImport(){
        this.wordTableLoading = true
        saveImportCatalog(this.importQuery).then(({data}) => {
          if (data && data.success) {
            this.wordTableLoading = false
            this.$message.success(data.msg)
            this.visible = false
            this.dialogVisible = false
            this.getCatalogListData()
          } else {
            this.$message('error', '保存失败')
          }
        })
      }


    },
    mounted() {
      // 获取列表数据
      this.getCatalogListData()
      this.getDictByType()
    }
  }
</script>
<style scoped>


</style>
