<template>
  <div class="supply bg-[#EFEFEF] p-9">
    <h1 class="text-[25px] font-bold mb-6">Danh sách nhập kho</h1>

    <div class="mb-8 flex justify-between">
      <div class="flex items-center mr-[50px]">
        <input
          class="category__input w-[200px] h-[35px] rounded-lg mr-2 p-3"
          placeholder="Tên..."
          @focus="handleFocus"
          v-model="searchName"
          @keyup.enter="handleAction"
        />
        <!-- <SearchIcon @click="handleAction" class="w-[20px] h-[20px]"></SearchIcon> -->
      </div>

      <div class="flex items-center">
        <input
          class="category__input w-[200px] h-[35px] rounded-lg mr-2 p-3"
          placeholder="Mã..."
          @focus="handleFocus"
          v-model="searchId"
          @keyup.enter="handleAction"
        />
      </div>

      <button @click="handleSearch" class="button text-[15px] ml-[80px] text-[#fff] bg-[#69C3A3] p-2 font-medium rounded-lg px-4">Tìm kiếm</button>

      <div class="flex">
        <button @click="handleExcel" class="flex justify-center w-[140px] h-[40px] items-center text-[16px] bg-[#27932c] text-[#fff] p-4 rounded-md mr-3">
          <span class="mr-2">xuất excel</span>
          <!-- <AddIcon class="w-[20px] h-[20px]"></AddIcon> -->
        </button>
      </div>
    </div>


    <div>
      <a-table
        :columns="columns"
        :data-source="exportData.dataSource"
        :pagination="{
          total: exportData.totalElements,
          current: exportData.currentPage,
          pageSize: exportData.pageSize
        }"
        @change="handleTableChange"
      >
        <template #bodyCell="{ column, record }">
          <span v-if="column.key === 'create_at'">
            {{ record[column.dataIndex] ? format(record[column.dataIndex], 'dd/MM/yyyy') : 'N/A' }}
          </span>
          <span v-else>
            {{ record[column.dataIndex] }}
          </span>
        </template>
      </a-table>
    </div>
  </div>
</template>

<script setup>
import { reactive, onMounted, ref } from 'vue';
import apiServices from '@/domain/apiServices';
import { format } from 'date-fns';
import _ from 'lodash';
const searchName = ref('');
const searchId = ref('');

const columns = [
  { title: 'STT', dataIndex: 'stt', key: 'stt' },
  // { title: 'Kho hàng', dataIndex: 'warehouse', key: 'warehouse' },
  { title: 'Nhà cung cấp', dataIndex: 'supplier', key: 'supplier' },
  { title: 'Tên sản phẩm', dataIndex: 'name', key: 'name' },
  { title: 'Mã sản phẩm', dataIndex: 'sku_code', key: 'sku_code' },
  { title: 'Vị trí', dataIndex: 'location', key: 'location' },
  { title: 'Số lượng', dataIndex: 'quantity', key: 'quantity' },
  { title: 'Ngày tạo', dataIndex: 'create_at', key: 'create_at' }
];

const exportData = reactive({
  dataSource: [],
  totalElements: 0,
  currentPage: 1,
  pageSize: 10
});

const fetchAPI = async (page = 1, name = '', id = '') => {
  try {
    const response = await apiServices.getAllImport(page, exportData.pageSize, name, id);
    const reversedData = response.data.data.content;

    exportData.dataSource = _.map(reversedData, (item, index) => ({
      ...item,
      stt: (page - 1) * exportData.pageSize + index + 1
    }));
    exportData.totalElements = response.data.data.totalElements;
    exportData.currentPage = page;
  } catch (error) {
    console.error('Error fetching warehouses:', error);
  }
};

const handleTableChange = (pagination) => {
  exportData.currentPage = pagination.current;
  fetchAPI(exportData.currentPage, searchName.value, searchId.value);
};

const handleAction = () => {
  exportData.currentPage = 1;
  fetchAPI(exportData.currentPage, searchName.value, searchId.value);
};

const handleSearch = () => {
  exportData.currentPage = 1;
  fetchAPI(exportData.currentPage, searchName.value, searchId.value);
};


const handleExcel = async () => {
  try {
    // Gọi API để lấy file Excel
    const response = await apiServices.handleExcelImport();

    // Kiểm tra xem response có dữ liệu hay không
    if (response && response.data) {
      const blob = new Blob([response.data], {
        type: response.headers['content-type'] || 'application/vnd.ms-excel',
      });

      // Tạo URL từ Blob
      const url = window.URL.createObjectURL(blob);

      // Tạo thẻ <a> ẩn để tải file
      const a = document.createElement('a');
      a.href = url;
      a.download = 'list-import.xlsx'; // Tên file khi tải xuống
      document.body.appendChild(a);
      a.click();

      // Dọn dẹp tài nguyên
      window.URL.revokeObjectURL(url);
      document.body.removeChild(a);
    }
  } catch (error) {
    console.error('Lỗi khi tải file Excel:', error);
  }
};

onMounted(async () => {
  fetchAPI();
});
</script>

<style scoped lang="scss">
.supply {
  box-shadow: rgba(100, 100, 111, 0.2) 0px 7px 29px 0px;
  border-radius: 10px;
  overflow: hidden;
}

.button {
  &:hover {
    background-color: #fff;
    color: #63b899;
    box-shadow:
      0 0 10px #63b899,
      0 0 20px #63b899;
  }
}
</style>
