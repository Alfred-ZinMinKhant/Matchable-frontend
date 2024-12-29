<template>
  <div class="grid gap-4">
    <div v-if="loading" class="loading-indicator">Loading sessions...</div>

    <SessionSelection
      :sessions="sessions"
      :cart="cart"
      @add-session="addToCart"
    />
    <CartOverview
      :cart="cart"
      @reorder-item="reorderItem"
      @remove-item="removeFromCart"
    />
    <BookingForm :cart="cart" @submit-booking="handleBooking" />
  </div>
</template>

<script>
import axios from "axios";
import SessionSelection from "../components/SessionSelection.vue";
import CartOverview from "../components/CartOverview.vue";
import BookingForm from "../components/BookingForm.vue";

export default {
  components: { SessionSelection, CartOverview, BookingForm },
  data() {
    return {
      sessions: [],
      cart: [],
      loading: false,
    };
  },
  methods: {
    async fetchSessions() {
      this.loading = true;

      const cachedSessions = localStorage.getItem("sessions");

      if (cachedSessions) {
        this.sessions = JSON.parse(cachedSessions);
        this.loading = false;
      } else {
        try {
          const response = await axios.get(
            "https://matchable-backend.onrender.com/sessions"
          );
          this.sessions = response.data;
          localStorage.setItem("sessions", JSON.stringify(this.sessions));
          this.loading = false;
        } catch (error) {
          console.error("Error fetching sessions:", error);
          this.loading = false;
        }
      }
    },
    addToCart(session) {
      this.cart.push(session);
    },
    removeFromCart(itemId) {
      this.cart = this.cart.filter((item) => item.id !== itemId);
    },
    reorderItem(fromIndex, toIndex) {
      const movedItem = this.cart[fromIndex];
      this.cart.splice(fromIndex, 1);
      this.cart.splice(toIndex, 0, movedItem);
    },
    async handleBooking(bookingDetails) {
      try {
        const payload = { ...bookingDetails, sessions: this.cart };
        await axios.post(
          "https://matchable-backend.onrender.com/bookings",
          payload
        );
        alert("Booking successful!");
        this.cart = [];
      } catch (error) {
        alert("Booking failed. Please try again.");
        console.error(error);
      }
    },
  },
  mounted() {
    this.fetchSessions();
  },
};
</script>

<style scoped>
.loading-indicator {
  text-align: center;
  font-size: 1.5rem;
  padding: 20px;
  color: #007bff;
  font-weight: bold;
}
</style>
