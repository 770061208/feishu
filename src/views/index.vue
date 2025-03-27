<script>
import { bitable, FieldType } from '@lark-base-open/js-sdk';
import { ref, onMounted } from 'vue';
import {
    ElButton,
    ElForm,
    ElFormItem,
    ElSelect,
    ElOption,
} from 'element-plus';

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
            if (tableId) {
                const table = await bitable.base.getTableById(tableId);

                const textField = await table.getField('文本'); // 选择某个多行文本字段
                const dateField = await table.getField('日期');
                const radioField = await table.getField('单选');

                // 处理单选字段
                const radioOptions = await radioField.getOptions();
                const randomRadioOption = radioOptions.length > 0
                    ? radioOptions[Math.floor(Math.random() * radioOptions.length)].id
                    : null;

                // 处理日期格式（改为 YYYY-MM-DD）
                const date = new Date();
                const formattedDate = `${date.getFullYear()}-${(date.getMonth() + 1).toString().padStart(2, '0')}-${date.getDate().toString().padStart(2, '0')}`;

                table.addRecord({
                    fields: {
                        [textField.id]: `随机文本 ${Math.random().toFixed(2)}`, // 生成随机文本
                        [radioField.id]: { id: randomRadioOption },
                        [dateField.id]: formattedDate, // 当前时间
                    }
                });
            }
        };

        onMounted(async () => {
            const [tableList, selection] = await Promise.all([bitable.base.getTableMetaList(), bitable.base.getSelection()]);
            formData.value.table = selection.tableId;
            tableMetaList.value = tableList;

            const table = await bitable.base.getActiveTable();
            const attachmentFieldList = await table.getFieldListByType(FieldType.Attachment);
            console.log('attachmentFieldList', attachmentFieldList);
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
        <el-form-item label="开发指南">
            <a href="https://bytedance.feishu.cn/docx/HazFdSHH9ofRGKx8424cwzLlnZc" target="_blank"
                rel="noopener noreferrer">
                多维表格插件开发指南
            </a>
            、
            <a href="https://lark-technologies.larksuite.com/docx/HvCbdSzXNowzMmxWgXsuB2Ngs7d" target="_blank"
                rel="noopener noreferrer">
                Base Extensions Guide
            </a>
        </el-form-item>
        <el-form-item label="API">
            <a href="https://bytedance.feishu.cn/docx/HjCEd1sPzoVnxIxF3LrcKnepnUf" target="_blank"
                rel="noopener noreferrer">
                多维表格插件API
            </a>
            、
            <a href="https://lark-technologies.larksuite.com/docx/Y6IcdywRXoTYSOxKwWvuLK09sFe" target="_blank"
                rel="noopener noreferrer">
                Base Extensions Front-end API
            </a>
        </el-form-item>
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
