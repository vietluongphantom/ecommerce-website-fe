<template>
    <div class="container mx-auto px-4 py-8">

        <h1 class="text-4xl font-bold text-center text-gray-800 mb-8 uppercase tracking-wide">
            Chi Tiết Đơn Hàng
        </h1>


        <div class="bg-white shadow-lg rounded-lg overflow-hidden">

            <div class="p-6 bg-gray-50 border-b">
                <div class="grid md:grid-cols-2 gap-4">
                    <div>
                        <div class="flex items-center mb-2">
                            <span class="font-semibold text-gray-600 mr-2">Mã đơn hàng:</span>
                            <span class="text-blue-600 font-bold">{{ orderDetail?.id }}</span>
                        </div>
                        <div class="flex items-center mb-2">
                            <span class="font-semibold text-gray-600 mr-2">Trạng thái:</span>
                            <span :class="{
                                'text-yellow-600': orderDetail?.status === 'PENDING',
                                'text-green-600': orderDetail?.status === 'COMPLETED',
                                'text-red-600': orderDetail?.status === 'CANCELLED'
                            }" class="font-bold">
                                {{ orderDetail?.status }}
                            </span>
                        </div>
                        <div class="flex items-center">
                            <span class="font-semibold text-gray-600 mr-2">Người mua:</span>
                            <span>{{ orderDetail?.buyer }}</span>
                        </div>
                    </div>
                    <div>
                        <div class="flex items-center mb-2">
                            <span class="font-semibold text-gray-600 mr-2">Địa chỉ:</span>
                            <span>{{ orderDetail?.address }} - {{ orderDetail?.address_detail }}</span>
                        </div>
                        <div class="flex items-center mb-2">
                            <span class="font-semibold text-gray-600 mr-2">Số điện thoại:</span>
                            <span>{{ orderDetail?.receiver_phone }}</span>
                        </div>
                        <div class="flex items-center">
                            <span class="font-semibold text-gray-600 mr-2">Ngày tạo:</span>
                            <span>{{ formatDate(orderDetail?.created_at) }}</span>
                        </div>
                    </div>
                </div>
            </div>


            <div class="p-6">
                <h2 class="text-2xl font-semibold text-gray-800 mb-4">Danh Sách Sản Phẩm</h2>
                <a-table v-if="orderItems.length > 0" :columns="columns" :data-source="orderItems" :pagination="false"
                    class="w-full"
                    @row-click="handleRowClick" :custom-row="(record) => ({
                        class: 'cursor-pointer hover:bg-gray-50',
                        onClick: () => handleRowClick(record)
                    })">
                    <template #bodyCell="{ column, record }">
                        <template v-if="column.dataIndex === 'unitPrice'">
                            <span class="text-blue-600 font-bold">{{ formatCurrency(record.unitPrice) }}</span>
                        </template>

                        <template v-else-if="column.key === 'product_name'">
                            <a @click="handleProductClick(record)">{{ record.productDetails.name }}</a>
                        </template>

                        <template v-else-if="column.key === 'image'">
                            <img :src="record.productDetails.image" alt="Product Image"
                                class="w-[60px] h-[60px] object-cover rounded-md p-1" />
                        </template>

                        <template v-else-if="column.key === 'attributes'">
                            <div>
                                <div v-for="attribute in record.productDetails.list_product_item" :key="attribute.id">
                                    <span class="text-[#8A8A8A] uppercase"> {{ attribute.value }}</span>
                                </div>
                            </div>
                        </template>

                        <template v-if="column.dataIndex === 'total'">
                            <span class="text-green-600 font-bold">{{ formatCurrency(record.unitPrice * record.quantity)
                                }}</span>
                        </template>
                    </template>
                </a-table>


                <a-empty v-else description="Không có sản phẩm trong đơn hàng" />
            </div>



            <div class="bg-gray-100 p-6 text-right">
                <p class="text-2xl font-bold text-red-600">
                    Tổng giá trị: {{ formatCurrency(orderDetail?.total_price || 0) }}
                </p>
            </div>
            <div class="bg-gray-100 p-6 text-right">
                <button v-if="orderDetail?.status !== 'COMPLETED' && orderDetail?.status !== 'CANCELLED' && orderDetail?.status !== 'SHIPPED'&& orderDetail?.status !== 'RETURNED' && orderDetail?.status !== 'PACKED' "
                    @click="handleCancelOrder"
                    class="ml-4 px-4 py-2 bg-red-600 text-white rounded-md hover:bg-red-700 transition duration-200 text-sm font-medium">
                    Hủy đơn hàng
                </button>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useOrderStore } from '@/stores/orderDetailStore';
import { useOrderStatusStore } from '@/stores/orderStatusStore';

const columns = [
    {
        title: 'Mã sản phẩm',
        dataIndex: 'productItemId',
        key: 'productItemId',
    },
    {
        title: 'Sản phẩm',
        dataIndex: ['productDetails', 'name'],
        key: 'product_name',
        // slots: { customRender: 'name' }
    },
    {
        title: 'Hình ảnh',
        key: 'image',
        dataIndex: ['productDetails', 'image']
    },
    {
        title: 'Phân loại',
        key: 'attributes',
        dataIndex: ['productDetails', 'list_product_item']
    },
    {
        title: 'Số lượng',
        dataIndex: 'quantity',
        key: 'quantity',
    },
    {
        title: 'Đơn giá',
        dataIndex: 'unitPrice',
        key: 'unitPrice',
    },
    {
        title: 'Tổng tiền',
        dataIndex: 'total',
        key: 'total',
    },
];

const editOrder = useOrderStatusStore();
const orderStore = useOrderStore();
const route = useRoute();
const router = useRouter();


const orderDetail = computed(() => orderStore.orders);


const orderItems = computed(() => {
    return orderDetail.value?.orderItems || [];
});


const formatCurrency = (value) => {
    return new Intl.NumberFormat('vi-VN', {
        style: 'currency',
        currency: 'VND'
    }).format(value);
};


const formatDate = (dateString) => {
    if (!dateString) return '';
    return new Date(dateString).toLocaleString('vi-VN');
};
const handleCancelOrder = async () => {
    if (orderDetail.value) {
        await editOrder.editOrderByUser(

            orderDetail.value.id,
            orderDetail.value.status
        );
      
        await orderStore.getOrderByUser(route.params.id);
    }
};
// const handleRowClick = (record) => {
//     if (!record?.productDetails?.id) {
//         console.warn('Không tìm thấy ID sản phẩm');
//         return;
//     }

//     const productItemId = record.productDetails.id;
//     const productId = record.productDetails.product_id; // Giả sử có trường productId

//     router.push({
//         name: 'product-detail-user',
//         params: { id: productId }
//     }).catch(error => {
//         console.error('Lỗi khi chuyển trang:', error);
//     });
// };

const handleProductClick= async (e) => {
  let id = e.productDetails.product_id
  router.push({ name: 'product-detail-user', params: { id } });
};


onMounted(async () => {
    try {

        const id = route.params.id;


        await orderStore.getOrderByUser(id);
    } catch (error) {
        console.error('Lỗi khi tải chi tiết đơn hàng:', error);

        router.push({ name: 'list-order' });
    }
});
</script>