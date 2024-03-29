<template>
  <div>
    <el-card>
      <template #header>
        <div class="card-header">
          <span>日常动态</span>
          <el-button
            @click="openDialog"
            :icon="useRenderIcon(EditPen)"
            type="primary"
            text
            >发动态</el-button
          >
        </div>
      </template>
      <el-row style="display: flex; justify-content: space-around">
        <div v-for="item in dynamicList" :key="item.id">
          <el-card
            shadow="always"
            style="width: 355px"
            body-style="height: 220px"
          >
            <template #header>
              <div style="display: flex; align-items: center">
                <el-avatar :size="50" shape="square" :src="avatar" fit="fill" />
                <span style="margin-left: 10px">{{ name }}</span>
              </div>
            </template>
            <p>
              <el-text size="default" type="info">{{ item.content }}</el-text>
            </p>
            <Upload :fileList="item.images" :hiddenDelete="true" />
            <template #footer>
              <div style="display: flex; justify-content: space-between">
                <el-text size="small">发布于 {{ item.create_time }}</el-text>
                <el-popconfirm
                  width="200"
                  :title="`是否删除此条动态 ?`"
                  :icon="useRenderIcon(Warning)"
                  icon-color="#f56c6c"
                  @confirm="deleteBtn(item.id)"
                >
                  <template #reference
                    ><el-button link type="danger" size="small">
                      删除</el-button
                    ></template
                  ></el-popconfirm
                >
              </div>
            </template>
          </el-card>
        </div>
      </el-row>
      <template #footer>
        <el-pagination
          v-model:current-page="queryParams.currentPage"
          v-model:page-size="queryParams.pageSize"
          :page-sizes="[6, 9, 15]"
          :small="true"
          background
          layout="->,total, sizes, prev, pager, next,jumper"
          :total="total"
          @size-change="getDynamicList"
          @current-change="getDynamicList"
        />
      </template>
    </el-card>
    <el-dialog
      title="发表日常"
      v-model="dialogVisible"
      width="30%"
      :before-close="closeDialog"
    >
      <el-form :inline="false" :model="dynamicForm" ref="dynamicFormRef">
        <el-form-item
          label="内容"
          prop="content"
          :rules="[
            {
              required: true,
              message: '内容不能为空 !',
              trigger: 'blur'
            }
          ]"
        >
          <el-input
            type="textarea"
            placeholder="这一刻的想法..."
            v-model="dynamicForm.content"
          />
        </el-form-item>
        <el-form-item
          label="照片"
          prop="images"
          :rules="[
            {
              required: true,
              message: '照片不能为空 !',
              trigger: 'blur'
            }
          ]"
        >
          <Upload
            multiple
            :limit="2"
            :fileList="imageList"
            @getFileList="getFile"
            :fileSize="4"
          />
        </el-form-item>
      </el-form>

      <template #footer>
        <div>
          <el-button @click="closeDialog">取消</el-button>
          <el-button type="primary" @click="submit(dynamicFormRef)"
            >确认</el-button
          >
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, reactive } from "vue";
import Upload from "@/components/ReUpload/index.vue";
import { useRenderIcon } from "@/components/ReIcon/src/hooks";
import EditPen from "@iconify-icons/ep/edit-pen";
import Warning from "@iconify-icons/ep/warning";
import { message } from "@/utils/message";
import { getDynamic, addDynamic, deleteDynamic } from "@/api/daily";
import type { DynamicInfo } from "@/api/daily/type";
import { uploadFileList } from "@/utils/upload";
import type { FormInstance, UploadUserFile } from "element-plus";
const dynamicForm = reactive<DynamicInfo>({
  content: "",
  images: []
});
const dynamicFormRef = ref<FormInstance>();
const imageList = ref<UploadUserFile[]>([]);
const dialogVisible = ref<boolean>(false);
const dynamicList = ref<DynamicInfo[]>([]);
const avatar = ref<string>();
const name = ref<string>();
const total = ref<number>();
const queryParams = reactive<any>({
  currentPage: 1,
  pageSize: 6
});
onMounted(() => {
  getDynamicList();
});
// 获取动态
const getDynamicList = () => {
  getDynamic(queryParams).then(response => {
    dynamicList.value = response.data.dynamicList;
    avatar.value = response.data.avatar;
    name.value = response.data.username;
    total.value = response.data.total;
  });
};
const openDialog = () => {
  dialogVisible.value = true;
};
const closeDialog = () => {
  dialogVisible.value = false;
  dynamicFormRef.value.resetFields();
  imageList.value = [];
};
const getFile = fileList => {
  imageList.value = fileList;
};
// 添加提交
const submit = async (formEl: FormInstance | undefined) => {
  if (!formEl) return;
  if (imageList.value.length != 0 && dynamicForm.content != "") {
    // 上传照片
    await uploadFileList(imageList.value).then(response => {
      dynamicForm.images = response;
    });
  }
  formEl.validate(async (valid, fields) => {
    if (valid) {
      addDynamic(dynamicForm).then(response => {
        if (response.code == 200) {
          message("发布成功", { type: "success" });
          getDynamicList();
        } else {
          message(response.message, { type: "error" });
        }
        closeDialog();
      });
    } else {
      return fields;
    }
  });
};
// 删除
const deleteBtn = (id: number) => {
  deleteDynamic({ id: id }).then(response => {
    if (response.code == 200) {
      message("删除成功", { type: "success" });
      getDynamicList();
    } else {
      message(response.message, { type: "error" });
    }
    closeDialog();
  });
};
</script>

<style scoped>
.el-card {
  margin: 5px;
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
/* .dialog {
  width: 30% !important;
}
@media screen and (max-width: 750px) {
  .el-dialog {
    --el-dialog-width: 100vw;
    margin: 0 auto;
  }
} */
</style>
