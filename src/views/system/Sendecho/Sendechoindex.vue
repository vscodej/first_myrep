<script setup lang="ts">
import { ref, onMounted, reactive, nextTick } from 'vue'
import axios from "axios";
import type {
  RoleResponseData,
  Records,
  RoleData,
  MenuData,
} from '@/api/system/role/type.ts'
import useLayOutSettingStore from '@/store/modules/setting'
import {ElMessage} from "element-plus";
let pageNo = ref<number>(1)

let pageSize = ref<number>(10)

let settingStore = useLayOutSettingStore()

let dialogVisible = ref<boolean>(false)

let RoleParams = reactive<RoleData>({
  roleName: '',
})

onMounted(() => {
  getHasRole()
})

let keyword = ref<string>('')

let allRole = ref<Records>([])

let total = ref<number>(0)
let form = ref<any>()
let selectIdArr = ref<RoleData[]>([])


const getHasRole = async (pager = 1) => {
  pageNo.value = pager
  let res:RoleResponseData = await  axios.get( ` http://localhost:8080/admin/system/sysRole/${pageNo.value}/${pageSize.value}/?roleName=${keyword.value}`)
    res =res.data
  if (res.code === 200) {
    total.value = res.data.total
    allRole.value = res.data.records
  }
}

const sizeHandler = () => {
  getHasRole()
}

const search = () => {
  getHasRole()
  keyword.value = ''
}

const reset = () => {
  settingStore.refsh = !settingStore.refsh
}

const addRole = () => {
  dialogVisible.value = true
  Object.assign(RoleParams, {
    roleName: '',
    id: 0,
  })
  nextTick(() => {
    form.value.clearValidate('roleName')
  })
}

const updateRole = (row: RoleData) => {
  dialogVisible.value = true
  Object.assign(RoleParams, row)
  nextTick(() => {
    form.value.clearValidate('roleName')
  })
}

const validateRoleName = (rule: any, value: any, callBack: any) => {
  if (value.trim().length >= 2) {
    callBack()
  } else {
    callBack(new Error('职位名称至少两位'))
  }
}
const selectChange = (value: any) => {
  selectIdArr.value = value
}

const rules = {
  roleName: [{ required: true, trigger: 'blur', validator: validateRoleName }],
}

const save = async () => {
  await form.value.validate()
  let res:any
  if(RoleParams.id){
     res= await axios.put( " http://localhost:8080/admin/system/sysRole/update",RoleParams)
     res =res.data
  }else{
     res =await axios.post( " http://localhost:8080/admin/system/sysRole/save",RoleParams)
     res =res.data
  }
  if (res.code === 200) {
    ElMessage({
      type: 'success',
      message: RoleParams.id ? '更新成功' : '添加成功',
    })
    dialogVisible.value = false
    getHasRole(RoleParams.id ? pageNo.value : 1)
  }
}

const deleteSelectUser = async () => {
  let idList: any = selectIdArr.value.map((item) => {
    return item.id
  })
  let res: any=await axios.delete( " http://localhost:8080/admin/system/sysRole/batchRemove" ,{data:idList} )
   res =res.data
  if (res.code === 200) {
    ElMessage({ type: 'success', message: '删除成功' })
    getHasRole(allRole.value.length > 1 ? pageNo.value : pageNo.value - 1)
  }
}

const filterSelectArr = (allData: any, initArr: any) => {
  allData.forEach((item: MenuData) => {
    if (item.select && item.children?.length  ===0) {
      initArr.push(item.id)
    }
    if (item.children && item.children.length > 0) {
      filterSelectArr(item.children, initArr)
    }
  })
  return initArr
}

const removeRole = async (id: number) => {
  let res: any = await axios.delete( ` http://localhost:8080/admin/system/sysRole/remove/${id}`  )
  res =res.data
  if (res.code === 200) {
    ElMessage({
      type: 'success',
      message: '删除成功',
    })
    getHasRole(allRole.value.length > 1 ? pageNo.value : pageNo.value - 1)
  }
}
</script>
<template>
  <el-card style="height: 80px">
    <el-form :inline="true" class="form" @keyup.enter.native="search"  >
      <el-form-item label="职位搜索">
        <el-input
            placeholder="请你输入搜索职位的关键字"
            v-model="keyword"
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button
            type="primary"
            size="default"
            :disabled="keyword ? false : true"
            @click="search"
        >
          搜索
        </el-button>
        <el-button size="default" @click="reset">重置</el-button>
      </el-form-item>
    </el-form>
  </el-card>
  <el-card style="margin: 10px 0">
    <el-button type="primary" size="default" icon="Plus" @click="addRole">
      添加职位
    </el-button>
    <el-button
        type="danger"
        size="default"
        :disabled="selectIdArr.length ? false : true"
        @click="deleteSelectUser"
    >
      批量删除
    </el-button>
    <el-table border style="margin: 10px 0" :data="allRole"  @selection-change="selectChange" >
      <el-table-column type="selection" align="center"></el-table-column>
      <el-table-column label="ID" align="center" prop="id"></el-table-column>
      <el-table-column
          label="职位名称"
          align="center"
          show-overflow-tooltip
          prop="roleName"
      ></el-table-column>
      <el-table-column
          label="创建时间"
          align="center"
          show-overflow-tooltip
          prop="createTime"
      ></el-table-column>
      <el-table-column
          label="更新时间"
          align="center"
          show-overflow-tooltip
          prop="updateTime"
      ></el-table-column>
      <el-table-column label="操作" width="280px" align="center">
        <template #="{ row, $index }">
          <el-button
              type="primary"
              size="small"
              icon="Edit"
              @click="updateRole(row)"
          >
            编辑
          </el-button>
          <el-popconfirm
              :title="`你确定要删除${row.rowName}?`"
              width="260px"
              @confirm="removeRole(row.id)"
          >
            <template #reference>
              <el-button type="danger" size="small" icon="Delete">
                删除
              </el-button>
            </template>
          </el-popconfirm>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
        v-model:current-page="pageNo"
        v-model:page-size="pageSize"
        :page-sizes="[10, 20, 30, 40]"
        :background="true"
        layout="prev, pager, next, jumper , ->, sizes, total, "
        :total="total"
        @current-change="getHasRole"
        @size-change="sizeHandler"
    />
  </el-card>
  <el-dialog
      v-model="dialogVisible"
      :title="RoleParams.id ? '更新职位' : '添加职位'"
  >
    <el-form :model="RoleParams" :rules="rules" ref="form">
      <el-form-item label="职位名称" prop="roleName">
        <el-input
            placeholder="请你输入职位名称"
            v-model="RoleParams.roleName"
        ></el-input>
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button size="default" @click="dialogVisible = false">取消</el-button>
      <el-button type="primary" size="default" @click="save">确定</el-button>
    </template>
  </el-dialog>
</template>
<style lang="scss" scoped>
.form {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>
