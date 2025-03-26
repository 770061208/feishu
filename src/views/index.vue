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
const generateFakeData = async (tableId) => {
    const table = await bitable.base.getTableById(tableId);
    const fieldMetaList = await table.getFieldMetaList();

    const fields = {};

    for (const field of fieldMetaList) {
        switch (field.name) {
            case '文本':
                fields['文本'] = String(Number((Math.random() * 10000).toFixed(2)));
                break;
            case '单选':
                if (field.type === 3) { // 单选字段
                    const options = field.property.options || [];
                    if (options.length > 0) {
                        const randomOption = options[Math.floor(Math.random() * options.length)];
                        fields['单选'] = randomOption.name;
                    } else {
                        fields['单选'] = '未知';
                    }
                }
                break;
            case '日期':
                fields['日期'] = new Date().toISOString();
                break;
            case '附件':
                fields['附件'] = [];
                break;
            default:
                break;
        }
    }

    return { fields };
};

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
                const fakeData = await generateFakeData(tableId);
                const recordId = await table.addRecord(fakeData);
                // 验证记录是否真的被添加
                const record = await table.getRecordById(recordId);
                if (record) {
                    ElMessage.success('成功添加记录');
                } else {
                    ElMessage.error('记录添加失败，可能没有权限');
                }
            } catch (error) {
                ElMessage.error('添加记录失败: ' + error.message);
                console.error(error);
            }
        };

        onMounted(async () => {
            try {
                const [tableList, selection] = await Promise.all([
                    bitable.base.getTableMetaList(),
                    bitable.base.getSelection()
                ]);
                formData.value.table = selection.tableId || '';
                tableMetaList.value = tableList;
            } catch (error) {
                ElMessage.error('加载数据表失败: ' + error.message);
                console.error(error);
            }
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
    <div class="container">
        <el-form ref="form" :model="formData" label-position="top" class="modern-form">
            <div class="form-content">
                <el-form-item label="选择数据表" prop="table" class="form-item">
                    <el-select v-model="formData.table" placeholder="请选择数据表" size="large" clearable
                        class="select-input">
                        <el-option v-for="meta in tableMetaList" :key="meta.id" :label="meta.name" :value="meta.id" />
                    </el-select>
                </el-form-item>
                <el-form-item class="button-item">
                    <el-button type="primary" size="large" @click="addRecord" :disabled="!formData.table"
                        class="submit-button">
                        新增一行记录
                    </el-button>
                </el-form-item>
            </div>
        </el-form>
    </div>
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
