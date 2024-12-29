<template>
  <div class="grid gap-4">
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
    };
  },
  methods: {
    async fetchSessions() {
      const cachedSessions = localStorage.getItem("sessions");

      if (cachedSessions) {
        this.sessions = JSON.parse(cachedSessions);
      } else {
        try {
          const response = await axios.get(
            "https://matchable-backend.onrender.com/sessions"
          );
          this.sessions = response.data;
          localStorage.setItem("sessions", JSON.stringify(this.sessions));
        } catch (error) {
          console.error("Error fetching sessions:", error);
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
