<template>
  <div class="flex h-screen bg-gray-100">
    <!-- Sidebar -->
    <aside class="w-64 bg-white shadow-md">
      <div class="flex items-center justify-between px-4 py-2 border-b">
        <h1 class="text-xl font-bold">My E-commerce Store</h1>
        <button @click="isSidebarOpen = !isSidebarOpen" class="lg:hidden">
          <MenuIcon v-if="!isSidebarOpen" class="h-6 w-6" />
          <XIcon v-else class="h-6 w-6" />
        </button>
      </div>
      <nav class="p-4">
        <ul class="space-y-2">
          <li v-for="category in categories" :key="category">
            <button
              @click="selectedCategory = category"
              :class="[
                'w-full text-left px-4 py-2 rounded-md transition-colors',
                selectedCategory === category
                  ? 'bg-primary text-primary-foreground'
                  : 'hover:bg-accent hover:text-accent-foreground'
              ]"
            >
              {{ category }}
            </button>
          </li>
        </ul>
      </nav>
      <div class="p-4 border-t">
        <button @click="isCartOpen = true" class="w-full bg-primary text-primary-foreground px-4 py-2 rounded-md hover:bg-primary/90 transition-colors">
          <ShoppingCartIcon class="inline-block mr-2 h-4 w-4" />
          View Cart ({{ cartItemCount }})
        </button>
      </div>
    </aside>

    <!-- Main Content -->
    <div class="flex-1 flex flex-col overflow-hidden">
      <header class="bg-white shadow-sm">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 flex justify-between items-center">
          <h1 class="text-2xl font-bold text-gray-900">Products</h1>
          <div class="flex items-center">
            <div class="relative">
              <input
                v-model="searchQuery"
                type="search"
                placeholder="Search products..."
                class="pl-10 pr-4 py-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-primary"
              />
              <SearchIcon class="absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400" />
            </div>
            <button
              @click="isCartOpen = true"
              class="ml-4 relative bg-transparent p-2 rounded-full hover:bg-gray-100 transition-colors"
            >
              <ShoppingCartIcon class="h-6 w-6 text-gray-700" />
              <span
                v-if="cartItemCount > 0"
                class="absolute -top-2 -right-2 bg-red-500 text-white rounded-full w-5 h-5 flex items-center justify-center text-xs"
              >
                {{ cartItemCount }}
              </span>
            </button>
          </div>
        </div>
      </header>

      <main class="flex-1 overflow-y-auto p-4">
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
          <div
            v-for="product in filteredProducts"
            :key="product.id"
            class="bg-white rounded-lg shadow-md overflow-hidden"
          >
            <img :src="product.image" :alt="product.name" class="w-full h-48 object-cover" />
            <div class="p-4">
              <h2 class="text-lg font-semibold text-gray-900">{{ product.name }}</h2>
              <div class="mt-1 flex items-center">
                <template v-if="product.discountPercentage > 0">
                  <p class="text-gray-600 line-through">${{ product.price.toFixed(2) }}</p>
                  <p class="ml-2 text-red-600 font-semibold">
                    ${{ (product.price * (1 - product.discountPercentage / 100)).toFixed(2) }}
                  </p>
                  <span class="ml-2 bg-red-100 text-red-800 text-xs font-medium px-2 py-0.5 rounded">
                    {{ product.discountPercentage }}% OFF
                  </span>
                </template>
                <p v-else class="text-gray-600">${{ product.price.toFixed(2) }}</p>
              </div>
              <p class="mt-1 text-sm text-gray-500">{{ product.category }}</p>
              <button
                @click="addToCart(product)"
                class="mt-4 w-full bg-primary text-primary-foreground px-4 py-2 rounded-md hover:bg-primary/90 transition-colors"
              >
                Add to Cart
              </button>
            </div>
          </div>
        </div>
      </main>
    </div>

    <!-- Shopping Cart Sidebar -->
    <div v-if="isCartOpen" class="fixed inset-0 overflow-hidden z-50">
      <div class="absolute inset-0 overflow-hidden">
        <div class="absolute inset-0 bg-gray-500 bg-opacity-75 transition-opacity" @click="isCartOpen = false"></div>
        <div class="fixed inset-y-0 right-0 pl-10 max-w-full flex">
          <div class="w-screen max-w-md">
            <div class="h-full flex flex-col bg-white shadow-xl">
              <div class="flex-1 py-6 overflow-y-auto px-4 sm:px-6">
                <div class="flex items-start justify-between">
                  <h2 class="text-lg font-medium text-gray-900">Shopping cart</h2>
                  <button
                    @click="isCartOpen = false"
                    class="ml-3 h-7 flex items-center"
                  >
                    <XIcon class="h-6 w-6 text-gray-400" />
                  </button>
                </div>

                <div class="mt-8">
                  <div v-if="cart.length === 0" class="text-center py-8">
                    <ShoppingBagIcon class="mx-auto h-12 w-12 text-gray-400" />
                    <h3 class="mt-2 text-sm font-medium text-gray-900">Your cart is empty</h3>
                    <p class="mt-1 text-sm text-gray-500">Start adding some items to your cart!</p>
                  </div>
                  <ul v-else role="list" class="divide-y divide-gray-200">
                    <li v-for="item in cart" :key="item.id" class="py-6 flex">
                      <div class="flex-shrink-0 w-24 h-24 rounded-md overflow-hidden">
                        <img :src="item.image" :alt="item.name" class="w-full h-full object-cover" />
                      </div>
                      <div class="ml-4 flex-1 flex flex-col">
                        <div>
                          <div class="flex justify-between text-base font-medium text-gray-900">
                            <h3>{{ item.name }}</h3>
                            <p class="ml-4">
                              ${{ ((item.price * (1 - item.discountPercentage / 100)) * item.quantity).toFixed(2) }}
                            </p>
                          </div>
                          <p v-if="item.discountPercentage > 0" class="mt-1 text-sm text-red-500">
                            {{ item.discountPercentage }}% OFF
                          </p>
                        </div>
                        <div class="flex-1 flex items-end justify-between text-sm">
                          <div class="flex items-center">
                            <button @click="decreaseQuantity(item)" class="text-gray-500 hover:text-gray-700">-</button>
                            <p class="mx-2 text-gray-500">Qty {{ item.quantity }}</p>
                            <button @click="increaseQuantity(item)" class="text-gray-500 hover:text-gray-700">+</button>
                          </div>
                          <button
                            @click="removeFromCart(item)"
                            class="font-medium text-primary hover:text-primary/80"
                          >
                            Remove
                          </button>
                        </div>
                      </div>
                    </li>
                  </ul>
                </div>
              </div>

              <div class="border-t border-gray-200 py-6 px-4 sm:px-6">
                <div class="flex justify-between text-base font-medium text-gray-900">
                  <p>Subtotal</p>
                  <p>${{ cartTotal.toFixed(2) }}</p>
                </div>
                <p class="mt-0.5 text-sm text-gray-500">Shipping and taxes calculated at checkout.</p>
                <div class="mt-6">
                  <button
                    @click="showCheckoutForm = true"
                    class="w-full bg-primary text-primary-foreground px-4 py-3 rounded-md hover:bg-primary/90 transition-colors"
                  >
                    Checkout
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Checkout Form -->
    <div v-if="showCheckoutForm" class="fixed inset-0 overflow-hidden z-50">
      <div class="absolute inset-0 overflow-hidden">
        <div class="absolute inset-0 bg-gray-500 bg-opacity-75 transition-opacity"></div>
        <div class="absolute inset-y-0 right-0 pl-10 max-w-full flex">
          <div class="w-screen max-w-md">
            <div class="h-full flex flex-col bg-white shadow-xl">
              <div class="flex-1 py-6 overflow-y-auto px-4 sm:px-6">
                <div class="flex items-start justify-between">
                  <h2 class="text-lg font-medium text-gray-900">Checkout</h2>
                  <button
                    @click="showCheckoutForm = false"
                    class="ml-3 h-7 flex items-center"
                  >
                    <XIcon class="h-6 w-6 text-gray-400" />
                  </button>
                </div>

                <form @submit.prevent="submitOrder" class="mt-8 space-y-6">
                  <div>
                    <label for="name" class="block text-sm font-medium text-gray-700">Full Name</label>
                    <input
                      type="text"
                      id="name"
                      v-model="checkoutForm.name"
                      required
                      class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-primary focus:border-primary sm:text-sm"
                    />
                  </div>
                  <div>
                    <label for="email" class="block text-sm font-medium text-gray-700">Email Address</label>
                    <input
                      type="email"
                      id="email"
                      v-model="checkoutForm.email"
                      required
                      class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-primary focus:border-primary sm:text-sm"
                    />
                  </div>
                  <div>
                    <label for="address" class="block text-sm font-medium text-gray-700">Shipping Address</label>
                    <textarea
                      id="address"
                      v-model="checkoutForm.address"
                      required
                      class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm py-2 px-3 focus:outline-none focus:ring-primary focus:border-primary sm:text-sm"
                    ></textarea>
                  </div>
                  <div>
                    <button
                      type="submit"
                      class="w-full bg-primary text-primary-foreground px-4 py-3 rounded-md hover:bg-primary/90 transition-colors"
                    >
                      Place Order
                    </button>
                  </div>
                </form>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { ShoppingCartIcon, XIcon, ShoppingBagIcon, SearchIcon, MenuIcon } from 'lucide-vue-next'

const products = ref([
  { id: 1, name: 'Laptop', price: 999.99, discountPercentage: 10, image: '/placeholder.svg?height=200&width=200', category: 'Electronics' },
  { id: 2, name: 'Smartphone', price: 499.99, discountPercentage: 5, image: '/placeholder.svg?height=200&width=200', category: 'Electronics' },
  { id: 3, name: 'Headphones', price: 99.99, discountPercentage: 0, image: '/placeholder.svg?height=200&width=200', category: 'Electronics' },
  { id: 4, name: 'T-shirt', price: 19.99, discountPercentage: 20, image: '/placeholder.svg?height=200&width=200', category: 'Clothing' },
  { id: 5, name: 'Jeans', price: 49.99, discountPercentage: 15, image: '/placeholder.svg?height=200&width=200', category: 'Clothing' },
  { id: 6, name: 'Sneakers', price: 79.99, discountPercentage: 0, image: '/placeholder.svg?height=200&width=200', category: 'Footwear' },
])

const cart = ref([])
const isCartOpen = ref(false)
const showCheckoutForm = ref(false)
const checkoutForm = ref({
  name: '',
  email: '',
  address: '',
})
const selectedCategory = ref('All')
const searchQuery = ref('')
const isSidebarOpen = ref(true)

const addToCart = (product) => {
  const existingItem = cart.value.find(item => item.id === product.id)
  if  (existingItem) {
    existingItem.quantity++
  } else {
    cart.value.push({ ...product, quantity: 1 })
  }
}

const removeFromCart = (item) => {
  const index = cart.value.findIndex(cartItem => cartItem.id === item.id)
  if (index !== -1) {
    cart.value.splice(index, 1)
  }
}

const increaseQuantity = (item) => {
  item.quantity++
}

const decreaseQuantity = (item) => {
  if (item.quantity > 1) {
    item.quantity--
  } else {
    removeFromCart(item)
  }
}

const cartItemCount = computed(() => {
  return cart.value.reduce((total, item) => total + item.quantity, 0)
})

const cartTotal = computed(() => {
  return cart.value.reduce((total, item) => {
    const discountedPrice = item.price * (1 - item.discountPercentage / 100)
    return total + discountedPrice * item.quantity
  }, 0)
})

const categories = computed(() => {
  const allCategories = products.value.map(product => product.category)
  return ['All', ...new Set(allCategories)]
})

const filteredProducts = computed(() => {
  return products.value.filter(product => 
    (selectedCategory.value === 'All' || product.category === selectedCategory.value) &&
    product.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

const submitOrder = () => {
  console.log('Order submitted:', { ...checkoutForm.value, items: cart.value, total: cartTotal.value })
  alert('Thank you for your order!')
  cart.value = []
  checkoutForm.value = { name: '', email: '', address: '' }
  showCheckoutForm.value = false
  isCartOpen.value = false
}
</script>