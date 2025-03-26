<script>
import { bitable } from '@lark-base-open/js-sdk';
import { ref, onMounted } from 'vue';
import {
    ElButton,
    ElForm,
    ElFormItem,
    ElSelect,
    ElOption,
    ElMessage
} from 'element-plus';

// 假数据生成函数
const generateFakeData = () => ({
    fields: {
        '文本': Number((Math.random() * 10000).toFixed(2)), // 随机生成 0-10000 的金额，保留两位小数
        '单选': Math.floor(Math.random() * 10) + 1, // 随机生成 1-10 的频度
        '日期': new Date().toISOString(), // 当前日期，ISO 格式
        '附件': '' // 附件字段留空（如果需要上传文件，可以后续扩展）
    }
});

export default {
    components: {
        ElButton,
        ElForm,
        ElFormItem,
        ElSelect,
        ElOption,
    },
    setup() {
        const formData = ref({ table: '' });
        const tableMetaList = ref([]);

        const addRecord = async () => {
            const tableId = formData.value.table;
            if (!tableId) {
                ElMessage.warning('请先选择数据表');
                return;
            }
            try {
                const table = await bitable.base.getTableById(tableId);
                await table.addRecord({
                    fields: {
                        '文本': Number((Math.random() * 10000).toFixed(2)), // 随机生成 0-10000 的金额，保留两位小数
                        '单选': Math.floor(Math.random() * 10) + 1, // 随机生成 1-10 的频度
                        '日期': new Date().toISOString(), // 当前日期，ISO 格式
                        '附件': '' // 附件字段留空（如果需要上传文件，可以后续扩展）
                    }
                });
                ElMessage.success('成功添加记录');
            } catch (error) {
                ElMessage.error('添加记录失败');
                console.error(error);
            }
        };

        onMounted(async () => {
            const [tableList, selection] = await Promise.all([bitable.base.getTableMetaList(), bitable.base.getSelection()]);
            formData.value.table = selection.tableId;
            tableMetaList.value = tableList;
        });

        return {
            formData,
            tableMetaList,
            addRecord,
        };
    },
};
</script>

<template>
    <el-form ref="form" class="form" :model="formData" label-position="top">
        <el-form-item label="选择数据表" size="large">
            <el-select v-model="formData.table" placeholder="请选择数据表" style="width: 100%">
                <el-option v-for="meta in tableMetaList" :key="meta.id" :label="meta.name" :value="meta.id" />
            </el-select>
        </el-form-item>
        <el-button type="primary" plain size="large" @click="addRecord">新增一行记录</el-button>
    </el-form>
</template>

<style scoped>
.form :deep(.el-form-item__label) {
    font-size: 16px;
    color: var(--el-text-color-primary);
    margin-bottom: 0;
}

.form :deep(.el-form-item__content) {
    font-size: 16px;
}
</style>
