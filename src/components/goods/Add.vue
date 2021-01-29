<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert title="消息提示的文案"
                type="info"
                show-icon
                :closable="false">
      </el-alert>
      <!-- 步骤条 -->
      <el-steps :space="200"
                :active="activeIndex - 0"
                align-center
                finish-status="success">
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- 竖向tab -->
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addFormRef"
               label-position="top"
               label-width="100px">
        <el-tabs tab-position="left"
                 :before-leave="beforeTabsLeave"
                 v-model="activeIndex"
                 @tab-click="tabClicked">
          <el-tab-pane label="基本信息"
                       name='0'>
            <el-form-item label="商品名称"
                          prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格"
                          prop="goods_price">
              <el-input v-model="addForm.goods_price"
                        type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量"
                          prop="goods_weight">
              <el-input v-model="addForm.goods_weight"
                        type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量"
                          prop="goods_number">
              <el-input v-model="addForm.goods_number"
                        type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类"
                          prop="goods_cat">
              <el-cascader v-model="addForm.goods_cat"
                           :options="cateList"
                           :props="cateProps"
                           @change="handleChange"></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数"
                       name='1'>
            <el-form-item :label="item.attr_name"
                          v-for="(item) in manyTableData"
                          :key="item.attr_id">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox border
                             :label="cb"
                             v-for="(cb, i) in item.attr_vals"
                             :key='i'></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性"
                       name='2'>
            <el-form-item :label="item.attr_name"
                          v-for="item in onlyTableData"
                          :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片"
                       name='3'>
            <el-upload :on-success="handleSuccess"
                       :headers="headers"
                       :action="uploadPath"
                       :on-preview="handlePreview"
                       :on-remove="handleRemove"
                       list-type="picture">
              <el-button size="small"
                         type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容"
                       name='4'>
            <quill-editor ref="myQuillEditor"
                          v-model="addForm.goods_introduce" />
            <el-button label="添加商品"
                       type="primary"
                       class="btnAdd"
                       @click="addGoods">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <!-- 上传图片预览对话框 -->
    <el-dialog title="图片预览"
               :visible.sync="previewDialogVisible"
               width="50%">
      <img :src="previewPath"
           class="previewImg">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  data () {
    return {
      // 激活的步骤
      activeIndex: '0',
      // 添加表单的数据
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      // 添加表单的校验规则
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请输入商品分类', trigger: 'blur' }
        ]
      },
      // 分类数据
      cateList: [],
      // 级联选择框配置
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children',
        expandTrigger: 'hover'
      },
      // 动态参数列表
      manyTableData: [],
      // 静态属性列表
      onlyTableData: [],
      // 上传图片的URL
      uploadPath: 'http://127.0.0.1:8888/api/private/v1/upload',
      headers: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewPath: '',
      previewDialogVisible: false
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类信息失败')
      }
      this.cateList = res.data
    },
    handleChange () {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    beforeTabsLeave (activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    async tabClicked () {
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`,
          { params: { sel: 'many' } })
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }

        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`,
          { params: { sel: 'only' } })
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        this.onlyTableData = res.data
      }
    },
    // 预览图片事件
    handlePreview (file) {
      debugger
      this.previewPath = file.response.data.url
      this.previewDialogVisible = true
    },
    // 删除图片事件
    handleRemove (file) {
      const filePath = file.response.data.tmp_path
      const index = this.addForm.pics.findIndex(x => {
        x.pic = filePath
      })
      this.addForm.pics.splice(index, 1)
    },
    // 监听文件上传成功
    handleSuccess (response) {
      const picInfo = { pic: response.data.tmp_path }
      this.addForm.pics.push(picInfo)
    },
    addGoods () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          this.$message.error('请添加必要的表单项')
        }

        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')

        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_vals: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })

        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_vals: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })

        form.attrs = this.addForm.attrs

        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        }

        this.$message.success('添加成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 0 10px 0;
}
.previewImg {
  width: 100%;
}
.btnAdd {
  margin-top: 15px;
}
</style>
