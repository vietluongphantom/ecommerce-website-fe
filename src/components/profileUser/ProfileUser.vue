<template>
  <div class="flex justify-center p-8">
    <div class="w-[70%] bg-[#ffff] flex rounded-md item overflow-hidden">
      <div class="gradient-custom w-[30%] flex flex-col items-center justify-center">
        <img v-if="formData.avatar == null"src="@/assets/images/user_1.png" class="w-[60px] h-[60px] object-cover rounded-full border-[3px] border-[#69C3A3]" />
        <img v-if="formData.avatar != null" class="w-[60px] h-[60px] object-cover rounded-full border-[3px] border-[#69C3A3]" :src="formData.avatar"/>
        <p class="mt-4 text-[22px] font-medium">Hồ sơ của tôi</p>
      </div>

      <div class="w-[70%] p-10 item">
        <h3 class="text-[20px] font-medium mb-1">Thông tin</h3>
        <hr class="border-none bg-[#8A8A8A] h-[2px]" />

        <div class="mt-6 mb-8">
          <form @submit.prevent="handleSubmit">
            <div style="text-align: left; font-size: 14px; margin-bottom: 10px">
              <label for="avatar" style="display: block; font-weight: bold; margin-bottom: 5px">Chọn hình ảnh:</label>
              <input
                type="file"
                id="avatar"
                class="swal2-file"
                @change="handleFileUpload"
                accept="image/*"
                style="width: 100%; padding: 5px; font-size: 14px; border-radius: 5px; border: 1px solid #ddd"
              />
            </div>

            <div class="flex justify-between">
              <div class="form-group w-[46%]">
                <label for="full_name">Họ tên</label>
                <input class="w-full" type="text" id="full_name" v-model="formData.full_name" required />
              </div>

              <div class="form-group w-[46%]">
                <label for="email">Email</label>
                <input class="w-full" type="email" id="email" v-model="formData.email" required readonly />
              </div>
            </div>

            <div class="flex justify-between">
              <div class="form-group w-[46%]">
                <label for="phone">Số điện thoại</label>
                <input class="w-full" type="text" id="phone" v-model="formData.phone" required />
              </div>

              <div class="form-group w-[46%]">
                <label for="gender">Giới tính</label>
                <select class="w-full" id="gender" v-model="formData.gender" required>
                  <option value="MALE">Nam</option>
                  <option value="FEMALE">Nữ</option>
                  <option value="OTHER">Khác</option>
                </select>
              </div>
            </div>

            <div class="flex justify-between">
              <div class="form-group w-[46%]">
                <label for="country">Quốc gia</label>
                <input class="w-full" type="text" id="country" v-model="formData.country" required />
              </div>
              <div class="form-group w-[46%]">
                <label for="province">Tỉnh / Thành phố</label>
                <input class="w-full" type="text" id="province" v-model="formData.province" required />
              </div>
            </div>

            <div class="flex justify-between">
              <div class="form-group w-[46%]">
                <label for="district">Huyện / Quận</label>
                <input class="w-full" type="text" id="district" v-model="formData.district" required />
              </div>
              <div class="form-group w-[46%]">
                <label for="commune">Xã</label>
                <input class="w-full" type="text" id="commune" v-model="formData.commune" required />
              </div>
            </div>

            <div class="form-group w-full">
              <label for="address_detail">Địa chỉ chi tiết</label>
              <textarea class="w-full" type="text" id="address_detail" v-model="formData.address_detail" required />
            </div>

            <div class="w-full justify-between flex">
              <button
                @click="handleSubmit"
                class="bg-[#EF8076] text-[#fff] font-medium text-[18px] p-6 pt-2 pb-2 rounded-lg mt-8 mb-[-20px]"
                type="submit"
              >
                Cập nhật
              </button>
              <button
                @click="handleBack"
                class="text-[#EF8076] font-medium text-[18px] p-6 pt-2 pb-2 rounded-lg mt-8 mb-[-20px] underline"
                type="submit"
              >
                Trang chủ
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue';
import apiServices from '@/domain/apiServices';
import Swal from 'sweetalert2';
import router from '@/router/index.js';
import { useImageStore } from '@/stores/imageStore';





const imageStore = useImageStore();


const selectedFile = ref(null);


const formData = ref({
  email: '',
  phone: '',
  gender: 'FEMALE',
  country: '',
  province: '',
  district: '',
  avatar:'',
  commune: '',
  full_name: '',
  address_detail: ''
});

const firstSlove = async () => {
  const response = await apiServices.getInfoUser();
  formData.value = response.data.data;
};

const handleSubmit = async () => {
  const response = await apiServices.updateProfileUser(formData.value);
  if (response.data.code === 200) {
    router.push({ name: 'Home user' });
    Swal.fire({
      position: 'top-end',
      icon: 'success',
      title: 'Cập nhật thành công!',
      showConfirmButton: false,
      timer: 1500
    });
  }
};

const handleBack = async () => {
  router.push({ name: 'Home user' });
};



const handleFileUpload = async(event) => {
  const file = event.target.files[0];
  if (file) {
    selectedFile.value = file;

    await imageStore.upLoadImage([file]);
    // formData.value.avatar
    // console.log("imageStore.image", imageStore.image)
    formData.value.avatar = imageStore.image
    // console.log("userInfo.value.avatar", userInfo.value)
  }
};


onMounted(() => {
  firstSlove();
});
</script>

<style lang="scss" scoped>
@import './style.scss';
</style>
