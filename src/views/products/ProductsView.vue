<script lang="ts" setup>
import { ref, nextTick, onMounted } from 'vue'
import { useProductStore } from '@/stores/productStore'
import { useCartStore } from '@/stores/cartStore'
import { storeToRefs } from 'pinia'
import { useNavigateToProduct } from '@/composables/useNavigateToProduct'
import BaseButton from '@/components/base/BaseButton.vue'


const cartStore = useCartStore()
const productStore = useProductStore()

const {products} = storeToRefs(productStore)
const {navigateToProduct} = useNavigateToProduct()


onMounted(async () => {
  await productStore.fetchProducts();
});

const normalizeTitle = (title: string) => {
  return title.split(' ').slice(0, 4).join(' ')
}

const addToCartAnimation = async (event: MouseEvent, product: any) => {
  event.stopPropagation()
  const productImg = (event.currentTarget as HTMLElement)
    .closest('div')
    ?.querySelector('img')

  if (!productImg) return

  const clonedImg = productImg.cloneNode(true) as HTMLElement
  clonedImg.style.position = 'fixed'
  clonedImg.style.zIndex = '1000'
  clonedImg.style.width = '80px'
  clonedImg.style.height = '80px'
  clonedImg.style.borderRadius = '50%'
  clonedImg.style.transition = 'transform 1.5s cubic-bezier(0.42, 0, 0.58, 1), opacity 1.2s ease-in-out'
  clonedImg.style.opacity = '1'
  clonedImg.style.pointerEvents = 'none'

  document.body.appendChild(clonedImg)

  const rect = productImg.getBoundingClientRect()
  clonedImg.style.left = `${rect.left}px`
  clonedImg.style.top = `${rect.top}px`

  await nextTick()

  const cartX = window.innerWidth - 80
  const cartY = 50

  clonedImg.style.transform = `translate(${cartX - rect.left}px, ${cartY - rect.top}px) scale(0.5)`
  clonedImg.style.opacity = '0.7'

  setTimeout(() => {
    clonedImg.style.transform = `translate(${cartX - rect.left}px, ${cartY - rect.top}px) scale(0)`
    clonedImg.style.opacity = '0'
  }, 1000)
  setTimeout(() => {
    document.body.removeChild(clonedImg)

  }, 1500)
  cartStore.addToCart(product)
}
</script>

<template>
  <div class="p-6 animate-fade-id-view">
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
      <div
        v-for="product in products"
        :key="product.id"
        class="p-4 bg-white shadow-lg rounded-xl relative"
        @click="navigateToProduct(product)"
      >
        <img
          :src="product.image"
          alt="product-image"
          class="cursor-pointer w-full h-80 object-contain rounded-lg mb-4"
          draggable="false"
        />
        <h2 class="text-lg font-poppins text-gray-500 dark:text-black">
          {{ normalizeTitle(product.title) }}
        </h2>
        <p class="text-gray-500 font-poppins text-xl font-semibold py-3">$ {{ product.price }}</p>

        <BaseButton
          label="Add To Cart"
          :grow="false"
          btn-class="text-base"
          @click="(event) => addToCartAnimation(event, product)"
        />
      </div>
    </div>
  </div>
</template>
