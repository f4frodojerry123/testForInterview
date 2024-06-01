<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <q-form
        class="q-mb-xl"
      >
        <q-input 
          v-model="tempData.name" label="姓名"
          lazy-rules
          :rules="[ val => val && val.length > 0 || '請輸入姓名']"
        />
        <q-input 
          v-model="tempData.age"
          label="年齡"
          type="number"
          lazy-rules
          :rules="[ val => val && val.length > 0 || '請輸入年齡']"
        />
        <q-btn v-if="actionType === 'add'" @click="update()" color="primary" class="q-mt-md" type="submit">新增</q-btn>
        <div v-else class="edit-btn">
          <q-btn @click="edit()" color="primary" class="q-mt-md" type="submit">更新</q-btn>
          <q-btn @click="changeActionType('add')" color="secondary" class="q-mt-md" type="reset">取消</q-btn>
        </div>
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
import { QTableProps, Dialog } from 'quasar';
import { onBeforeMount, ref } from 'vue';
import { useQuasar } from 'quasar'

const $q = useQuasar()
interface btnType {
  label: string;
  icon: string;
  status: string;
}

interface parmasType {
  id: string;
  name: string;
  age: number;
}
const blockData = ref([]);
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
    sortable: true,
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

const actionType = ref('add');
const editDataId = ref('')

const tempData = ref({
  name: '',
  age: '',
});

const getData = async() => {
  axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a').then((response) => {
    if(response.status === 200) {
      blockData.value = response.data;
    }
  })
}

const update = async() => {
  if(tempData.value.name && tempData.value.age) {
    const params = {
      name: tempData.value.name,
      age: +tempData.value.age,
    }
    axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', params).then((response) => {
      if(response.status === 200) {
       getData();
      }
    })
  }
}

const deleteData = (params: parmasType) => {
  axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${params.id}`).then((response) => {
    if(response.status === 200) {
      getData();
    }
  })
}

const edit = () => {
  // $q.dialog({
  //       title: 'Alert',
  //       message: '確定要更新嗎?'
  //     }).onOk(() => {
  //       
  //     }).onCancel(() => {
  //       
  //     }).onDismiss(() => {
  //       
  //     })
  if(tempData.value.name && tempData.value.age) {
  const params = { 
    id: editDataId.value,
    name: tempData.value.name,
    age: +tempData.value.age,
  }
  axios.patch('https://dahua.metcfire.com.tw/api/CRUDTest', params).then((response) => {
    if(response.status === 200) {
      getData();
    }
  })
  }

}

const changeActionType = (type: string, params?: parmasType) => {
  actionType.value = type;
  if (type === 'edit') {
    editDataId.value = params?.id || '';
    Object.assign(tempData.value, params)
  } else {
    tempData.value.name = '';
    tempData.value.age = '';
  }
}

const handleClickOption = (btn: any, data: parmasType) => {
  switch (btn.status) {
    case 'edit':
      changeActionType('edit', data);
      break;
    case 'delete':
      deleteData(data);
    default:
      break;
  }
}

onBeforeMount(async() => {
  await getData();
})
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}

.edit-btn {
  display: flex;
  .q-mt-md {
    margin-right: 10px;
  }
}
</style>
