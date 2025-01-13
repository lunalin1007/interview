<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <q-form ref="formRef" @submit="onSubmit" class="q-mb-xl">
        <q-input
          v-model="tempData.name"
          label="姓名"
          lazy-rules
          :rules="nameRules"
        />
        <q-input
          v-model="tempData.age"
          label="年齡"
          lazy-rules
          :rules="ageRules"
        />
        <q-btn
          :label="isEditing ? '修改' : '新增'"
          type="submit"
          color="primary"
          class="q-mt-md"
        />
      </q-form>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps, QForm } from 'quasar';
import { onMounted, ref } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
interface BlockData {
  id?: string;
  name: string;
  age: number;
}
const formRef = ref<InstanceType<typeof QForm> | null>(null);
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);
const tempData = ref({
  name: '',
  age: '',
});

const isEditing = ref(false);
const currentEditId = ref<string | null>(null);
const nameRules = [(val: string | number) => !!val || '請輸入姓名'];

const ageRules = [
  (val: string | number) => !!val || '請輸入年齡',
  (val: string | number) => Number.isInteger(Number(val)) || '請輸入整數',
];
function handleClickOption(btn: btnType, data: BlockData) {
  const { id, name, age } = data;
  if (btn.status === 'edit') {
    isEditing.value = true;
    tempData.value = { name, age: age.toString() };

    if (id) {
      currentEditId.value = id;
    }
  }
  if (btn.status === 'delete') {
    if (!data.id) return;

    deleteData(data.id);
  }
}
async function fetchData() {
  axios
    .get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
    .then((response) => {
      blockData.value = response.data;
    })
    .catch((error) => {
      console.log(error);
    });
}
function addData(blockData: BlockData) {
  axios
    .post(' https://dahua.metcfire.com.tw/api/CRUDTest', blockData)
    .then(() => {
      fetchData();
      resetForm();
    })
    .catch((error) => {
      console.log(error);
    });
}
function updateData(blockData: BlockData) {
  axios
    .patch('https://dahua.metcfire.com.tw/api/CRUDTest', blockData)
    .then(() => {
      fetchData();
      resetForm();
    })
    .catch((error) => {
      console.log(error);
    });
}
function deleteData(id: string) {
  axios
    .delete(` https://dahua.metcfire.com.tw/api/CRUDTest/${id}`)
    .then(() => {
      fetchData();
    })
    .catch((error) => {
      console.log(error);
    });
}
async function resetForm() {
  formRef.value?.resetValidation();

  tempData.value = {
    name: '',
    age: '',
  };
}
function onSubmit() {
  if (isEditing.value) {
    if (!currentEditId.value) return;
    const { name, age } = tempData.value;
    const newData = {
      id: currentEditId.value,
      name,
      age: parseInt(age) || 0,
    };
    updateData(newData);
    isEditing.value = false;
    return;
  } else {
    const { name, age } = tempData.value;
    const newData = {
      name,
      age: parseInt(age) || 0,
    };
    addData(newData);
  }
}
onMounted(() => {
  fetchData();
});
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
