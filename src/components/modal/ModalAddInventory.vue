<template>
  <div>
    <div>
        <a-button @click="showModal" class="flex w-[140px] h-[40px] items-center text-[16px] bg-[#0397D6] text-[#fff] p-4 rounded-md mr-3">
          <span class="mr-2">Nhập hàng</span>
          <AddIcon class="w-[20px] h-[20px]"></AddIcon>
        </a-button>
    </div>
    <a-modal v-model:open="open" title="Basic Modal" @ok="handleOk">
        <p>Chọn nhà kho</p>
        <a-select
            v-model:value="inventory.warehouse_id"
            show-search
            placeholder="Chọn giá trị"
            style="width: 100%"
            :options="name"
            :filter-option="(input, option) => option.label.toLowerCase().includes(input.toLowerCase())"
            @change="handleChangeWarehouse"
        ></a-select>
        <p>Chọn sản phẩm</p>
        <a-select
            v-model:value="inventory.product_id"
            show-search
            placeholder="Chọn giá trị"
            style="width: 100%"
            :options="product"
            :filter-option="(input, option) => option.label.toLowerCase().includes(input.toLowerCase())"
            @change="handleChangeProduct"
            @search="debouncedFetchBrand"
        ></a-select>
        <p>Chọn mã sản phẩm</p>
        <a-select
            v-model:value="inventory.sku_code"
            show-search
            placeholder="Chọn giá trị"
            style="width: 100%"
            :options="productItem"
            :filter-option="(input, option) => option.label.toLowerCase().includes(input.toLowerCase())"
            @change="handleChangeWareProductItem"
        ></a-select>
      <p>Số lượng</p>
      <a-input v-model:value="inventory.quantity" placeholder="nhập số lượng..." />
      <p>Nhà cung cấp</p>
      <a-input v-model:value="inventory.supplier" placeholder="nhập nhà cung cấp..." />
      <p>Vị trí</p>
      <a-input v-model:value="inventory.location"  placeholder="Nhập vị trí để hàng..." />
    </a-modal>
  </div>
</template>

<script setup>
import { ref , reactive, onMounted  } from 'vue';
import { SearchIcon, AddIcon, EditIcon, TrashIcon } from '@/assets/icons/icon.js';
import { useWarehouseStore } from '@/stores/warehouseStore';
import { useProductStore } from '@/stores/productSellerStore';
import { useProductItemStore } from '@/stores/productItemStore';
import {useInventoryStore} from '@/stores/inventoryStore';
import debounce from 'lodash/debounce';
import apiServices from '@/domain/apiServices';

const inventoryStore = useInventoryStore()
const productStore = useProductStore()
const warehouseStore = useWarehouseStore()
const productItemStore = useProductItemStore()
const name = ref()
const product = ref()
const productItem = ref()
const inventory = reactive({
    warehouse_id: '',
    supplier: '',
    location: '',
    product_id:'',
    quantity: '',
    sku_code: ''
});

const open = ref(false);

const showModal = () => {
  open.value = true;
};

const handleOk = (e) => {
    inventoryStore.addInventory(inventory);
    open.value = false;
};

const handleChangeWarehouse = (id) => {
  inventory.warehouse_id = id;
};
const handleChangeWareProductItem = (id) => {
  inventory.sku_code = id;
};

const handleChangeProduct = async (id) => {
    inventory.product_id = id;
    await productItemStore.getListProductItemByProductId(id);
    console.log("Anbcbđb dddd",productItemStore.productItems )
    productItem.value = productItemStore.productItems.map((productItem) => ({
        value: productItem.sku_code,
        label: productItem.sku_code
    }))

};

const debouncedFetchBrand = debounce((searchText) => {
  fetchProducts(searchText);
}, 500);


const fetchProducts = async (searchText="") => {
  const response = await apiServices.getListProductS(1, 10, searchText);
  console.log("có vào kl")
  product.value = response.data.data.productResponses.map((product) => ({
    value: product.id,
    label: product.name
  }));
};
// const debouncedFetchWarehouse = debounce((searchText) => {
//     useWarehouseStore.fetchWarehouses(searchText);
// }, 1000);

onMounted(async () => {
  await warehouseStore.fetchWarehouses();
    name.value = warehouseStore.warehouses.map((warehouse) => ({
    value: warehouse.id,
    label: warehouse.name
  }));
   await productStore.fetchProducts();
    product.value = productStore.products.map((product) => ({
        value: product.id,
        label: product.name
    }))
});
</script>

<style scoped lang="scss">
.modal {
  &__border {
    border: 3px solid #9e9e9e;
    outline: none;
    &:focus {
      border: 3px solid #3884E1;
    }
  }
}
// ::v-deep .ant-modal-body{
//   width: 250px!important;
//   padding: 20px
// }
</style>
