<template>
  <div class="bg-white shadow-lg rounded-lg p-6 mx-auto w-full">
    <h2 class="text-2xl font-semibold text-blue-700 mb-6">Cart Overview</h2>

    <!-- Cart Items List -->
    <ul class="space-y-4">
      <li
        v-for="(item, index) in cart"
        :key="item.id"
        class="flex justify-between items-center p-4 border-b border-gray-200 rounded-lg hover:bg-gray-50"
        :draggable="true"
        @dragstart="dragStart(index)"
        @dragover.prevent
        @drop="handleDrop(index)"
      >
        <div class="flex-1">
          <p class="text-lg font-medium text-blue-600">
            {{ capitalizeFirstLetter(item.type) }} with {{ item.trainer }} at
            {{ item.time_slot }}
          </p>
        </div>
        <div class="text-lg font-semibold text-gray-800">
          ${{ formatPrice(item.price) }}
        </div>
        <!-- Remove Button -->
        <button
          @click="removeItem(item.id)"
          class="ml-4 text-red-600 hover:text-red-800"
        >
          <i class="fas fa-trash-alt"></i> Remove
        </button>
      </li>
    </ul>

    <!-- Total Cost -->
    <div class="mt-4 flex justify-between items-center">
      <p class="text-xl font-bold text-gray-900">Total:</p>
      <p class="text-xl font-semibold text-green-600">
        ${{ formatPrice(totalCost) }}
      </p>
    </div>
  </div>
</template>

<script>
export default {
  props: ["cart"],
  computed: {
    totalCost() {
      return this.cart.reduce(
        (sum, item) => sum + parseFloat(item.price || 0),
        0
      );
    },
  },
  methods: {
    capitalizeFirstLetter(string) {
      return string.charAt(0).toUpperCase() + string.slice(1).toLowerCase();
    },
    formatPrice(price) {
      return parseFloat(price).toFixed(2);
    },
    removeItem(itemId) {
      this.$emit("remove-item", itemId); // Emit event to parent with the item ID
    },
    dragStart(index) {
      this.draggedItemIndex = index;
    },
    handleDrop(index) {
      if (index === this.draggedItemIndex) return;
      this.$emit("reorder-item", this.draggedItemIndex, index);
    },
  },
};
</script>

<style scoped>
/* Styling for the Cart Overview container */
.bg-white {
  background-color: #fff;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  border-radius: 1rem;
  padding: 1.5rem;
  max-width: 100%;
}

/* List item styling */
ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  border-bottom: 1px solid #e2e8f0;
  border-radius: 0.375rem;
}

li:hover {
  background-color: #f3f4f6;
}

button.ml-4 {
  color: #ef4444;
  font-weight: bold;
  border: none;
  background: none;
  cursor: pointer;
}

button.ml-4:hover {
  color: #dc2626;
}

.fas {
  font-size: 1rem;
}
</style>
