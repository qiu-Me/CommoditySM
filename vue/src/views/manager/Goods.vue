<template>
  <div>
    <div class="search">
      <el-input placeholder="请输入商品名称查询" style="width: 200px" v-model="name"></el-input>
      <el-input placeholder="请输入供应商名称查询" style="width: 200px; margin-left: 10px" v-model="supplierName"></el-input>
      <el-input placeholder="请输入生产批号查询" style="width: 200px; margin-left: 10px" v-model="productNo"></el-input>
      <el-input placeholder="请输入批准文号查询" style="width: 200px; margin-left: 10px" v-model="approveNo"></el-input>
      <el-button type="info" plain style="margin-left: 10px" @click="load(1)">查询</el-button>
      <el-button type="warning" plain style="margin-left: 10px" @click="reset">重置</el-button>
    </div>

    <div class="operation">
      <el-button type="primary" plain @click="handleAdd">新增</el-button>
      <el-button type="danger" plain @click="delBatch">批量删除</el-button>
      <el-button type="primary" plain @click="exp">批量导出</el-button>
      <el-upload :show-file-list="false" style="display: inline-block; margin-left: 10px" :action="$baseUrl + '/goods/import'" :headers="{ token: user.token }" :on-success="handleImport" >
        <el-button type="info" plain>批量导入</el-button>
      </el-upload>
    </div>

    <div class="table">
      <el-table :data="tableData" strip @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55" align="center"></el-table-column>
        <el-table-column prop="id" label="序号" width="70" align="center" sortable></el-table-column>
        <el-table-column prop="name" label="商品名称"></el-table-column>
        <el-table-column prop="supplierName" label="供应商"></el-table-column>
        <el-table-column prop="producer" label="商品产地"></el-table-column>
        <el-table-column prop="descr" label="商品描述"></el-table-column>
        <el-table-column prop="price" label="销售价格"></el-table-column>
        <el-table-column prop="num" label="商品数量"></el-table-column>
        <el-table-column prop="img" label="商品图片">
          <template v-slot="scope">
            <div style="display: flex; align-items: center">
              <el-image style="width: 40px; height: 40px;" v-if="scope.row.img"
                        :src="scope.row.img" :preview-src-list="[scope.row.img]"></el-image>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="unit" label="商品规格"></el-table-column>
        <el-table-column prop="pack" label="包装单位"></el-table-column>
        <el-table-column prop="productNo" label="生产批号"></el-table-column>
        <el-table-column prop="approveNo" label="批准文号"></el-table-column>
        <el-table-column prop="status" label="状态">
          <template v-slot="scope">
            <el-tag type="success" v-if="scope.row.status === '可用'">可用</el-tag>
            <el-tag type="danger" v-if="scope.row.status === '不可用'">不可用</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" align="center" width="180">
          <template v-slot="scope">
            <el-button size="mini" type="primary" plain @click="handleEdit(scope.row)">编辑</el-button>
            <el-button size="mini" type="danger" plain @click="del(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <div class="pagination">
        <el-pagination
            background
            @current-change="handleCurrentChange"
            :current-page="pageNum"
            :page-sizes="[5, 10, 20]"
            :page-size="pageSize"
            layout="total, prev, pager, next"
            :total="total">
        </el-pagination>
      </div>
    </div>


    <el-dialog title="商品信息" :visible.sync="fromVisible" width="40%" :close-on-click-modal="false" destroy-on-close>
      <el-form :model="form" label-width="100px" style="padding-right: 50px" :rules="rules" ref="formRef">
        <el-form-item label="商品名称" prop="name">
          <el-input v-model="form.name" placeholder="商品名称"></el-input>
        </el-form-item>
        <el-form-item label="供应商" prop="supplierId">
          <el-select style="width: 100%" v-model="form.supplierId">
            <el-option v-for="item in suppliers" :key="item.id" :value="item.id" :label="item.name"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="商品产地" prop="producer">
          <el-input v-model="form.producer" placeholder="商品产地"></el-input>
        </el-form-item>
        <el-form-item label="商品描述" prop="descr">
          <el-input v-model="form.descr" placeholder="商品描述"></el-input>
        </el-form-item>
        <el-form-item label="销售价格" prop="price">
          <el-input v-model="form.price" placeholder="销售价格"></el-input>
        </el-form-item>
        <el-form-item label="商品数量" prop="num">
          <el-input v-model="form.num" placeholder="商品数量"></el-input>
        </el-form-item>
        <el-form-item label="商品图片" prop="img">
          <el-upload
              :action="$baseUrl + '/files/upload'"
              :headers="{ token: user.token }"
              :on-success="handleFileSuccess"
          >
            <el-button type="primary">上传</el-button>
          </el-upload>
        </el-form-item>
        <el-form-item label="商品规格" prop="unit">
          <el-input v-model="form.unit" placeholder="商品规格"></el-input>
        </el-form-item>
        <el-form-item label="包装单位" prop="pack">
          <el-input v-model="form.pack" placeholder="包装单位"></el-input>
        </el-form-item>
        <el-form-item label="生产批号" prop="productNo">
          <el-input v-model="form.productNo" placeholder="生产批号"></el-input>
        </el-form-item>
        <el-form-item label="批准文号" prop="approveNo">
          <el-input v-model="form.approveNo" placeholder="批准文号"></el-input>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-select style="width: 100%" v-model="form.status">
            <el-option v-for="item in ['可用', '不可用']" :key="item" :value="item" :label="item"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="fromVisible = false">取 消</el-button>
        <el-button type="primary" @click="save">确 定</el-button>
      </div>
    </el-dialog>


  </div>
</template>

<script>
export default {
  name: "Goods",
  data() {
    return {
      tableData: [],  // 所有的数据
      suppliers: [],  // 供应商数据
      pageNum: 1,   // 当前的页码
      pageSize: 10,  // 每页显示的个数
      total: 0,
      name: null,
      supplierName: null,
      productNo: null,
      approveNo: null,
      fromVisible: false,
      form: {},
      user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
      rules: {
        name: [
          {required: true, message: '请输入名称', trigger: 'blur'},
        ],
        supplierId: [
          {required: true, message: '请选择供应商', trigger: 'blur'},
        ],
        price: [
          {required: true, message: '请输入价格', trigger: 'blur'},
        ]
      },
      ids: []
    }
  },
  created() {
    this.load(1)
  },
  methods: {
    handleImport(res) {
      if (res.code === '200') {  // 成功
        this.$message.success('导入成功')
        this.load(1)  // 刷新表格数据
      } else {
        this.$message.error(res.msg)
      }
    },
    exp() {
      window.open(this.$baseUrl + '/goods/export')
    },
    handleAdd() {   // 新增数据
      this.form = {}  // 新增数据的时候清空数据
      this.fromVisible = true   // 打开弹窗
    },
    handleEdit(row) {   // 编辑数据
      this.form = JSON.parse(JSON.stringify(row))  // 给form对象赋值  注意要深拷贝数据
      this.fromVisible = true   // 打开弹窗
    },
    save() {   // 保存按钮触发的逻辑  它会触发新增或者更新
      this.$refs.formRef.validate((valid) => {
        if (valid) {
          this.$request({
            url: this.form.id ? '/goods/update' : '/goods/add',
            method: this.form.id ? 'PUT' : 'POST',
            data: this.form
          }).then(res => {
            if (res.code === '200') {  // 表示成功保存
              this.$message.success('保存成功')
              this.load(1)
              this.fromVisible = false
            } else {
              this.$message.error(res.msg)  // 弹出错误的信息
            }
          })
        }
      })
    },
    del(id) {   // 单个删除
      this.$confirm('您确定删除吗？', '确认删除', {type: "warning"}).then(response => {
        this.$request.delete('/goods/delete/' + id).then(res => {
          if (res.code === '200') {   // 表示操作成功
            this.$message.success('操作成功')
            this.load(1)
          } else {
            this.$message.error(res.msg)  // 弹出错误的信息
          }
        })
      }).catch(() => {
      })
    },
    handleSelectionChange(rows) {   // 当前选中的所有的行数据
      this.ids = rows.map(v => v.id)   //  [1,2]
    },
    delBatch() {   // 批量删除
      if (!this.ids.length) {
        this.$message.warning('请选择数据')
        return
      }
      this.$confirm('您确定批量删除这些数据吗？', '确认删除', {type: "warning"}).then(response => {
        this.$request.delete('/goods/delete/batch', {data: this.ids}).then(res => {
          if (res.code === '200') {   // 表示操作成功
            this.$message.success('操作成功')
            this.load(1)
          } else {
            this.$message.error(res.msg)  // 弹出错误的信息
          }
        })
      }).catch(() => {
      })
    },
    load(pageNum) {  // 分页查询
      if (pageNum) this.pageNum = pageNum
      this.$request.get('/goods/selectPage', {
        params: {
          pageNum: this.pageNum,
          pageSize: this.pageSize,
          name: this.name,
          supplierName: this.supplierName,
          productNo: this.productNo,
          approveNo: this.approveNo,
        }
      }).then(res => {
        this.tableData = res.data?.list
        this.total = res.data?.total
      })

      this.$request.get('/supplier/selectAll').then(res => {
        this.suppliers = res.data || []
      })
    },
    reset() {
      this.name = null
      this.supplierName = null
      this.productNo = null
      this.approveNo = null
      this.load(1)
    },
    handleCurrentChange(pageNum) {
      this.load(pageNum)
    },
    handleFileSuccess(res) {
      this.form.img = res.data
    }
  }
}
</script>

<style scoped>

</style>