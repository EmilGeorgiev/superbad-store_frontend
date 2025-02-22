<template>
  <div class="flex w-full px-5 pt-7 justify-center gap-5">
    <div class="flex w-full flex-col gap-5 max-w-[1200px] max-[1254px]:w-full h-fit rounded-[8px] justify-center py-5">
      <div class="w-full">
        <BreadCrumb :routes="routes" />
      </div>
      <div class="w-full flex flex-col gap-10">
        <!-- thumbnail -->
        <p class="text-3xl font-bold">Checkout(1)</p>
        <div class="flex flex-row-reverse gap-20">
          <div class="shadow-d-20 w-[370px] h-fit rounded-lg border-[1px]">
            <div class="p-5 flex justify-between items-center border-b-[1px]">
              <p class="text-base font-semibold">Order Summary</p>
            </div>
            <div class="p-5 flex flex-col gap-5 mt-3">
              <div class="flex justify-between items-center">
                <p class="text-base font-medium">Subtotal</p>
                <p class="text-base font-semibold">${{ totalSubPrice }}</p>
              </div>
              <div class="flex justify-between items-center">
                <p class="text-base font-medium">Shipping</p>
                <p class="text-base font-semibold">$0</p>
              </div>
              <div class="flex justify-between items-center">
                <p class="text-base font-medium">Tax</p>
                <p class="text-base font-semibold">$0</p>
              </div>
            </div>
            <div class="p-5 py-3 border-t-[1px] flex justify-between items-center mt-3">
              <p class="text-base font-medium">Total</p>
              <p class="text-xl font-bold">${{ totalSubPrice }}</p>
            </div>
            <div class="p-5 pt-2">
              <AButton
                title="checkout"
                class="text-white text-lg font-medium flex justify-center py-3"
                @click="checkout"
              />
            </div>
          </div>
          <div class="flex-auto flex flex-col gap-7">
            <div v-if="product">
              <div class="flex gap-4">
                <!-- <input
                    :id="item.id"
                    v-model="selectItems"
                    class="mt-3 rounded-sm w-5 h-5 text-[#ff7050]"
                    type="checkbox"
                    :value="item.id"
                    name=""
                  /> -->
                <img class="w-[80px] h-[80px] rounded-md" :src="typeSelected?.imageUrl" alt="" />
                <div>
                  <RouterLink :to="`/products/${product.id}`" class="font-medium text-base">{{
                    product.name
                  }}</RouterLink>
                  <p>({{ quantity }} items) type: {{ typeSelected.name }}</p>
                  <!-- <ANumberInput @update:modelValue="onAddQuantity" v-model="item.quantity" /> -->
                </div>
                <p class="ml-auto text-base font-semibold">${{ typeSelected.price * quantity }}</p>
              </div>
              <div class="flex justify-between items-center mt-3 pl-7">
                <p class="text-third-100 font-medium">
                  <i class="ri-add-line font-bold"></i>
                  Add notes
                </p>
                <div class="flex items-center gap-4 text-base">
                  <p>
                    <i class="ri-heart-line"></i>
                    Add to white list
                  </p>
                  <span class="h-[20px] border-r-[1px]"></span>
                  <i class="ri-delete-bin-6-line"></i>
                </div>
              </div>
              <!-- line -->
              <div class="border-b-[1px] mt-7"></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref, onBeforeMount, computed } from 'vue'
// components
import AButton from '@/components/commons/atoms/AButton.vue'
import BreadCrumb from '@/components/commons/BreadCrumb.vue'
// services
import { getProductApi } from '@/services/product.service'
import { orderApi } from '@/services/order.service'
// stores
import { usePopupStore } from '@/stores/common.store'
const popupStore = usePopupStore()
import { useAuthStore } from '@/stores/auth.store'
const authStore = useAuthStore()
import { useRoute, useRouter } from 'vue-router'
const route = useRoute()
const router = useRouter()
// breadcrumb
const routes = ref([
  {
    name: 'Home',
    path: '/',
  },
  {
    name: 'Checkout',
    path: '/checkout',
  },
])

const quantity = ref(route.query.quantity)
const typeId = ref(route.query.typeId)

const typeSelected = computed(() => {
  return product.value?.types.find((type) => type.id === typeId.value)
})
onBeforeMount(async () => {
  try {
    if (!route.query.typeId || !route.query.quantity) {
      router.push({ name: 'product-detail', params: { id: route.params.id } })
    }
    const res = await getProductApi(route.params.id)
    product.value = res.data
    console.log(product.value)
  } catch (error) {
    console.log(error)
  }
})

const product = ref(null)

const selectItems = ref([])
const totalSubPrice = computed(() => {
  return typeSelected.value?.price * quantity?.value
})

const checkout = async () => {
  try {
    popupStore.showPopup({
      content: 'Do you want to checkout now?',
      type: 'confirm',
      isAsync: true,
      onConfirm: async () => {
        try {
          if (!authStore.state.isLoggedIn) {
            localStorage.setItem('redirect', route.fullPath)
            router.push({ name: 'login' })
            return
          }
          const method = 'TakeFromCart'
          const body = {
            shippingAddressId: '3fa85f64-5717-4562-b3fc-2c963f66afa6',
            cartItemIds: selectItems.value,
          }
          await orderApi(body, method)
        } catch (error) {
          console.log(error)
          router.push({ name: 'checkout-success', params: { id: route.params.id } })
        }
      },
    })
  } catch (error) {
    console.log(error)
  }
}
</script>
<style scoped></style>
