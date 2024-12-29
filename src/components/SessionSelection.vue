<template>
  <div class="bg-white shadow-lg rounded-lg p-6 mx-auto w-full">
    <h2 class="text-3xl font-semibold text-blue-700 mb-6">
      Available Sessions
    </h2>

    <!-- Filters for selecting time slot and type -->
    <div class="mb-4 flex space-x-4">
      <div class="flex-1">
        <label for="time-slot-select" class="font-medium text-blue-600">
          Select Time Slot:
        </label>
        <select
          id="time-slot-select"
          v-model="selectedTimeSlot"
          class="mt-2 p-2 border rounded w-full"
        >
          <option value="">All Time Slots</option>
          <option
            v-for="timeSlot in uniqueTimeSlots"
            :key="timeSlot"
            :value="timeSlot"
            :disabled="isOverlapping(timeSlot)"
          >
            {{ timeSlot }}
          </option>
        </select>
      </div>

      <div class="flex-1">
        <label for="type-select" class="font-medium text-blue-600">
          Select Type:
        </label>
        <select
          id="type-select"
          v-model="selectedType"
          class="mt-2 p-2 border rounded w-full"
        >
          <option value="">All Types</option>
          <option v-for="type in uniqueTypes" :key="type" :value="type">
            {{ type.toUpperCase() }}
          </option>
        </select>
      </div>
    </div>

    <!-- Display sessions -->
    <div
      v-for="session in filteredSessions"
      :key="session.id"
      class="session-card"
    >
      <div class="session-info">
        <div class="text-gray-700">
          <p class="font-medium text-blue-600 flex items-center">
            <i class="fas fa-chalkboard-teacher mr-2 text-lg"></i>
            <span class="text-base">Type:</span>
            <span class="ml-2 text-lg font-semibold text-gray-800">
              {{ capitalizeFirstLetter(session.type) }}
            </span>
          </p>
        </div>
        <div class="text-gray-700">
          <p class="font-medium text-blue-600 flex items-center">
            <i class="fas fa-user-tie mr-2 text-lg"></i>
            <span class="text-base">Trainer:</span>
            <span class="ml-2 text-lg font-semibold text-gray-800">
              {{ capitalizeFirstLetter(session.trainer) }}
            </span>
          </p>
        </div>
        <div class="text-gray-700">
          <p class="font-medium text-blue-600 flex items-center">
            <i class="fas fa-clock mr-2 text-lg"></i>
            <span class="text-base">Time:</span>
            <span class="ml-2 text-lg font-semibold text-gray-800">
              {{ session.time_slot }}
            </span>
          </p>
        </div>
        <div class="text-gray-700">
          <p class="font-medium text-blue-600 flex items-center">
            <i class="fas fa-dollar-sign mr-2 text-lg"></i>
            <span class="text-base">Price:</span>
            <span class="ml-2 text-lg font-semibold text-green-600">
              {{ session.price | currency }}
            </span>
          </p>
        </div>
      </div>

      <button
        @click="addToCart(session)"
        :disabled="isInCart(session)"
        class="mt-4 bg-green-600 text-white text-sm font-medium py-2 px-6 rounded-lg hover:bg-green-700 transition-colors duration-300"
      >
        {{ isInCart(session) ? "Already in Cart" : "Add to Cart" }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: ["sessions", "cart"], // Use cart as a prop from the parent
  data() {
    return {
      selectedTimeSlot: "", // Selected time slot filter
      selectedType: "", // Selected type filter
    };
  },
  computed: {
    uniqueTimeSlots() {
      const timeSlots = this.sessions.map((session) => session.time_slot);
      return [...new Set(timeSlots)];
    },
    uniqueTypes() {
      const types = this.sessions.map((session) => session.type);
      return [...new Set(types)];
    },
    filteredSessions() {
      return this.sessions
        .filter((session) => {
          if (
            this.selectedTimeSlot &&
            session.time_slot !== this.selectedTimeSlot
          ) {
            return false;
          }
          if (this.selectedType && session.type !== this.selectedType) {
            return false;
          }
          return true;
        })
        .sort(
          (a, b) =>
            a.type.localeCompare(b.type) ||
            a.time_slot.localeCompare(b.time_slot)
        );
    },
  },
  methods: {
    addToCart(session) {
      const existingSession = this.cart.find(
        (cartItem) => cartItem.time_slot === session.time_slot
      );

      if (existingSession && existingSession.type !== session.type) {
        alert(
          "You cannot add another session with the same time slot but a different type."
        );
        return;
      }

      this.$emit("add-session", session); // Notify parent to update cart
    },
    capitalizeFirstLetter(string) {
      return string ? string.charAt(0).toUpperCase() + string.slice(1) : string;
    },
    parseTimeSlot(timeSlot) {
      const [start, end] = timeSlot.split(" - ");
      const [startHour, startMinute] = start.split(":");
      const [endHour, endMinute] = end.split(":");

      return {
        startInMinutes: +startHour * 60 + +startMinute,
        endInMinutes: +endHour * 60 + +endMinute,
      };
    },
    isOverlapping(timeSlot) {
      const { startInMinutes, endInMinutes } = this.parseTimeSlot(timeSlot);
      return this.cart.some((cartItem) => {
        const { startInMinutes: cartStart, endInMinutes: cartEnd } =
          this.parseTimeSlot(cartItem.time_slot);

        return (
          (startInMinutes < cartEnd && endInMinutes > cartStart) ||
          (startInMinutes === cartStart && endInMinutes === cartEnd)
        );
      });
    },
    isInCart(session) {
      return this.cart.some((cartItem) => cartItem.id === session.id);
    },
  },
};
</script>

<style scoped>
/* Styling for session cards */
.session-card {
  border: 1px solid #e2e8f0;
  border-radius: 0.375rem;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
  transition: box-shadow 0.3s ease;
  background-color: #f9fafb;
}

/* Hover effect for the session cards */
.session-card:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  background-color: #f1f5f9;
}

/* Grid layout for session details */
.session-info {
  display: grid;
  gap: 1.5rem;
  grid-template-columns: 1fr;
}

/* For tablet and larger screens, adjust the grid layout */
@media (min-width: 640px) {
  .session-info {
    grid-template-columns: 1fr 1fr;
  }
}

@media (min-width: 1024px) {
  .session-info {
    grid-template-columns: 1fr 1fr 1fr;
  }
}

/* Ensuring the container takes up full width */
.container {
  width: 100%;
  max-width: 100%;
}
</style>
